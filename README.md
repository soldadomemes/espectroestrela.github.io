<!DOCTYPE html>
<html lang="pt-BR">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Espectrograma Compacto</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link
        href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;600;800&family=Space+Mono:wght@400;700&display=swap"
        rel="stylesheet">
    <style>
        :root {
            --primary-color: #00f2ff;
            --bg-color: transparent;
            /* Transparent for slide integration */
            --card-bg: #050508;
            --text-color: #e0e0e0;
            --star-tint: #ffffff;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'Outfit', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
        }

        /* Main Compact Card */
        .slide-card {
            background: var(--card-bg);
            width: 500px;
            max-width: 500px;
            /* Compact width */
            border-radius: 20px;
            padding: 1.5rem;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            border: 1px solid rgba(255, 255, 255, 0.1);
            display: flex;
            flex-direction: column;
            gap: 1rem;
        }

        /* Star Visualization - Zoomed In */
        .star-display {
            position: relative;
            /* Fixed height for the visual */
            border-radius: 12px;
            overflow: hidden;
            background: #000;
        }

        .star-container {
            width: 100%;
            height: 100%;
            position: relative;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .star-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
            /* Zoom in to make star occupy the frame */
            filter: contrast(1.2) brightness(1.1);
        }

        .star-tint-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: var(--star-tint);
            mix-blend-mode: multiply;
            z-index: 2;
            pointer-events: none;
        }

        /* Controls */
        .controls-container {
            display: flex;
            flex-direction: column;
            gap: 0.8rem;
        }

        .control-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-family: 'Space Mono', monospace;
            font-size: 0.8rem;
            color: #aaa;
        }

        .value-display {
            color: var(--primary-color);
            font-weight: bold;
        }

        input[type="range"] {
            -webkit-appearance: none;
            width: 100%;
            height: 4px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 2px;
            outline: none;
            margin-top: 0.3rem;
        }

        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            background: var(--primary-color);
            border-radius: 50%;
            cursor: pointer;
            transition: transform 0.1s;
        }

        input[type="range"]::-webkit-slider-thumb:hover {
            transform: scale(1.2);
        }

        /* Graph */
        .graph-container {
            width: 100%;
            height: 120px;
            /* Smaller graph */
            background: rgba(255, 255, 255, 0.02);
            border-radius: 8px;
            padding: 0.5rem;
            position: relative;
        }

        canvas {
            width: 100%;
            height: 100%;
        }

        /* Stats Row */
        .stats-row {
            display: flex;
            justify-content: space-between;
            font-size: 0.75rem;
            color: #888;
            border-top: 1px solid rgba(255, 255, 255, 0.1);
            padding-top: 0.8rem;
        }

        .stat-item strong {
            color: #fff;
            font-family: 'Space Mono', monospace;
        }
    </style>
</head>

<body>

    <div class="slide-card">
        <!-- Star Visual -->
        <div class="star-display">
            <div class="star-container">
                <img src="https://media.discordapp.net/attachments/1443362129565450370/1443399265249399046/white-dwarf-in-space-a-star-at-the-end-of-its-life-cycle.png?ex=6928edd1&is=69279c51&hm=495e89425f3c649a642c557450d9eb98bd20d0eeb72bb3a069e8c4f96ce91558&=&format=webp&quality=lossless"
                    alt="Estrela" class="star-image">
                <div class="star-tint-overlay" id="starTint"></div>
            </div>
        </div>

        <!-- Controls -->
        <div class="controls-container">
            <div>
                <div class="control-row">
                    <span>TEMPERATURA</span>
                    <span class="value-display" id="tempValue">5778 K</span>
                </div>
                <input type="range" id="tempSlider" min="1000" max="40000" value="5778" step="100">
            </div>

            <div>
                <div class="control-row">
                    <span>RESOLUÇÃO (ONDAS)</span>
                    <span class="value-display" id="waveValue">100</span>
                </div>
                <input type="range" id="waveSlider" min="10" max="300" value="100" step="10">
            </div>
        </div>

        <!-- Graph -->
        <div class="graph-container">
            <canvas id="spectrogramCanvas"></canvas>
        </div>

        <!-- Footer Stats -->
        <div class="stats-row">
            <div class="stat-item">Pico: <strong id="peakDisplay">501 nm</strong></div>
            <div class="stat-item">Cor: <strong id="colorDisplay">#FFFFFF</strong></div>
            <div class="stat-item">Classe: <strong id="classDisplay">G</strong></div>
        </div>
    </div>

    <script>
        // Constants
        const h = 6.626e-34;
        const c = 3.0e8;
        const k = 1.38e-23;
        const b = 2.898e-3;

        // DOM Elements
        const tempSlider = document.getElementById('tempSlider');
        const waveSlider = document.getElementById('waveSlider');
        const tempValue = document.getElementById('tempValue');
        const waveValue = document.getElementById('waveValue');
        const peakDisplay = document.getElementById('peakDisplay');
        const colorDisplay = document.getElementById('colorDisplay');
        const classDisplay = document.getElementById('classDisplay');
        const starTint = document.getElementById('starTint');
        const canvas = document.getElementById('spectrogramCanvas');
        const ctx = canvas.getContext('2d');

        // State
        let temperature = 5778;
        let numWaves = 100;

        function init() {
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);

            tempSlider.addEventListener('input', (e) => {
                temperature = parseInt(e.target.value);
                update();
            });

            waveSlider.addEventListener('input', (e) => {
                numWaves = parseInt(e.target.value);
                update();
            });

            update();
        }

        function resizeCanvas() {
            canvas.width = canvas.parentElement.clientWidth;
            canvas.height = canvas.parentElement.clientHeight;
            update();
        }

        function planckLaw(wavelength, T) {
            const w = wavelength;
            const p1 = (2 * h * Math.pow(c, 2)) / Math.pow(w, 5);
            const p2 = 1 / (Math.exp((h * c) / (w * k * T)) - 1);
            return p1 * p2;
        }

        function wienDisplacement(T) {
            return b / T;
        }

        function kelvinToRGB(kelvin) {
            let temp = kelvin / 100;
            let r, g, b;

            if (temp <= 66) {
                r = 255;
                g = temp;
                g = 99.4708025861 * Math.log(g) - 161.1195681661;
                if (temp <= 19) b = 0;
                else {
                    b = temp - 10;
                    b = 138.5177312231 * Math.log(b) - 305.0447927307;
                }
            } else {
                r = temp - 60;
                r = 329.698727446 * Math.pow(r, -0.1332047592);
                g = temp - 60;
                g = 288.1221695283 * Math.pow(g, -0.0755148492);
                b = 255;
            }
            return { r: clamp(r, 0, 255), g: clamp(g, 0, 255), b: clamp(b, 0, 255) };
        }

        function clamp(val, min, max) { return Math.min(Math.max(val, min), max); }

        function rgbToHex(r, g, b) {
            return "#" + ((1 << 24) + (Math.round(r) << 16) + (Math.round(g) << 8) + Math.round(b)).toString(16).slice(1).toUpperCase();
        }

        function getStarClass(temp) {
            if (temp >= 30000) return 'O';
            if (temp >= 10000) return 'B';
            if (temp >= 7500) return 'A';
            if (temp >= 6000) return 'F';
            if (temp >= 5200) return 'G';
            if (temp >= 3700) return 'K';
            return 'M';
        }

        function drawSpectrogram() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            const minLambda = 100e-9;
            const maxLambda = 2000e-9;
            const range = maxLambda - minLambda;
            const peakLambda = wienDisplacement(temperature);
            const maxIntensity = planckLaw(peakLambda, temperature);
            const barWidth = canvas.width / numWaves;

            for (let i = 0; i < numWaves; i++) {
                const x = i * barWidth;
                const lambda = minLambda + (i / numWaves) * range;
                const intensity = planckLaw(lambda, temperature);
                const normalizedHeight = (intensity / maxIntensity) * (canvas.height * 0.9);

                const nm = lambda * 1e9;
                let fillStyle;
                if (nm >= 380 && nm <= 780) {
                    const hue = 270 - ((nm - 380) / (780 - 380)) * 270;
                    fillStyle = `hsl(${hue}, 100%, 50%)`;
                } else if (nm < 380) fillStyle = '#4b0082';
                else fillStyle = '#8b0000';

                ctx.fillStyle = fillStyle;
                ctx.fillRect(x, canvas.height - normalizedHeight, barWidth - 0.5, normalizedHeight);
            }
        }

        function update() {
            tempValue.textContent = `${temperature} K`;
            waveValue.textContent = numWaves;

            const peakLambda = wienDisplacement(temperature);
            peakDisplay.textContent = `${(peakLambda * 1e9).toFixed(0)} nm`;
            classDisplay.textContent = getStarClass(temperature);

            const rgb = kelvinToRGB(temperature);
            const hex = rgbToHex(rgb.r, rgb.g, rgb.b);
            colorDisplay.textContent = hex;
            document.documentElement.style.setProperty('--star-tint', hex);

            drawSpectrogram();
        }

        init();
    </script>
</body>

</html>
