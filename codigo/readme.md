<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jogo da Cobrinha (Google Snake Style)</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      background-color: #4a752c;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

    .game-container {
      background-color: #4a752c;
      border-radius: 8px;
      padding: 10px;
      box-shadow: 0 10px 20px rgba(0,0,0,0.3);
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    .header {
      width: 100%;
      background-color: #4a752c;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 5px 15px 15px 15px;
      color: white;
      font-size: 24px;
      font-weight: bold;
    }

    .score-box {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .apple-icon {
      width: 30px;
      height: 30px;
      background-color: #e74c3c;
      border-radius: 50%;
      position: relative;
    }

    .apple-icon::after {
      content: '';
      position: absolute;
      top: -2px;
      right: 12px;
      width: 4px;
      height: 8px;
      background-color: #27ae60;
      border-radius: 2px;
    }

    canvas {
      border-radius: 4px;
    }
  </style>
</head>
<body>

  <div class="game-container">
    <div class="header">
      <div class="score-box">
        <div class="apple-icon"></div>
        <span id="score">0</span>
      </div>
    </div>
    
    <canvas id="snakeGame" width="400" height="400"></canvas>
  </div>

  <script>
    const canvas = document.getElementById("snakeGame");
    const ctx = canvas.getContext("2d");
    const scoreElement = document.getElementById("score");

    const gridSize = 20;
    const tileCount = canvas.width / gridSize;

    // Lógica do jogo (grade)
    let snakeGrid = [{ x: 10, y: 10 }, { x: 9, y: 10 }, { x: 8, y: 10 }];
    let food = { x: 15, y: 10 };
    
    let dx = 1;
    let dy = 0;
    let nextDx = 1;
    let nextDy = 0;
    
    let score = 0;
    let lastLogicTime = 0;
    const MOVE_INTERVAL = 140; // Tempo em ms entre cada passo lógico

    function gameLoop(currentTime) {
      if (!lastLogicTime) lastLogicTime = currentTime;
      const elapsed = currentTime - lastLogicTime;

      // Se passou o tempo de 1 passo, atualiza a lógica do jogo
      if (elapsed >= MOVE_INTERVAL) {
        updateGameLogic();
        lastLogicTime = currentTime;
      }

      // Progresso da transição suave entre a posição anterior e a atual (0 a 1)
      const progress = Math.min((currentTime - lastLogicTime) / MOVE_INTERVAL, 1);

      drawBoard();
      drawFood();
      drawSmoothSnake(progress);

      requestAnimationFrame(gameLoop);
    }

    function updateGameLogic() {
      dx = nextDx;
      dy = nextDy;

      const head = { x: snakeGrid[0].x + dx, y: snakeGrid[0].y + dy };

      // Colisão com as paredes
      if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
        resetGame();
        return;
      }

      // Colisão com o próprio corpo
      for (let i = 0; i < snakeGrid.length; i++) {
        if (head.x === snakeGrid[i].x && head.y === snakeGrid[i].y) {
          resetGame();
          return;
        }
      }

      snakeGrid.unshift(head);

      if (head.x === food.x && head.y === food.y) {
        score++;
        scoreElement.innerText = score;
        generateFood();
      } else {
        snakeGrid.pop();
      }
    }

    function drawBoard() {
      for (let r = 0; r < tileCount; r++) {
        for (let c = 0; c < tileCount; c++) {
          ctx.fillStyle = (r + c) % 2 === 0 ? "#aad751" : "#a2d149";
          ctx.fillRect(c * gridSize, r * gridSize, gridSize, gridSize);
        }
      }
    }

    function drawSmoothSnake(progress) {
      // Interpola a posição da cabeça e dos segmentos para transição suave
      snakeGrid.forEach((part, index) => {
        let renderX = part.x;
        let renderY = part.y;

        if (index === 0) {
          // A cabeça se move da posição anterior para a atual
          renderX = part.x - dx * (1 - progress);
          renderY = part.y - dy * (1 - progress);
        } else {
          // O corpo segue o segmento da frente
          const prevPart = snakeGrid[index - 1];
          renderX = part.x + (prevPart.x - part.x) * progress;
          renderY = part.y + (prevPart.y - part.y) * progress;
        }

        const x = renderX * gridSize;
        const y = renderY * gridSize;

        ctx.fillStyle = "#467bec";

        if (index === 0) {
          // Cabeça
          ctx.beginPath();
          ctx.roundRect(x, y, gridSize, gridSize, 8);
          ctx.fill();

          // Olhos
          ctx.fillStyle = "white";
          let eye1X = x + 5, eye1Y = y + 5;
          let eye2X = x + 5, eye2Y = y + 12;

          if (dx === 1) { eye1X = x + 12; eye2X = x + 12; eye1Y = y + 4; eye2Y = y + 12; }
          if (dx === -1) { eye1X = x + 4; eye2X = x + 4; eye1Y = y + 4; eye2Y = y + 12; }
          if (dy === 1) { eye1X = x + 4; eye2X = x + 12; eye1Y = y + 12; eye2Y = y + 12; }
          if (dy === -1) { eye1X = x + 4; eye2X = x + 12; eye1Y = y + 4; eye2Y = y + 4; }

          ctx.beginPath();
          ctx.arc(eye1X, eye1Y, 3, 0, Math.PI * 2);
          ctx.arc(eye2X, eye2Y, 3, 0, Math.PI * 2);
          ctx.fill();

          // Pupilas
          ctx.fillStyle = "black";
          ctx.beginPath();
          ctx.arc(eye1X, eye1Y, 1.5, 0, Math.PI * 2);
          ctx.arc(eye2X, eye2Y, 1.5, 0, Math.PI * 2);
          ctx.fill();
        } else {
          // Corpo
          ctx.beginPath();
          ctx.roundRect(x + 1, y + 1, gridSize - 2, gridSize - 2, 6);
          ctx.fill();
        }
      });
    }

    function drawFood() {
      const x = food.x * gridSize + gridSize / 2;
      const y = food.y * gridSize + gridSize / 2;

      ctx.fillStyle = "#e74c3c";
      ctx.beginPath();
      ctx.arc(x, y, gridSize / 2 - 2, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = "#27ae60";
      ctx.fillRect(x - 1, y - gridSize / 2, 2, 4);
    }

    function generateFood() {
      food.x = Math.floor(Math.random() * tileCount);
      food.y = Math.floor(Math.random() * tileCount);

      snakeGrid.forEach(part => {
        if (part.x === food.x && part.y === food.y) {
          generateFood();
        }
      });
    }

    function resetGame() {
      alert("Fim de Jogo! Pontuação: " + score);
      snakeGrid = [{ x: 10, y: 10 }, { x: 9, y: 10 }, { x: 8, y: 10 }];
      dx = 1; dy = 0;
      nextDx = 1; nextDy = 0;
      score = 0;
      scoreElement.innerText = score;
      generateFood();
    }

    // Controles WASD e Setas
    document.addEventListener("keydown", (e) => {
      const key = e.key.toLowerCase();

      if ((key === "arrowup" || key === "w") && dy !== 1) { nextDx = 0; nextDy = -1; }
      if ((key === "arrowdown" || key === "s") && dy !== -1) { nextDx = 0; nextDy = 1; }
      if ((key === "arrowleft" || key === "a") && dx !== 1) { nextDx = -1; nextDy = 0; }
      if ((key === "arrowright" || key === "d") && dx !== -1) { nextDx = 1; nextDy = 0; }
    });

    generateFood();
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
