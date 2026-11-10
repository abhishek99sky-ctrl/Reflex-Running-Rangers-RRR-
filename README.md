<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Cyber Dash: Neon City</title>
    <style>
        * { box-sizing: border-box; user-select: none; -webkit-user-select: none; touch-action: none; }
        body, html { margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden; background: #0b0f19; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: #fff; }
        #game-container { position: relative; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; background: #000; }
        canvas { display: block; width: 100%; height: 100%; object-fit: cover; }
        .screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; flex-direction: column; justify-content: center; align-items: center; background: rgba(11, 15, 25, 0.85); backdrop-filter: blur(8px); z-index: 10; transition: opacity 0.3s ease; }
        .hidden { opacity: 0; pointer-events: none; display: none !important; }
        h1 { font-size: 3rem; margin: 0 0 10px 0; background: linear-gradient(45deg, #00f3ff, #ff0055); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-shadow: 0 0 20px rgba(0,243,255,0.4); text-align: center; }
        p { font-size: 1.1rem; color: #a5b4fc; margin-bottom: 25px; text-align: center; max-width: 80%; }
        .btn { background: linear-gradient(135deg, #00f3ff, #0066ff); color: #000; font-weight: bold; font-size: 1.2rem; padding: 15px 40px; border: none; border-radius: 30px; cursor: pointer; box-shadow: 0 0 20px rgba(0,243,255,0.5); transition: transform 0.1s, box-shadow 0.1s; margin: 10px; text-transform: uppercase; }
        .btn:active { transform: scale(0.95); }
        .btn-secondary { background: linear-gradient(135deg, #ff0055, #7000ff); color: #fff; box-shadow: 0 0 20px rgba(255,0,85,0.5); }
        .hud { position: absolute; top: 0; left: 0; width: 100%; padding: 20px; display: flex; justify-content: space-between; align-items: flex-start; z-index: 5; pointer-events: none; font-weight: bold; font-size: 1.2rem; text-shadow: 0 2px 4px rgba(0,0,0,0.8); }
        .hud-group { display: flex; gap: 20px; align-items: center; pointer-events: auto; }
        .hud-box { background: rgba(15, 23, 42, 0.7); border: 1px solid rgba(255,255,255,0.1); padding: 8px 16px; border-radius: 12px; backdrop-filter: blur(4px); display: flex; align-items: center; gap: 8px; }
        .icon-btn { background: rgba(15, 23, 42, 0.7); border: 1px solid rgba(255,255,255,0.2); color: #fff; width: 45px; height: 45px; border-radius: 50%; display: flex; justify-content: center; align-items: center; font-size: 1.2rem; cursor: pointer; backdrop-filter: blur(4px); pointer-events: auto; transition: background 0.2s; }
        .icon-btn:hover { background: rgba(255,255,255,0.2); }
        .modal { background: rgba(15, 23, 42, 0.95); border: 1px solid rgba(0,243,255,0.3); border-radius: 24px; padding: 30px; width: 90%; max-width: 500px; max-height: 85vh; overflow-y: auto; display: flex; flex-direction: column; box-shadow: 0 0 40px rgba(0,0,0,0.8); }
        .grid-cards { display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; margin: 20px 0; width: 100%; }
        .card { background: rgba(30, 41, 59, 0.7); border: 2px solid rgba(255,255,255,0.1); border-radius: 16px; padding: 15px; text-align: center; cursor: pointer; transition: all 0.2s; }
        .card.selected { border-color: #00f3ff; background: rgba(0,243,255,0.1); box-shadow: 0 0 15px rgba(0,243,255,0.2); }
        .back-btn { align-self: flex-start; background: transparent; border: 1px solid rgba(255,255,255,0.3); color: #fff; padding: 8px 16px; border-radius: 8px; cursor: pointer; margin-bottom: 15px; font-weight: bold; }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="gameCanvas"></canvas>

        <!-- HUD -->
        <div id="hud" class="hud hidden">
            <div class="hud-group">
                <div class="hud-box">🪙 <span id="hud-coins">0</span></div>
                <div class="hud-box">📏 <span id="hud-distance">0</span>m</div>
            </div>
            <div class="hud-group">
                <div class="hud-box">⭐ <span id="hud-score">0</span></div>
                <button class="icon-btn" id="pause-btn">⏸️</button>
            </div>
        </div>

        <!-- MAIN MENU -->
        <div id="main-menu" class="screen">
            <h1>CYBER DASH</h1>
            <p>Neon City Endless Runner</p>
            <button class="btn" id="start-btn">Play Game</button>
            <button class="btn btn-secondary" id="menu-chars-btn">Characters 🐾</button>
            <button class="btn btn-secondary" id="menu-shop-btn">Shop 🛍️</button>
            <button class="btn btn-secondary" id="menu-missions-btn">Missions 🎯</button>
            <button class="btn btn-secondary" id="menu-leaderboard-btn">Leaderboard 🏆</button>
        </div>

        <!-- CHARACTER SELECTION -->
        <div id="char-screen" class="screen hidden">
            <div class="modal">
                <button class="back-btn" id="char-back-btn">← Back</button>
                <h2>Choose Animal Runner</h2>
                <div class="grid-cards" id="char-grid"></div>
            </div>
        </div>

        <!-- SHOP -->
        <div id="shop-screen" class="screen hidden">
            <div class="modal">
                <button class="back-btn" id="shop-back-btn">← Back</button>
                <h2>Upgrades & Shop</h2>
                <div class="grid-cards" id="shop-grid"></div>
            </div>
        </div>

        <!-- MISSIONS -->
        <div id="missions-screen" class="screen hidden">
            <div class="modal">
                <button class="back-btn" id="missions-back-btn">← Back</button>
                <h2>Daily Missions</h2>
                <div id="missions-list" style="display:flex; flex-direction:column; gap:10px; margin:15px 0;"></div>
            </div>
        </div>

        <!-- LEADERBOARD -->
        <div id="leaderboard-screen" class="screen hidden">
            <div class="modal">
                <button class="back-btn" id="leaderboard-back-btn">← Back</button>
                <h2>Leaderboard</h2>
                <div id="leaderboard-list" style="display:flex; flex-direction:column; gap:8px; margin:15px 0;"></div>
            </div>
        </div>

        <!-- PAUSE MENU -->
        <div id="pause-screen" class="screen hidden">
            <div class="modal" style="align-items: center; text-align: center;">
                <h2>Game Paused</h2>
                <button class="btn" id="resume-btn" style="width: 100%;">Resume</button>
                <button class="btn btn-secondary" id="restart-btn" style="width: 100%;">Restart</button>
                <button class="back-btn" id="home-btn" style="width: 100%; margin-top: 10px; text-align: center;">Home</button>
            </div>
        </div>

        <!-- GAME OVER -->
        <div id="game-over-screen" class="screen hidden">
            <div class="modal" style="align-items: center; text-align: center;">
                <h1>GAME OVER</h1>
                <p id="final-stats">Score: 0 | Coins: 0</p>
                <button class="btn" id="play-again-btn" style="width: 100%;">Play Again</button>
                <button class="btn btn-secondary" id="gameover-home-btn" style="width: 100%;">Main Menu</button>
            </div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // Game State
        let gameState = 'MENU'; // MENU, PLAYING, PAUSED, GAMEOVER
        let score = 0, coins = 0, distance = 0;
        let gameSpeed = 4.5;
        let baseSpeed = 4.5;
        let maxSpeed = 12;

        // Player / Animal Characters
        const characters = [
            { id: 'cheetah', name: 'Cheetah', unlocked: true, price: 0, color: '#f59e0b', emoji: '🐆' },
            { id: 'lion', name: 'Lion', unlocked: false, price: 200, color: '#d97706', emoji: '🦁' },
            { id: 'horse', name: 'Horse', unlocked: false, price: 350, color: '#b45309', emoji: '🐎' },
            { id: 'panther', name: 'Cyber Panther', unlocked: false, price: 500, color: '#4f46e5', emoji: '🐈‍⬛' },
            { id: 'wolf', name: 'Alpha Wolf', unlocked: false, price: 750, color: '#64748b', emoji: '🐺' }
        ];
        let selectedChar = characters[0];

        let player = {
            x: 100, y: 0, w: 50, h: 60,
            vy: 0, jumpPower: -12, gravity: 0.6,
            groundY: 0, isJumping: false, isSliding: false, slideTimer: 0,
            reset(groundLevel) {
                this.groundY = groundLevel - this.h;
                this.y = this.groundY;
                this.vy = 0;
                this.isJumping = false;
                this.isSliding = false;
            }
        };

        let obstacles = [];
        let collectibleItems = [];
        let particles = [];

        // Input Handling
        window.addEventListener('keydown', e => {
            if (e.code === 'Space' || e.code === 'ArrowUp') {
                if (gameState === 'PLAYING') jump();
            } else if (e.code === 'ArrowDown') {
                if (gameState === 'PLAYING') slide();
            } else if (e.code === 'Escape') {
                if (gameState === 'PLAYING') pauseGame();
                else if (gameState === 'PAUSED') resumeGame();
            }
        });

        let touchStartY = 0;
        window.addEventListener('touchstart', e => { touchStartY = e.touches[0].clientY; });
        window.addEventListener('touchend', e => {
            if (gameState !== 'PLAYING') return;
            let diffY = e.changedTouches[0].clientY - touchStartY;
            if (diffY < -40) jump();
            else if (diffY > 40) slide();
            else jump();
        });

        function jump() {
            if (!player.isJumping) {
                player.vy = player.jumpPower;
                player.isJumping = true;
                player.isSliding = false;
            }
        }

        function slide() {
            if (!player.isJumping) {
                player.isSliding = true;
                player.slideTimer = 40;
            }
        }

        // UI navigation bindings
        document.getElementById('start-btn').onclick = startGame;
        document.getElementById('pause-btn').onclick = pauseGame;
        document.getElementById('resume-btn').onclick = resumeGame;
        document.getElementById('restart-btn').onclick = startGame;
        document.getElementById('play-again-btn').onclick = startGame;
        document.getElementById('home-btn').onclick = goHome;
        document.getElementById('gameover-home-btn').onclick = goHome;

        document.getElementById('menu-chars-btn').onclick = () => showScreen('char-screen');
        document.getElementById('char-back-btn').onclick = () => showScreen('main-menu');
        document.getElementById('menu-shop-btn').onclick = () => showScreen('shop-screen');
        document.getElementById('shop-back-btn').onclick = () => showScreen('main-menu');
        document.getElementById('menu-missions-btn').onclick = () => { buildMissions(); showScreen('missions-screen'); };
        document.getElementById('missions-back-btn').onclick = () => showScreen('main-menu');
        document.getElementById('menu-leaderboard-btn').onclick = () => { buildLeaderboard(); showScreen('leaderboard-screen'); };
        document.getElementById('leaderboard-back-btn').onclick = () => showScreen('main-menu');

        function showScreen(screenId) {
            document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
            document.getElementById(screenId).classList.remove('hidden');
            if(screenId === 'char-screen') buildCharGrid();
            if(screenId === 'shop-screen') buildShopGrid();
        }

        function startGame() {
            document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
            document.getElementById('hud').classList.remove('hidden');
            score = 0; coins = 0; distance = 0;
            gameSpeed = baseSpeed;
            obstacles = [];
            collectibleItems = [];
            player.reset(canvas.height - 100);
            gameState = 'PLAYING';
        }

        function pauseGame() {
            gameState = 'PAUSED';
            showScreen('pause-screen');
            document.getElementById('hud').classList.add('hidden');
        }

        function resumeGame() {
            document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
            document.getElementById('hud').classList.remove('hidden');
            gameState = 'PLAYING';
        }

        function goHome() {
            gameState = 'MENU';
            document.getElementById('hud').classList.add('hidden');
            showScreen('main-menu');
        }

        function gameOver() {
            gameState = 'GAMEOVER';
            document.getElementById('hud').classList.add('hidden');
            document.getElementById('final-stats').innerText = `Score: ${Math.floor(score)} | Coins: ${coins}`;
            showScreen('game-over-screen');
            saveGameData();
        }

        // Grid Builder for Characters & Shop
        function buildCharGrid() {
            const grid = document.getElementById('char-grid');
            grid.innerHTML = '';
            characters.forEach(char => {
                const div = document.createElement('div');
                div.className = `card ${selectedChar.id === char.id ? 'selected' : ''}`;
                div.innerHTML = `<div style="font-size:2.5rem">${char.emoji}</div><h3>${char.name}</h3><p>${char.unlocked ? 'Unlocked' : 'Locked'}</p>`;
                div.onclick = () => {
                    if (char.unlocked) {
                        selectedChar = char;
                        buildCharGrid();
                    }
                };
                grid.appendChild(div);
            });
        }

        function buildShopGrid() {
            const grid = document.getElementById('shop-grid');
            grid.innerHTML = '';
            characters.forEach(char => {
                if (!char.unlocked) {
                    const div = document.createElement('div');
                    div.className = 'card';
                    div.innerHTML = `<div style="font-size:2rem">${char.emoji}</div><h3>${char.name}</h3><p>🪙 ${char.price}</p>`;
                    div.onclick = () => {
                        if (coins >= char.price) {
                            coins -= char.price;
                            char.unlocked = true;
                            buildShopGrid();
                            saveGameData();
                        } else {
                            alert('Not enough coins!');
                        }
                    };
                    grid.appendChild(div);
                }
            });
            if(grid.children.length === 0) {
                grid.innerHTML = '<p style="grid-column: span 2;">All animals unlocked!</p>';
            }
        }

        function buildMissions() {
            const list = document.getElementById('missions-list');
            list.innerHTML = `
                <div class="card" style="text-align:left;"><b>Run 500 Meters</b><br>Progress: ${Math.min(500, Math.floor(distance))}/500m</div>
                <div class="card" style="text-align:left;"><b>Collect 20 Coins</b><br>Progress: ${Math.min(20, coins)}/20</div>
            `;
        }

        function buildLeaderboard() {
            const list = document.getElementById('leaderboard-list');
            list.innerHTML = `
                <div class="card" style="display:flex; justify-content:space-between;"><span>1. CyberRunner</span><span>5,420 pts</span></div>
                <div class="card" style="display:flex; justify-content:space-between;"><span>2. NeonDash</span><span>3,890 pts</span></div>
                <div class="card" style="display:flex; justify-content:space-between; border-color:#00f3ff;"><span>3. You (Local Best)</span><span>${Math.floor(score)} pts</span></div>
            `;
        }

        // Persistence
        function saveGameData() {
            const saveData = {
                coins,
                unlocked: characters.map(c => c.unlocked)
            };
            localStorage.setItem('cyber_dash_save', JSON.stringify(saveData));
        }

        function loadGameData() {
            const data = localStorage.getItem('cyber_dash_save');
            if (data) {
                try {
                    const parsed = JSON.parse(data);
                    coins = parsed.coins || 0;
                    if(parsed.unlocked) {
                        parsed.unlocked.forEach((val, idx) => { if(characters[idx]) characters[idx].unlocked = val; });
                    }
                } catch(e){}
            }
        }
        loadGameData();

        // Game Loop & Spawning
        let obstacleTimer = 0;
        let coinTimer = 0;

        function update(dt) {
            if (gameState !== 'PLAYING') return;

            distance += gameSpeed * 0.1;
            score += gameSpeed * 0.2;
            gameSpeed = Math.min(maxSpeed, baseSpeed + distance * 0.002);

            // Player physics
            player.vy += player.gravity;
            player.y += player.vy;
            let groundLevel = canvas.height - 100;
            if (player.y >= groundLevel - player.h) {
                player.y = groundLevel - player.h;
                player.vy = 0;
                player.isJumping = false;
            }

            if (player.isSliding) {
                player.slideTimer--;
                if (player.slideTimer <= 0) player.isSliding = false;
            }

            // Obstacles spawning
            obstacleTimer++;
            if (obstacleTimer > Math.max(60, 120 - gameSpeed * 6)) {
                obstacleTimer = 0;
                let isLow = Math.random() > 0.5;
                obstacles.push({
                    x: canvas.width,
                    y: isLow ? groundLevel - 40 : groundLevel - 110,
                    w: 40,
                    h: isLow ? 40 : 50,
                    passed: false
                });
            }

            // Coins spawning
            coinTimer++;
            if (coinTimer > 50) {
                coinTimer = 0;
                collectibleItems.push({
                    x: canvas.width,
                    y: groundLevel - 80 - Math.random() * 60,
                    radius: 12
                });
            }

            // Move Obstacles
            for (let i = obstacles.length - 1; i >= 0; i--) {
                let obs = obstacles[i];
                obs.x -= gameSpeed;

                // Collision detection
                let pH = player.isSliding ? 30 : player.h;
                let pY = player.isSliding ? player.y + 30 : player.y;
                if (player.x < obs.x + obs.w && player.x + player.w > obs.x && pY < obs.y + obs.h && pY + pH > obs.y) {
                    gameOver();
                }

                if (obs.x + obs.w < 0) obstacles.splice(i, 1);
            }

            // Move Coins
            for (let i = collectibleItems.length - 1; i >= 0; i--) {
                let c = collectibleItems[i];
                c.x -= gameSpeed;

                let dx = (player.x + player.w/2) - c.x;
                let dy = (player.y + player.h/2) - c.y;
                if (Math.hypot(dx, dy) < 35) {
                    coins += 1;
                    collectibleItems.splice(i, 1);
                } e# Reflex-Running-Rangers-RRR-
⚡ Cyber Dash: Neon City is a fast-paced 2.5D endless runner playable directly in your browser! Dash through dynamic neon worlds, dodge obstacles, collect coins, and unlock awesome animal runners like the Cheetah, Lion, and Cyber Panther. Features smooth tap/swipe &amp; keyboard controls, power-ups, and high-score tracking. Play now! 🐾
