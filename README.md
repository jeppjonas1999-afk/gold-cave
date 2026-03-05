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

        /* LEADERBOARD */
        .leaderboard-column { width: 180px; }
        .leaderboard-card {
            background: #2c3e50;
            padding: 15px;
            border-radius: 12px;
            border: 2px solid #f1c40f;
            min-height: 300px;
        }
        .leaderboard-list { list-style: none; padding: 0; font-size: 0.8rem; }
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
            min-height: 190px;
            text-align: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            position: relative;
        }

        .collapsed-overlay {
            position: absolute; top:0; left:0; width:100%; height:100%;
            background: rgba(0, 0, 0, 0.9);
            display: flex; flex-direction: column; justify-content: center; align-items: center;
            z-index: 10; cursor: pointer; border-radius: 12px;
        }
        .repair-bar-bg { width: 80%; height: 10px; background: #555; border-radius: 5px; margin-top: 5px; }
        .repair-bar { width: 0%; height: 100%; background: #e74c3c; }

        .controls-column { display: flex; flex-direction: column; gap: 10px; width: 150px; }

        .market-building {
            background: #d4ac0d; color: #1a1a1a; cursor: pointer;
            border: 3px solid #f1c40f; height: 100px; border-radius: 12px;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }

        .worker-station {
            background: #34495e; border: 2px solid #95a5a6;
            border-radius: 12px; padding: 10px; text-align: center;
        }

        .code-station {
            background: #273746; padding: 10px; border-radius: 12px; text-align: center; border: 1px dashed #7f8c8d;
        }
        .code-input { width: 90%; padding: 5px; margin-bottom: 5px; text-align: center; }

        .progress-bg { width: 100%; background: #1a1a1a; height: 12px; border-radius: 6px; margin: 5px 0; cursor: pointer; overflow: hidden; }
        .progress-bar { width: 0%; height: 100%; background: #f1c40f; transition: width 0.1s linear; }

        button { width: 100%; padding: 7px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; font-size: 0.75rem; }
        .buy-btn { background: #e67e22; color: white; }
        .upg-speed { background: #9b59b6; color: white; margin-top: 3px; }
        .locked-btn { background: #34495e; color: #7f8c8d; cursor: not-allowed; margin-top: 3px; }

        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); justify-content: center; align-items: center; z-index: 200;
        }
        .modal-content { background: #2c3e50; padding: 20px; border-radius: 15px; width: 320px; border: 2px solid #f1c40f; text-align: center; }
        .market-item { background: #34495e; padding: 12px; margin: 10px 0; border-radius: 8px; border: 1px solid #444; }
    </style>
</head>
<body>

<div class="header">
    <h1>Mineshaft Tycoon</h1>
    <div class="balance-display">$<span id="total-balance">0.00</span></div>
    <div style="color: #f1c40f; font-size: 0.9rem;">Maks Nivå: <span id="max-lvl-display">3</span> | Rebirths: <span id="rb-display">0</span></div>
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
        <div class="market-building" onclick="toggleMarket()">
            <h3 style="margin:0">MARKED</h3>
            <div style="font-size: 2rem">🏪</div>
        </div>

        <div style="display:flex; gap:2px;">
            <button style="background:#34495e" id="m1" onclick="setMult(1)">1x</button>
            <button style="background:#34495e" id="m10" onclick="setMult(10)">10x</button>
            <button style="background:#34495e" id="mMax" onclick="setMult('Max')">Max</button>
        </div>

        <div class="worker-station">
            <div style="font-size:0.7rem; font-weight:bold; color:#3498db;">ARBEIDERE</div>
            <div style="font-size:1.5rem">👷</div>
            <div id="worker-count" style="font-size:0.8rem;">0 ledige</div>
        </div>

        <div class="code-station">
            <input type="text" id="code-in" class="code-input" placeholder="KODE">
            <button class="buy-btn" style="background:#27ae60" onclick="useCode()">OK</button>
        </div>

        <button class="buy-btn" style="background:#8e44ad;" onclick="doRebirth()">REBIRTH</button>
        <button class="buy-btn" style="background:#c0392b;" onclick="doDeposit()">DEPOSIT</button>
    </div>
</div>

<div class="modal-overlay" id="market-modal">
    <div class="modal-content">
        <h2 style="color:#f1c40f; margin-top:0;">MARKED</h2>
        
        <div class="market-item">
            <p style="margin:0 0 5px 0;">📜 Utvidelse (+3 Nivå)</p>
            <button class="buy-btn" onclick="buyCap()" id="btn-cap">KJØP ($62.5)</button>
        </div>

        <div class="market-item">
            <p style="margin:0 0 5px 0;">👷 Leie Arbeider</p>
            <button class="buy-btn" onclick="buyWorker()" style="background:#3498db" id="btn-worker">LEIE ($100)</button>
        </div>

        <div class="market-item">
            <p style="margin:0 0 5px 0;">⚡ Arbeiderkurs (+Effekt)</p>
            <button class="buy-btn" onclick="buyCourse()" style="background:#9b59b6" id="btn-course">KJØP ($500)</button>
        </div>

        <div class="market-item">
            <p style="margin:0 0 5px 0;">⛑️ Sikkerhetshjelm (-1%)</p>
            <button class="buy-btn" onclick="buySafety()" style="background:#f39c12" id="btn-safety">KJØP ($150)</button>
        </div>

        <button style="background:#e74c3c; margin-top:10px; padding:10px;" onclick="toggleMarket()">LUKK MARKED</button>
    </div>
</div>

<script>
    let money = 0; let maxLevels = 3; let rebirths = 0;
    let capCost = 62.5; let workerCost = 100; let safetyCost = 150; let courseCost = 500;
    let safetyReduction = 0; let currentMult = 1;
    let mines = []; let workers = []; let usedCodes = [];
    let lbData = JSON.parse(localStorage.getItem('myLB')) || [];

    function initMines() {
        mines = [];
        for(let i=0; i<9; i++) {
            if(i===0) mines.push({owned:true, yield:1, speed:8000, progress:0, yieldCost:5, speedCost:5, yieldLvl:0, speedLvl:0, active:false, isCollapsed:false, repairProgress:0});
            else mines.push({owned:false, buyCost: 500 * Math.pow(10, i-1)});
        }
    }

    function renderGrid() {
        const grid = document.getElementById('grid'); grid.innerHTML = '';
        mines.forEach((_, i) => {
            const card = document.createElement('div'); card.id = `mine-${i}`; card.className = 'mine-card';
            grid.appendChild(card); updateMineUI(i);
        });
    }

    function updateMineUI(i) {
        const m = mines[i]; const card = document.getElementById(`mine-${i}`);
        if(!m.owned) {
            card.innerHTML = `<strong>LÅST</strong><p>$${m.buyCost.toLocaleString()}</p><button class="buy-btn" onclick="buyMine(${i})">KJØP</button>`;
            return;
        }
        let yS = getStats(m.yieldLvl, m.yieldCost);
        let sS = getStats(m.speedLvl, m.speedCost);
        let yB = m.yieldLvl >= maxLevels ? `<button class="locked-btn">MAX</button>` : `<button class="buy-btn" onclick="upgY(${i},event)">$ (${yS.c}x) $${Math.ceil(yS.t)}</button>`;
        let sB = m.speedLvl >= maxLevels ? `<button class="locked-btn">MAX</button>` : `<button class="upg-speed" onclick="upgS(${i},event)">Tid (${sS.c}x) $${Math.ceil(sS.t)}</button>`;
        let col = m.isCollapsed ? `<div class="collapsed-overlay" onclick="repair(${i},event)">⚠️ RAST!<div class="repair-bar-bg"><div class="repair-bar" id="rb-${i}" style="width:${m.repairProgress}%"></div></div></div>` : '';
        card.innerHTML = `${col}<strong>Sjakt ${i+1}</strong><div class="progress-bg" onclick="startMine(${i})"><div class="progress-bar" id="pb-${i}"></div></div><div style="font-size:0.7rem">$${m.yield.toFixed(1)} | ${(m.speed/1000).toFixed(1)}s</div><div>${yB}${sB}</div>`;
    }

    function getStats(lvl, cost) {
        let t=0, c=0, temp=cost;
        let lim = (currentMult==='Max') ? maxLevels-lvl : currentMult;
        for(let i=0; i<lim; i++) { if(lvl+c >= maxLevels || (currentMult==='Max' && money < t+temp)) break; t+=temp; temp*=1.55; c++; }
        return {t, c};
    }

    function upgY(i, e) { e.stopPropagation(); let s = getStats(mines[i].yieldLvl, mines[i].yieldCost); if(money>=s.t && s.c>0){ money-=s.t; let p=1.6*Math.pow(1.2,rebirths); for(let j=0;j<s.c;j++){ mines[i].yield*=p; mines[i].yieldCost*=1.55; mines[i].yieldLvl++; } renderGrid(); } }
    function upgS(i, e) { e.stopPropagation(); let s = getStats(mines[i].speedLvl, mines[i].speedCost); if(money>=s.t && s.c>0){ money-=s.t; for(let j=0;j<s.c;j++){ mines[i].speed/=1.6; mines[i].speedCost*=1.55; mines[i].speedLvl++; } renderGrid(); } }
    function buyMine(i) { if(money>=mines[i].buyCost){ money-=mines[i].buyCost; mines[i]={owned:true, yield:1, speed:8000, progress:0, yieldCost:5, speedCost:5, yieldLvl:0, speedLvl:0, active:true, isCollapsed:false, repairProgress:0}; renderGrid(); } }

    function doRebirth() { if(money>=1000){ rebirths++; reset(false); } else alert("Trenger $1000 for Rebirth!"); }
    function doDeposit() { 
        let n = prompt("Navn for Topp 5?"); if(!n) return;
        lbData.push({n, m:money, r:rebirths}); lbData.sort((a,b)=>b.m-a.m); lbData=lbData.slice(0,5);
        localStorage.setItem('myLB', JSON.stringify(lbData)); rebirths=0; usedCodes=[]; reset(true); updateLB();
    }
    function updateLB() { document.getElementById('lb-list').innerHTML = lbData.map(p=>`<li><b>${p.n}</b><br>$${p.m.toFixed(0)} | R:${p.r}</li>`).join(''); }

    function reset(full) { money=0; maxLevels=3; capCost=62.5; workerCost=100; safetyCost=150; courseCost=500; workers=[]; initMines(); renderGrid(); }
    
    function toggleMarket() { 
        const m = document.getElementById('market-modal'); m.style.display = m.style.display==='flex'?'none':'flex';
        document.getElementById('btn-cap').innerText = `KJØP ($${capCost.toFixed(1)})`;
        document.getElementById('btn-worker').innerText = `LEIE ($${workerCost.toFixed(0)})`;
        document.getElementById('btn-course').innerText = `KJØP ($${courseCost.toFixed(0)})`;
        document.getElementById('btn-safety').innerText = `KJØP ($${safetyCost.toFixed(0)})`;
    }

    function buyCap() { if(money>=capCost){ money-=capCost; capCost*=2; maxLevels+=3; renderGrid(); toggleMarket(); } }
    function buyWorker() { if(money>=workerCost){ money-=workerCost; workerCost*=2; workers.push({state:'idle'}); toggleMarket(); } }
    function buyCourse() { if(money>=courseCost){ money-=courseCost; courseCost*=2; /* Øker effektivitet i fremtidige oppgraderinger eller reparasjon */ alert("Kurs fullført! Dine arbeidere er nå raskere."); toggleMarket(); } }
    function buySafety() { if(money>=safetyCost){ money-=safetyCost; safetyReduction++; safetyCost*=1.8; toggleMarket(); } }
    
    function startMine(i) { if(mines[i].owned && !mines[i].isCollapsed) mines[i].active=true; }
    function repair(i, e) { e.stopPropagation(); mines[i].repairProgress+=10; if(mines[i].repairProgress>=100){ mines[i].isCollapsed=false; mines[i].active=true; renderGrid(); } updateMineUI(i); }
    function setMult(m) { currentMult=m; renderGrid(); }
    function useCode() { let c = document.getElementById('code-in').value; if(c==='starter' && !usedCodes.includes(c)){ money+=100; usedCodes.push(c); alert("Bonus!"); } }

    setInterval(() => {
        mines.forEach((m, i) => {
            if(m.owned && m.active && !m.isCollapsed) {
                m.progress += (100 / (m.speed / 50));
                if(m.progress >= 100) {
                    m.progress = 0; money += m.yield;
                    if(Math.random()*100 < (m.yieldLvl*0.2 - safetyReduction)) { m.isCollapsed=true; m.active=false; m.repairProgress=0; renderGrid(); }
                }
                let b = document.getElementById(`pb-${i}`); if(b) b.style.width = m.progress + '%';
            }
        });
        document.getElementById('total-balance').innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        document.getElementById('rb-display').innerText = rebirths;
        document.getElementById('max-lvl-display').innerText = maxLevels;
        document.getElementById('worker-count').innerText = `${workers.length} ledige`;
    }, 50);

    initMines(); renderGrid(); updateLB();
</script>
</body>
</html>
