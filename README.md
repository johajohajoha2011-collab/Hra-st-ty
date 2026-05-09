<html lang="cs">
<head>
    <meta charset="UTF-8">
    <style>
        body { margin: 0; background: white; overflow: hidden; }
        canvas { display: block; background: white; }
    </style>
</head>
<body>

<canvas id="c"></canvas>

<script>
    const canvas = document.getElementById('c');
    const ctx = canvas.getContext('2d');
    let drawing = false;

    // Nastavení velikosti a vynucení bílého pozadí
    function setup() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        ctx.fillStyle = "white";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    setup();

    // Blokace pravého kliku
    window.oncontextmenu = (e) => e.preventDefault();

    canvas.onmousedown = (e) => {
        if (e.button === 2) {
            drawing = true;
            ctx.beginPath();
            ctx.moveTo(e.clientX, e.clientY);
        }
    };

    canvas.onmouseup = () => {
        drawing = false;
        ctx.closePath();
    };

    canvas.onmousemove = (e) => {
        if (drawing) {
            ctx.lineTo(e.clientX, e.clientY);
            ctx.strokeStyle = 'black';
            ctx.lineWidth = 3;
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';
            ctx.stroke();
        }
    };

    window.onresize = setup;
</script>

</body>
</html>
