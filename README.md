<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - The Slow Grind</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: white;
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            padding: 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .header { text-align: center; margin-bottom: 10px; }
        .balance-display { font-size: 2.5rem; color: #2ecc71; font-weight: bold; }
        
        .game-area {
            display: flex;
            flex-direction: row;
            gap: 20px;
            align-items: flex-start;
            justify-content: center;
        }

        .grid-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            width: 580px;
        }

        .mine-card {
            background: #2c3e50;
            padding: 10px;
            border-radius: 12px;
            min-height: 180px;
            text-align: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            font-size: 0.85rem;
        }

        .market-building {
            background: #d4ac0d;
            color: #1a1a1a;
            cursor: pointer;
            border: 3px solid #f1c40f;
            width: 130px;
            height: 130px;
            border-radius: 12px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: transform 0.2s;
        }
        .market-building:hover { transform: scale(1.05); }

        .for-sale { background: #34495e; border: 2px dashed #7f8c8d; cursor: pointer; }
        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; margin: 5px 0; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; transition: width 0.1s linear; }

        button { width: 100%; padding: 6px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; font-size: 0.75rem; }
        .buy-btn { background: #e67e22; color: white; }
        .upg-speed { background: #9b59b6; color: white; margin-top: 3px; }
        
        .locked-btn { 
            background: #34495e; color: #e74c3c; cursor: default; 
            border: 1px solid #e74c3c; opacity: 0.8; margin-top: 3px;
        }
        
        button:disabled { opacity: 0.4; cursor: not-allowed; }

        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 100;
        }
        .modal-content {
            background: #2c3e50; padding: 20px; border-radius: 15px; width: 320px;
            border: 2px solid #f1c40f; text-align: center;
        }
        .market-ui-box {
            background: #34495e;
            padding: 15px;
            margin: 15px 0;
            border-radius: 10px;
            border: 1px solid #f1c40f;
        }
    </style>
</head>
<body>

<div class="header">
    <h1 style="margin: 5px 0;">Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
    <div style="color: #f1c40f; font-size: 0.85rem;">Maks Nivå: <span id="max-lvl-display">3</span></div>
</div>

<div class="game-area">
    <div class="grid-container" id="grid"></div>
    <div class="market-building" onclick="toggleMarket()">
        <h3 style="margin:0">MARKED</h3>
        <div style="font-size: 2rem">🏪</div>
    </div>
</div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color: #f1c40f; margin:0">GLOBALT MARKED</h2>
        <div class="market-ui-box">
            <h3 style="margin-top:0">📜 Utvidelsestillatelse</h3>
            <p style="font-size: 0.85rem;">Øker grensen for oppgraderinger med +3 på alle sjakter.</p>
            <p id="cap-price-display" style="color: #2ecc71; font-weight: bold; font-size: 1.1rem;">Pris: $250</p>
            <button class="buy-btn" onclick="buyCapUpgrade()" id="btn-buy-cap">KJØP OPPGRADERING</button>
        </div>
        <button style="background:#e74c3c; color:white; padding: 10px;" onclick="toggleMarket()">AVSLUTT</button>
    </div>
</div>

<script>
    let money = 0; 
    let maxLevels = 3; 
    let capCost = 250; 
    let mines = [];

    // NYE START-STATS
    const START_YIELD = 1;      // $1 per gang
    const START_SPEED = 10000;  // 10 sekunder
    const START_UPG_COST = 5; 
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
                ? `<button class="locked-btn" disabled>LÅST (Maks)</button>` 
                : `<button class="buy-btn" id="y-btn-${index}" onclick="upgradeYield(${index}, event)">$ +60% ($${Math.ceil(mine.yieldCost)})</button>`;

            let sBtn = mine.speedLvl >= maxLevels 
                ? `<button class="locked-btn" disabled>LÅST (Maks)</button>` 
                : `<button class="upg-speed" id="s-btn-${index}" onclick="upgradeSpeed(${index}, event)">Fart +60% ($${Math.ceil(mine.speedCost)})</button>`;

            card.innerHTML = `
                <strong style="color:#f1c40f">Sjakt ${index + 1}</strong>
                <div class="progress-bg">
                    <div class="progress-bar" id="bar-${index}"></div>
                </div>
                <div style="font-size: 0.75rem; margin: 3px 0;">
                    Verdi: $${mine.yield.toLocaleString(undefined, {maximumFractionDigits: 1})}<br>
                    Syklus: ${(mine.speed/1000).toFixed(1)}s
                </div>
                <div>${yBtn}${sBtn}</div>
            `;
        } else {
            card.innerHTML = `<strong>LÅST SJAKT</strong><p style="margin:5px 0;">Pris: $${mine.buyCost.toLocaleString()}</p><button class="buy-btn">KJØP</button>`;
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
        const priceEl = document.getElementById('cap-price-display');
        if(priceEl) priceEl.innerText = `Pris: $${capCost.toLocaleString()}`;
        const btn = document.getElementById('btn-buy-cap');
        if(btn) btn.disabled = money < capCost;
    }

    function buyCapUpgrade() {
        if (money >= capCost) {
            money -= capCost;
            capCost *= 2;      
            maxLevels += 3;    
            document.getElementById('max-lvl-display').innerText = maxLevels;
            updateMarketUI();
            mines.forEach((_, index) => updateMineUI(index));
        }
    }

    let lastTime = Date.now();
    function gameLoop() {
        let now = Date.now();
        let diff = now - lastTime;
        lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                mine.progress += (diff / mine.speed) * 100;
                
                if (mine.progress >= 100) {
                    mine.progress = 0;
                    money += mine.yield;
                }
                
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = Math.min(mine.progress, 100) + "%";
                
                const yBtn = document.getElementById(`y-btn-${index}`);
                const sBtn = document.getElementById(`s-btn-${index}`);
                if(yBtn) yBtn.disabled = money < mine.yieldCost;
                if(sBtn) sBtn.disabled = money < mine.speedCost;
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
