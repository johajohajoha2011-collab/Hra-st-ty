<html lang="cs">
<head>
    <meta charset="UTF-8">
    <style>
        body { margin: 0; padding: 0; overflow: hidden; background: white; }
        canvas { display: block; }
    </style>
</head>
<body>

<canvas id="c"></canvas>

<script>
    const canvas = document.getElementById('c');
    const ctx = canvas.getContext('2d');
    let drawing = false;

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    window.oncontextmenu = (e) => e.preventDefault();

    canvas.onmousedown = (e) => {
        if (e.button === 2) {
            drawing = true;
            ctx.beginPath();
            ctx.moveTo(e.clientX, e.clientY);
        }
    };

    canvas.onmouseup = () => drawing = false;

    canvas.onmousemove = (e) => {
        if (drawing) {
            ctx.lineTo(e.clientX, e.clientY);
            ctx.strokeStyle = 'black';
            ctx.lineWidth = 2;
            ctx.stroke();
        }
    };

    window.onresize = () => {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    };
</script>

</body>
</html>
