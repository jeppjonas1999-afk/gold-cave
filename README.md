<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Market Edition</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: white;
            font-family: sans-serif;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .header { text-align: center; margin-bottom: 20px; }
        .balance-display { font-size: 3rem; color: #2ecc71; font-weight: bold; }
        
        /* Layout for spill og marked */
        .main-container {
            display: flex;
            gap: 30px;
            align-items: flex-start;
            max-width: 1250px;
        }

        .grid-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
        }

        /* Marked-styling */
        .market-container {
            background: #34495e;
            padding: 20px;
            border-radius: 15px;
            width: 250px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            border: 2px solid #f1c40f;
        }

        .mine-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 15px;
            width: 200px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        .for-sale { background: #34495e; border: 2px dashed #7f8c8d; cursor: pointer; justify-content: center; }
        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 10px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; transition: 0.2s; }
        .buy-btn { background: #e67e22; color: white; }
        .upg-btn { background: #3498db; color: white; }
        .speed-btn { background: #9b59b6; color: white; }
        
        /* LÅST-knapp styling */
        .locked { background: #1a1a1a; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; text-transform: uppercase; }
        
        button:disabled { background: #7f8c8d !important; cursor: not-allowed; opacity: 0.5; }
        
        .market-item {
            background: #2c3e50;
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 8px;
            font-size: 0.9rem;
        }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
</div>

<div class="main-container">
    <div class="grid-container" id="grid"></div>

    <div class="market-container">
        <h2 style="margin-top:0; color: #f1c40f; text-align: center;">MARKED</h2>
        <div id="market-items">
            <div class="market-item">
                <p><b>Super-Hakke</b><br>+25% yield på alle sjakter.</p>
                <button class="buy-btn" onclick="buyGlobalMultiplier()" id="market-mult-btn">Pris: $5,000</button>
            </div>
            <div class="market-item">
                <p><b>Kaffe-maskin</b><br>Gjør alle sjakter 10% raskere.</p>
                <button class="buy-btn" onclick="buyGlobalSpeed()" id="market-speed-btn">Pris: $10,000</button>
            </div>
        </div>
    </div>
</div>

<script>
    let money = 10;
    let globalMultiplier = 1;
    let globalSpeedBonus = 1;
    let mines = [];

    function initMines() {
        mines.push({
            owned: true, yield: 1, speed: 5000, progress: 0, 
            yieldCost: 5, speedCost: 5, yieldUpgraded: false, speedUpgraded: false
        });

        for(let i = 1; i < 9; i++) {
            mines.push({ owned: false, buyCost: 50 * Math.pow(5, i) });
        }
    }

    const gridEl = document.getElementById('grid');
    const balEl = document.getElementById('total-balance');

    function renderGrid() {
        gridEl.innerHTML = '';
        mines.forEach((mine, index) => {
            const card = document.createElement('div');
            card.id = `mine-${index}`;
            card.className = mine.owned ? 'mine-card' : 'mine-card for-sale';
            gridEl.appendChild(card);
            updateCardUI(index);
        });
    }

    function updateCardUI(index) {
        const mine = mines[index];
        const card = document.getElementById(`mine-${index}`);
        if (!card) return;

        if (mine.owned) {
            const yieldHTML = mine.yieldUpgraded 
                ? `<button class="locked" disabled>LÅST</button>` 
                : `<button class="upg-btn" onclick="upgradeYield(${index})" id="upg-y-${index}">+60% Yield ($${mine.yieldCost})</button>`;

            const speedHTML = mine.speedUpgraded 
                ? `<button class="locked" disabled>LÅST</button>` 
                : `<button class="speed-btn" onclick="upgradeSpeed(${index})" id="upg-s-${index}">-20% Tid ($${mine.speedCost})</button>`;

            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.8rem">Inntekt: $${(mine.yield * globalMultiplier).toFixed(1)}</div>
                ${yieldHTML}
                ${speedHTML}
            `;
        } else {
            card.onclick = () => buyMine(index);
            card.innerHTML = `<h3>LÅST</h3><p>$${mine.buyCost.toLocaleString()}</p><button class="buy-btn">KJØP</button>`;
        }
    }

    function buyMine(index) {
        if (money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = {
                owned: true, yield: (index + 1) * 2, speed: 8000, progress: 0,
                yieldCost: 20 * (index + 1), speedCost: 20 * (index + 1),
                yieldUpgraded: false, speedUpgraded: false
            };
            renderGrid();
        }
    }

    function upgradeYield(index) {
        if (!mines[index].yieldUpgraded && money >= mines[index].yieldCost) {
            money -= mines[index].yieldCost;
            mines[index].yield *= 1.6;
            mines[index].yieldUpgraded = true;
            updateCardUI(index);
        }
    }

    function upgradeSpeed(index) {
        if (!mines[index].speedUpgraded && money >= mines[index].speedCost) {
            money -= mines[index].speedCost;
            mines[index].speed *= 0.8;
            mines[index].speedUpgraded = true;
            updateCardUI(index);
        }
    }

    // Markeds-funksjoner
    function buyGlobalMultiplier() {
        if (money >= 5000) {
            money -= 5000;
            globalMultiplier += 0.25;
            document.getElementById('market-mult-btn').disabled = true;
            document.getElementById('market-mult-btn').innerText = "KJØPT";
            mines.forEach((_, i) => updateCardUI(i));
        }
    }

    function buyGlobalSpeed() {
        if (money >= 10000) {
            money -= 10000;
            globalSpeedBonus = 1.1; // 10% raskere
            document.getElementById('market-speed-btn').disabled = true;
            document.getElementById('market-speed-btn').innerText = "KJØPT";
        }
    }

    function gameLoop() {
        let now = Date.now();
        let diff = (now - (window.lastTime || now));
        window.lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                // Legger til global speed bonus her
                mine.progress += (diff / (mine.speed / globalSpeedBonus)) * 100;
                
                if (mine.progress >= 100) {
                    mine.progress = 0;
                    money += (mine.yield * globalMultiplier);
                }
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = mine.progress + "%";
                
                const btnY = document.getElementById(`upg-y-${index}`);
                const btnS = document.getElementById(`upg-s-${index}`);
                if(btnY) btnY.disabled = money < mine.yieldCost;
                if(btnS) btnS.disabled = money < mine.speedCost;
            }
        });

        document.getElementById('market-mult-btn').disabled = money < 5000 && document.getElementById('market-mult-btn').innerText !== "KJØPT";
        document.getElementById('market-speed-btn').disabled = money < 10000 && document.getElementById('market-speed-btn').innerText !== "KJØPT";

        balEl.innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        requestAnimationFrame(gameLoop);
    }

    initMines();
    renderGrid();
    gameLoop();
</script>

</body>
</html>
