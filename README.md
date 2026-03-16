<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Brainrot Racing</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Bungee&family=Inter:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Inter', sans-serif;
            transition: background-color 0.3s, color 0.3s;
        }
        .bungee { font-family: 'Bungee', cursive; }
        .theme-dark { background-color: #0f172a; color: #f8fafc; }
        .theme-light { background-color: #f8fafc; color: #0f172a; }

        #game-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: none;
        }

        .ui-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 50; /* High z-index to ensure it is on top */
            padding: 20px;
            pointer-events: auto;
        }

        .card {
            background: rgba(15, 23, 42, 0.9);
            backdrop-filter: blur(12px);
            border: 2px solid rgba(255, 255, 255, 0.2);
            border-radius: 24px;
            padding: 2rem;
            width: 100%;
            max-width: 450px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }

        .btn { transition: all 0.2s; cursor: pointer; }
        .btn:hover { transform: scale(1.05); filter: brightness(1.2); }
        .btn:active { transform: scale(0.95); }

        .selected { border-color: #3b82f6 !important; background: rgba(59, 130, 246, 0.2) !important; }

        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            right: 20px;
            display: none;
            justify-content: space-between;
            pointer-events: none;
            z-index: 5;
        }

        #ai-shoutout-container {
            position: absolute;
            bottom: 120px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 20;
            text-align: center;
            pointer-events: none;
        }

        #shoutout-text {
            background: rgba(0, 0, 0, 0.8);
            color: #00ff00;
            padding: 10px 24px;
            border-radius: 30px;
            font-weight: bold;
            display: none;
            animation: bounce 0.5s infinite alternate;
            border: 2px solid #00ff00;
        }

        @keyframes bounce { from { transform: translateY(0); } to { transform: translateY(-10px); } }

        .loader {
            border: 3px solid rgba(255,255,255,0.3);
            border-top: 3px solid #3b82f6;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
            display: inline-block;
        }

        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

        #powerup-bar {
            width: 100%;
            height: 8px;
            background: rgba(255,255,255,0.2);
            border-radius: 4px;
            overflow: hidden;
            margin-top: 8px;
            display: none;
        }
        #powerup-fill {
            width: 100%;
            height: 100%;
            background: #facc15;
            transition: width 0.1s linear;
        }
    </style>
</head>
<body class="theme-dark">

    <!-- SIGN UP -->
    <div id="setup-screen" class="ui-overlay">
        <div class="card text-center">
            <h1 class="bungee text-4xl mb-6 text-blue-500">BRAINROT RACING</h1>
            <div class="space-y-4">
                <input type="text" id="player-name" placeholder="Skibidi Name..." class="w-full p-3 rounded-lg bg-white/10 border border-white/20 outline-none focus:border-blue-400 text-white">
                <input type="number" id="player-age" placeholder="Age (Gyatt level)" class="w-full p-3 rounded-lg bg-white/10 border border-white/20 outline-none focus:border-blue-400 text-white">
                <div class="flex gap-4">
                    <button onclick="setTheme('light')" class="flex-1 p-3 rounded-lg border-2 border-white/20 hover:bg-white/10 text-white">Light</button>
                    <button onclick="setTheme('dark')" class="flex-1 p-3 rounded-lg border-2 border-blue-500 bg-blue-500/20 text-white">Dark</button>
                </div>
                <button onclick="toSelection()" class="w-full bungee bg-blue-600 hover:bg-blue-500 text-white py-4 rounded-xl text-xl mt-4 btn">NEXT</button>
            </div>
        </div>
    </div>

    <!-- SELECTION -->
    <div id="selection-screen" class="ui-overlay" style="display: none;">
        <div class="card max-w-2xl">
            <h2 class="bungee text-2xl mb-4 text-center text-white">CHOOSE YOUR BRAINROT</h2>
            <div class="grid grid-cols-3 gap-4 mb-6">
                <div onclick="selectChar('toilet', this)" class="character-option p-2 rounded-xl text-center bg-white/5 border-2 border-transparent selected">
                    <div class="text-4xl mb-1">🚽</div><div class="text-xs font-bold text-white">Skibidi</div>
                </div>
                <div onclick="selectChar('cat', this)" class="character-option p-2 rounded-xl text-center bg-white/5 border-2 border-transparent">
                    <div class="text-4xl mb-1">😺</div><div class="text-xs font-bold text-white">Maxwell</div>
                </div>
                <div onclick="selectChar('sigma', this)" class="character-option p-2 rounded-xl text-center bg-white/5 border-2 border-transparent">
                    <div class="text-4xl mb-1">🗿</div><div class="text-xs font-bold text-white">Sigma</div>
                </div>
            </div>

            <h2 class="bungee text-2xl mb-4 text-center text-white">SELECT WHIP</h2>
            <div class="grid grid-cols-2 gap-4 mb-6">
                <div onclick="selectCar('neon', this)" class="car-option p-4 rounded-xl text-center bg-white/5 border-2 border-transparent selected">
                    <div class="text-blue-400 font-bold">NEON DRIFTER</div>
                </div>
                <div onclick="selectCar('tank', this)" class="car-option p-4 rounded-xl text-center bg-white/5 border-2 border-transparent">
                    <div class="text-red-400 font-bold">CHAD TRUCK</div>
                </div>
            </div>
            <button onclick="startGame()" class="w-full bungee bg-green-600 hover:bg-green-500 text-white py-4 rounded-xl text-2xl btn">RACE!</button>
        </div>
    </div>

    <!-- HUD -->
    <div id="hud">
        <div class="flex flex-col gap-2">
            <div class="bg-black/50 p-4 rounded-xl backdrop-blur-md border border-white/20">
                <div class="text-xs text-white opacity-70">DISTANCE</div>
                <div id="distance-val" class="bungee text-2xl text-yellow-400">0m</div>
                <div id="powerup-bar"><div id="powerup-fill"></div></div>
            </div>
            <button onclick="generateShoutout()" class="pointer-events-auto bg-purple-600 text-white text-xs p-2 rounded-lg bungee hover:bg-purple-500 btn">
                ✨ AI SHOUTOUT
            </button>
        </div>
        <div class="bg-black/50 p-4 rounded-xl backdrop-blur-md border border-white/20 text-right">
            <div class="text-xs text-white opacity-70">SPEED</div>
            <div id="speed-val" class="bungee text-2xl text-blue-400">0</div>
        </div>
    </div>

    <div id="ai-shoutout-container">
        <div id="shoutout-text"></div>
    </div>

    <!-- GAME OVER -->
    <div id="game-over" class="ui-overlay" style="display: none;">
        <div class="card text-center max-w-md">
            <h1 class="bungee text-5xl text-red-500 mb-2">L RIZZ!</h1>
            <p id="final-stats" class="mb-4 opacity-80 text-white"></p>
            <div id="ai-assessment" class="bg-black/20 p-4 rounded-xl mb-6 text-sm italic border border-white/10 min-h-[80px] flex items-center justify-center text-white">
                <div id="assessment-loading" style="display:none;"><div class="loader"></div> Calculating Rizz...</div>
                <div id="assessment-content"></div>
            </div>
            <button id="rizz-btn" onclick="getAIAssessment()" class="w-full mb-4 bg-purple-600 p-3 rounded-xl bungee text-white btn">✨ AI RIZZ ASSESSMENT</button>
            <button onclick="handleRetry()" class="w-full bungee bg-blue-600 p-4 rounded-xl text-white btn">RETRY</button>
        </div>
    </div>

    <div id="game-container"></div>

    <script>
        const apiKey = ""; 

        let gameState = {
            name: "", age: 0, theme: "dark", char: "toilet", car: "neon",
            distance: 0, speed: 0, targetSpeed: 0.5, isRacing: false,
            lane: 0, clock: new THREE.Clock(),
            powerup: null, powerupTime: 0, maxPowerupTime: 5000,
            animationId: null
        };

        async function callGemini(prompt, systemInstruction = "") {
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: { parts: [{ text: systemInstruction }] }
            };
            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const data = await response.json();
                return data.candidates?.[0]?.content?.parts?.[0]?.text || "No rizz found.";
            } catch (e) { return "L connection."; }
        }

        async function generateShoutout() {
            const shoutoutEl = document.getElementById('shoutout-text');
            shoutoutEl.style.display = 'block';
            shoutoutEl.innerText = "✨ VIBE CHECKING...";
            const powerStr = gameState.powerup ? `Currently using ${gameState.powerup} superpower!` : "No superpowers right now.";
            const prompt = `Player ${gameState.name} is at ${Math.floor(gameState.distance)}m as ${gameState.char}. ${powerStr}. One short brainrot sentence (max 8 words). Use slang like Skibidi, Gyatt, Sigma, Fanum Tax.`;
            const text = await callGemini(prompt, "You are a Gen Alpha commentator.");
            shoutoutEl.innerText = text.toUpperCase();
            setTimeout(() => { if(shoutoutEl.innerText === text.toUpperCase()) shoutoutEl.style.display = 'none'; }, 4000);
        }

        async function getAIAssessment() {
            document.getElementById('rizz-btn').style.display = 'none';
            document.getElementById('assessment-loading').style.display = 'block';
            const prompt = `Roast this run: ${gameState.name}, Score: ${Math.floor(gameState.distance)}. Give a Rizz Grade and 2 roasting sentences.`;
            const result = await callGemini(prompt, "You are the Ultimate Rizz Judge.");
            document.getElementById('assessment-loading').style.display = 'none';
            document.getElementById('assessment-content').innerText = result;
        }

        function setTheme(t) {
            gameState.theme = t;
            document.body.className = `theme-${t}`;
            event.target.parentElement.querySelectorAll('button').forEach(b => b.classList.remove('border-blue-500', 'bg-blue-500/20'));
            event.target.classList.add('border-blue-500', 'bg-blue-500/20');
        }

        function toSelection() {
            gameState.name = document.getElementById('player-name').value || "Noob";
            gameState.age = document.getElementById('player-age').value || "9";
            document.getElementById('setup-screen').style.display = 'none';
            document.getElementById('selection-screen').style.display = 'flex';
        }

        function selectChar(type, el) {
            gameState.char = type;
            document.querySelectorAll('.character-option').forEach(o => o.classList.remove('selected'));
            el.classList.add('selected');
        }

        function selectCar(type, el) {
            gameState.car = type;
            document.querySelectorAll('.car-option').forEach(o => o.classList.remove('selected'));
            el.classList.add('selected');
        }

        function handleRetry() {
            // Stop animation loop immediately
            if (gameState.animationId) cancelAnimationFrame(gameState.animationId);
            // Full clean reload to reset state
            window.location.reload();
        }

        // 3D Engine
        let scene, camera, renderer, player, road, obstacles = [], powerups = [], light;
        const laneWidth = 4;

        function init3D() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(gameState.theme === 'dark' ? 0x0f172a : 0xf8fafc);
            scene.fog = new THREE.FogExp2(scene.background, 0.02);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 5, 10);
            camera.lookAt(0, 2, 0);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            document.getElementById('game-container').innerHTML = ''; // Clear previous
            document.getElementById('game-container').appendChild(renderer.domElement);

            scene.add(new THREE.AmbientLight(0xffffff, 0.6));
            light = new THREE.DirectionalLight(0xffffff, 1);
            light.position.set(10, 20, 10);
            light.castShadow = true;
            scene.add(light);

            const roadGeo = new THREE.PlaneGeometry(20, 2000);
            const roadMat = new THREE.MeshPhongMaterial({ color: gameState.theme === 'dark' ? 0x1e293b : 0xe2e8f0 });
            road = new THREE.Mesh(roadGeo, roadMat);
            road.rotation.x = -Math.PI / 2;
            road.receiveShadow = true;
            scene.add(road);

            createPlayer();
            for(let i=0; i<8; i++) spawnObstacle(i * -50 - 60);
            for(let i=0; i<2; i++) spawnPowerup(i * -150 - 100);

            window.addEventListener('resize', onResize);
            window.addEventListener('keydown', handleInput);
        }

        function createPlayer() {
            player = new THREE.Group();
            const bodyGeo = new THREE.BoxGeometry(2, 0.8, 4);
            const bodyMat = new THREE.MeshPhongMaterial({ 
                color: gameState.car === 'neon' ? 0x00ffff : 0xff4444,
                transparent: true,
                opacity: 1
            });
            const carBody = new THREE.Mesh(bodyGeo, bodyMat);
            carBody.castShadow = true;
            player.add(carBody);

            const headMap = { toilet: '🚽', cat: '😺', sigma: '🗿' };
            const canvas = document.createElement('canvas');
            canvas.width = 128; canvas.height = 128;
            const ctx = canvas.getContext('2d');
            ctx.font = '80px Arial'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(headMap[gameState.char], 64, 64);
            const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: new THREE.CanvasTexture(canvas) }));
            sprite.scale.set(2, 2, 1); sprite.position.y = 2;
            player.add(sprite);
            scene.add(player);
        }

        function spawnObstacle(zPos) {
            const obs = new THREE.Mesh(new THREE.BoxGeometry(2.5, 3, 2.5), new THREE.MeshPhongMaterial({ color: 0x475569 }));
            obs.position.set((Math.floor(Math.random() * 3) - 1) * laneWidth, 1.5, zPos);
            obs.castShadow = true;
            scene.add(obs);
            obstacles.push(obs);
        }

        function spawnPowerup(zPos) {
            const type = Math.random() > 0.5 ? 'GHOST' : 'BOOST';
            const color = type === 'GHOST' ? 0xfacc15 : 0x22c55e;
            const geo = new THREE.OctahedronGeometry(1);
            const mat = new THREE.MeshPhongMaterial({ color, emissive: color, emissiveIntensity: 0.5 });
            const pup = new THREE.Mesh(geo, mat);
            pup.userData = { type };
            pup.position.set((Math.floor(Math.random() * 3) - 1) * laneWidth, 1.5, zPos);
            scene.add(pup);
            powerups.push(pup);
        }

        function handleInput(e) {
            if(!gameState.isRacing) return;
            if((e.key === 'ArrowLeft' || e.key === 'a') && gameState.lane > -1) gameState.lane--;
            if((e.key === 'ArrowRight' || e.key === 'd') && gameState.lane < 1) gameState.lane++;
        }

        function onResize() {
            if (!camera || !renderer) return;
            camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function startGame() {
            document.getElementById('selection-screen').style.display = 'none';
            document.getElementById('game-container').style.display = 'block';
            document.getElementById('hud').style.display = 'flex';
            init3D();
            gameState.isRacing = true;
            animate();
        }

        function triggerPowerup(type) {
            gameState.powerup = type;
            gameState.powerupTime = Date.now() + gameState.maxPowerupTime;
            document.getElementById('powerup-bar').style.display = 'block';
            document.getElementById('powerup-fill').style.backgroundColor = type === 'GHOST' ? '#facc15' : '#22c55e';
            if(type === 'GHOST') {
                player.children[0].material.opacity = 0.5;
            }
        }

        function gameOver() {
            gameState.isRacing = false;
            cancelAnimationFrame(gameState.animationId);
            document.getElementById('hud').style.display = 'none';
            document.getElementById('game-over').style.display = 'flex';
            document.getElementById('final-stats').innerText = `${gameState.name} reached ${Math.floor(gameState.distance)}m.`;
        }

        function animate() {
            if(!gameState.isRacing) return;
            gameState.animationId = requestAnimationFrame(animate);

            // Powerup Logic
            if(gameState.powerup) {
                const remaining = gameState.powerupTime - Date.now();
                if(remaining <= 0) {
                    if(gameState.powerup === 'GHOST') {
                        player.children[0].material.opacity = 1;
                    }
                    gameState.powerup = null;
                    document.getElementById('powerup-bar').style.display = 'none';
                } else {
                    document.getElementById('powerup-fill').style.width = `${(remaining / gameState.maxPowerupTime) * 100}%`;
                }
            }

            const currentSpeed = (gameState.powerup === 'BOOST' ? gameState.speed * 1.8 : gameState.speed);
            gameState.speed = THREE.MathUtils.lerp(gameState.speed, gameState.targetSpeed, 0.05);
            gameState.distance += currentSpeed;
            
            player.position.x = THREE.MathUtils.lerp(player.position.x, gameState.lane * laneWidth, 0.1);
            player.rotation.z = -(player.position.x - (gameState.lane * laneWidth)) * 0.1;
            road.position.z = (gameState.distance % 100);
            
            obstacles.forEach(obs => {
                obs.position.z += currentSpeed * 2;
                if(player.position.distanceTo(obs.position) < 2.5 && gameState.powerup !== 'GHOST') {
                    gameOver();
                }
                if(obs.position.z > 20) {
                    obs.position.z = -400;
                    obs.position.x = (Math.floor(Math.random() * 3) - 1) * laneWidth;
                }
            });

            powerups.forEach(pup => {
                pup.position.z += currentSpeed * 2;
                pup.rotation.y += 0.05;
                if(player.position.distanceTo(pup.position) < 2.5) {
                    triggerPowerup(pup.userData.type);
                    pup.position.z = -600; 
                }
                if(pup.position.z > 20) {
                    pup.position.z = -600 - Math.random() * 200;
                    pup.position.x = (Math.floor(Math.random() * 3) - 1) * laneWidth;
                }
            });

            gameState.targetSpeed += 0.0001;
            document.getElementById('distance-val').innerText = `${Math.floor(gameState.distance)}m`;
            document.getElementById('speed-val').innerText = `${Math.floor(currentSpeed * 400)}`;
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
