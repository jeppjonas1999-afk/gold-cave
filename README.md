<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Gold Cave Edition</title>
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
            min-height: 400px; /* Sikrer at plassen er der */
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
        }
        .worker-count { font-size: 0.8rem; color: #bdc3c7; }
        
        .code-station {
            background: #273746;
            border: 2px dashed #7f8c8d;
            border-radius: 12px;
            padding: 10px;
            text-align: center;
        }
        .code-input {
            width: 90%;
            padding: 5px;
            border-radius: 4px;
            border: none;
            margin-bottom: 5px;
            font-size: 0.7rem;
            text-align: center;
        }

        .worker-unit {
            position: absolute;
            font-size: 1.5rem;
            z-index: 100;
            pointer-events: none;
        }

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
    <div style="color: #f1c40f; font-size: 0.85rem;">Maks Nivå: <span id="max-lvl-display">3</span> | Risiko-reduksjon: <span id="safety-display">0</span>%</div>
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
            <div style="font-size: 1.5rem">👷</div>
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
        <h2 style="color: #f1c40f; margin:0">MARKED</h2>
        
        <div class="market-item">
            <p>📜 <strong>Utvidelsestillatelse</strong> (+$62.5)</p>
            <button class="buy-btn" onclick="buyCapUpgrade()" id="btn-buy-cap">KJØP (+3 Nivå)</button>
        </div>

        <div class="market-item">
            <p>⛑️ <strong>Sikkerhetshjelm</strong> ($<span id="safety-cost">150</span>)</p>
            <button class="buy-btn" onclick="buySafetyHelmet()" id="btn-buy-safety" style="background:#f39c12;">KJØP (-1% Risiko)</button>
        </div>

        <div class="market-item">
            <p>👷 <strong>Arbeider</strong> ($<span id="worker-cost">100</span>)</p>
            <button class="buy-btn" onclick="buyWorker()" id="btn-buy-worker" style="background:#3498db;">LEIE</button>
        </div>

        <button style="background:#e74c3c; color:white; padding: 10px; margin-top:10px;" onclick="toggleMarket()">LUKK</button>
    </div>
</div>

<script>
    let money = 0; 
    let maxLevels = 3; 
    let capCost = 62.5; 
    let mines = [];
    let currentMultiplier = 1;
    let workers = []; 
    let workerCost = 100;
    let safetyReduction = 0;
    let safetyCost = 150;
    let usedCodes = [];

    const START_YIELD = 1;      
    const START_SPEED = 10000;  
    const PRICE_MULT = 1.55; 
    const POWER_MULT = 1.6;

    function initMines() {
        mines = []; // Reset
        for(let i = 0; i < 9; i++) {
            if (i === 0) {
                mines.push({
                    owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0,
                    yieldCost: 5, speedCost: 5, yieldLvl: 0, speedLvl: 0, active: false,
                    isCollapsed: false, repairProgress: 0, graceTimer: 0 
                });
            } else {
                mines.push({ owned: false, buyCost: 500 * Math.pow(10, i-1) });
            }
        }
    }

    function renderGrid() {
        const gridEl = document.getElementById('grid');
        gridEl.innerHTML = '';
        mines.forEach((mine, index) => {
            const card = document.createElement('div');
            card.id = `mine-${index}`;
            card.className = 'mine-card';
            gridEl.appendChild(card);
            updateMineUI(index);
        });
    }

    function updateMineUI(index) {
        const mine = mines[index];
        const card = document.getElementById(`mine-${index}`);
        if (!card) return;

        if (mine.owned) {
            let yStats = getMultiUpgradeStats(mine.yieldLvl, mine.yieldCost);
            let sStats = getMultiUpgradeStats(mine.speedLvl, mine.speedCost);

            let yBtn = mine.yieldLvl >= maxLevels ? `<button class="locked-btn" disabled>LÅST</button>` 
                : `<button class="buy-btn" onclick="upgradeYield(${index}, event)">$ (${yStats.count}x) $${Math.ceil(yStats.totalCost)}</button>`;

            let sBtn = mine.speedLvl >= maxLevels ? `<button class="locked-btn" disabled>LÅST</button>` 
                : `<button class="upg-speed" onclick="upgradeSpeed(${index}, event)">Tid (${sStats.count}x) $${Math.ceil(sStats.totalCost)}</button>`;

            let overlay = mine.isCollapsed ? `
                <div class="collapsed-overlay" onclick="manualRepair(${index}, event)">
                    <div class="warning-sign">⚠️</div>
                    <div class="repair-bar-bg"><div class="repair-bar" id="repair-bar-${index}" style="width:${mine.repairProgress}%"></div></div>
                </div>` : '';

            card.innerHTML = `${overlay}
                <strong style="color:#f1c40f">Sjakt ${index + 1}</strong>
                <div class="progress-bg" onclick="activateMine(${index})">
                    <div class="progress-bar" id="bar-${index}" style="width:${mine.progress}%"></div>
                </div>
                <div style="font-size:0.7rem">Verdi: $${mine.yield.toFixed(1)} | Tid: ${(mine.speed/1000).toFixed(1)}s</div>
                ${yBtn}${sBtn}`;
        } else {
            card.innerHTML = `<div style="margin:auto"><strong>LÅST</strong><br>$${mine.buyCost.toLocaleString()}<br>
                <button class="buy-btn" onclick="buyMine(${index})">KJØP</button></div>`;
        }
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

    function upgradeYield(idx, e) {
        e.stopPropagation(); let m = mines[idx]; let s = getMultiUpgradeStats(m.yieldLvl, m.yieldCost);
        if (money >= s.totalCost && s.count > 0) {
            money -= s.totalCost; for(let i=0; i<s.count; i++) { m.yield *= POWER_MULT; m.yieldCost *= PRICE_MULT; m.yieldLvl++; }
            updateMineUI(idx);
        }
    }

    function upgradeSpeed(idx, e) {
        e.stopPropagation(); let m = mines[idx]; let s = getMultiUpgradeStats(m.speedLvl, m.speedCost);
        if (money >= s.totalCost && s.count > 0) {
            money -= s.totalCost; for(let i=0; i<s.count; i++) { m.speed /= POWER_MULT; m.speedCost *= PRICE_MULT; m.speedLvl++; }
            updateMineUI(idx);
        }
    }

    function buyMine(idx) {
        if (money >= mines[idx].buyCost) {
            money -= mines[idx].buyCost;
            mines[idx] = { owned: true, yield: START_YIELD, speed: START_SPEED, progress: 0, yieldCost: 5, speedCost: 5, yieldLvl: 0, speedLvl: 0, active: true, isCollapsed: false, repairProgress: 0, graceTimer: 0 };
            updateMineUI(idx);
        }
    }

    function buySafetyHelmet() {
        if (money >= safetyCost) { money -= safetyCost; safetyReduction++; safetyCost *= 1.8; updateMarketUI(); }
    }

    function buyWorker() {
        if (money >= workerCost) { money -= workerCost; workerCost *= 2; workers.push(createWorker()); updateMarketUI(); }
    }

    function buyCapUpgrade() {
        if (money >= capCost) { money -= capCost; capCost *= 2; maxLevels += 3; document.getElementById('max-lvl-display').innerText = maxLevels; updateMarketUI(); mines.forEach((_, i) => updateMineUI(i)); }
    }

    function redeemCode() {
        let input = document.getElementById('code-input');
        if (input.value.toLowerCase() === 'starterpakke' && !usedCodes.includes('starterpakke')) {
            money += 5; usedCodes.push('starterpakke'); alert('Du fikk $5!'); input.value = '';
        }
    }

    function spawnParticle(x, y, text) {
        let p = document.createElement('div'); p.className = 'particle'; p.innerText = text;
        p.style.left = x + 'px'; p.style.top = y + 'px'; document.body.appendChild(p);
        setTimeout(() => p.remove(), 800);
    }

    function toggleMarket() {
        let m = document.getElementById('market-modal');
        m.style.display = (m.style.display === 'flex') ? 'none' : 'flex';
        updateMarketUI();
    }

    function updateMarketUI() {
        document.getElementById('safety-cost').innerText = Math.ceil(safetyCost);
        document.getElementById('worker-cost').innerText = Math.ceil(workerCost);
        document.getElementById('safety-display').innerText = safetyReduction;
    }

    function setMultiplier(v) {
        currentMultiplier = v;
        document.querySelectorAll('.mult-btn').forEach(b => b.classList.remove('active'));
        document.getElementById('m' + v).classList.add('active');
        mines.forEach((_, i) => updateMineUI(i));
    }

    function createWorker() {
        let id = 'w-' + Math.random();
        let el = document.createElement('div'); el.className = 'worker-unit'; el.innerText = '👷'; el.id = id;
        document.getElementById('worker-layer').appendChild(el);
        return { id, state: 'idle', x: 0, y: 0, targetIndex: null, workTimer: 0 };
    }

    function activateMine(idx) { if (mines[idx].owned && !mines[idx].isCollapsed) mines[idx].active = true; }

    function manualRepair(idx, e) {
        e.stopPropagation(); mines[idx].repairProgress += 5;
        if (mines[idx].repairProgress >= 100) { mines[idx].isCollapsed = false; mines[idx].active = true; mines[idx].repairProgress = 0; updateMineUI(idx); }
        else { document.getElementById(`repair-bar-${idx}`).style.width = mines[idx].repairProgress + '%'; }
    }

    let lastTime = Date.now();
    function gameLoop() {
        let now = Date.now(); let diff = now - lastTime; lastTime = now;
        
        mines.forEach((m, i) => {
            if (m.owned && m.active && !m.isCollapsed) {
                m.progress += (diff / m.speed) * 100;
                if (m.progress >= 100) {
                    m.progress = 0; money += m.yield;
                    let rect = document.getElementById(`mine-${i}`).getBoundingClientRect();
                    spawnParticle(rect.left + 20, rect.top, `+$${m.yield.toFixed(1)}`);
                    
                    let risk = Math.max(1, ((m.yieldLvl + m.speedLvl) * 0.2) - safetyReduction);
                    if (Math.random() * 100 < risk) { m.isCollapsed = true; m.active = false; updateMineUI(i); }
                }
                let bar = document.getElementById(`bar-${i}`);
                if (bar) bar.style.width = m.progress + '%';
            }
        });

        document.getElementById('total-balance').innerText = money.toLocaleString(undefined, {minimumFractionDigits:2});
        requestAnimationFrame(gameLoop);
    }

    initMines();
    renderGrid();
    gameLoop();
</script>

</body>
</html>
