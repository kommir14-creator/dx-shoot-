<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DX SHOOT</title>
  <style>
    :root {
      --bg-color: #080710;
      --panel-bg: rgba(255, 255, 255, 0.05);
      --panel-border: rgba(255, 255, 255, 0.1);
      --primary-neon: #00f0ff;
      --accent-neon: #ff0055;
      --text-color: #ffffff;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
      -webkit-user-select: none;
    }

    body {
      background-color: var(--bg-color);
      color: var(--text-color);
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }

    #game-wrapper {
      position: relative;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    #hud {
      position: absolute;
      top: 15px;
      left: 15px;
      right: 15px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 12px 24px;
      background: var(--panel-bg);
      backdrop-filter: blur(10px);
      -webkit-backdrop-filter: blur(10px);
      border: 1px solid var(--panel-border);
      border-radius: 12px;
      pointer-events: none;
      z-index: 5;
    }

    .hud-card {
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    .hud-label {
      font-size: 0.75rem;
      text-transform: uppercase;
      letter-spacing: 1px;
      color: #aaa;
    }

    .hud-value {
      font-size: 1.5rem;
      font-weight: 800;
      text-shadow: 0 0 8px currentColor;
    }

    #score-val { color: var(--primary-neon); }
    #wave-val { color: #ffeb3b; }

    #canvas-container {
      position: relative;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 0 40px rgba(0, 240, 255, 0.15), 0 0 80px rgba(0, 0, 0, 0.8);
      border: 1px solid var(--panel-border);
    }

    canvas {
      display: block;
      background: radial-gradient(circle at center, #111025 0%, #05040a 100%);
      cursor: none;
    }

    .overlay-screen {
      position: absolute;
      inset: 0;
      background: rgba(8, 7, 16, 0.88);
      backdrop-filter: blur(8px);
      -webkit-backdrop-filter: blur(8px);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 20px;
      z-index: 10;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.3s ease;
    }

    .overlay-screen.active {
      opacity: 1;
      pointer-events: auto;
    }

    .overlay-title {
      font-size: 3.5rem;
      font-weight: 900;
      background: linear-gradient(135deg, var(--primary-neon), var(--accent-neon));
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 0 0 20px rgba(0, 240, 255, 0.3);
      text-align: center;
      letter-spacing: 3px;
    }

    .btn {
      padding: 14px 32px;
      font-size: 1.1rem;
      font-weight: 700;
      color: #fff;
      background: linear-gradient(135deg, #00f0ff, #0077ff);
      border: none;
      border-radius: 50px;
      cursor: pointer;
      box-shadow: 0 0 15px rgba(0, 240, 255, 0.4);
      transition: all 0.2s ease;
    }

    .btn:hover {
      transform: translateY(-2px) scale(1.05);
      box-shadow: 0 0 25px rgba(0, 240, 255, 0.7);
    }

    .btn:active {
      transform: translateY(0) scale(0.98);
    }
  </style>
</head>
<body>

  <div id="game-wrapper">
    <div id="hud">
      <div class="hud-card">
        <span class="hud-label">النقاط</span>
        <span id="score-val" class="hud-value">0</span>
      </div>
      <div class="hud-card">
        <span class="hud-label">المستوى</span>
        <span id="wave-val" class="hud-value">1</span>
      </div>
    </div>

    <div id="canvas-container">
      <canvas id="gameCanvas" width="800" height="600"></canvas>

      <!-- شاشة البداية -->
      <div id="start-screen" class="overlay-screen active">
        <h1 class="overlay-title">DX SHOOT</h1>
        <p style="color: #aaa; text-align: center; max-width: 80%;">
          وجه المدفع واصطد الكرات الساقطة!<br>إذا سقطت أي كرة إلى الأسفل تخسر فوراً.
        </p>
        <button class="btn" id="start-btn">ابدأ اللعب</button>
      </div>

      <!-- شاشة الخسارة -->
      <div id="game-over-screen" class="overlay-screen">
        <h1 class="overlay-title" style="background: linear-gradient(135deg, #ff0055, #ff5500); -webkit-background-clip: text;">DX SHOOT</h1>
        <p style="font-size: 1.2rem; color: #ddd; text-align: center;">
          انتهت اللعبة! سقطت إحدى الكرات.<br>مجموع نقاطك: <span id="final-score" style="color: var(--primary-neon); font-weight: bold;">0</span>
        </p>
        <button class="btn" id="restart-btn">إعادة المحاولة</button>
      </div>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const scoreEl = document.getElementById('score-val');
    const waveEl = document.getElementById('wave-val');
    const finalScoreEl = document.getElementById('final-score');

    const startScreen = document.getElementById('start-screen');
    const gameOverScreen = document.getElementById('game-over-screen');
    const startBtn = document.getElementById('start-btn');
    const restartBtn = document.getElementById('restart-btn');

    // حالة اللعبة
    let score = 0;
    let wave = 1;
    let isRunning = false;
    let animationId;
    let spawnTimer;

    // موقع النيشان/الماوس
    const mouse = {
      x: canvas.width / 2,
      y: canvas.height / 2
    };

    // مدفع الإطلاق (أسفل الشاشة)
    const shooter = {
      x: canvas.width / 2,
      y: canvas.height - 20,
      radius: 25,
      angle: 0,
      color: '#00f0ff'
    };

    // المصفوفات البرمجية
    let bullets = [];
    let targets = [];
    let particles = [];

    // تتبع موقع الماوس داخل اللعبة
    canvas.addEventListener('mousemove', (e) => {
      const rect = canvas.getBoundingClientRect();
      mouse.x = e.clientX - rect.left;
      mouse.y = e.clientY - rect.top;

      // حساب زاوية توجيه المدفع نحو الماوس
      shooter.angle = Math.atan2(mouse.y - shooter.y, mouse.x - shooter.x);
    });

    // فئة الرصاصة
    class Bullet {
      constructor(x, y, angle) {
        this.x = x;
        this.y = y;
        this.radius = 5;
        this.color = '#00f0ff';
        this.speed = 16;
        this.vx = Math.cos(angle) * this.speed;
        this.vy = Math.sin(angle) * this.speed;
      }

      draw() {
        ctx.save();
        ctx.shadowColor = this.color;
        ctx.shadowBlur = 12;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fillStyle = this.color;
        ctx.fill();
        ctx.restore();
      }

      update() {
        this.x += this.vx;
        this.y += this.vy;
      }
    }

    // فئة الهدف (الكرة)
    class Target {
      constructor() {
        this.radius = Math.random() * 12 + 16;
        this.x = Math.random() * (canvas.width - this.radius * 2) + this.radius;
        this.y = -this.radius;

        const types = [
          { color: '#ff0055', speed: 1.5 + wave * 0.2, score: 10 },
          { color: '#ffeb3b', speed: 2.2 + wave * 0.25, score: 20 },
          { color: '#00f0ff', speed: 1.8 + wave * 0.2, score: 15 }
        ];

        const selected = types[Math.floor(Math.random() * types.length)];
        this.color = selected.color;
        this.speed = selected.speed;
        this.scoreVal = selected.score;
      }

      draw() {
        ctx.save();
        ctx.shadowColor = this.color;
        ctx.shadowBlur = 12;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fillStyle = this.color;
        ctx.fill();
        ctx.restore();
      }

      update() {
        this.y += this.speed;
      }
    }

    // فئة الجزيئات (الانفجار)
    class Particle {
      constructor(x, y, color) {
        this.x = x;
        this.y = y;
        this.radius = Math.random() * 4 + 1;
        this.color = color;
        const angle = Math.random() * Math.PI * 2;
        const speed = Math.random() * 7 + 2;
        this.vx = Math.cos(angle) * speed;
        this.vy = Math.sin(angle) * speed;
        this.alpha = 1;
        this.decay = Math.random() * 0.03 + 0.015;
      }

      draw() {
        ctx.save();
        ctx.globalAlpha = this.alpha;
        ctx.shadowColor = this.color;
        ctx.shadowBlur = 8;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fillStyle = this.color;
        ctx.fill();
        ctx.restore();
      }

      update() {
        this.x += this.vx;
        this.y += this.vy;
        this.alpha -= this.decay;
      }
    }

    // رسم مدفع النيون الموجه
    function drawShooter() {
      ctx.save();
      ctx.translate(shooter.x, shooter.y);

      // رسم قاعدة المدفع الدائرية
      ctx.beginPath();
      ctx.arc(0, 0, shooter.radius, 0, Math.PI * 2);
      ctx.fillStyle = '#111025';
      ctx.strokeStyle = shooter.color;
      ctx.lineWidth = 3;
      ctx.shadowColor = shooter.color;
      ctx.shadowBlur = 15;
      ctx.fill();
      ctx.stroke();

      // رسم فوهة المدفع الموجهة نحو النيشان
      ctx.rotate(shooter.angle);
      ctx.fillStyle = shooter.color;
      ctx.fillRect(0, -6, 35, 12);

      ctx.restore();
    }

    // رسم النيشان وخط الليزر
    function drawCrosshair() {
      ctx.save();

      // خط ليزر وهمي من المدفع للنيشان
      ctx.beginPath();
      ctx.moveTo(shooter.x, shooter.y);
      ctx.lineTo(mouse.x, mouse.y);
      ctx.strokeStyle = 'rgba(0, 240, 255, 0.15)';
      ctx.lineWidth = 1;
      ctx.setLineDash([4, 4]);
      ctx.stroke();

      // رسم دائرة النيشان عند موقع الماوس
      ctx.shadowColor = shooter.color;
      ctx.shadowBlur = 10;
      ctx.beginPath();
      ctx.arc(mouse.x, mouse.y, 12, 0, Math.PI * 2);
      ctx.strokeStyle = shooter.color;
      ctx.lineWidth = 2;
      ctx.stroke();

      // نقطة المنتصف
      ctx.beginPath();
      ctx.arc(mouse.x, mouse.y, 2, 0, Math.PI * 2);
      ctx.fillStyle = '#ff0055';
      ctx.fill();

      ctx.restore();
    }

    // توليد الانفجار
    function triggerExplosion(x, y, color, count = 20) {
      for (let i = 0; i < count; i++) {
        particles.push(new Particle(x, y, color));
      }
    }

    // توليد الأهداف بانتظام
    function spawnLoop() {
      if (!isRunning) return;
      targets.push(new Target());
      
      const nextSpawn = Math.max(450, 1200 - wave * 70);
      spawnTimer = setTimeout(spawnLoop, nextSpawn);
    }

    // إطلاق النار عند الضغط
    canvas.addEventListener('click', () => {
      if (!isRunning) return;
      
      const muzzleX = shooter.x + Math.cos(shooter.angle) * 35;
      const muzzleY = shooter.y + Math.sin(shooter.angle) * 35;
      bullets.push(new Bullet(muzzleX, muzzleY, shooter.angle));
    });

    // الحلقة الرئيسية للعبة
    function gameLoop() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // رسم عناصر المدفع والنيشان
      drawShooter();

      // تحديث الجزيئات
      particles.forEach((p, index) => {
        if (p.alpha <= 0) {
          particles.splice(index, 1);
        } else {
          p.update();
          p.draw();
        }
      });

      // تحديث الرصاص
      bullets.forEach((b, bIndex) => {
        b.update();
        b.draw();

        if (b.x < 0 || b.x > canvas.width || b.y < 0 || b.y > canvas.height) {
          bullets.splice(bIndex, 1);
        }
      });

      // تحديث الأهداف والاصطدامات
      for (let tIndex = targets.length - 1; tIndex >= 0; tIndex--) {
        const t = targets[tIndex];
        t.update();
        t.draw();

        // الخسارة عند السقوط
        if (t.y + t.radius >= canvas.height) {
          triggerExplosion(t.x, canvas.height, t.color, 30);
          endGame();
          return;
        }

        // الاصطدام بالرصاص
        for (let bIndex = bullets.length - 1; bIndex >= 0; bIndex--) {
          const b = bullets[bIndex];
          const dist = Math.hypot(b.x - t.x, b.y - t.y);

          if (dist - t.radius - b.radius < 0) {
            triggerExplosion(t.x, t.y, t.color, 20);
            score += t.scoreVal;
            scoreEl.textContent = score;

            targets.splice(tIndex, 1);
            bullets.splice(bIndex, 1);

            const newWave = Math.floor(score / 100) + 1;
            if (newWave !== wave) {
              wave = newWave;
              waveEl.textContent = wave;
            }
            break;
          }
        }
      }

      // رسم النيشان فوق الكرات
      drawCrosshair();

      if (isRunning) {
        animationId = requestAnimationFrame(gameLoop);
      }
    }

    // بدء اللعبة
    function startGame() {
      score = 0;
      wave = 1;
      bullets = [];
      targets = [];
      particles = [];
      isRunning = true;

      scoreEl.textContent = score;
      waveEl.textContent = wave;

      startScreen.classList.remove('active');
      gameOverScreen.classList.remove('active');

      spawnLoop();
      gameLoop();
    }

    // إنهاء اللعبة
    function endGame() {
      isRunning = false;
      clearTimeout(spawnTimer);
      cancelAnimationFrame(animationId);
      finalScoreEl.textContent = score;
      gameOverScreen.classList.add('active');
    }

    // ربط الأزرار
    startBtn.addEventListener('click', startGame);
    restartBtn.addEventListener('click', startGame);
  </script>
</body>
</html>
