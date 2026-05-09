<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <title>Kreslení pravým tlačítkem</title>
    <style>
        body { margin: 0; overflow: hidden; background: #1a1a1a; }
        canvas { display: block; cursor: crosshair; }
    </style>
</head>
<body>

<canvas id="canvas"></canvas>

<script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    // Nastavení velikosti plátna
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    let drawing = false;

    // Zakázání kontextového menu (pravý klik), aby nepřekáželo
    window.addEventListener('contextmenu', (e) => e.preventDefault());

    function startDrawing(e) {
        // Kontrola, zda jde o pravé tlačítko (button 2)
        if (e.button === 2) {
            drawing = true;
            draw(e);
        }
    }

    function stopDrawing() {
        drawing = false;
        ctx.beginPath(); // Reset cesty pro nový tah
    }

    function draw(e) {
        if (!drawing) return;

        ctx.lineWidth = 5;
        ctx.lineCap = 'round';
        ctx.strokeStyle = '#00ffcc';

        ctx.lineTo(e.clientX, e.clientY);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(e.clientX, e.clientY);
    }

    // Event listenery
    canvas.addEventListener('mousedown', startDrawing);
    canvas.addEventListener('mouseup', stopDrawing);
    canvas.addEventListener('mousemove', draw);
    
    // Responzivita
    window.addEventListener('resize', () => {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    });
</script>

</body>
</html>
