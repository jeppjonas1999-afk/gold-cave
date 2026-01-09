<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: white;
            font-family: sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .game-box {
            background: #2c3e50;
            padding: 30px;
            border-radius: 20px;
            text-align: center;
            width: 350px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }
        .balance-display {
            font-size: 2.5rem;
            color: #2ecc71;
            margin-bottom: 10px;
        }
        .progress-bg {
            width: 100%;
            background: #34495e;
            height: 20px;
            border-radius: 10px;
            margin: 20px 0;
            overflow: hidden;
        }
        #progress-bar {
            width: 0%;
            height: 100%;
            background: #f1c40f;
        }
        .shop {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        button {
            padding: 15px;
            font-size: 1rem;
            font-weight: bold;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: 0.2s;
        }
        .speed-btn { background: #3498db; color: white; }
        .yield-btn { background: #9b59b6; color: white; }
        
        button:hover:not(:disabled) { opacity: 0.8; transform: translateY(-2px); }
        button:disabled { background: #7f8c8d; cursor: not-allowed; }
        
        .stats-text {
            font-size: 0.9rem;
            color: #bdc3c7;
            margin-bottom: 15px;
        }
    </style>
</head>
<body>

<div class="game-box">
    <h1>Mineshaft</h1>
    <div class="balance-display">$<span id="balance">0.00</span></div>
    
    <div class="stats-text">
        Speed: <span id="speed-label">10.0</span>s | 
        Earn: $<span id="yield-label">1.0</span>
    </div>

    <div class="progress-bg">
        <div id="progress-bar"></div>
    </div>

    <div class="shop">
        <button id="speedBtn" class="speed-btn">
            Upgrade Speed (Cost: $<span id="s-cost">5.00</span>)
        </button>
        <button id="yieldBtn" class="yield-btn">
            Upgrade Yield (Cost: $<span id="y-cost">5.00</span>)
        </button>
    </div>
</div>

<script>
    // Variables
    let money = 0;
    let yieldAmount = 1;
    let speedMs = 10000; // 10 seconds
    let speedCost = 5;
    let yieldCost = 5;
    
    let progress = 0;
    let lastTime = Date.now();

    // Elements
    const balEl = document.getElementById('balance');
    const barEl = document.getElementById('progress-bar');
    const sCostEl = document.getElementById('s-cost');
    const yCostEl = document.getElementById('y-cost');
    const sLabEl = document.getElementById('speed-label');
    const yLabEl = document.getElementById('yield-label');
    const speedBtn = document.getElementById('speedBtn');
    const yieldBtn = document.getElementById('yieldBtn');

    function mainLoop() {
        let now = Date.now();
        let diff = now - lastTime;
        lastTime = now;

        // Progress the bar
        progress += (diff / speedMs) * 100;

        if (progress >= 100) {
            progress = 0;
            money += yieldAmount;
        }

        // Update UI
        balEl.innerText = money.toFixed(2);
        barEl.style.width = progress + "%";
        sCostEl.innerText = speedCost.toFixed(2);
        yCostEl.innerText = yieldCost.toFixed(2);
        sLabEl.innerText = (speedMs / 1000).toFixed(2);
        yLabEl.innerText = yieldAmount.toFixed(2);

        // Check if buttons should be disabled
        speedBtn.disabled = (money < speedCost);
        yieldBtn.disabled = (money < yieldCost);

        requestAnimationFrame(mainLoop);
    }

    // Button Click Events
    speedBtn.onclick = () => {
        if (money >= speedCost) {
            money -= speedCost;
            speedMs /= 2;
            speedCost *= 1.5;
            progress = 0;
        }
    };

    yieldBtn.onclick = () => {
        if (money >= yieldCost) {
            money -= yieldCost;
            yieldAmount *= 2;
            yieldCost *= 1.5;
        }
    };

    // Start
    requestAnimationFrame(mainLoop);
</script>

</body>
</html>
