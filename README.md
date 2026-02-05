<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Dynamic Pricing</title>
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

        .for-sale { background: #34495e; border: 2px dashed #7f8c8d; cursor: pointer; transition: 0.2s; }
        .for-sale:hover { background: #3e5871; }
        
        .progress-bg { width: 100%; background: #1a1a1a; height: 12px; border-radius: 6px; overflow: hidden; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 8px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        .buy-btn { background: #e67e22; color: white; }
        .locked-btn { background: #1a1a1a; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; }
        
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
    let money = 500; // Starter med nok til første gruve
    let globalMultiplier = 1;
    let globalSpeedBonus = 1;
    let baseBuyCost = 500; 
    let mines = [];

    function initMines() {
        for(let i = 0; i < 9; i++) {
            if (i === 4) { 
                mines.push({ type: 'market' });
            } else if (i === 0) {
                // Første gruve er gratis/allerede eid for å komme i gang
                mines.push({
                    type: 'mine', owned: true, yield: 2, speed: 5000, progress: 0,
                    yieldCost: 50, speedCost: 50, yieldUpgraded: false, speedUpgraded: false
                });
            } else {
                // Alle andre starter på 500
                mines.push({ type: 'mine', owned: false, buyCost: baseBuyCost });
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
                card.innerHTML = `<h2>MARKED</h2><p>Oppgraderinger</p><div style="font-size: 3rem">🏪</div>`;
            } else {
                card.className = mine.owned ? 'mine-card' : 'mine-card for-sale';
                if (!mine.owned) card.onclick = () => buyMine(index);
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
            const yBtn = mine.yieldUpgraded ? `<button class="locked-btn" disabled>LÅST</button>` : `<button class="buy-btn" onclick="upgradeYield(${index}, event)">+60% $ ($${mine.yieldCost})</button>`;
            const sBtn = mine.speedUpgraded ? `<button class="locked-btn" disabled>LÅST</button>` : `<button class="buy-btn" style="background:#9b59b6" onclick="upgradeSpeed(${index}, event)">-20% Tid ($${mine.speedCost})</button>`;

            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.9rem">$${(mine.yield * globalMultiplier).toFixed(1)} / ${(mine.speed/1000).toFixed(1)}s</div>
                ${yBtn} ${sBtn}
            `;
        } else {
            card.innerHTML = `<h3>LÅST</h3><p>Pris: <b>$${mine.buyCost.toLocaleString()}</b></p><button class="buy-btn">KJØP</button>`;
        }
    }

    function buyMine(index) {
        if (!mines[index].owned && money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            
            // Konfigurer den nye gruven
            mines[index].owned = true;
            mines[index].yield = (index + 1) * 5;
            mines[index].speed = 6000;
            mines[index].progress = 0;
            mines[index].yieldCost = 200;
            mines[index].speedCost = 200;
            mines[index].yieldUpgraded = false;
            mines[index].speedUpgraded = false;

            // ØK PRISEN PÅ ALLE ANDRE GRUVER MED 10x
            mines.forEach(m => {
                if (m.type === 'mine' && !m.owned) {
                    m.buyCost *= 10;
                }
            });

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
            mines.forEach((m, i) => { if(m.type==='mine' && m.owned) updateMineUI(i); });
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
            if (mine.type === 'mine' && mine.owned) {
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
