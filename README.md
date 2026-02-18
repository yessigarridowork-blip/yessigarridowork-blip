<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Generador de Video Anonymous - Yessi</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow-y: auto;
            background-color: #0a0a0a;
            font-family: 'Segoe UI', sans-serif;
            color: white;
        }

        #container {
            position: relative;
            width: 100%;
            height: 350px; 
            background: #000;
            display: flex;
            align-items: center;
            justify-content: center;
            border-bottom: 3px solid #ff2d55;
        }

        canvas {
            display: block;
            max-width: 100%;
            max-height: 100%;
        }

        .ui-panel {
            padding: 40px 20px;
            text-align: center;
        }

        .btn-record {
            background: #ff2d55;
            color: white;
            border: none;
            padding: 18px 40px;
            border-radius: 50px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 6px 20px rgba(255, 45, 85, 0.5);
            transition: all 0.3s ease;
        }

        .btn-record:hover {
            transform: scale(1.05);
            background: #ff4d73;
        }

        .btn-record:disabled {
            background: #333;
            cursor: not-allowed;
            box-shadow: none;
        }

        #status {
            margin-top: 20px;
            font-size: 18px;
            font-weight: bold;
            color: #ff2d55;
            display: none;
        }

        .progress-bar {
            width: 200px;
            height: 4px;
            background: #222;
            margin: 10px auto;
            display: none;
            border-radius: 2px;
            overflow: hidden;
        }

        .progress-fill {
            width: 0%;
            height: 100%;
            background: #ff2d55;
            transition: width 0.1s linear;
        }
    </style>
</head>
<body>

<div id="container">
    <canvas id="matrixCanvas"></canvas>
</div>

<div class="ui-panel">
    <h3>Exportador de Video Corregido</h3>
    <p>Ahora el texto se dibuja dentro del canvas para que aparezca en la grabación.</p>
    <button id="btn-record" class="btn-record">GRABAR VIDEO (.WEBM)</button>
    
    <div id="status">GRABANDO...</div>
    <div class="progress-bar" id="progress-container">
        <div class="progress-fill" id="progress-bar"></div>
    </div>
</div>

<script>
    const canvas = document.getElementById('matrixCanvas');
    const ctx = canvas.getContext('2d');
    
    // Resolución para el banner
    canvas.width = 1200;
    canvas.height = 450;

    const letters = "ｱｲｳｴｵｶｷｸｹｺｻｼｽｾｿﾀﾁﾂﾃﾄﾅﾆﾇﾈﾉﾊﾋﾌﾍﾎﾏﾐﾑﾒﾓﾔﾕﾖﾗﾘﾙﾚﾛﾜﾝ0123456789";
    const fontSize = 18;
    const columns = Math.floor(canvas.width / fontSize);
    const drops = Array(columns).fill(1);

    function draw() {
        // Fondo con rastro
        ctx.fillStyle = "rgba(0, 0, 0, 0.12)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        // Dibujar lluvia de código
        ctx.font = fontSize + "px monospace";
        for (let i = 0; i < drops.length; i++) {
            const text = letters.charAt(Math.floor(Math.random() * letters.length));
            ctx.fillStyle = Math.random() > 0.98 ? "#fff" : "#ff2d55";
            ctx.fillText(text, i * fontSize, drops[i] * fontSize);

            if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) {
                drops[i] = 0;
            }
            drops[i]++;
        }

        // --- DIBUJAR EL OVERLAY DENTRO DEL CANVAS ---
        const boxWidth = 500;
        const boxHeight = 120;
        const centerX = canvas.width / 2;
        const centerY = canvas.height / 2;

        // Fondo del cuadro central
        ctx.fillStyle = "rgba(0, 0, 0, 0.85)";
        ctx.fillRect(centerX - boxWidth/2, centerY - boxHeight/2, boxWidth, boxHeight);
        
        // Borde del cuadro
        ctx.strokeStyle = "#ff2d55";
        ctx.lineWidth = 3;
        ctx.strokeRect(centerX - boxWidth/2, centerY - boxHeight/2, boxWidth, boxHeight);
        
        // Sombra/Resplandor del texto
        ctx.shadowBlur = 15;
        ctx.shadowColor = "#ff2d55";
        
        // Texto
        ctx.fillStyle = "#ff2d55";
        ctx.font = "bold 45px Courier New";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText("Hello, I'm Yessi", centerX, centerY);
        
        // Limpiar sombra para que no afecte a la lluvia en el siguiente frame
        ctx.shadowBlur = 0;

        requestAnimationFrame(draw);
    }

    // Iniciar animación
    draw();

    // Lógica de grabación
    const btnRecord = document.getElementById('btn-record');
    const statusText = document.getElementById('status');
    const progressContainer = document.getElementById('progress-container');
    const progressBar = document.getElementById('progress-bar');

    btnRecord.addEventListener('click', async () => {
        btnRecord.disabled = true;
        statusText.style.display = 'block';
        progressContainer.style.display = 'block';

        const stream = canvas.captureStream(60); 
        const recorder = new MediaRecorder(stream, {
            mimeType: 'video/webm;codecs=vp9',
            videoBitsPerSecond: 6000000 
        });

        const chunks = [];
        recorder.ondataavailable = e => chunks.push(e.data);
        
        recorder.onstop = () => {
            const blob = new Blob(chunks, { type: 'video/webm' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'banner_yessi_anonymous.webm';
            a.click();
            
            statusText.innerText = "¡GRABACIÓN LISTA!";
            btnRecord.disabled = false;
            setTimeout(() => {
                statusText.style.display = 'none';
                progressContainer.style.display = 'none';
                statusText.innerText = "GRABANDO...";
            }, 3000);
        };

        recorder.start();
        
        let progress = 0;
        const duration = 5000;
        const interval = 50;
        const step = (interval / duration) * 100;

        const progressInterval = setInterval(() => {
            progress += step;
            progressBar.style.width = Math.min(progress, 100) + '%';
            if (progress >= 100) {
                clearInterval(progressInterval);
                recorder.stop();
            }
        }, interval);
    });
</script>

</body>
</html>
