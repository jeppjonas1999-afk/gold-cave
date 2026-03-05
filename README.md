<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Gold Cave</title>
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
            overflow-x: hidden;
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

        /* LEADERBOARD STIL */
        .leaderboard-column { width: 180px; }
        .leaderboard-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 12px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            border: 2px solid #f1c40f;
            min-height: 350px;
        }
        .leaderboard-list { list-style: none; padding: 0; margin: 10px 0 0 0; }
        .leaderboard-list li {
            background: #34495e;
            margin-bottom: 8px;
            padding: 8px;
            border-radius: 6px;
            font-size: 0.75rem;
            border-left: 3px solid #f1c40f;
        }
        .lb-name { font-weight: bold; color: #3498db; }

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
            position: relative;
            overflow: hidden;
        }

        .collapsed-overlay {
            position: absolute; top:0; left:0; width:100%; height:100%;
            background: rgba(0, 0, 0, 0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            cursor: pointer;
            border: 2px solid #e74c3c;
        }
        .repair-bar-bg { width: 80%; height: 8px; background: #555; border-radius: 4px; margin-top: 5px; }
        .repair-bar { width: 0%; height: 100%; background: #e74c3c; }

        .controls-column { display: flex; flex-direction: column; gap: 10px; width: 140px; }

        .market-building {
            background: #d4ac0d;
            cursor: pointer;
            height: 100px;
            border-radius: 12px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            border: 3px solid #f1c40f;
        }

        .multiplier-row { display: flex; gap: 4px; }
        .mult-btn { flex: 1; padding: 5px; background: #34495e; color: white; border-radius: 4px; cursor: pointer; font-size: 0.7rem; }
        .mult-btn.active { background: #f1c40f; color: black; }

        .code-station, .prestige-station {
            background: #273746;
            padding: 10px;
            border-radius: 12px;
            text-align: center;
            border: 1px solid #7f8c8d;
        }
        .code-input { width: 80%; font-size: 0.7rem; text-align: center; margin-bottom: 5px; }

        .particle {
            position: fixed; pointer-events: none; color: #f1c40f; font-weight: bold;
            z-index: 1000; animation: floatUp 0.8s ease-out forwards;
        }
        @keyframes floatUp { 0% { transform: translateY(0); opacity: 1; } 100% { transform: translateY(-50px); opacity: 0; } }

        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; margin: 5px 0; cursor: pointer; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 6px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; font-size: 0.75rem; }
        .buy-btn { background: #e67e22; color: white; }
        .upg-speed { background: #9b59b6; color: white; margin-top: 3px; }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
    <div style="color: #f1c40f; font-size: 0.8rem;">Rebirths: <span id="rebirth-count">0</span> | Effekt: <span id="effekt-mult">1.0</span>x</div>
</div>

<div class="game-area">
    <div class="leaderboard-column">
        <div class="leaderboard-card">
            <h3 style="margin:0; text-align:center; color:#f1c40f;">🏆 TOPP 5</h3>
            <ul class="leaderboard-list" id="lb-list"></ul>
        </div>
    </div>

    <div class="grid-container" id="grid"></div>
    
    <div class="controls-column">
        <div class="market-building" onclick="alert('Markedet åpner...')">
            <h3>MARKED</h3>
            <div style="font-size: 1.5rem">🏪</div>
        </div>

        <div class="multiplier-row">
            <button class="mult-btn active" onclick="setMult(1)">1x</button>
            <button class="mult-btn" onclick="setMult(10)">10x</button>
            <button class="mult-btn" onclick="setMult('Max')">Max</button>
        </div>

        <div class="code-station">
            <input type="text" id="code-in" class="code-input" placeholder="KODE">
            <button class="buy-btn" style="background:#27ae60" onclick="useCode()">OK</button>
        </div>

        <div class="prestige-station">
            <button class="buy-btn" style="background:#8e44ad; margin-bottom:5px;" onclick="rebirth()">REBIRTH</button>
            <button class="buy-btn" style="background:#c0392b" onclick="deposit()">DEPOSIT</button>
        </div>
    </div>
</div>

<script>
    let money = 0;
    let rebirths = 0;
    let currentMult = 1;
    let mines = [];
    let usedCodes = [];

    // Start-verdier for leaderboard (Bots)
    let lbData = JSON.parse(localStorage.getItem('goldCaveLB')) || [
        {name: "Gull-Graver Nils", money: 10000, rebirths: 2},
        {name: "Sjakt-Kongen", money: 5000, rebirths: 1},
        {name: "Gruve-Bror", money: 2500, rebirths: 0},
        {name: "NoobMiner", money: 1000, rebirths: 0},
        {name: "Lærling", money: 500, rebirths: 0}
    ];

    function init() {
        mines = [];
        for(let i=0; i<9; i++) {
            mines.push(i===0 ? {owned:true, yield:1, speed:5000, yieldLvl:0, speedLvl:0, progress:0, active:true} : {owned:false, cost:500 * Math.pow(5, i-1)});
        }
        render();
        updateLB();
    }

    function render() {
        const grid = document.getElementById('grid');
        grid.innerHTML = '';
        mines.forEach((m, i) => {
            const card = document.createElement('div');
            card.className = 'mine-card';
            if(m.owned) {
                card.innerHTML = `
                    <strong>Sjakt ${i+1}</strong>
                    <div class="progress-bg"><div class="progress-bar" id="p${i}"></div></div>
                    <div style="font-size:0.7rem">$${m.yield.toFixed(1)} | ${(m.speed/1000).toFixed(1)}s</div>
                    <button class="buy-btn" onclick="upgYield(${i})">OPPGRADER $</button>
                    <button class="upg-speed" onclick="upgSpeed(${i})">BEDRE TID</button>
                `;
            } else {
                card.innerHTML = `<div style="margin:auto">LÅST<br>$${m.cost}<br><button class="buy-btn" onclick="buy(${i})">KJØP</button></div>`;
            }
            grid.appendChild(card);
        });
    }

    function upgYield(i) {
        let power = 1.2 * Math.pow(1.2, rebirths);
        if(money >= 10) { money -= 10; mines[i].yield *= power; mines[i].yieldLvl++; render(); }
    }

    function rebirth() {
        if(money >= 1000) {
            rebirths++;
            money = 0;
            init();
            alert("REBIRTH! Oppgraderinger er nå sterkere.");
        } else { alert("Du trenger $1000 for Rebirth!"); }
    }

    function deposit() {
        let name = prompt("Navn for leaderboard?");
        if(!name) return;
        lbData.push({name: name, money: money, rebirths: rebirths});
        lbData.sort((a,b) => b.money - a.money);
        lbData = lbData.slice(0, 5);
        localStorage.setItem('goldCaveLB', JSON.stringify(lbData));
        
        // Fullstendig reset
        money = 0; rebirths = 0; usedCodes = [];
        init();
        updateLB();
    }

    function updateLB() {
        const list = document.getElementById('lb-list');
        list.innerHTML = lbData.map(p => `<li><span class="lb-name">${p.name}</span><br>$${p.money.toLocaleString()} | R:${p.rebirths}</li>`).join('');
    }

    setInterval(() => {
        mines.forEach((m, i) => {
            if(m.owned && m.active) {
                m.progress += (100 / (m.speed / 50));
                if(m.progress >= 100) { m.progress = 0; money += m.yield; }
                let bar = document.getElementById('p'+i);
                if(bar) bar.style.width = m.progress + '%';
            }
        });
        document.getElementById('total-balance').innerText = money.toFixed(2);
        document.getElementById('rebirth-count').innerText = rebirths;
        document.getElementById('effekt-mult').innerText = Math.pow(1.2, rebirths).toFixed(1);
    }, 50);

    init();
</script>
</body>
</html>
