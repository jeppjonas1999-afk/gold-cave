<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - 3-Step Upgrades</title>
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
            width: 750px;
        }

        .mine-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 15px;
            min-height: 220px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }

        .market-building {
            background: #d4ac0d;
            color: #1a1a1a;
            cursor: pointer;
            border: 4px solid #f1c40f;
            width: 200px;
            height: 180px;
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
        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; margin: 8px 0; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 8px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        .buy-btn { background: #e67e22; color: white; margin-top: 4px; }
        .upg-speed { background: #9b59b6; color: white; margin-top: 4px; }
        
        .locked-btn { 
            background: #34495e; color: #e74c3c; cursor: default; 
            border: 1px solid #e74c3c; opacity: 0.8; margin-top: 4px;
        }
        
        button:disabled { opacity: 0.5; cursor: not-allowed; }

        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 100;
        }
        .modal-content {
            background: #2c3e50; padding: 25px; border-radius: 20px; width: 350px;
            border: 2px solid #f1c40f; text-align: center;
        }
        .market-item { background: #34495e; padding: 15px; margin: 10px 0; border-radius: 10px; border: 1px solid #7f8c8d; }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
    <div style="margin-top: 10px; color: #f1c40f;">Maks Nivå per Gruve: <span id="max-lvl-display">3</span></div>
</div>

<div class="game-area">
    <div class="grid-container" id="grid"></div>
    
    <div class="market-building" onclick="toggleMarket()">
        <h2 style="margin:0">MARKED</h2>
        <p style="font-size: 0.9rem">Utvid Kapasitet</p>
        <div style="font-size: 3rem">📜</div>
    </div>
</div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color: #f1c40f; margin:0">MARKED</h2>
        
        <div class="market-item">
            <h3>Utvidelsestillatelse</h3>
            <p>Øker maks oppgraderinger på alle gruver med <b>+3</b>.</p>
            <p id="cap-price-display" style="color: #2ecc71; font-weight: bold;">Pris: $250</p>
            <button class="buy-btn" onclick="buyCapUpgrade()" id="btn-buy-cap">KJØP +3 NIVÅER</button>
        </div>

        <button style="background:#e74c3c; color:white; margin-top:10px" onclick="toggleMarket()">LUKK</button>
    </div>
</div>

<script>
    let money = 500;
    let maxLevels = 3; // Startgrense endret til 3
    let capCost = 250; 
    let mines = [];

    const START_YIELD = 10;
    const START_SPEED = 5000;
    const START_UPG_COST = 250;
    const PRICE_MULT = 1.55; 
    const POWER_MULT = 1.6;

    function initMines() {
        for(let i = 0; i < 9; i++) {
            if (i === 0) {
                mines.push({
                    owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0,
                    yieldCost: START_UPG_COST, speedCost: START_UPG_COST,
                    yieldLvl: 0, speedLvl: 0
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
            let yBtn = mine.yieldLvl >= maxLevels 
                ? `<button class="locked-btn" disabled>LÅST (Maks Lvl ${maxLevels})</button>` 
                : `<button class="buy-btn" id="y-btn-${index}" onclick="upgradeYield(${index}, event)">Oppgradere $ (Lvl ${mine.yieldLvl})<br>$${mine.yieldCost.toLocaleString(undefined, {maximumFractionDigits: 0})}</button>`;

            let sBtn = mine.speedLvl >= maxLevels 
                ? `<button class="locked-btn" disabled>LÅST (Maks Lvl ${maxLevels})</button>` 
                : `<button class="upg-speed" id="s-btn-${index}" onclick="upgradeSpeed(${index}, event)">Oppgradere Tid (Lvl ${mine.speedLvl})<br>$${mine.speedCost.toLocaleString(undefined, {maximumFractionDigits: 0})}</button>`;

            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.85rem; margin-bottom: 5px;">
                    Inntekt: $${mine.yield.toLocaleString(undefined, {maximumFractionDigits: 1})}<br>
                    Tid: ${(mine.speed/1000).toFixed(2)}s
                </div>
                ${yBtn}
                ${sBtn}
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
                yieldCost: START_UPG_COST, speedCost: START_UPG_COST,
                yieldLvl: 0, speedLvl: 0
            };
            mines.forEach(m => { if(!m.owned) m.buyCost *= 10; });
            renderGrid();
        }
    }

    function upgradeYield(index, event) {
        event.stopPropagation();
        let mine = mines[index];
        if (mine.yieldLvl < maxLevels && money >= mine.yieldCost) {
            money -= mine.yieldCost;
            mine.yield *= POWER_MULT;
            mine.yieldCost *= PRICE_MULT;
            mine.yieldLvl++;
            updateMineUI(index);
        }
    }

    function upgradeSpeed(index, event) {
        event.stopPropagation();
        let mine = mines[index];
        if (mine.speedLvl < maxLevels && money >= mine.speedCost) {
            money -= mine.speedCost;
            mine.speed /= POWER_MULT;
            mine.speedCost *= PRICE_MULT;
            mine.speedLvl++;
            updateMineUI(index);
        }
    }

    function toggleMarket() {
        const modal = document.getElementById('market-modal');
        modal.style.display = (modal.style.display === 'flex') ? 'none' : 'flex';
        updateMarketUI();
    }

    function updateMarketUI() {
        document.getElementById('cap-price-display').innerText = `Pris: $${capCost.toLocaleString()}`;
        const btn = document.getElementById('btn-buy-cap');
        if(btn) btn.disabled = money < capCost;
    }

    function buyCapUpgrade() {
        if (money >= capCost) {
            money -= capCost;
            capCost *= 2;      
            maxLevels += 3;    // Legger til 3 nye nivåer
            
            document.getElementById('max-lvl-display').innerText = maxLevels;
            updateMarketUI();
            mines.forEach((_, index) => updateMineUI(index));
        }
    }

    function gameLoop() {
        let now = Date.now();
        let diff = now - (window.lastTime || now);
        window.lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                mine.progress += (diff / mine.speed) * 100;
                if (mine.progress >= 100) {
                    mine.progress = 0;
                    money += mine.yield;
                }
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = mine.progress + "%";
                
                const yBtn = document.getElementById(`y-btn-${index}`);
                const sBtn = document.getElementById(`s-btn-${index}`);
                if(yBtn) yBtn.disabled = money < mine.yieldCost;
                if(sBtn) sBtn.disabled = money < mine.speedCost;
            }
        });

        const marketModal = document.getElementById('market-modal');
        if (marketModal.style.display === 'flex') {
            const btn = document.getElementById('btn-buy-cap');
            if(btn) btn.disabled = money < capCost;
        }

        document.getElementById('total-balance').innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        requestAnimationFrame(gameLoop);
    }

    initMines();
    renderGrid();
    gameLoop();
</script>

</body>
</html>
