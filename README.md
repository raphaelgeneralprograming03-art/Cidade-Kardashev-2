
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Resident Evil UI - Simulação de Cidade Kardashev</title>
  <style>
    * {
      box-sizing: border-box;
      user-select: none;
    }
    body, html {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: #020406;
      font-family: 'Consolas', 'Courier New', monospace;
      color: #00ff66;
    }

    #simCanvas {
      display: block;
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
    }

    /* Linhas de Varredura CRT (Estilo Resident Evil) */
    .scanlines {
      position: absolute;
      top: 0; left: 0; width: 100%; height: 100%;
      background: linear-gradient(
        rgba(18, 16, 16, 0) 50%, 
        rgba(0, 0, 0, 0.4) 50%
      );
      background-size: 100% 4px;
      z-index: 10;
      pointer-events: none;
    }

    /* Moldura HUD Tática */
    .hud-overlay {
      position: absolute;
      top: 0; left: 0; width: 100%; height: 100%;
      z-index: 20;
      pointer-events: none;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      padding: 20px;
    }

    .hud-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: rgba(6, 12, 8, 0.9);
      border: 1px solid #00ff66;
      border-left: 8px solid #ff0033;
      padding: 10px 20px;
      box-shadow: 0 0 15px rgba(0, 255, 102, 0.2);
    }

    .status-alert {
      color: #ff0033;
      font-weight: bold;
      animation: blink 1s infinite;
    }

    @keyframes blink {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.3; }
    }

    .hud-bottom {
      display: flex;
      justify-content: space-between;
      align-items: flex-end;
    }

    .panel-box {
      background: rgba(4, 10, 12, 0.9);
      border: 1px solid #00ff66;
      padding: 12px 18px;
      width: 320px;
      box-shadow: inset 0 0 10px rgba(0, 255, 102, 0.15);
    }

    .panel-title {
      font-size: 11px;
      color: #ffffff;
      background: #ff0033;
      padding: 2px 6px;
      margin-bottom: 8px;
      display: inline-block;
      font-weight: bold;
      letter-spacing: 1px;
    }

    .row {
      font-size: 11px;
      margin: 4px 0;
      display: flex;
      justify-content: space-between;
      border-bottom: 1px stroke rgba(0, 255, 102, 0.2);
    }

    .val {
      color: #ffffff;
      font-weight: bold;
    }

    .reticle {
      position: absolute;
      top: 50%; left: 50%;
      transform: translate(-50%, -50%);
      width: 60px; height: 60px;
      border: 1px solid rgba(0, 255, 102, 0.3);
      pointer-events: none;
      z-index: 15;
    }
    .reticle::after {
      content: '';
      position: absolute;
      top: 29px; left: -10px; width: 80px; height: 2px;
      background: rgba(255, 0, 51, 0.6);
    }
  </style>
</head>
<body>

  <div class="scanlines"></div>
  <div class="reticle"></div>

  <canvas id="simCanvas"></canvas>

  <div class="hud-overlay">
    <div class="hud-header">
      <div>
        <span style="color: #fff; font-weight: bold; font-size: 14px;">TACTICAL MONITORING: METROPOLIS SECTOR 01</span>
        <div style="font-size: 10px; color: #00ff66;">CIVILIAN POPULATION & KARDASHEV GRID SIMULATION</div>
      </div>
      <div class="status-alert">[ BIO-GRID: OPTIMAL ]</div>
    </div>

    <div class="hud-bottom">
      <div class="panel-box">
        <div class="panel-title">POPULATION & ENERGY</div>
        <div class="row"><span>👥 CIDADÃOS EM TRÂNSITO:</span><span class="energy-val" id="pedCount">250 AGENTES</span></div>
        <div class="row"><span>🏎️ VEÍCULOS TERRESTRES:</span><span class="energy-val">MAGLEV / FUSÃO</span></div>
        <div class="row"><span>🚁 TRÁFEGO AÉREO:</span><span class="energy-val">ANTIGRAVITACIONAL</span></div>
        <div class="row"><span>🚢 INFRAESTRUTURA NAVAL:</span><span class="energy-val">HIDROFÓLIO SOLAR</span></div>
      </div>

      <div class="panel-box">
        <div class="panel-title">SYSTEM DIAGNOSTIC</div>
        <div class="row"><span>REDES ENERGÉTICAS:</span><span class="val" style="color: #00ff66;">100% RENOVÁVEL</span></div>
        <div class="row"><span>MODO DE CÂMERA:</span><span class="val">VISTA DE SEGURANÇA</span></div>
        <div class="row"><span>INTEGRIDADE DA CIDADE:</span><span class="val" style="color: #00ff66;">100% SECURE</span></div>
      </div>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('simCanvas');
    const ctx = canvas.getContext('2d');

    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    // 1. Estruturas Urbanas e Edifícios Futuristas
    const buildings = [];
    for (let i = 0; i < 25; i++) {
      buildings.push({
        x: Math.random() * canvas.width,
        y: canvas.height * 0.3 + Math.random() * (canvas.height * 0.4),
        width: 40 + Math.random() * 60,
        height: 120 + Math.random() * 200,
        color: '#0d1a15'
      });
    }

    // 2. Agentes de Pessoas (Cidadãos caminhando nas calçadas/ruas)
    const pedestrians = [];
    for (let i = 0; i < 120; i++) {
      pedestrians.push({
        x: Math.random() * canvas.width,
        y: canvas.height * 0.7 + (Math.random() * 80),
        vx: (Math.random() - 0.5) * 1.2,
        vy: (Math.random() - 0.5) * 0.4,
        size: 3,
        color: '#00ffcc'
      });
    }

    // 3. Veículos Avançados (Aéreos e Terrestres)
    const vehicles = [];
    // Carros aéreos
    for (let i = 0; i < 15; i++) {
      vehicles.push({
        x: Math.random() * canvas.width,
        y: 100 + Math.random() * 200,
        speed: 3 + Math.random() * 4,
        type: 'air',
        color: '#ff0055'
      });
    }
    // Trens/Carros Maglev na Terra
    for (let i = 0; i < 10; i++) {
      vehicles.push({
        x: Math.random() * canvas.width,
        y: canvas.height * 0.68,
        speed: 5 + Math.random() * 3,
        type: 'land',
        color: '#00ff66'
      });
    }

    // Loop de Animação Principal
    function render() {
      // Limpeza de tela com rastro suave
      ctx.fillStyle = 'rgba(2, 4, 6, 0.3)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // --- Desenhar Mar/Oceano no Fundo ---
      ctx.fillStyle = 'rgba(0, 40, 80, 0.4)';
      ctx.fillRect(0, canvas.height * 0.85, canvas.width, canvas.height * 0.15);

      // --- Desenhar Prédios e Megastruturas ---
      buildings.forEach(b => {
        ctx.fillStyle = b.color;
        ctx.strokeStyle = '#00ff66';
        ctx.lineWidth = 1;
        ctx.fillRect(b.x, b.y - b.height, b.width, b.height);
        ctx.strokeRect(b.x, b.y - b.height, b.width, b.height);

        // Janelas Energizadas
        ctx.fillStyle = 'rgba(0, 255, 102, 0.6)';
        for (let wy = b.y - b.height + 10; wy < b.y - 10; wy += 20) {
          ctx.fillRect(b.x + 8, wy, 6, 6);
          ctx.fillRect(b.x + b.width - 14, wy, 6, 6);
        }
      });

      // --- Desenhar Pontes e Vias Magnéticas ---
      ctx.strokeStyle = 'rgba(0, 255, 102, 0.3)';
      ctx.lineWidth = 4;
      ctx.beginPath();
      ctx.moveTo(0, canvas.height * 0.68);
      ctx.lineTo(canvas.width, canvas.height * 0.68);
      ctx.stroke();

      // --- Atualizar e Desenhar Pessoas (Cidadãos) ---
      pedestrians.forEach(p => {
        p.x += p.vx;
        p.y += p.vy;

        if (p.x < 0 || p.x > canvas.width) p.vx *= -1;
        if (p.y < canvas.height * 0.70 || p.y > canvas.height * 0.82) p.vy *= -1;

        // Representação de figura humana tática
        ctx.fillStyle = p.color;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2); // Corpo
        ctx.fill();

        ctx.fillStyle = '#ffffff';
        ctx.beginPath();
        ctx.arc(p.x, p.y - 4, 1.5, 0, Math.PI * 2); // Cabeça
        ctx.fill();
      });

      // --- Atualizar e Desenhar Veículos ---
      vehicles.forEach(v => {
        v.x += v.speed;
        if (v.x > canvas.width + 50) v.x = -50;

        ctx.fillStyle = v.color;
        ctx.shadowColor = v.color;
        ctx.shadowBlur = 8;

        if (v.type === 'air') {
          // Carro Voador / Aeronave
          ctx.fillRect(v.x, v.y, 14, 5);
        } else {
          // Trem Maglev / Carro Terrestre
          ctx.fillRect(v.x, v.y - 6, 22, 6);
        }
        ctx.shadowBlur = 0;
      });

      requestAnimationFrame(render);
    }

    render();
  </script>
</body>
</html>
