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
        .worker-count { font-size: 0.8rem; color: #bdc3c7; }
        .worker-icon-static { font-size: 1.5rem; }

        /* KODEBOKS STIL */
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
