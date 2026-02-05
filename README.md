<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - 3x3 Grid + Market</title>
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
        
        /* Hovedcontainer for å ha rutenett og marked ved siden av hverandre */
        .game-area {
            display: flex;
            flex-direction: row;
            gap: 30px;
            align-items: flex-start;
            justify-content: center;
        }

        .grid-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            width: 700px;
        }

        .mine-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 15px;
            min-height: 160px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }

        /* Marked-bygningen på siden */
        .market-building {
            background: #d4ac0d;
            color: #1a1a1a;
            cursor: pointer;
            border: 4px solid #f1c40f;
            width: 180px;
            height: 160px;
            border-radius: 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            text-align: center;
            transition: transform 0.2s;
        }
        .market-building:hover { transform: scale(1.05); }

        .for-sale { background: #34495e; border: 2px dashed #7f8c8d; cursor: pointer; }
        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; margin: 5px 0; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 8px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        .buy-btn { background: #e67e22; color: white; }
        .locked-btn { background: #1a1a1a; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; }
        
        /* Modal */
        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 100;
        }
        .modal-content {
            background: #2c3e50; padding: 25px; border-radius: 20px; width: 320px;
            border: 2px solid #f1c40f; text-align: center;
        }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
</div>

<div class="game-area">
    <div class="grid-container" id="grid"></div>

    <div class="market-building" onclick="toggleMarket()">
        <h2 style="margin:0">MARKED</h2>
        <p style="font-size: 0.8rem">Globale bonuser</p>
        <div style="font-size: 2.5rem">🏪</div>
    </div>
</div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color: #f1c40f; margin:0">MARKED</h2>
        <div style="background:#34495e; padding:10px; margin:10px 0; border-radius:10px;">
            <p>Super-Hakke (+25% $)</p>
            <button class="buy-btn" onclick="buyGlobalMultiplier()" id="m-mult">Pris: $5,000</button>
        </div>
        <div style="background:#34495e; padding:10px; margin:10px 0; border-radius:10px;">
            <p>Kaffemaskin (+15% Fart)</p>
            <button class="buy-btn" onclick="buyGlobalSpeed()" id="m-speed">Pris: $10,000</button>
        </div>
        <button style="background:#e74c3c; color:white; margin-top:10px" onclick="toggleMarket()">LUKK</button>
    </div>
</div>

<script>
    let money = 500;
    let globalMultiplier = 1;
    let globalSpeedBonus = 1;
    let mines = [];

    // Felles stats for alle gruver
    const START_YIELD = 10;
    const START_SPEED = 5000;
    const UPG_YIELD_COST = 250;
    const UPG_SPEED_COST = 250;

    function initMines() {
        for(let i = 0; i < 9; i++) {
            if (i === 0) {
                mines.push({
                    owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0,
                    yieldCost: UPG_YIELD_COST, speedCost: UPG_SPEED_COST, 
                    yieldUpgraded: false, speedUpgraded: false, buyCost: 0
                });
            } else {
                mines.push({ owned: false, buyCost: 500 });
            }
        }
    }

    function renderGrid() {
        const gridEl = document.getElementById('grid');
        gridEl.innerHTML = '';
        mines.forEach((mine, index) => {
            const card = document.createElement('div');
            card.id = `mine-${index}`;
            card.className = mine.owned ? 'mine-card' : 'mine-card for-sale';
            if (!mine.owned) card.onclick = () => buyMine(index);
            gridEl.appendChild(card);
            updateMineUI(index);
        });
    }

    function updateMineUI(index) {
        const mine = mines[index];
        const card = document.getElementById(`mine-${index}`);
        if (!card) return;

        if (mine.owned) {
            const yBtn = mine.yieldUpgraded ? `<button class="locked-btn" disabled>LÅST</button>` : `<button class="buy-btn" onclick="upgradeYield(${index}, event)">+60% $ ($${mine.yieldCost})</button>`;
            const sBtn = mine.speedUpgraded ? `<button class="locked-btn" disabled>LÅST</button>` : `<button class="buy-btn" style="background:#9b59b6" onclick="upgradeSpeed(${index}, event)">-20% Tid ($${mine.speedCost})</button>`;

            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.9rem">$${(mine.yield * globalMultiplier).toFixed(1)} / ${(mine.speed/1000).toFixed(1)}s</div>
                <div style="display:flex; gap:5px; flex-direction:column">${yBtn}${sBtn}</div>
            `;
        } else {
            card.innerHTML = `<h3>LÅST</h3><p>Pris: <b>$${mine.buyCost.toLocaleString()}</b></p><button class="buy-btn">KJØP</button>`;
        }
    }

    function buyMine(index) {
        if (!mines[index].owned && money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = {
                owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0,
                yieldCost: UPG_YIELD_COST, speedCost: UPG_SPEED_COST,
                yieldUpgraded: false, speedUpgraded: false
            };
            // 10x pris på alle låste gruver
            mines.forEach(m => { if(!m.owned) m.buyCost *= 10; });
            renderGrid();
        }
    }

    function upgradeYield(index, event) {
        event.stopPropagation();
        if (money >= mines[index].yieldCost) {
            money -= mines[index].yieldCost;
            mines[index].yield *= 1.6;
            mines[index].yieldUpgraded = true;
            updateMineUI(index);
        }
    }

    function upgradeSpeed(index, event) {
        event.stopPropagation();
        if (money >= mines[index].speedCost) {
            money -= mines[index].speedCost;
            mines[index].speed *= 0.8;
            mines[index].speedUpgraded = true;
            updateMineUI(index);
        }
    }

    function toggleMarket() {
        const modal = document.getElementById('market-modal');
        modal.style.display = (modal.style.display === 'flex') ? 'none' : 'flex';
    }

    function buyGlobalMultiplier() {
        if (money >= 5000) {
            money -= 5000;
            globalMultiplier += 0.25;
            document.getElementById('m-mult').innerText = "KJØPT";
            document.getElementById('m-mult').disabled = true;
            mines.forEach((m, i) => { if(m.owned) updateMineUI(i); });
        }
    }

    function buyGlobalSpeed() {
        if (money >= 10000) {
            money -= 10000;
            globalSpeedBonus = 1.15;
            document.getElementById('m-speed').innerText = "KJØPT";
            document.getElementById('m-speed').disabled = true;
        }
    }

    function gameLoop() {
        let now = Date.now();
        let diff = now - (window.lastTime || now);
        window.lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                mine.progress += (diff / (mine.speed / globalSpeedBonus)) * 100;
                if (mine.progress >= 100) {
                    mine.progress = 0;
                    money += (mine.yield * globalMultiplier);
                }
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = mine.progress + "%";
            }
        });

        document.getElementById('total-balance').innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        requestAnimationFrame(gameLoop);
    }

    initMines();
    renderGrid();
    gameLoop();
</script>

</body>
</html>
