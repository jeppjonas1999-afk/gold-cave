<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Fixed Grid</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: white;
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .header { text-align: center; margin-bottom: 20px; }
        .balance-display { font-size: 3rem; color: #2ecc71; font-weight: bold; }
        
        .grid-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            width: 100%;
            max-width: 750px;
        }

        .mine-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 15px;
            min-height: 180px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            gap: 8px;
        }

        .market-building {
            background: #d4ac0d;
            color: #1a1a1a;
            cursor: pointer;
            border: 4px solid #f1c40f;
        }

        .for-sale { background: #34495e; border: 2px dashed #7f8c8d; cursor: pointer; }
        .progress-bg { width: 100%; background: #1a1a1a; height: 12px; border-radius: 6px; overflow: hidden; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 8px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        .buy-btn { background: #e67e22; color: white; }
        .locked { background: #1a1a1a; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; }
        
        /* Modal */
        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 100;
        }
        .modal-content {
            background: #2c3e50; padding: 25px; border-radius: 20px; width: 320px;
            border: 2px solid #f1c40f; text-align: center;
        }
        .market-item { background: #34495e; padding: 10px; margin: 10px 0; border-radius: 10px; }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
</div>

<div class="grid-container" id="grid"></div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color: #f1c40f; margin:0">MARKED</h2>
        <div class="market-item">
            <p>Super-Hakke (+25% $)</p>
            <button class="buy-btn" onclick="buyGlobalMultiplier()" id="m-mult">Pris: $5,000</button>
        </div>
        <div class="market-item">
            <p>Kaffemaskin (+10% Fart)</p>
            <button class="buy-btn" onclick="buyGlobalSpeed()" id="m-speed">Pris: $10,000</button>
        </div>
        <button style="background:#e74c3c; color:white; margin-top:10px" onclick="toggleMarket()">LUKK</button>
    </div>
</div>

<script>
    let money = 10;
    let globalMultiplier = 1;
    let globalSpeedBonus = 1;
    let mines = [];

    function initMines() {
        for(let i = 0; i < 9; i++) {
            if (i === 4) { // Setter markedet i midten (index 4)
                mines.push({ type: 'market' });
            } else if (i === 0) {
                mines.push({
                    type: 'mine', owned: true, yield: 1, speed: 5000, progress: 0,
                    yieldCost: 5, speedCost: 5, yieldUpgraded: false, speedUpgraded: false
                });
            } else {
                // Justerer prisen så de ikke blir ekstremt dyre med en gang
                mines.push({ type: 'mine', owned: false, buyCost: 50 * Math.pow(4, i) });
            }
        }
    }

    function renderGrid() {
        const gridEl = document.getElementById('grid');
        gridEl.innerHTML = '';
        
        mines.forEach((mine, index) => {
            const card = document.createElement('div');
            card.id = `node-${index}`;
            
            if (mine.type === 'market') {
                card.className = 'mine-card market-building';
                card.onclick = toggleMarket;
                card.innerHTML = `<h2>MARKED</h2><p>Klikk for globale bonuser</p><div style="font-size: 3rem">🏪</div>`;
            } else {
                card.className = mine.owned ? 'mine-card' : 'mine-card for-sale';
                if (!mine.owned) card.onclick = () => buyMine(index);
                // Vi kaller update funksjonen med en gang for å fylle innholdet
                gridEl.appendChild(card);
                updateMineUI(index);
            }
            gridEl.appendChild(card);
        });
    }

    function updateMineUI(index) {
        const mine = mines[index];
        const card = document.getElementById(`node-${index}`);
        if (!mine || mine.type !== 'mine' || !card) return;

        if (mine.owned) {
            const yBtn = mine.yieldUpgraded ? `<button class="locked" disabled>LÅST</button>` : `<button class="buy-btn" onclick="upgradeYield(${index}, event)">+60% $ ($${mine.yieldCost})</button>`;
            const sBtn = mine.speedUpgraded ? `<button class="locked" disabled>LÅST</button>` : `<button class="buy-btn" style="background:#9b59b6" onclick="upgradeSpeed(${index}, event)">-20% Tid ($${mine.speedCost})</button>`;

            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.9rem">Inntekt: $${(mine.yield * globalMultiplier).toFixed(1)}</div>
                ${yBtn} ${sBtn}
            `;
        } else {
            card.innerHTML = `<h3>LÅST</h3><p>Pris: $${mine.buyCost.toLocaleString()}</p><button class="buy-btn">KJØP</button>`;
        }
    }

    function toggleMarket() {
        const modal = document.getElementById('market-modal');
        modal.style.display = (modal.style.display === 'flex') ? 'none' : 'flex';
    }

    function buyMine(index) {
        if (!mines[index].owned && money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index].owned = true;
            mines[index].yield = (index + 1) * 2;
            mines[index].speed =
