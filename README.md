# MR-BEAN
Mr bean game
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Jogo Mr. Bean com Imagens de Game Over por Pontuação</title>
  <style>
    body {
      margin: 0;
      background: #d0e7f9;
      overflow: hidden;
      font-family: Arial, sans-serif;
      user-select: none;
    }
    #game {
      position: relative;
      width: 400px;
      height: 600px;
      margin: 20px auto;
      background: #88b3e0;
      border: 3px solid #444;
      overflow: hidden;
      border-radius: 10px;
    }
    #mrbean {
      position: absolute;
      bottom: 10px;
      width: 80px;
      height: 80px;
      left: 160px;
      background: url('https://static.wikia.nocookie.net/mrbean/images/c/c2/Mr._Bean_Cartoon.webp/revision/latest/scale-to-width/360?cb=20250430000419') no-repeat center/contain;
      z-index: 2;
    }
    .objeto {
      position: absolute;
      width: 120px;
      height: 120px;
      background: url('https://static.wikia.nocookie.net/mrbean/images/9/99/Scrapper.gif/revision/latest?cb=20241228064420') no-repeat center/contain;
      top: -120px;
      left: 0;
      z-index: 1;
    }
    #cat-image {
      position: absolute;
      bottom: 10px;
      left: 160px;
      width: 80px;
      height: 80px;
      background: url('https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRwepJH990FRSX9v9UvQpOwuizunCI3As02AQ&s') no-repeat center/contain;
      display: none;
      z-index: 3;
    }
    #gameover {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(255,255,255,0.95);
      padding: 30px;
      border-radius: 10px;
      font-size: 24px;
      display: none;
      border: 2px solid #444;
      width: 80%;
      text-align: center;
      z-index: 5;
      user-select: none;
    }
    #gameover img {
      width: 200px;
      margin-bottom: 15px;
      border-radius: 10px;
      box-shadow: 0 0 12px rgba(0,0,0,0.3);
    }
    button {
      margin-top: 15px;
      padding: 8px 16px;
      font-size: 18px;
      cursor: pointer;
      border-radius: 6px;
      border: none;
      background: #4caf50;
      color: white;
      transition: background 0.3s ease;
    }
    button:hover {
      background: #388e3c;
    }
    #scoreboard {
      text-align: center;
      margin-top: 10px;
      font-size: 20px;
      color: #333;
      user-select: none;
    }
  </style>
</head>
<body>
  <div id="game">
    <div id="mrbean"></div>
    <div id="cat-image"></div>
    <div id="gameover">
      <img src="https://i.pinimg.com/236x/65/bc/95/65bc95e680feb2a428483191e2be2fb8.jpg" alt="Mr Bean assustado com Mrs. Wicked" />
      <div id="finalscore"></div>
      <button onclick="restartGame()">Reiniciar</button>
    </div>
  </div>
  <div id="scoreboard">Pontuação: 0 | Vidas: 3 | Nível: 1</div>

  <script>
    const game = document.getElementById('game');
    const mrbean = document.getElementById('mrbean');
    const catImage = document.getElementById('cat-image');
    const scoreboard = document.getElementById('scoreboard');
    const gameoverScreen = document.getElementById('gameover');
    const finalscore = document.getElementById('finalscore');

    const gameWidth = 400;
    const gameHeight = 600;
    const mrbeanWidth = 80;
    const mrbeanHeight = 80;

    let mrbeanX = 160;
    const mrbeanY = gameHeight - mrbeanHeight - 10;
    let speed = 14;
    let objetos = [];
    let spawnInterval;
    let gameLoopInterval;

    let score = 0;
    let lives = 3;
    let level = 1;
    let isGameOver = false;

    function createObjeto() {
      const objeto = document.createElement('div');
      objeto.classList.add('objeto');
      objeto.style.left = Math.floor(Math.random() * (gameWidth - 120)) + 'px';
      objeto.style.top = '-120px';
      game.appendChild(objeto);

      const fallSpeed = 3 + Math.random() * 2 + (level - 1) * 1.5;

      objetos.push({
        element: objeto,
        y: -120,
        x: parseInt(objeto.style.left),
        speed: fallSpeed,
      });
    }

    function moveMrBean(direction) {
      if (direction === 'left') {
        mrbeanX -= speed;
        if (mrbeanX < 0) mrbeanX = 0;
      } else if (direction === 'right') {
        mrbeanX += speed;
        if (mrbeanX > gameWidth - mrbeanWidth) mrbeanX = gameWidth - mrbeanWidth;
      }
      mrbean.style.left = mrbeanX + 'px';
      catImage.style.left = mrbeanX + 'px';
    }

    function checkCollision(obj) {
      const objX = obj.x;
      const objY = obj.y;
      const objWidth = 120;
      const objHeight = 120;

      if (
        objX < mrbeanX + mrbeanWidth &&
        objX + objWidth > mrbeanX &&
        objY + objHeight > mrbeanY &&
        objY < mrbeanY + mrbeanHeight
      ) {
        return true;
      }
      return false;
    }

    function updateLevel() {
      const newLevel = Math.floor(score / 10) + 1;
      if (newLevel !== level) {
        level = newLevel;
        updateScoreboard();

        clearInterval(spawnInterval);
        const newSpawnTime = Math.max(700, 1500 - (level - 1) * 200);
        spawnInterval = setInterval(() => {
          if (!isGameOver) createObjeto();
        }, newSpawnTime);
      }
    }

    function updateScoreboard() {
      scoreboard.textContent = `Pontuação: ${score} | Vidas: ${lives} | Nível: ${level}`;
    }

    function update() {
      if (isGameOver) return;

      objetos.forEach((obj, index) => {
        obj.y += obj.speed;
        obj.element.style.top = obj.y + 'px';

        if (checkCollision(obj)) {
          score++;
          updateLevel();
          updateScoreboard();
          game.removeChild(obj.element);
          objetos.splice(index, 1);
        } else if (obj.y > gameHeight) {
          game.removeChild(obj.element);
          objetos.splice(index, 1);
          lives--;
          updateScoreboard();
          if (lives <= 0) endGame();
        }
      });
    }

    function endGame() {
      isGameOver = true;
      catImage.style.display = 'none';

      // Define a imagem do game over conforme pontuação
      let imgSrc = '';
      if (score < 10) {
        imgSrc = 'https://i.pinimg.com/236x/65/bc/95/65bc95e680feb2a428483191e2be2fb8.jpg';
      } else if (score >= 11 && score <= 20) {
        imgSrc = 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSHf_0KFs9sb3e2Jg8mO5i1y_7WL7cHVwjaNw&s';
      } else if (score >= 21 && score <= 30) {
        imgSrc = 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSgMxnlwlrSBW2EAEtnmFC-mXCHgrdPXtGZOw&s';
      } else if (score > 30) {
        imgSrc = 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTld38MlyngO1MOuf6rVYTP0U_sZhmfxYcCRQ&s';
      }

      const imgElement = gameoverScreen.querySelector('img');
      imgElement.src = imgSrc;

      gameoverScreen.style.display = 'block';
      finalscore.textContent = `Game Over! Sua pontuação: ${score}`;
    }

    function restartGame() {
      objetos.forEach(obj => game.removeChild(obj.element));
      objetos = [];
      score = 0;
      lives = 3;
      level = 1;
      isGameOver = false;
      mrbeanX = 160;
      mrbean.style.left = mrbeanX + 'px';
      catImage.style.left = mrbeanX + 'px';
      catImage.style.display = 'none';
      updateScoreboard();
      gameoverScreen.style.display = 'none';

      clearInterval(spawnInterval);
      spawnInterval = setInterval(() => {
        if (!isGameOver) createObjeto();
      }, 1500);
    }

    document.addEventListener('keydown', e => {
      if (isGameOver) return;
      if (e.key === 'ArrowLeft' || e.key === 'a') {
        moveMrBean('left');
      } else if (e.key === 'ArrowRight' || e.key === 'd') {
        moveMrBean('right');
      }
    });

    spawnInterval = setInterval(() => {
      if (!isGameOver) createObjeto();
    }, 1500);

    gameLoopInterval = setInterval(() => {
      update();
    }, 1000 / 60);

    updateScoreboard();
  </script>
</body>
</html>
