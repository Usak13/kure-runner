<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8" />
  <title>Skákací kuře s animací a zvuky</title>
  <style>
    body { margin: 0; background: #87ceeb; font-family: Arial, sans-serif; text-align: center; }
    #gameCanvas { background: #3a6; display: block; margin: 20px auto; border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.3); }
    #score { font-size: 24px; color: #222; margin-top: 10px; font-weight: bold; }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="600" height="300"></canvas>
  <div id="score">Skóre: 0</div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // Sprite sheet kuřete (4 snímky chůze vedle sebe, 64x64 px každý)
    const chickenSprite = new Image();
    chickenSprite.src = 'https://i.imgur.com/CSvEFAz.png';

    // Zvuky
    const jumpSound = new Audio('https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg');
    const landSound = new Audio('https://actions.google.com/sounds/v1/impacts/wood_thud.ogg');
    const scoreSound = new Audio('https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg');

    // Hráč (kuře)
    const player = {
      x: 50,
      y: 220,
      width: 64,
      height: 64,
      vy: 0,
      gravity: 0.8,
      jumpForce: -15,
      grounded: false,
      frame: 0,
      frameCount: 4,
      frameTimer: 0,
      frameInterval: 10
    };

    // Překážky
    const obstacles = [];
    const obstacleWidth = 30;
    let gameSpeed = 5;
    let frameCounter = 0;
    let score = 0;
    let gameOver = false;

    // Skok - klávesnice a kliknutí
    document.addEventListener('keydown', e => {
      if ((e.code === 'Space' || e.code === 'ArrowUp') && !gameOver) jump();
    });
    document.addEventListener('click', () => { if (!gameOver) jump(); });

    function jump() {
      if (player.grounded) {
        player.vy = player.jumpForce;
        player.grounded = false;
        jumpSound.play();
      }
    }

    function createObstacle() {
      const y = 220;
      obstacles.push({ x: canvas.width, y, width: obstacleWidth, height: 50 });
    }

    function update() {
      if (gameOver) return;

      frameCounter++;
      if (frameCounter % 90 === 0) {
        createObstacle();
        if (gameSpeed < 12) gameSpeed += 0.2;
      }

      // Pohyb překážek
      obstacles.forEach(obs => { obs.x -= gameSpeed; });

      // Odstranění překážek mimo obrazovku a zvýšení skóre
      while (obstacles.length && obstacles[0].x + obstacleWidth < 0) {
        obstacles.shift();
        score++;
        scoreSound.play();
        document.getElementById('score').textContent = 'Skóre: ' + score;
      }

      // Gravitační pohyb hráče
      player.vy += player.gravity;
      player.y += player.vy;

      // Podlaha
      if (player.y > 220) {
        if (!player.grounded) landSound.play();
        player.y = 220;
        player.vy = 0;
        player.grounded = true;
      }

      // Kolize s překážkami
      obstacles.forEach(obs => {
        if (
          player.x < obs.x + obs.width &&
          player.x + player.width > obs.x &&
          player.y < obs.y + obs.height &&
          player.y + player.height > obs.y
        ) {
          gameOver = true;
          alert('Konec hry! Skóre: ' + score);
          location.reload();
        }
      });

      // Animace kuřete (chodící frames)
      player.frameTimer++;
      if (player.frameTimer >= player.frameInterval) {
        player.frame = (player.frame + 1) % player.frameCount;
        player.frameTimer = 0;
      }
    }

    function draw() {
      // Pozadí oblohy
      ctx.fillStyle = '#87ceeb';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Podlaha
      ctx.fillStyle = '#654321';
      ctx.fillRect(0, 270, canvas.width, 30);

      // Kuře (sprite animace)
      ctx.drawImage(
        chickenSprite,
        player.frame * player.width, 0, player.width, player.height,
        player.x, player.y, player.width, player.height
      );

      // Překážky
      ctx.fillStyle = '#d44';
      obstacles.forEach(obs => {
        ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
      });
    }

    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    chickenSprite.onload = () => {
      gameLoop();
    };
  </script>
</body>
</html>
