<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <title>EMPIRE CREATOR: Total War & Conquest</title>
    <style>
        :root {
            --p-blue: #00a2ff; --p-red: #ff3c00; --bg: #020617;
            --panel: #1e293b; --text: #f8fafc; --gold: #fbbf24;
        }
        body { margin: 0; padding: 0; background: var(--bg); color: var(--text); font-family: 'Segoe UI', Tahoma, sans-serif; display: flex; height: 100vh; overflow: hidden; }
        
        /* Levý panel - Statistiky a Budovy */
        #left-panel { width: 300px; background: var(--panel); border-right: 2px solid #334155; display: flex; flex-direction: column; z-index: 10; }
        .header { padding: 15px; background: #0f172a; text-align: center; font-weight: bold; font-size: 1.2rem; border-bottom: 2px solid var(--p-blue); }
        .resource-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; padding: 10px; background: #000; }
        .res { padding: 8px; font-size: 0.9rem; text-align: center; border: 1px solid #334155; border-radius: 4px; }
        .gold { color: var(--gold); } .oil { color: #10b981; } .army { color: #f43f5e; }

        /* Centrální herní plocha */
        #game-area { flex-grow: 1; position: relative; display: flex; flex-direction: column; background: #000; }
        canvas { background: #0f172a; cursor: crosshair; image-rendering: auto; box-shadow: 0 0 20px rgba(0,0,0,1); }
        
        /* Pravý panel - Obchod a Technologie */
        #right-panel { width: 300px; background: var(--panel); border-left: 2px solid #334155; padding: 15px; overflow-y: auto; }
        .section-title { font-size: 0.9rem; color: var(--p-blue); text-transform: uppercase; margin: 15px 0 5px; border-bottom: 1px solid #475569; }
        
        /* Karty v obchodě */
        .card { background: #334155; padding: 12px; margin-bottom: 10px; border-radius: 6px; cursor: pointer; border: 1px solid transparent; transition: 0.2s; position: relative; }
        .card:hover { border-color: var(--p-blue); background: #475569; }
        .card.active { border-color: var(--gold); background: #1e3a8a; }
        .card-title { font-weight: bold; font-size: 0.95rem; display: block; }
        .card-cost { font-size: 0.8rem; color: var(--gold); }
        .card-desc { font-size: 0.75rem; color: #cbd5e1; display: block; margin-top: 4px; }

        /* Spodní konzole */
        #console { height: 120px; background: rgba(0,0,0,0.8); border-top: 2px solid #334155; padding: 10px; font-family: monospace; font-size: 0.8rem; overflow-y: hidden; }
        .log-entry { margin-bottom: 4px; border-left: 3px solid var(--p-blue); padding-left: 8px; }

        /* Tlačítka */
        .btn-main { width: 100%; padding: 15px; background: #0ea5e9; border: none; color: white; font-weight: bold; cursor: pointer; margin-top: auto; }
        .btn-main:hover { background: #38bdf8; }
        .btn-main:disabled { background: #475569; cursor: not-allowed; }
    </style>
</head>
<body>

<aside id="left-panel">
    <div class="header">IMPÉRIUM</div>
    <div class="resource-grid">
        <div class="res gold">🪙 Zlato: <span id="res-gold">100</span></div>
        <div class="res oil">🛢️ Ropa: <span id="res-oil">50</span></div>
        <div class="res army">🪖 Armáda: <span id="res-army">0</span></div>
        <div class="res">🌍 Území: <span id="res-area">0%</span></div>
    </div>
    
    <div style="padding: 15px;">
        <div class="section-title">Stavby na území</div>
        <div class="card" onclick="setMode('build_mine', 40, 10)">
            <span class="card-title">Zlatý důl</span>
            <span class="card-cost">40🪙, 10🛢️</span>
            <span class="card-desc">+5 zlata / kolo. Umísti na své území.</span>
        </div>
        <div class="card" onclick="setMode('build_rig', 60, 5)">
            <span class="card-title">Ropná plošina</span>
            <span class="card-cost">60🪙, 5🛢️</span>
            <span class="card-desc">+8 ropy / kolo.</span>
        </div>
        <div class="card" onclick="setMode('build_fort', 100, 30)">
            <span class="card-title">Pevnost</span>
            <span class="card-cost">100🪙, 30🛢️</span>
            <span class="card-desc">Automaticky brání okolí.</span>
        </div>
    </div>
    <button class="btn-main" id="turn-btn" onclick="endTurn()">UKONČIT TAH (1)</button>
</aside>

<main id="game-area">
    <canvas id="mapCanvas"></canvas>
    <div id="console">
        <div class="log-entry">Vítej, veliteli. Zakresli první území kolem modré základny.</div>
    </div>
</main>

<aside id="right-panel">
    <div class="section-title">Vojenská technika</div>
    <div class="card" onclick="setMode('draw_infantry', 5, 0)">
        <span class="card-title">Pěchota (Tužka)</span>
        <span class="card-cost">5🪙 za tah</span>
        <span class="card-desc">Standardní zakreslování území.</span>
    </div>
    <div class="card" onclick="setMode('draw_tanks', 15, 2)">
        <span class="card-title">Tanková divize</span>
        <span class="card-cost">15🪙, 2🛢️</span>
        <span class="card-desc">Tlustá čára, rychlejší dobývání.</span>
    </div>
    
    <div class="section-title">Speciální operace</div>
    <div class="card" onclick="setMode('action_nuke', 300, 100)">
        <span class="card-title">Atomový úder</span>
        <span class="card-cost">300🪙, 100🛢️</span>
        <span class="card-desc">Totální vymazání plochy.</span>
    </div>
    <div class="card" onclick="setMode('action_spy', 50, 0)">
        <span class="card-title">Špionáž</span>
        <span class="card-cost">50🪙</span>
        <span class="card-desc">Odkryje kus nepřátelského území.</span>
    </div>
</aside>

<script>
    const canvas = document.getElementById('mapCanvas');
    const ctx = canvas.getContext('2d', { willReadFrequently: true });
    const consoleEl = document.getElementById('console');

    // Inicializace rozměrů
    canvas.width = window.innerWidth - 600;
    canvas.height = window.innerHeight - 120;

    // Herní data
    let player = { gold: 100, oil: 50, army: 0, area: 0, mines: 0, rigs: 0 };
    let enemy = { gold: 500, oil: 200 };
    let game = { 
        turn: 1, 
        isPlayerTurn: true, 
        mode: 'draw_infantry', 
        brushSize: 12, 
        isDrawing: false,
        lastX: 0, lastY: 0
    };

    let buildings = []; // {x, y, type, owner}

    function init() {
        // Pozadí (oceán)
        ctx.fillStyle = '#020617';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Generování pevniny
        ctx.fillStyle = '#1e293b';
        for(let i=0; i<12; i++) {
            drawSeed(Math.random()*canvas.width, Math.random()*canvas.height, 100 + Math.random()*150);
        }

        // Základny
        drawBase(80, canvas.height/2, 'blue');
        drawBase(canvas.width-80, canvas.height/2, 'red');
        
        updateUI();
    }

    function drawSeed(x, y, r) {
        ctx.beginPath();
        ctx.arc(x, y, r, 0, Math.PI*2);
        ctx.fill();
    }

    function drawBase(x, y, color) {
        ctx.shadowBlur = 15; ctx.shadowColor = color;
        ctx.fillStyle = color;
        ctx.fillRect(x-25, y-25, 50, 50);
        ctx.strokeStyle = "white";
        ctx.strokeRect(x-25, y-25, 50, 50);
        ctx.shadowBlur = 0;
        // Označení nezničitelného středu
        ctx.beginPath(); ctx.fillStyle = "white"; ctx.arc(x, y, 5, 0, Math.PI*2); ctx.fill();
    }

    // Ovládání
    function setMode(mode, goldCost, oilCost) {
        if (!game.isPlayerTurn) return;
        if (player.gold < goldCost || player.oil < oilCost) {
            log("Nedostatek surovin!", "red");
            return;
        }
        game.mode = mode;
        game.brushSize = mode === 'draw_tanks' ? 35 : (mode === 'action_nuke' ? 120 : 12);
        log(`Režim změněn na: ${mode}`);
        
        document.querySelectorAll('.card').forEach(c => c.classList.remove('active'));
        event.currentTarget.classList.add('active');
    }

    canvas.onmousedown = (e) => {
        if (!game.isPlayerTurn || player.gold <= 0) return;
        game.isDrawing = true;
        [game.lastX, game.lastY] = [e.offsetX, e.offsetY];
        
        // Pokud stavíme budovu
        if (game.mode.startsWith('build_')) {
            placeBuilding(e.offsetX, e.offsetY);
            game.isDrawing = false;
        }
    };

    canvas.onmousemove = (e) => {
        if (!game.isDrawing || !game.isPlayerTurn) return;
        
        ctx.lineJoin = 'round';
        ctx.lineCap = 'round';
        ctx.lineWidth = game.brushSize;

        if (game.mode === 'action_nuke') {
            ctx.globalCompositeOperation = 'destination-out';
            ctx.beginPath(); ctx.arc(e.offsetX, e.offsetY, game.brushSize/2, 0, Math.PI*2); ctx.fill();
            ctx.globalCompositeOperation = 'source-over';
            player.gold -= 2; player.oil -= 1;
        } else {
            ctx.strokeStyle = 'rgba(0, 162, 255, 0.3)';
            ctx.beginPath();
            ctx.moveTo(game.lastX, game.lastY);
            ctx.lineTo(e.offsetX, e.offsetY);
            ctx.stroke();
            player.gold -= 0.2;
            if (game.mode === 'draw_tanks') player.oil -= 0.05;
        }

        [game.lastX, game.lastY] = [e.offsetX, e.offsetY];
        updateUI();
    };

    window.onmouseup = () => {
        game.isDrawing = false;
        calculateArea();
    };

    function placeBuilding(x, y) {
        // Kontrola, zda je na vlastním (modrém) území
        const pixel = ctx.getImageData(x, y, 1, 1).data;
        if (pixel[2] > 100 && pixel[0] < 100) { // Je tam modrá
            let costG = game.mode === 'build_mine' ? 40 : (game.mode === 'build_rig' ? 60 : 100);
            let costO = game.mode === 'build_mine' ? 10 : (game.mode === 'build_rig' ? 5 : 30);
            
            player.gold -= costG; player.oil -= costO;
            buildings.push({x, y, type: game.mode, owner: 'player'});
            
            ctx.fillStyle = "white";
            ctx.font = "20px Arial";
            let icon = game.mode === 'build_mine' ? "💰" : (game.mode === 'build_rig' ? "⛽" : "🏰");
            ctx.fillText(icon, x-10, y+10);
            
            if (game.mode === 'build_mine') player.mines++;
            if (game.mode === 'build_rig') player.rigs++;
            
            log(`Stavba ${game.mode} dokončena.`);
            updateUI();
        } else {
            log("Budovy lze stavět jen na vlastním vybarveném území!", "red");
        }
    }

    function endTurn() {
        game.isPlayerTurn = false;
        document.getElementById('turn-btn').disabled = true;
        log("--- TAH NEPŘÍTELE ---", "orange");

        setTimeout(() => {
            // Bot AI logika
            simulateEnemy();
            
            // Produkce
            player.gold += 15 + (player.mines * 10);
            player.oil += 5 + (player.rigs * 8);
            game.turn++;
            
            game.isPlayerTurn = true;
            document.getElementById('turn-btn').disabled = false;
            document.getElementById('turn-btn').innerText = `UKONČIT TAH (${game.turn})`;
            log(`Tvůj tah. Příjem: +${15 + player.mines*10}🪙, +${5 + player.rigs*8}🛢️`);
            updateUI();
        }, 1500);
    }

    function simulateEnemy() {
        ctx.strokeStyle = 'rgba(255, 60, 0, 0.3)';
        ctx.lineWidth = 25;
        ctx.beginPath();
        let ex = canvas.width - 80;
        let ey = canvas.height / 2;
        ctx.moveTo(ex, ey);
        
        for(let i=0; i<20; i++) {
            ex -= Math.random()*50;
            ey += (Math.random()-0.5)*80;
            ctx.lineTo(ex, ey);
        }
        ctx.stroke();
        calculateArea();
    }

    function calculateArea() {
        const data = ctx.getImageData(0, 0, canvas.width, canvas.height).data;
        let p = 0;
        for(let i=0; i<data.length; i+=4) {
            if(data[i+2] > 100 && data[i] < 100) p++;
        }
        player.area = ((p / (canvas.width*canvas.height)) * 100).toFixed(1);
        updateUI();
    }

    function updateUI() {
        document.getElementById('res-gold').innerText = Math.floor(player.gold);
        document.getElementById('res-oil').innerText = Math.floor(player.oil);
        document.getElementById('res-area').innerText = player.area + "%";
        
        if (player.gold <= 0) log("Došlo zlato! Musíš ukončit tah pro zisk surovin.", "red");
    }

    function log(msg, color = "white") {
        const entry = document.createElement('div');
        entry.className = 'log-entry';
        entry.style.borderLeftColor = color === "red" ? "red" : (color === "orange" ? "orange" : "var(--p-blue)");
        entry.innerText = `[Kolo ${game.turn}] ${msg}`;
        consoleEl.prepend(entry);
    }

    init();
</script>
</body>
</html>
