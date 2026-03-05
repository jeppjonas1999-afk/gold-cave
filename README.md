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
            transition: transform 0.2s;
        }
        .market-building:hover { transform: scale(1.05); }

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

        /* KODEBOKS */
        .code-station {
            background: #273746;
            border: 2px dashed #7f8c8d;
            border-radius: 12px;
            padding: 10px;
            text-align: center;
            margin-top: 5px;
        }
        .code-input {
            width: 80%;
            padding: 5px;
            border-radius: 4px;
            border: none;
            margin-bottom: 5px;
            font-size: 0.7rem;
            text-align: center;
        }

        .worker-count { font-size: 0.8rem; color: #bdc3c7; }
        .worker-icon-static { font-size: 1.5rem; }

        .worker-unit {
            position: absolute;
            font-size: 1.5rem;
            z-index: 100;
            pointer-events: none;
        }

        /* PARTIKLER */
        .particle {
            position: fixed;
            pointer-events: none;
            color: #f1c40f;
            font-weight: bold;
            font-size: 1.2rem;
            z-index: 1000;
            animation: floatUp 0.8s ease-out forwards;
        }
        @keyframes floatUp {
            0% { transform: translateY(0) scale(1); opacity: 1; }
            100% { transform: translateY(-50px) scale(1.5); opacity: 0; }
        }

        .progress-bg { width: 100%; background: #1a1a1a; height: 10px; border-radius: 5px; overflow: hidden; margin: 5px 0; cursor: pointer; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; }

        button { width: 100%; padding: 6px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; font-size: 0.75rem; }
        .buy-btn { background: #e67e22; color: white; }
        .upg-speed { background: #9b59b6; color: white; margin-top: 3px; }
        .locked-btn { background: #34495e; color: #e74c3c; cursor: default; border: 1px solid #e74c3c; opacity: 0.8; margin-top: 3px; }
        button:disabled { opacity: 0.4; cursor: not-allowed; }

        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 200;
        }
        .modal-content {
            background: #2c3e50; padding: 20px; border-radius: 15px; width: 340px;
            border: 2px solid #f1c40f; text-align: center;
            max-height: 80vh; overflow-y: auto;
        }
        .market-item {
            background: #34495e; padding: 10px; margin: 10px 0; border-radius: 8px; border: 1px solid #7f8c8d;
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
            <div class="worker-icon-static">👷</div>
            <div class="worker-count" id="worker-count-display">0 ledige</div>
        </div>

        <div class="code-station">
            <div style="font-size:0.7rem; font-weight:bold; color:#95a5a6; margin-bottom:2px;">KODER</div>
            <input type="text" id="code-input" class="code-input" placeholder="Skriv kode...">
            <button class="buy-btn" style="background:#27ae60; padding:3px;" onclick="redeemCode()">OK</button>
        </div>
    </div>
</div>

<div id="worker-layer"></div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color: #f1c40f; margin:0">GLOBALT MARKED</h2>
        
        <div class="market-item">
            <h3 style="margin:0">📜 Utvidelsestillatelse</h3>
            <p style="font-size: 0.8rem;">+3 nivåer til alle gruver.</p>
            <p id="cap-price-display" style="color: #2ecc71; font-weight: bold;">Pris: $62.5</p>
            <button class="buy-btn" onclick="buyCapUpgrade()" id="btn-buy-cap">KJØP</button>
        </div>

        <div class="market-item">
            <h3 style="margin:0">⛑️ Sikkerhetshjelm</h3>
            <p style="font-size: 0.8rem;">Senker rase-sjanse med 1%. (Min. 1%)</p>
            <p id="safety-price-display" style="color: #2ecc71; font-weight: bold;">Pris: $150</p>
            <button class="buy-btn" onclick="buySafetyHelmet()" id="btn-buy-safety" style="background:#f39c12;">KJØP</button>
        </div>

        <div class="market-item">
            <h3 style="margin:0">👷 Leie Arbeider</h3>
            <p style="font-size: 0.8rem;">Reparerer automatisk ødelagte gruver.</p>
            <p id="worker-price-display" style="color: #2ecc71; font-weight: bold;">Pris: $100</p>
            <button class="buy-btn" onclick="buyWorker()" id="btn-buy-worker" style="background:#3498db;">LEIE</button>
        </div>

        <div class="market-item">
            <h3 style="margin:0">⚡ Arbeidskurs</h3>
            <p style="font-size: 0.8rem;">Reparasjon +20%, Bevegelse +25%.</p>
            <p id="wspeed-price-display" style="color: #2ecc71; font-weight: bold;">Pris: $500</p>
            <button class="buy-btn" onclick="buyWorkerSpeed()" id="btn-buy-wspeed" style="background:#9b59b6;">OPPGRADER</button>
        </div>

        <button style="background:#e74c3c; color:white; padding: 10px;" onclick="toggleMarket()">AVSLUTT</button>
    </div>
</div>

<script>
    let money = 0; 
    let maxLevels = 3; 
    let capCost = 62.5; 
    let mines = [];
    let currentMultiplier = 1;
    let usedCodes = [];
    let safetyReduction = 0;
    let safetyCost = 150;

    let workers = []; 
    let workerCost = 100;
    let workerSpeedCost = 500;
    let workerRepairMult = 1.0; 
    let workerMoveMult = 1.0;
    let baseRepairTime = 5000; 

    const START_YIELD = 1;      
    const START_SPEED = 10000;  
    const START_UPG_COST = 5; 
    const PRICE_MULT = 1.55; 
    const POWER_MULT = 1.6;

    function initMines() {
        for(let i = 0; i < 9; i++) {
            if (i === 0) {
                mines.push({
                    owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0,
                    yieldCost: START_UPG_COST, speedCost: START_UPG_COST,
                    yieldLvl: 0, speedLvl: 0, active: false,
                    isCollapsed: false, repairProgress: 0,
                    graceTimer: 0 
                });
            } else {
                mines.push({ owned: false, buyCost: 500 });
            }
        }
    }

    function checkCollapse(index) {
        let mine = mines[index];
        if (mine.graceTimer > 0) return;

        let baseRisk = (mine.yieldLvl + mine.speedLvl) * 0.2; 
        let risk = Math.max(1, baseRisk - safetyReduction);
        if (baseRisk <= 0) risk = 0; 

        if (Math.random() * 100 < Math.min(risk, 30)) {
            triggerCollapse(index);
        }
    }

    function triggerCollapse(index) {
        let mine = mines[index];
        mine.isCollapsed = true;
        mine.repairProgress = 0;
        mine.active = false; 
        updateMineUI(index);
    }

    function manualRepair(index, event) {
        if(event) event.stopPropagation();
        let mine = mines[index];
        if (mine.isCollapsed) {
            mine.repairProgress += 4;
            if (mine.repairProgress >= 100) {
                completeRepair(index);
            }
            updateMineUI(index);
        }
    }

    function completeRepair(index) {
        let mine = mines[index];
        mine.isCollapsed = false;
        mine.repairProgress = 0;
        mine.active = true; 
        mine.graceTimer = 2500; 
        
        let worker = workers.find(w => w.targetIndex === index);
        if (worker) {
            worker.state = 'returning';
            worker.targetIndex = null;
        }
        updateMineUI(index);
    }

    function createWorker() {
        let id = 'worker-' + Date.now() + Math.random();
        let el = document.createElement('div');
        el.className = 'worker-unit';
        el.innerText = '👷';
        el.id = id;
        document.getElementById('worker-layer').appendChild(el);
        let station = document.getElementById('worker-station').getBoundingClientRect();
        return { id: id, state: 'idle', targetIndex: null, workTimer: 0, x: station.left, y: station.top };
    }

    function updateWorkers(diff) {
        let stationRect = document.getElementById('worker-station').getBoundingClientRect();
        let stationX = stationRect.left + 30 + window.scrollX;
        let stationY = stationRect.top + 30 + window.scrollY;

        document.getElementById('worker-count-display').innerText = `${workers.filter(w => w.state === 'idle').length}/${workers.length} ledige`;

        workers.forEach(w => {
            let el = document.getElementById(w.id);
            let speed = 0.3 * diff * workerMoveMult; 

            if (w.state === 'idle') {
                let collapsedIndex = mines.findIndex((m, idx) => m.isCollapsed && !workers.some(worker => worker.targetIndex === idx));
                if (collapsedIndex !== -1) { w.targetIndex = collapsedIndex; w.state = 'moving_to'; }
                else { moveTowards(w, stationX, stationY, speed); }
            }
            else if (w.state === 'moving_to') {
                let card = document.getElementById(`mine-${w.targetIndex}`);
                if (card) {
                    let rect = card.getBoundingClientRect();
                    if (moveTowards(w, rect.left + rect.width/2 + window.scrollX, rect.top + rect.height/2 + window.scrollY, speed)) {
                        w.state = 'working'; w.workTimer = 0;
                    }
                }
            }
            else if (w.state === 'working') {
                let requiredTime = baseRepairTime / workerRepairMult;
                w.workTimer += diff;
                let mine = mines[w.targetIndex];
                if(mine && mine.isCollapsed) {
                    mine.repairProgress = Math.max(mine.repairProgress, (w.workTimer / requiredTime) * 100);
                    let bar = document.getElementById(`repair-bar-${w.targetIndex}`);
                    if(bar) bar.style.width = mine.repairProgress + "%";
                }
                if (w.workTimer >= requiredTime) completeRepair(w.targetIndex);
            }
            else if (w.state === 'returning') {
                if (moveTowards(w, stationX, stationY, speed)) w.state = 'idle';
            }
            el.style.left = w.x + 'px'; el.style.top = w.y + 'px';
        });
    }

    function moveTowards(obj, tx, ty, step) {
        let dx = tx - obj.x; let dy = ty - obj.y; let dist = Math.sqrt(dx*dx + dy*dy);
        if (dist <= step) { obj.x = tx; obj.y = ty; return true; }
        obj.x += (dx / dist) * step; obj.y += (dy / dist) * step; return false;
    }

    function updateMineUI(index) {
        const mine = mines[index];
        const card = document.getElementById(`mine-${index}`);
        if (!card) return;

        if (mine.owned) {
            let yStats = getMultiUpgradeStats(mine.yieldLvl, mine.yieldCost);
            let sStats = getMultiUpgradeStats(mine.speedLvl, mine.speedCost);

            let yBtn = mine.yieldLvl >= maxLevels 
                ? `<button class="locked-btn" disabled>LÅST</button>` 
                : `<button class="buy-btn" id="y-btn-${index}" onclick="upgradeYield(${index}, event)">$ (${yStats.count}x) $${Math.ceil(yStats.totalCost)}</button>`;

            let sBtn = mine.speedLvl >= maxLevels 
                ? `<button class="locked-btn" disabled>LÅST</button>` 
                : `<button class="upg-speed" id="s-btn-${index}" onclick="upgradeSpeed(${index}, event)">Tid (${sStats.count}x) $${Math.ceil(sStats.totalCost)}</button>`;

            let overlayHTML = mine.isCollapsed ? `
                <div class="collapsed-overlay" onclick="manualRepair(${index}, event)">
                    <div class="warning-sign">⚠️</div>
                    <div style="font-weight:bold; color:#e74c3c;">GRUVEN RASTE!</div>
                    <div style="font-size:0.7rem; margin-top:5px;">Klikk raskt eller bruk arbeider</div>
                    <div class="repair-bar-bg"><div class="repair-bar" id="repair-bar-${index}" style="width: ${mine.repairProgress}%"></div></div>
                </div>` : '';

            card.innerHTML = `${overlayHTML}
                <strong style="color:#f1c40f">Sjakt ${index + 1}</strong>
                <div class="progress-bg" onclick="activateMine(${index})">
                    <div class="progress-bar" id="bar-${index}" style="width: ${mine.progress}%"></div>
                </div>
                <div style="font-size: 0.75rem; margin: 3px 0;">
                    Verdi: $${mine.yield.toLocaleString(undefined, {maximumFractionDigits: 1})}<br>
                    Syklus: ${(mine.speed/1000).toFixed(1)}s
                </div>
                <div>${yBtn}${sBtn}</div>`;
        } else {
            card.innerHTML = `<div style="display:flex; flex-direction:column; justify-content:center; height:100%;">
                <strong style="font-size:1.1rem; color:#bdc3c7;">LÅST SJAKT</strong>
                <p style="margin:10px 0; color:#f1c40f; font-weight:bold; font-size:1.2rem;">$${mine.buyCost.toLocaleString()}</p>
                <button class="buy-btn" onclick="buyMine(${index})">KJØP EIENDOM</button></div>`;
        }
    }

    function spawnParticle(x, y, text) {
        const p = document.createElement('div');
        p.className = 'particle'; p.innerText = text;
        p.style.left = x + 'px'; p.style.top = y + 'px';
        document.body.appendChild(p);
        setTimeout(() => p.remove(), 800);
    }

    function redeemCode() {
        const input = document.getElementById('code-input');
        const code = input.value.trim().toLowerCase();
        if (code === 'starterpakke' && !usedCodes.includes('starterpakke')) {
            money += 5; usedCodes.push('starterpakke'); alert('Kode aktivert! Du fikk $5.'); input.value = '';
        }
    }

    function buySafetyHelmet() {
        if (money >= safetyCost) { money -= safetyCost; safetyReduction += 1; safetyCost *= 1.8; updateMarketUI(); }
    }

    function getMultiUpgradeStats(lvl, cost) {
        let totalCost = 0; let count = 0; let tempCost = cost;
        let limit = (currentMultiplier === 'Max') ? maxLevels - lvl : currentMultiplier;
        for (let i = 0; i < limit; i++) {
            if (lvl + count >= maxLevels) break;
            if (currentMultiplier === 'Max' && money < totalCost + tempCost) break;
            totalCost += tempCost; tempCost *= PRICE_MULT; count++;
        }
        return { totalCost, count };
    }

    function upgradeYield(index, event) {
        event.stopPropagation(); let m = mines[index]; let s = getMultiUpgradeStats(m.yieldLvl, m.yieldCost);
        if (s.count > 0 && money >= s.totalCost) {
            money -= s.totalCost; for(let i=0; i<s.count; i++) { m.yield *= POWER_MULT; m.yieldCost *= PRICE_MULT; m.yieldLvl++; }
            updateMineUI(index);
        }
    }

    function upgradeSpeed(index, event) {
        event.stopPropagation(); let m = mines[index]; let s = getMultiUpgradeStats(m.speedLvl, m.speedCost);
        if (s.count > 0 && money >= s.totalCost) {
            money -= s.totalCost; for(let i=0; i<s.count; i++) { m.speed /= POWER_MULT; m.speedCost *= PRICE_MULT; m.speedLvl++; }
            updateMineUI(index);
        }
    }

    function buyMine(index) {
        if (!mines[index].owned && money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = { owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0, yieldCost: START_UPG_COST, speedCost: START_UPG_COST, yieldLvl: 0, speedLvl: 0, active: true, isCollapsed: false, repairProgress: 0, graceTimer: 0 };
            mines.forEach(m => { if(!m.owned) m.buyCost *= 10; });
            renderGrid();
        }
    }

    function setMultiplier(val) {
        currentMultiplier = val;
        document.querySelectorAll('.mult-btn').forEach(b => b.classList.remove('active'));
        document.getElementById('m' + val).classList.add('active');
        mines.forEach((_, i) => updateMineUI(i));
    }

    function renderGrid() {
        const gridEl = document.getElementById('grid');
        gridEl.innerHTML = '';
        mines.forEach((m, i) => {
            const card = document.createElement('div');
            card.id = `mine-${i}`; card.className = 'mine-card';
            gridEl.appendChild(card); updateMineUI(i);
        });
    }

    function toggleMarket() {
        const modal = document.getElementById('market-modal');
        modal.style.display = (modal.style.display === 'flex') ? 'none' : 'flex';
        updateMarketUI();
    }

    function updateMarketUI() {
        document.getElementById('cap-price-display').innerText = `Pris: $${capCost.toLocaleString()}`;
        document.getElementById('worker-price-display').innerText = `Pris: $${workerCost.toLocaleString()}`;
        document.getElementById('wspeed-price-display').innerText = `Pris: $${workerSpeedCost.toLocaleString()}`;
        document.getElementById('safety-price-display').innerText = `Pris: $${Math.ceil(safetyCost).toLocaleString()}`;
    }

    function buyCapUpgrade() {
        if (money >= capCost) { money -= capCost; capCost *= 2; maxLevels += 3; document.getElementById('max-lvl-display').innerText = maxLevels; updateMarketUI(); mines.forEach((_, i) => updateMineUI(i)); }
    }

    function buyWorker() {
        if (money >= workerCost) { money -= workerCost; workerCost *= 2; workers.push(createWorker()); updateMarketUI(); }
    }

    function buyWorkerSpeed() {
        if (money >= workerSpeedCost) { money -= workerSpeedCost; workerSpeedCost *= 1.5; workerRepairMult += 0.2; workerMoveMult += 0.25; updateMarketUI(); }
    }

    function activateMine(index) {
        if (mines[index].owned && !mines[index].active && !mines[index].isCollapsed) { mines[index].active = true; updateMineUI(index); }
    }

    let lastTime = Date.now();
    function gameLoop() {
        let now = Date.now(); let diff = now - lastTime; lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                if (mine.graceTimer > 0) mine.graceTimer -= diff;
                if (!mine.isCollapsed && mine.active) {
                    mine.progress += (diff / mine.speed) * 100;
                    if (mine.progress >= 100) {
                        mine.progress = 0; money += mine.yield;
                        const rect = document.getElementById(`mine-${index}`).getBoundingClientRect();
                        spawnParticle(rect.left + rect.width/2, rect.top, `+$${mine.yield.toFixed(1)}`);
                        checkCollapse(index); 
                    }
                } else if (mine.isCollapsed) {
                    if (!workers.some(w => w.targetIndex === index && w.state === 'working')) {
                        mine.repairProgress = Math.max(0, mine.repairProgress - (0.05 * (diff/16)));
                    }
                }
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = Math.min(mine.progress, 100) + "%";
            }
        });

        updateWorkers(diff); 
        document.getElementById('total-balance').innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        requestAnimationFrame(gameLoop);
    }

    initMines();
    renderGrid();
    gameLoop();
</script>

</body>
</html>
