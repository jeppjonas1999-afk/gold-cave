# gold-cave
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon</title>
    <style>
        body {
            background-color: #2c3e50;
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }

        .game-container {
            background-color: #34495e;
            padding: 2rem;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            text-align: center;
            width: 300px;
        }

        #mineshaft {
            font-size: 5rem;
            margin: 20px 0;
            display: block;
        }

        .stats {
            font-size: 1.5rem;
            margin-bottom: 20px;
        }

        .info {
            font-size: 0.9rem;
            color: #bdc3c7;
            margin-bottom: 20px;
        }

        button {
            background-color: #e67e22;
            border: none;
            color: white;
            padding: 15px 25px;
            font-size: 1rem;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.2s;
            width: 100%;
        }

        button:hover {
            background-color: #d35400;
        }

        button:disabled {
            background-color: #7f8c8d;
            cursor: not-allowed;
        }

        .income-text {
            color: #2ecc71;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div class="game-container">
        <h1>Mineshaft Tycoon</h1>
        
        <div id="mineshaft">🕳️</div>
        
        <div class="stats">
            Balance: $<span id="balance">0</span>
        </div>
        
        <div class="info">
            Mining Speed: $<span id="rate">1.00</span> / 10s<br>
            Level: <span id="level">1</span>
        </div>

        <button id="upgradeBtn">
            Upgrade Mineshaft (Cost: $<span id="cost">5.00</span>)
        </button>
    </div>

    <script>
        // Game State
        let balance = 0;
        let incomeRate = 1;
        let upgradeCost = 5;
        let level = 1;

        // Elements
        const balanceEl = document.getElementById('balance');
        const rateEl = document.getElementById('rate');
        const levelEl = document.getElementById('level');
        const costEl = document.getElementById('cost');
        const upgradeBtn = document.getElementById('upgradeBtn');

        // Passive Income Logic (Every 10 seconds)
        setInterval(() => {
            balance += incomeRate;
            updateDisplay();
        }, 10000);

        // Upgrade Logic
        upgradeBtn.addEventListener('click', () => {
            if (balance >= upgradeCost) {
                balance -= upgradeCost;
                
                // Logic: Double mining speed, increase cost by 1.5x
                incomeRate *= 2;
                upgradeCost *= 1.5;
                level++;

                updateDisplay();
            }
        });

        function updateDisplay() {
            balanceEl.innerText = balance.toFixed(2);
            rateEl.innerText = incomeRate.toFixed(2);
            levelEl.innerText = level;
            costEl.innerText = upgradeCost.toFixed(2);

            // Disable button if player can't afford it
            upgradeBtn.disabled = balance < upgradeCost;
        }

        // Run once to initialize button state
        updateDisplay();
    </script>
</body>
</html>
