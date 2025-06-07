<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8" />
  <title>Skákací kuře</title>
  <style>
    body { margin: 0; overflow: hidden; background: #87ceeb; font-family: Arial, sans-serif; }
    #gameCanvas { background: #3a6; display: block; margin: 0 auto; }
    #score { color: #333; font-size: 24px; text-align: center; margin-top: 10px; }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="600" height="300"></canvas>
  <div id="score">Skóre: 0</div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // Kuře jako obrázek
    const chicken = new Image();
    chicken.src = 'https://i.imgur.com/dw7aQPw.png'; // Jednoduchý obrázek kuřete (můžeš nahradit vlastním)

    // Hráč (kuře)
    const player = {
      x: 50,
      y: 220,
      width: 50,
      height: 50,
      vy: 0,
      gravity: 0.8,
      jumpForce: -15,
      grounded: false,
    };

    // Překážky
    const obstacles = [];
    const obstacleWidth = 30;
    let gameSpeed = 5;
    let frameCount = 0;
    let score = 0;
    let gameOver = false;

    // Skákání na mezerník / kliknutí
    document.addEventListener('keydown', e => {
      if (e.code === 'Space' || e.code === 'ArrowUp') jump();
    });
    document.addEventListener('click', jump);

    function jump() {
      if (player.grounded && !gameOver) {
        player.vy = player.jumpForce;
        player.grounded = false;
      }
    }

    function createObstacle() {
      const y = 220;
      obstacles.push({ x: canvas.width, y, width: obstacleWidth, height: 50 });
    }

    function update() {
      if (gameOver) return;

      frameCount++;
      if (frameCount % 90 === 0) {
        createObstacle();
        if (gameSpeed < 12) gameSpeed += 0.2; // postupně zrychluj
      }

      // Pohyb překážek
      obstacles.forEach(obs => {
        obs.x -= gameSpeed;
      });

      // Odstranění překážek mimo obrazovku
      while (obstacles.length && obstacles[0].x + obstacleWidth < 0) {
        obstacles.shift();
        score++;
        document.getElementById('score').textContent = 'Skóre: ' + score;
      }

      // Gravitační pohyb hráče
      player.vy += player.gravity;
      player.y += player.vy;

      // Podlaha
      if (player.y > 220) {
        player.y = 220;
        player.vy = 0;
        player.grounded = true;
      }

      // Kolize s překážkou
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
    }

    function draw() {
      // Pozadí
      ctx.fillStyle = '#87ceeb';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Podlaha
      ctx.fillStyle = '#654321';
      ctx.fillRect(0, 270, canvas.width, 30);

      // Kuře
      ctx.drawImage(chicken, player.x, player.y, player.width, player.height);

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

    chicken.onload = () => {
      gameLoop();
    };
  </script>
</body>
</html>
