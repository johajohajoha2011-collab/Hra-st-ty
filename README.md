<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <title>STÁTY - Strategická nadvláda</title>
    <style>
        :root { --p1: #00d4ff; --p2: #ff4d4d; --bg: #1a1a2e; }
        body { background: var(--bg); color: white; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; overflow: hidden; display: flex; }
        
        /* Menu a UI */
        #menu-overlay { position: fixed; inset: 0; background: radial-gradient(circle, #16213e 0%, #0f3460 100%); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 100; }
        .btn { padding: 20px 50px; font-size: 24px; background: none; border: 3px solid var(--p1); color: var(--p1); cursor: pointer; border-radius: 50px; transition: 0.3s; text-transform: uppercase; letter-spacing: 2px; }
        .btn:hover { background: var(--p1); color: white; box-shadow: 0 0 30px var(--p1); }
        
        #side-panel { width: 300px; background: #16213e; border-right: 2px solid #0f3460; padding: 20px; display: flex; flex-direction: column; gap: 15px; }
        .stat-card { background: #0f3460; padding: 15px; border-radius: 10px; border-bottom: 4px solid var(--p1); }
        
        /* Obchod */
        .shop-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .item { background: #1a1a2e; border: 1px solid #444; padding: 10px; text-align: center; border-radius: 5px; cursor: pointer; font-size: 13px; }
        .item:hover { border-color: var(--p1); background: #252545; }
        .item.active { border-color: #f1c40f; box-shadow: 0 0 10px #f1c40f; }

        canvas { flex-grow: 1; background: #0f3460; }
        #log { position: fixed; bottom: 20px; right: 20px; width: 300px; height: 150px; background: rgba(0,0,0,0.6); padding: 10px; font-size: 12px; overflow-y: auto; border-radius: 10px; pointer-events: none; }
    </style>
</head>
<body>

<div id="menu-overlay">
    <h1 style="font-size: 80px; margin-bottom: 10px; text-shadow: 0 0 20px var(--p1);">STÁTY</h1>
    <p style="margin-bottom: 40px; color: #aaa;">Dobij hlavní město nepřítele silou nebo technologií.</p>
    <button class="btn" onclick="startGame()">Hrát proti botovi</button>
</div>

<div id="side-panel">
    <div class="stat-card">
        <h2 id="turn-txt" style="margin: 0; color: var(--p1);">Tvůj Tah</h2>
        <div style="font-size: 24px; margin-top: 10px;">🪙 <span id="m-val">1</span></div>
        <div>V záloze: 🪖 <span id="s-val">0</span></div>
    </div>

    <h3 style="margin-bottom: 5px;">Ceník (Akce)</h3>
    <div class="shop-grid">
        <div class="item" onclick="selectAction('soldier')">Voják<br><b>1 🪙</b></div>
        <div class="item" onclick="selectAction('port')">Přístav<br><b>1 🪙</b></div>
        <div class="item" onclick="selectAction('airport')">Letiště<br><b>1 🪙</b></div>
        <div class="item" onclick="selectAction('tower')">Věž<br><b>3 🪙</b></div>
        <div class="item" onclick="selectAction('nuke_silo')">Silo<br><b>3 🪙</b></div>
        <div class="item" onclick="selectAction('nuke_bomb')">A-Bomba<br><b>2 🪙</b></div>
    </div>
    
    <div style="margin-top: auto; font-size: 12px; color: #888;">
        <b>Tip:</b> Letiště a přístavy zvyšují tvůj dosah pro zabírání a útoky na 300px.
    </div>
</div>

<canvas id="gameCanvas"></canvas>
<div id="log"></div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let territories = [];
let player = { money: 1, soldiers: 0 };
let bot = { money: 1, soldiers: 0 };
let currentTurn = 1;
let selectedAction = null;
const RANGE_DEFAULT = 150;
const RANGE_EXTENDED = 350;

function startGame() {
    document.getElementById('menu-overlay').style.display = 'none';
    canvas.width = window.innerWidth - 300;
    canvas.height = window.innerHeight;
    initMap();
    render();
}

function initMap() {
    territories = [];
    for(let i=0; i<25; i++) {
        territories.push({
            x: 50 + Math.random() * (canvas.width - 100),
            y: 50 + Math.random() * (canvas.height - 100),
            owner: 0,
            type: 'neutral',
            soldiers: 0,
            id: i
        });
    }
    // Hráč start (vlevo dole), Bot start (vpravo nahoře)
    territories[0].owner = 1; territories[0].type = 'city'; territories[0].x = 100; territories[0].y = canvas.height - 100;
    territories[24].owner = 2; territories[24].type = 'city'; territories[24].x = canvas.width - 100; territories[24].y = 100;
}

function selectAction(type) {
    if(currentTurn !== 1) return;
    selectedAction = type;
    const items = document.querySelectorAll('.item');
    items.forEach(i => i.classList.remove('active'));
    event.currentTarget.classList.add('active');
}

canvas.addEventListener('click', (e) => {
    if(currentTurn !== 1) return;
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    
    const target = territories.find(t => Math.hypot(t.x - x, t.y - y) < 40);
    if(target) executeAction(target);
});

function executeAction(target) {
    const costs = { soldier: 1, port: 1, airport: 1, tower: 3, nuke_silo: 3, nuke_bomb: 2 };
    
    // Kontrola dosahu (zda je v blízkosti nějakého našeho území)
    const canReach = territories.some(t => {
        if(t.owner !== 1) return false;
        let r = (t.type === 'airport' || t.type === 'port') ? RANGE_EXTENDED : RANGE_DEFAULT;
        return Math.hypot(t.x - target.x, t.y - target.y) < r;
    });

    if(!canReach && target.owner !== 1) {
        addLog("Moc daleko! Potřebuješ letiště nebo přístav.");
        return;
    }

    if(selectedAction === 'soldier' && player.money >= 1) {
        if(target.owner === 0) {
            target.owner = 1;
            target.soldiers += 1;
            player.money -= 1;
            addLog("Zabráno nové území.");
            endTurn();
        } else if(target.owner === 2) {
            // Útok
            if(confirm(`Poslat 1 vojáka na útok? (Nepřítel má ${target.soldiers})`)) {
                if(target.soldiers < 1) {
                    target.owner = 1;
                    target.soldiers = 1;
                    addLog("Území dobyto!");
                    if(target.type === 'city') alert("VÍTĚZSTVÍ! Dobyl jsi hlavní město.");
                } else {
                    target.soldiers--;
                    addLog("Bitva proběhla, nepřítel oslaben.");
                }
                player.money -= 1;
                endTurn();
            }
        }
    } else if(selectedAction && target.owner === 1 && player.money >= costs[selectedAction]) {
        if(target.type !== 'neutral') { addLog("Zde už budova stojí!"); return; }
        target.type = selectedAction;
        player.money -= costs[selectedAction];
        addLog(`Postaveno: ${selectedAction}`);
        endTurn();
    }
    render();
    updateUI();
}

function endTurn() {
    currentTurn = 2;
    updateUI();
    setTimeout(botTurn, 1000);
}

function botTurn() {
    bot.money++;
    // Botova logika: Najdi nejbližší volné nebo hráčovo území
    let botTerritories = territories.filter(t => t.owner === 2);
    let targets = territories.filter(t => t.owner !== 2);
    
    // Bot prostě zaútočí na první dostupný cíl v dosahu
    for(let t of botTerritories) {
        let reachable = targets.find(trg => Math.hypot(t.x - trg.x, t.y - trg.y) < RANGE_DEFAULT);
        if(reachable && bot.money >= 1) {
            if(reachable.owner === 0) {
                reachable.owner = 2;
                reachable.soldiers = 1;
            } else {
                reachable.soldiers = Math.max(0, reachable.soldiers - 1);
                if(reachable.soldiers === 0 && Math.random() > 0.5) reachable.owner = 2;
            }
            bot.money--;
            addLog("Bot provedl akci.");
            break;
        }
    }
    
    currentTurn = 1;
    player.money++;
    updateUI();
    render();
}

function render() {
    ctx.fillStyle = "#0f3460";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Vykreslení spojnic dosahu
    territories.forEach(t => {
        if(t.owner !== 0) {
            ctx.beginPath();
            ctx.strokeStyle = t.owner === 1 ? "rgba(0, 212, 255, 0.1)" : "rgba(255, 77, 77, 0.1)";
            let r = (t.type === 'airport' || t.type === 'port') ? RANGE_EXTENDED : RANGE_DEFAULT;
            ctx.arc(t.x, t.y, r, 0, Math.PI*2);
            ctx.stroke();
        }
    });

    // Vykreslení území
    territories.forEach(t => {
        ctx.beginPath();
        ctx.arc(t.x, t.y, 25, 0, Math.PI*2);
        ctx.fillStyle = t.owner === 1 ? "#00d4ff" : (t.owner === 2 ? "#ff4d4d" : "#444");
        ctx.fill();
        ctx.strokeStyle = "white";
        ctx.lineWidth = 2;
        ctx.stroke();

        ctx.fillStyle = "white";
        ctx.font = "bold 14px Arial";
        ctx.textAlign = "center";
        
        let icon = "";
        if(t.type === 'city') icon = "▣";
        if(t.type === 'port') icon = "⚓";
        if(t.type === 'airport') icon = "✈";
        if(t.type === 'tower') icon = "🗼";
        if(t.type === 'nuke_silo') icon = "☢";
        
        ctx.fillText(icon, t.x, t.y + 5);
        ctx.font = "10px Arial";
        ctx.fillText(`🪖 ${t.soldiers}`, t.x, t.y + 40);
    });
}

function updateUI() {
    document.getElementById('m-val').innerText = player.money;
    document.getElementById('turn-txt').innerText = currentTurn === 1 ? "Tvůj Tah" : "Tah Bota...";
    document.getElementById('turn-txt').style.color = currentTurn === 1 ? "var(--p1)" : "var(--p2)";
}

function addLog(msg) {
    const logDiv = document.getElementById('log');
    logDiv.innerHTML = `> ${msg}<br>${logDiv.innerHTML}`;
}
</script>
</body>
</html>
