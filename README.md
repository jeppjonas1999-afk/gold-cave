let money = 0;
    // Start-innstillinger for den første sjakten
    let mines = [{ 
        owned: true, 
        yield: 1, 
        speed: 10000, 
        progress: 0, 
        yieldCost: 5, 
        speedCost: 5 
    }];

    // Opprett de resterende sjaktene
    for(let i = 0; i < 8; i++) {
        mines.push({ owned: false, buyCost: 500 * Math.pow(10, i) });
    }

    const gridEl = document.getElementById('grid');
    const balEl = document.getElementById('total-balance');

    function renderGrid() {
        gridEl.innerHTML = '';
        mines.forEach((mine, index) => {
            const card = document.createElement('div');
            card.id = `mine-${index}`;
            card.className = mine.owned ? 'mine-card' : 'mine-card for-sale';
            gridEl.appendChild(card);
            updateCard(index);
        });
    }

    function updateCard(index) {
        const mine = mines[index];
        const card = document.getElementById(`mine-${index}`);
        if (!card) return;

        if (mine.owned) {
            card.classList.remove('for-sale');
            card.onclick = null;
            card.innerHTML = `
                <h3>Sjakt ${index + 1}</h3>
                <div class="progress-bg"><div class="progress-bar" id="bar-${index}" style="width: ${mine.progress}%"></div></div>
                <p>$<span id="yield-val-${index}">${mine.yield.toFixed(1)}</span> / ${(mine.speed/1000).toFixed(2)}s</p>
                
                <button class="upg-btn" onclick="upgradeYield(${index})">
                    +60% Yield ($${mine.yieldCost.toFixed(0)})
                </button>
                <button class="upg-btn" style="background: #9b59b6;" onclick="upgradeSpeed(${index})">
                    +80% Speed ($${mine.speedCost.toFixed(0)})
                </button>
            `;
        } else {
            card.onclick = () => buyMine(index);
            card.innerHTML = `
                <h2 style="color: #e74c3c;">FOR SALG</h2>
                <p>Pris: $${mine.buyCost.toLocaleString()}</p>
                <button id="buy-btn-${index}" class="buy-btn" ${money < mine.buyCost ? 'disabled' : ''}>KJØP</button>
            `;
        }
    }

    function buyMine(index) {
        if (money >= mines[index].buyCost) {
            money -= mines[index].buyCost;
            mines[index] = { 
                owned: true, 
                yield: (index + 1) * 5, 
                speed: 10000, 
                progress: 0, 
                yieldCost: 5, 
                speedCost: 5 
            };
            renderGrid();
        }
    }

    function upgradeYield(index) {
        let mine = mines[index];
        if (money >= mine.yieldCost) {
            money -= mine.yieldCost;
            mine.yield *= 1.60; // +60% økning
            mine.yieldCost *= 1.75; // +75% prisøkning
            updateCard(index);
        }
    }

    function upgradeSpeed(index) {
        let mine = mines[index];
        if (money >= mine.speedCost) {
            money -= mine.speedCost;
            mine.speed *= 0.80; // Gjør tiden 20% kortere (80% av original tid)
            mine.speedCost *= 1.75; // +75% prisøkning
            updateCard(index);
        }
    }

    let lastTime = Date.now();
    function gameLoop() {
        let now = Date.now();
        let diff = now - lastTime;
        lastTime = now;

        mines.forEach((mine, index) => {
            if (mine.owned) {
                mine.progress += (diff / mine.speed) * 100;
                if (mine.progress >= 100) {
                    mine.progress = 0;
                    money += mine.yield;
                }
                const bar = document.getElementById(`bar-${index}`);
                if (bar) bar.style.width = mine.progress + "%";
            } else {
                const btn = document.getElementById(`buy-btn-${index}`);
                if (btn) btn.disabled = money < mine.buyCost;
            }
        });

        balEl.innerText = money.toLocaleString(undefined, {minimumFractionDigits: 2});
        requestAnimationFrame(gameLoop);
    }

    renderGrid();
    requestAnimationFrame(gameLoop);
