# gold-cave
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - High Speed</title>
    <style>
        body {
            background-color: #1a1a1a;
            color: #ecf0f1;
            font-family: 'Segoe UI', sans-serif;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }

        .game-container {
            background-color: #2c3e50;
            padding: 2.5rem;
            border-radius: 20px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.6);
            text-align: center;
            width: 320px;
            border: 2px solid #34495e;
        }

        .mining-icon {
            font-size: 4rem;
            margin: 10px 0;
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }

        .stats {
            font-size: 2rem;
            font-weight: bold;
            color: #2ecc71;
            margin: 15px 0;
        }

        /* Progress bar styling */
        .progress-container {
            width: 100%;
            background-color: #34495e;
            border-radius: 10px;
            height: 12px;
            margin: 15px 0;
            overflow: hidden;
        }

        #progress-bar {
            width: 0%;
            height: 100%;
            background-color: #f1c40f;
            transition: width 0.1s linear;
        }

        .info-panel {
            background: rgba(0,0,0,0.2);
            padding: 10px;
            border-radius: 8px;
            margin-bottom: 20px;
            font-size: 0.9rem;
        }

        button {
            background-color: #e67e22;
            border: none;
            color: white;
            padding: 15px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            width: 100%;
            transition: transform 0.1s, background 0.2s;
        }

        button:active { transform: scale(0.98); }
        button:hover { background-color: #d35400; }
        button:disabled { background-color: #7f8c8d; cursor: not-allowed; opacity: 0.6; }
    </style>
</head>
<body>

    <div class="game-container">
        <h1>Mineshaft</h1>
        <div class="mining-icon">⛏️</div>
        
        <div class="stats">$<span id="balance">0.00</span></div>

        <div class="progress-container">
            <div id="progress-bar"></div>
        </div>
        
        <div class="info-panel">
            Speed: <span id="speed-display">10.00</span>s per $1<br>
            Level: <span id="level">1</span>
        </div>

        <button id="upgradeBtn" onclick="upgrade()">
            Upgrade Speed (Cost: $<span id="cost">5.00</span>)
        </button>
    </div>

    <script>
        let balance = 0;
        let upgradeCost = 5;
        let level = 1;
        
        // Speed settings (in milliseconds)
        let miningInterval = 10000; 
        let lastTick = Date.now();
        let progress = 0;

        const balanceEl = document.getElementById('balance');
        const costEl = document.getElementById('cost');
        const levelEl = document.getElementById('level');
        const speedEl = document.getElementById('speed-display');
        const bar = document.getElementById('progress-bar');
        const upgradeBtn = document.getElementById('upgradeBtn');

        // Main Game Loop (runs 60 times per second for smooth visuals)
        function gameLoop() {
            let now = Date.now();
            let deltaTime = now - lastTick;
            lastTick = now;

            // Increase progress based on time passed
            progress += (deltaTime / miningInterval) * 100;

            if (progress >= 100) {
                progress = 0;
                balance += 1; // Always earn 1 dollar
            }

            updateUI();
            requestAnimationFrame(gameLoop);
        }

        function upgrade() {
            if (balance >= upgradeCost) {
                balance -= upgradeCost;
                
                // MATH: Divide interval by 2 (double speed)
                miningInterval = miningInterval / 2;
                
                // MATH: Cost increases by 1.5x
                upgradeCost = upgradeCost * 1.5;
                
                level++;
                progress = 0; // Reset bar on upgrade to sync new speed
            }
        }

        function updateUI() {
            balanceEl.innerText = balance.toFixed(2);
            costEl.innerText = upgradeCost.toFixed(2);
            levelEl.innerText = level;
            speedEl.innerText = (miningInterval / 1000).toFixed(3);
            bar.style.width = progress + "%";

            upgradeBtn.disabled = balance < upgradeCost;
        }

        // Start the game
        requestAnimationFrame(gameLoop);
    </script>
</body>
</html>
