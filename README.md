<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <title>STÁTY - Kreslící strategie</title>
    <style>
        body { margin: 0; background: #111; color: white; font-family: sans-serif; display: flex; overflow: hidden; }
        #ui { width: 250px; background: #222; padding: 20px; border-right: 2px solid #444; z-index: 10; }
        canvas { cursor: crosshair; touch-action: none; }
        .stat { font-size: 1.2rem; margin-bottom: 10px; color: #00ff00; }
        button { width: 100%; padding: 10px; margin-top: 10px; cursor: pointer; background: #444; color: white; border: none; }
        button:hover { background: #666; }
        #log { font-size: 0.8rem; color: #aaa; margin-top: 20px; }
    </style>
</head>
<body>

<div id="ui">
    <h1>STÁTY</h1>
    <div class="stat">Peníze: <span id="money">1</span> 🪙</div>
    <div id="info">Tah: <b>Hráč</b></div>
    <hr>
    <p><b>Instrukce:</b><br>Táhni myší po mapě a "vybarvi" si nové území u svých hranic.</p>
    <button onclick="endTurn()">UKONČIT TAH</button>
    <div id="log">Vítej ve hře! Zakresli své první území kolem modrého čtverečku.</div>
</div>

<canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const uiMoney = document.getElementById('money');
const uiInfo = document.getElementById('info');

canvas.width = window.innerWidth - 250;
canvas.height = window.innerHeight;

let money = 1;
let isDrawing = false;
let turn = "player"; // player / bot
let points = []; // Historie kreslení

// Inicializace mapy (náhodné ostrovy)
function drawMap() {
    ctx.fillStyle = "#001133"; // Voda
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.fillStyle = "#223311"; // Souše
    for(let i=0; i<6; i++) {
        ctx.beginPath();
        ctx.arc(Math.random()*canvas.width, Math.random()*canvas.height, 100 + Math.random()*200, 0, Math.PI*2);
        ctx.fill();
    }

    // Hlavní města
    // Hráč (Modré)
    ctx.fillStyle = "blue";
    ctx.fillRect(50, canvas.height/2 - 25, 50, 50);
    ctx.strokeStyle = "white";
    ctx.strokeRect(50, canvas.height/2 - 25, 50, 50);

    // Bot (Červené)
    ctx.fillStyle = "red";
    ctx.fillRect(canvas.width - 100, canvas.height/2 - 25, 50, 50);
    ctx.strokeRect(canvas.width - 100, canvas.height/2 - 25, 50, 50);
}

drawMap();

// Kreslení
canvas.addEventListener('mousedown', () => { if(turn === "player" && money > 0) isDrawing = true; });
canvas.addEventListener('mouseup', () => { 
    if(isDrawing) {
        isDrawing = false;
        money--; // Každé kreslení stojí 1 minci
        uiMoney.innerText = money;
        if(money <= 0) endTurn();
    }
});

canvas.addEventListener('mousemove', (e) => {
    if(!isDrawing) return;

    const x = e.clientX - 250;
    const y = e.clientY;

    ctx.fillStyle = "rgba(0, 100, 255, 0.5)";
    ctx.beginPath();
    ctx.arc(x, y, 15, 0, Math.PI*2);
    ctx.fill();
});

function endTurn() {
    if(turn === "player") {
        turn = "bot";
        uiInfo.innerHTML = "Tah: <b style='color:red'>Počítač</b>";
        setTimeout(botPlay, 1500);
    } else {
        turn = "player";
        money = 1; // Každé kolo 1 nová mince
        uiMoney.innerText = money;
        uiInfo.innerHTML = "Tah: <b style='color:blue'>Hráč</b>";
    }
}

function botPlay() {
    // Bot náhodně zakreslí červenou čáru u své základny
    ctx.fillStyle = "rgba(255, 0, 0, 0.5)";
    let startX = canvas.width - 100;
    let startY = canvas.height / 2;
    
    for(let i=0; i<50; i++) {
        setTimeout(() => {
            ctx.beginPath();
            ctx.arc(startX - (i*2), startY + Math.sin(i/5)*20, 15, 0, Math.PI*2);
            ctx.fill();
        }, i * 20);
    }
    
    setTimeout(endTurn, 1200);
}

</script>
</body>
</html>
