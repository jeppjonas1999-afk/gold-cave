<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mineshaft Tycoon - Professional</title>
    <style>
        body {
            background-color: #121212;
            color: #e0e0e0;
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }

        .game-card {
            background-color: #1e1e1e;
            padding: 2rem;
            border-radius: 24px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.8);
            text-align: center;
            width: 350px;
            border: 1px solid #333;
        }

        .mining-area {
            font-size: 5rem;
            margin: 10px 0;
            position: relative;
        }

        .stats-main {
            font-size: 2.5rem;
            font-weight: 800;
            color: #2ecc71;
            margin: 10px 0;
        }

        .progress-container {
            width: 100%;
            background-color: #333;
            border-radius: 20px;
            height: 15px;
            margin: 20px 0;
            overflow: hidden;
            border: 2px solid #444;
        }

        #progress-bar {
            width: 0%;
            height: 100%;
            background: linear-gradient(90deg, #f1c40f, #f39c12);
            transition: width 0.05s linear;
        }

        .upgrade-shop {
            display: flex;
            flex-direction: column;
            gap: 12px;
            margin-top: 20px;
        }

        .upgrade-btn {
            padding: 14px;
            border: none;
            border-radius: 12px;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 4px;
        }

        /* Speed Upgrade Blue */
        #speedBtn { background-color: #3498db; }
        #speedBtn:hover { background-color: #2980b9; }

        /* Yield Upgrade Purple */
        #yieldBtn { background-color: #9b59b6; }
        #yieldBtn:hover { background-color: #8e44ad; }

        .upgrade-btn:disabled {
            background-color: #444 !important;
            color: #888;
            cursor: not-allowed;
            transform: scale(1) !important;
        }

        .upgrade-btn:active { transform: scale(0.96); }

        .btn-subtext {
            font-size: 0.75rem;
            opacity: 0.9;
            font-weight: normal;
        }

        .info-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin: 15px 0;
            font-size: 0.85rem;
            background: rgba(255,255,255,0.05);
            padding: 10px;
            border-radius: 10px;
        }
    </style>
</head>
<body>

    <div class="game-card">
        <div class="mining-area" id="mine-icon">⛏️</div>
        <div class="stats-main">$<span id="balance">0.00</span></div>

        <div class="progress-container">
            <div id="progress-bar"></div>
        </div>

        <div class="info-grid">
            <div>Speed: <span id="speed-val">10.0</span>s</div>
            <div>Yield: $<span id="yield-val">1.0</span></div>
        </div>

        <div class="upgrade-shop">
            <button id="speedBtn" class="upgrade-btn" onclick="upgradeSpeed()">
                Upgrade Speed
                <span id="speed-cost
