<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Fixed & Balanced</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: white;
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .header {
            text-align: center;
            margin-bottom: 20px;
        }
        .balance-display {
            font-size: 3rem;
            color: #2ecc71;
            font-weight: bold;
        }
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
        }
        .for-sale {
            background: #34495e;
            border: 2px dashed #7f8c8d;
            cursor: pointer;
            transition: 0.3s;
            justify-content: center;
        }
        .for-sale:hover {
            background: #3e5871;
            transform: scale(1.02);
        }
        .progress-bg {
            width: 100%;
            background: #1a1a1a;
            height: 12px;
            border-radius: 6px;
            overflow: hidden;
        }
        .progress-bar {
            width: 0%;
            height: 100%;
            background: #f1c40f;
        }
        button {
            width: 100%;
            padding: 10px;
            border-radius: 5px;
            border: none;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }
        .buy-btn { background: #e67e22; color: white; }
        .upg-btn { background: #3498db; color: white; font-size: 0.85rem; }
        .speed-btn { background: #9b59b6; color: white; font-size: 0.85rem; }
        button:disabled { background: #7f8c8d !important; cursor: not-allowed; opacity: 0.5; }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
</div>

<div class="grid-container" id="grid"></div>

<script>
    let money = 10; // Starter med litt penger så du kan teste
    let mines = [];

    // Initialiserer 9 sjakter (3x3)
    function initMines() {
        // Første sjakt er eid
        mines.push({
            owned: true,
            yield: 1,
            speed: 5000, // 5 sekunder start
            progress: 0,
            yieldCost: 5,
            speedCost: 5
        });

        // Resten er til salgs
        for(let i = 1; i < 9; i++) {
            mines.push({
                owned: false,
                buyCost: 50 * Math.pow(5, i) // Justert prisstigning for bedre spillbarhet
            });
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
            card.innerHTML = `
                <h3 style="margin:0">Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}"></div></div>
                <div style="font-size: 0.9rem">
                    $${mine.yield.toFixed(1)} / ${(mine.speed/1000).toFixed(2)}s
                </div>
                <button class="upg-btn" onclick="upgradeYield(${index})" id="upg-y-${index}">
                    +60% Yield ($${mine.yieldCost.toFixed(0)})
                </button>
                <button class="speed-btn" onclick="upgradeSpeed(${index})" id="upg-s-${index}">
                    -20% Tid ($${mine.speedCost.toFixed(0)})
                </button>
            `;
        } else {
            card.onclick = () => buyMine(index);
            card.innerHTML = `
                <h2 style="color: #e74c3c; margin:0">LÅST</h2>
                <p>Pris: $${mine.buyCost.toLocaleString()}</p>
                <button class="buy-btn" id="buy-btn-${index}">KJØP</button>
            `;
        }
    }

    function buyMine(index) {
        if (money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = {
                owned: true,
                yield: (index + 1) * 2,
                speed: 8000,
                progress: 0,
                yieldCost: 10,
                speedCost: 10
            };
            renderGrid();
        }
    }

    function upgradeYield(index) {
        if (money >= mines[index].yieldCost) {
            money -= mines[index].yieldCost;
            mines[index].yield *= 1.6;
            mines[index].yieldCost *= 1.75;
            updateCardUI(index);
        }
    }

    function upgradeSpeed(index) {
        if (money >= mines[index].speedCost) {
            money -= mines[index].speedCost;
            mines[index].speed *= 0.8; // Hastigheten øker ved at tiden minker
            mines[index].speedCost *= 1.75;
            updateCardUI(index);
        }
    }

    let lastTime = Date.now();
    function gameLoop() {
        let now = Date.now();
        let diff = now - lastTime;
        lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                // Oppdater progress
                mine.progress += (diff / mine.speed) * 100;
                if (mine.progress >= 100) {
                    mine.progress = 0;
                    money += mine.yield;
                }
                
                // Oppdater bar uten å tegne om hele kortet (viktig for ytelse)
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = mine.progress + "%";

                // Sjekk om upgrade-knapper skal disables
                const btnY = document.getElementById(`upg-y-${index}`);
                const btnS = document.getElementById(`upg-s-${index}`);
                if(btnY) btnY.disabled = money < mine.yieldCost;
                if(btnS) btnS.disabled = money < mine.speedCost;
            } else {
                const btnBuy = document.getElementById(`buy-btn-${index}`);
                if(btnBuy) btnBuy.disabled = money < mine.buyCost;
            }
        });

        balEl.innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        requestAnimationFrame(gameLoop);
    }

    initMines();
    renderGrid();
    requestAnimationFrame(gameLoop);
</script>

</body>
</html>
