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
            position: relative;
        }

        /* LEADERBOARD KOLONNE */
        .leaderboard-column {
            width: 180px;
            display: flex;
            flex-direction: column;
        }
        .leaderboard-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 12px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            min-height: 250px;
            border: 2px solid #f1c40f;
        }
        .leaderboard-list {
            list-style: none;
            padding: 0;
            margin: 10px 0 0 0;
            font-size: 0.8rem;
        }
        .leaderboard-list li {
            background: #34495e;
            margin-bottom: 8px;
            padding: 8px;
            border-radius: 6px;
            border-left: 3px solid #f1c40f;
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
            position: relative;
            overflow: hidden;
        }

        .collapsed-overlay {
            position: absolute; top:0; left:0; width:100%; height:100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            cursor: pointer;
            border: 2px solid #e74c3c;
            border-radius: 12px;
        }
        .warning-sign { font-size: 2rem; color: #e74c3c; margin-bottom: 5px; animation: blink 1s infinite; }
        @keyframes blink { 50% { opacity: 0.5; } }

        .repair-bar-bg { width: 80%; height: 10px; background: #555; border-radius: 5px; overflow: hidden; margin-top: 5px; }
        .repair-bar { width: 0%; height: 100%; background: #e74c3c; transition: width 0.1s linear; }

        .controls-column { display: flex; flex-direction: column; gap: 10px; width: 140px; }

        .market-building {
            background: #d4ac0d;
            color: #1a1a1a;
            cursor: pointer;
            border: 3px solid #f1c40f;
            width: 100%;
            height: 100px;
            border-radius: 12px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        .multiplier-row { display: flex; gap: 4px; width: 100%; }
        .mult-btn {
            flex: 1; padding: 8px 0; background: #34495e; color: white;
            border: 1px solid #7f8c8d; font-size: 0.7rem; border-radius: 5px;
            cursor: pointer; font-weight: bold;
        }
        .mult-btn.active { background: #f1c40f; color: #1a1a1a; border-color: #d4ac0d; }

        .worker-station {
            background: #34495e;
            border: 2px solid #95a5a6;
            border-radius: 12px;
            padding: 10px;
            min-height: 60px;
            text-align: center;
            position: relative;
        }

        .code-station {
            background: #273746;
            border: 2px dashed #7f8c8d;
            border-radius: 12px;
            padding: 10px;
            text-align: center;
        }
        .code-input { width: 80%; padding: 5px; border-radius: 4px; border: none; margin-bottom: 5px; font-size: 0.7rem; text-align: center; }

        .prestige-buttons { display: flex; flex-direction: column; gap: 5px; }

        .particle {
            position: fixed; pointer-events: none; color: #f1c40f; font-weight: bold;
            font-size: 1.2rem; z-index: 1000; animation: floatUp 0.8s ease-out forwards;
        }
        @keyframes floatUp { 0% { transform: translateY(0); opacity: 1; } 100% { transform: translateY(-50px); opacity: 0; } }

        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; margin: 5px 0; cursor: pointer; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 6px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; font-size: 0.75rem; }
        .buy-btn { background: #e67e22; color: white; }
        .upg-speed { background: #9b59b6; color: white; margin-top: 3px; }
        .locked-btn { background: #34495e; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; opacity: 0.8; margin-top: 3px; }

        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 200;
        }
        .modal-content {
            background: #2c3e50; padding: 20px; border-radius: 15px; width: 340px;
            border: 2px solid #f1c40f; text-align: center;
        }
        .market-item { background: #34495e; padding: 10px; margin: 10px 0; border-radius: 8px; }
        .worker-unit { position: absolute; font-size: 1.5rem; z-index: 100; pointer-events: none; }
    </style>
</head>
<body>

<div class="header">
    <h1 style="margin: 5px 0;">Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
    <div style="color: #f1c40f; font-size: 0.85rem;">Maks Nivå: <span id="max-lvl-display">3</span> | Rebirths: <span id="rb-display">0</span></div>
</div>

<div class="game-area">
    <div class="leaderboard-column">
        <div class="leaderboard-card">
            <h3 style="margin:0; color:#f1c40f; text-align:center;">🏆 TOPP 5</h3>
            <ul class="leaderboard-list" id="leaderboard-list"></ul>
        </div>
    </div>

    <div class="grid-container" id="grid"></div>
    
    <div class="controls-column">
        <div class="market-building" onclick="toggleMarket()">
            <h3 style="margin:0">MARKED</h3>
            <div style="font-size: 2rem">🏪</div>
        </div>

        <div class="multiplier-row">
            <button class="mult-btn active" id="m1" onclick="setMultiplier(1)">1x</button>
            <button class="mult-btn" id="m10" onclick="setMultiplier(10)">10x</button>
            <button class="mult-btn" id="mMax" onclick="setMultiplier('Max')">Max</button>
        </div>

        <div class="worker-station" id="worker-station">
            <div style="font-size:0.8rem; font-weight:bold; color:#3498db;">ARBEIDERE</div>
            <div id="worker-count-display" style="font-size:0.8rem;">0 ledige</div>
        </div>

        <div class="code-station">
            <input type="text" id="code-input" class="code-input" placeholder="Kode...">
            <button class="buy-btn" style="background:#27ae60; padding:3px;" onclick="redeemCode()">OK</button>
        </div>

        <div class="prestige-buttons">
            <button class="buy-btn" style="background:#8e44ad;" onclick="doRebirth()">REBIRTH</button>
            <button class="buy-btn" style="background:#c0392b;" onclick="doDeposit()">DEPOSIT</button>
        </div>
    </div>
</div>

<div id="worker-layer"></div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color: #f1c40f;">MARKED</h2>
        <div class="market-item">
            <p>📜 Utvidelse: +3 Nivåer</p>
            <button class="buy-btn" onclick="buyCapUpgrade()">KJØP ($<span id="cap-p">62.5</span>)</button>
        </div>
        <div class="market-item">
            <p>👷 Leie Arbeider</p>
            <button class="buy-btn" onclick="buyWorker()" style="background:#3498db;">LEIE ($<span id="work-p">100</span>)</button>
        </div>
        <div class="market-item">
            <p>⛑️ Sikkerhetshjelm (-1% rase)</p>
            <button class="buy-btn" onclick="buySafety()" style="background:#f39c12;">KJØP ($<span id="safe-p">150</span>)</button>
        </div>
        <button style="background:#e74c3c; color:white; margin-top:10px;" onclick="toggleMarket()">LUKK</button>
    </div>
</div>

<script>
    let money = 0; let maxLevels = 3; let rebirths = 0;
    let capCost = 62.5; let workerCost = 100; let safetyCost = 150;
    let safetyReduction = 0; let currentMultiplier = 1;
    let mines = []; let workers = []; let usedCodes = [];
    
    let lbData = JSON.parse(localStorage.getItem('goldLB')) || [];

    const START_YIELD = 1; const START_SPEED = 10000; const START_UPG_COST = 5;
    const PRICE_MULT = 1.55; const BASE_POWER = 1.6;

    function initMines() {
        mines = [];
        for(let i=0; i<9; i++) {
            if(i===0) mines.push({owned:true, yield:START_YIELD, speed:START_SPEED, progress:0, yieldCost:START_UPG_COST, speedCost:START_UPG_COST, yieldLvl:0, speedLvl:0, active:false, isCollapsed:false, repairProgress:0, graceTimer:0});
            else mines.push({owned:false, buyCost: 500 * Math.pow(10, i-1)});
        }
    }

    function updateMineUI(index) {
        const mine = mines[index]; const card = document.getElementById(`mine-${index}`);
        if(!card) return;
        if(mine.owned) {
            let yStats = getMultiStats(mine.yieldLvl, mine.yieldCost);
            let sStats = getMultiStats(mine.speedLvl, mine.speedCost);
            let yBtn = mine.yieldLvl >= maxLevels ? `<button class="locked-btn" disabled>LÅST</button>` : `<button class="buy-btn" onclick="upgradeYield(${index}, event)">$ (${yStats.count}x) $${Math.ceil(yStats.totalCost)}</button>`;
            let sBtn = mine.speedLvl >= maxLevels ? `<button class="locked-btn" disabled>LÅST</button>` : `<button class="upg-speed" onclick="upgradeSpeed(${index}, event)">Tid (${sStats.count}x) $${Math.ceil(sStats.totalCost)}</button>`;
            let collapse = mine.isCollapsed ? `<div class="collapsed-overlay" onclick="manualRepair(${index}, event)"><div class="warning-sign">⚠️</div><div style="font-weight:bold; color:#e74c3c;">GRUVEN RASTE!</div><div class="repair-bar-bg"><div class="repair-bar" id="rb-${index}" style="width:${mine.repairProgress}%"></div></div></div>` : '';
            card.innerHTML = `${collapse}<strong style="color:#f1c40f">Sjakt ${index+1}</strong><div class="progress-bg" onclick="activateMine(${index})"><div class="progress-bar" id="bar-${index}" style="width:${mine.progress}%"></div></div><div style="font-size:0.7rem;">Verdi: $${mine.yield.toFixed(1)}<br>Tid: ${(mine.speed/1000).toFixed(1)}s</div><div>${yBtn}${sBtn}</div>`;
        } else {
            card.innerHTML = `<strong>LÅST</strong><p>$${mine.buyCost.toLocaleString()}</p><button class="buy-btn" onclick="buyMine(${index})">KJØP</button>`;
        }
    }

    function getMultiStats(lvl, cost) {
        let total = 0; let count = 0; let tempCost = cost;
        let limit = (currentMultiplier === 'Max') ? maxLevels - lvl : currentMultiplier;
        for(let i=0; i<limit; i++) {
            if(lvl + count >= maxLevels) break;
            if(currentMultiplier === 'Max' && money < total + tempCost) break;
            total += tempCost; tempCost *= PRICE_MULT; count++;
        }
        return { totalCost: total, count: count };
    }

    function upgradeYield(index, e) {
        e.stopPropagation(); let m = mines[index]; let s = getMultiStats(m.yieldLvl, m.yieldCost);
        if(s.count > 0 && money >= s.totalCost) {
            money -= s.totalCost; let power = BASE_POWER * Math.pow(1.2, rebirths);
            for(let i=0; i<s.count; i++) { m.yield *= power; m.yieldCost *= PRICE_MULT; m.yieldLvl++; }
            updateMineUI(index);
        }
    }

    function upgradeSpeed(index, e) {
        e.stopPropagation(); let m = mines[index]; let s = getMultiStats(m.speedLvl, m.speedCost);
        if(s.count > 0 && money >= s.totalCost) {
            money -= s.totalCost;
            for(let i=0; i<s.count; i++) { m.speed /= BASE_POWER; m.speedCost *= PRICE_MULT; m.speedLvl++; }
            updateMineUI(index);
        }
    }

    function buyMine(index) {
        if(money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = {owned:true, yield:START_YIELD, speed:START_SPEED, progress:0, yieldCost:START_UPG_COST, speedCost:START_UPG_COST, yieldLvl:0, speedLvl:0, active:true, isCollapsed:false, repairProgress:0, graceTimer:0};
            renderGrid();
        }
    }

    function doRebirth() {
        if(money >= 1000) { rebirths++; resetGame(false); alert("Rebirth! Dine oppgraderinger er nå 20% sterkere."); }
        else alert("Du trenger $1000 for Rebirth!");
    }

    function doDeposit() {
        let name = prompt("Navn for Topp 5?");
        if(!name) return;
        lbData.push({name: name, money: money, rb: rebirths});
        lbData.sort((a,b) => b.money - a.money);
        lbData = lbData.slice(0, 5);
        localStorage.setItem('goldLB', JSON.stringify(lbData));
        rebirths = 0; usedCodes = []; resetGame(true);
        updateLB();
    }

    function resetGame(full) {
        money = 0; maxLevels = 3; capCost = 62.5; safetyReduction = 0;
        workers.forEach(w => document.getElementById(w.id)?.remove()); workers = [];
        initMines(); renderGrid(); updateLB();
    }

    function updateLB() {
        const list = document.getElementById('leaderboard-list');
        list.innerHTML = lbData.map(p => `<li><b>${p.name}</b><br>$${p.money.toFixed(0)} | R:${p.rb}</li>`).join('');
    }

    function renderGrid() {
        const g = document.getElementById('grid'); g.innerHTML = '';
        mines.forEach((_, i) => { const c = document.createElement('div'); c.id = `mine-${i}`; c.className = 'mine-card'; g.appendChild(c); updateMineUI(i); });
    }

    function toggleMarket() { 
        const m = document.getElementById('market-modal');
        m.style.display = m.style.display === 'flex' ? 'none' : 'flex';
        document.getElementById('cap-p').innerText = capCost.toFixed(1);
        document.getElementById('work-p').innerText = workerCost.toFixed(0);
        document.getElementById('safe-p').innerText = safetyCost.toFixed(0);
    }

    function buyWorker() { if(money >= workerCost) { money -= workerCost; workerCost *= 2; workers.push(createWorker()); toggleMarket(); } }
    function buyCapUpgrade() { if(money >= capCost) { money -= capCost; capCost *= 2; maxLevels += 3; renderGrid(); toggleMarket(); } }
    function buySafety() { if(money >= safetyCost) { money -= safetyCost; safetyReduction++; safetyCost *= 1.8; toggleMarket(); } }

    function createWorker() {
        let id = 'w-' + Date.now(); let el = document.createElement('div'); el.id = id; el.className = 'worker-unit'; el.innerText = '👷';
        document.getElementById('worker-layer').appendChild(el);
        return {id: id, state: 'idle', target: null, x: 0, y: 0};
    }

    function activateMine(i) { if(mines[i].owned && !mines[i].isCollapsed) { mines[i].active = true; updateMineUI(i); } }
    function manualRepair(i, e) { e.stopPropagation(); mines[i].repairProgress += 5; if(mines[i].repairProgress >= 100) { mines[i].isCollapsed = false; mines[i].active = true; updateMineUI(i); } updateMineUI(i); }

    function setMultiplier(m) { currentMultiplier = m; document.querySelectorAll('.mult-btn').forEach(b => b.classList.remove('active')); event.target.classList.add('active'); mines.forEach((_, i) => updateMineUI(i)); }

    let last = Date.now();
    function loop() {
        let now = Date.now(); let diff = now - last; last = now;
        mines.forEach((m, i) => {
            if(m.owned && m.active && !m.isCollapsed) {
                m.progress += (diff / m.speed) * 100;
                if(m.progress >= 100) {
                    m.progress = 0; money += m.yield;
                    if(Math.random() * 100 < (m.yieldLvl * 0.2 - safetyReduction)) { m.isCollapsed = true; m.active = false; m.repairProgress = 0; }
                    updateMineUI(i);
                }
                document.getElementById(`bar-${i}`).style.width = m.progress + '%';
            }
        });
        document.getElementById('total-balance').innerText = money.toLocaleString();
        document.getElementById('rb-display').innerText = rebirths;
        requestAnimationFrame(loop);
    }

    initMines(); renderGrid(); updateLB(); loop();
</script>
</body>
</html>
