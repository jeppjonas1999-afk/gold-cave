<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - 3x3 Grid</title>
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
        /* 3x3 Rutenett */
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
            width: 200px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            justify-content: center;
        }
        .for-sale {
            background: #34495e;
            border: 2px dashed #7f8c8d;
            cursor: pointer;
            transition: 0.3s;
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
            margin: 10px 0;
            overflow: hidden;
        }
        .progress-bar {
            width: 0%;
            height: 100%;
            background: #f1c40f;
        }
        button {
            width: 100%;
            padding: 8px;
            margin: 2px 0;
            border-radius: 5px;
            border: none;
            cursor: pointer;
            font-weight: bold;
        }
        .buy-btn { background: #e67e22; color: white; }
        .upg-btn { background: #3498db; color: white; font-size: 0.8rem; }
        button:disabled { background: #7f8c8d; cursor: not-allowed; }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
</div>

<div class="grid-container" id="grid">
    </div>

<script>
    let money = 0;
    let mines = [
        { owned: true, yield: 1, speed: 10000, progress: 0, upgCost: 5 }
    ];
    
    // Opprett 8 ekstra tomme sjakter
    for(let i = 0; i < 8; i++) {
        mines.push({ owned: false, buyCost: 500 * Math.pow(10, i) });
    }

    const gridEl = document.getElementById('grid');
    const balEl = document.getElementById('total-balance');

    function updateUI() {
        balEl.innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        gridEl.innerHTML = '';

        mines.forEach((mine, index) => {
            const card = document.createElement('div');
            card.className = 'mine-card';

            if (mine.owned) {
                card.innerHTML = `
                    <h3>Sjakt ${index + 1}</h3>
                    <div class="progress-bg"><div class="progress-bar" id="bar-${index}" style="width: ${mine.progress}%"></div></div>
                    <p>$${mine.yield.toFixed(1)} / ${(mine.speed/1000).toFixed(1)}s</p>
                    <button class="upg-btn" onclick="upgradeMine(${index})">Upgrade ($${mine.upgCost.toFixed(0)})</button>
                `;
            } else {
                card.classList.add('for-sale');
                card.onclick = () => buyMine(index);
                card.innerHTML = `
                    <h2 style="color: #e74c3c;">FOR SALG</h2>
                    <p>Pris: $${mine.buyCost.toLocaleString()}</p>
                    <button class="buy-btn" ${money < mine.buyCost ? 'disabled' : ''}>KJØP</button>
                `;
            }
            gridEl.appendChild(card);
        });
    }

    function buyMine(index) {
        if (money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = { 
                owned: true, 
                yield: (index + 1) * 5, // Nye gruver starter sterkere
                speed: 10000, 
                progress: 0, 
                upgCost: mines[index].buyCost * 0.1 
            };
            updateUI();
        }
    }

    function upgradeMine(index) {
        if (money >= mines[index].upgCost) {
            money -= mines[index].upgCost;
            mines[index].yield *= 1.5;
            mines[index].upgCost *= 2;
            updateUI();
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
                // Oppdater bare fremgangsmåleren for ytelse
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = mine.progress + "%";
            }
        });

        balEl.innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        
        // Sjekk om kjøpe-knapper skal aktiveres
        document.querySelectorAll('.buy-btn').forEach((btn, i) => {
            // Logikk her for å finne riktig index
        });

        requestAnimationFrame(gameLoop);
    }

    // Kjør UI-oppdatering regelmessig (ikke hver frame for å spare krefter)
    setInterval(updateUI, 1000);
    
    updateUI();
    requestAnimationFrame(gameLoop);
</script>

</body>
</html>
