<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Market Building</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
            max-width: 1000px;
        }

        .mine-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 15px;
            width: 220px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            gap: 8px;
            position: relative;
        }

        /* Spesial-stil for Markeds-bygningen */
        .market-building {
            background: #d4ac0d; /* Gull-aktig farge */
            color: #1a1a1a;
            cursor: pointer;
            transition: transform 0.2s;
            justify-content: center;
            border: 4px solid #f1c40f;
        }
        .market-building:hover { transform: scale(1.05); }

        .for-sale { background: #34495e; border: 2px dashed #7f8c8d; cursor: pointer; justify-content: center; }
        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 10px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        .buy-btn { background: #e67e22; color: white; }
        .locked { background: #1a1a1a; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; }
        
        /* Modal (Oppgraderings UI) */
        .modal-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            justify-content: center;
            align-items: center;
            z-index: 100;
        }
        .modal-content {
            background: #2c3e50;
            padding: 30px;
            border-radius: 20px;
            width: 400px;
            border: 2px solid #f1c40f;
            text-align: center;
        }
        .market-item {
            background: #34495e;
            padding: 15px;
            margin: 10px 0;
            border-radius: 10px;
        }
        .close-btn {
            background: #e74c3c;
            margin-top: 15px;
        }
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
        <h2 style="color: #f1c40f;">GLOBAL MARKED</h2>
        <div class="market-item">
            <p><b>Super-Hakke</b> (+25% Inntekt)</p>
            <button class="buy-btn" onclick="buyGlobalMultiplier()" id="m-mult">Pris: $5,000</button>
        </div>
        <div class="market-item">
            <p><b>Kaffemaskin</b> (+10% Fart)</p>
            <button class="buy-btn" onclick="buyGlobalSpeed()" id="m-speed">Pris: $10,000</button>
        </div>
        <button class="close-btn" onclick="toggleMarket()">LUKK</button>
    </div>
</div>

<script>
    let money = 10;
    let globalMultiplier = 1;
    let globalSpeedBonus = 1;
    let mines = [];

    function initMines() {
        // Vi lager 9 plasser. Plass nr 4 (index 3) blir markedet.
        for(let i = 0; i < 9; i++) {
            if (i === 3) {
                mines.push({ type: 'market' });
            } else if (i === 0) {
                mines.push({
                    type: 'mine', owned: true, yield: 1, speed: 5000, progress: 0,
                    yieldCost: 5, speedCost: 5, yieldUpgraded: false, speedUpgraded: false
                });
            } else {
                mines.push({ type: 'mine', owned: false, buyCost: 50 * Math.pow(5, i) });
            }
        }
    }

    const gridEl = document.getElementById('grid');

    function renderGrid() {
        gridEl.innerHTML = '';
        mines.forEach((obj, index) => {
            const card = document.createElement('div');
            card.id = `node-${index}`;
            
            if (obj.type === 'market') {
                card.className = 'mine-card market-building';
                card.onclick = toggleMarket;
                card.innerHTML = `<h2>MARKED</h2><p>Klikk for å oppgradere</p><div style="font-size: 2rem">🏪</div>`;
            } else {
                card.className = obj.owned ? 'mine-card' : 'mine-card for-sale';
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
            const yBtn = mine.yieldUpgraded ? `<button class="locked" disabled>LÅST</button>` : `<button class="buy-btn" onclick="upgradeYield(${index})">+60% Yield ($${mine.yieldCost})</button>`;
            const sBtn = mine.speedUpgraded ? `<button class="locked" disabled>LÅST</button>` : `<button class="buy-btn" style="background:#9b59b6" onclick="upgradeSpeed(${index})">-20% Tid ($${mine.speedCost})</button>`;

            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.8rem">$${(mine.yield * globalMultiplier).toFixed(1)} / ${(mine.speed/1000).toFixed(1)}s</div>
                ${yBtn} ${sBtn}
            `;
        } else {
            card.onclick = () => buyMine(index);
            card.innerHTML = `<h3>LÅST</h3><p>$${mine.buyCost.toLocaleString()}</p><button class="buy-btn">KJØP</button>`;
        }
    }

    function toggleMarket() {
        const modal = document.getElementById('market-modal');
        modal.style.display = (modal.style.display === 'flex') ? 'none' : 'flex';
    }

    function buyMine(index) {
        if (money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index].owned = true;
            mines[index].yield = (index + 1) * 2;
            mines[index].speed = 8000;
            mines[index].progress = 0;
            mines[index].yieldCost = 20 * (index + 1);
            mines[index].speedCost = 20 * (index + 1);
            mines[index].yieldUpgraded = false;
            mines[index].speedUpgraded = false;
            renderGrid();
        }
    }

    function upgradeYield(index) {
        if (money >= mines[index].yieldCost) {
            money -= mines[index].yieldCost;
            mines[index].yield *= 1.6;
            mines[index].yieldUpgraded = true;
            updateMineUI(index);
        }
    }

    function upgradeSpeed(index) {
        if (money >= mines[index].speedCost) {
            money -= mines[index].speedCost;
            mines[index].speed *= 0.8;
            mines[index].speedUpgraded = true;
            updateMineUI(index);
        }
    }

    function buyGlobalMultiplier() {
        if (money >= 5000) {
            money -= 5000;
            globalMultiplier += 0.25;
            document.getElementById('m-mult').innerText = "KJØPT";
            document.getElementById('m-mult').disabled = true;
            mines.forEach((_, i) => updateMineUI(i));
        }
    }

    function buyGlobalSpeed() {
        if (money >= 10000) {
            money -= 10000;
            globalSpeedBonus = 1.1;
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
