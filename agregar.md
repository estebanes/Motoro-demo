<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>No hay Internet</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
    }
    body {
      background: #fff;
      color: #333;
      font-family: Roboto, image.pngArial, sans-serif;
      margin: 0;
      padding: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
    }
    .container {
      text-align: center;
      margin-top: 20px;
      width: 100%;
      max-width: 900px;
    }
    #game {
      width: 100%;
      max-width: 800px;
      height: auto;
      aspect-ratio: 4/3;
      background: #fff;
      display: block;
      margin: 0 auto 20px auto;
      border: 1px solid #eee;
      box-sizing: border-box;
    }
    .msg {
      font-size: 1.5em;
      margin-bottom: 10px;
    }
    .desc {
      color: #666;
      margin-bottom: 20px;
    }
    .error {
      color: #a0a0a0;
      font-size: 0.9em;
    }
    @media (max-width: 600px) {
      .msg {
        font-size: 1.1em;
      }
      #game {
        max-width: 100vw;
      }
    }
  </style>
</head>
<body>
  <audio id="fartSound" src="fart-6.mp3" preload="auto"></audio>
  <audio id="runningSound" src="running.mp3" preload="auto" loop></audio>
  <div class="container">
    <canvas id="game"></canvas>
    <div class="msg">No tenimos Internet</div>
    <div class="desc">Tu ordenador está desconectado.</div>
    <div class="error">ERR_INTERNET_DISCONNECTED</div>
  </div>
  <div style="position:fixed; right:24px; bottom:16px; color:#444; font-size:16px; font-family:Arial, sans-serif; opacity:0.7; z-index:10; pointer-events:none;">Powerby Benja Luc</div>
  <script>
    let canvas = document.getElementById('game');
    let ctx = canvas.getContext('2d');
    let groundY;
    let dino;
    let cactusGroup;
    let score;
    let gameOver;
    let gravity;
    let jumpPower;
    let speed;
    let explosion;
    let nightMode;
    let autoJump;
    let stars;
    let houses;
    let houseType;
    let houseLightsInterval;
    let trees;
    let birds;
    let bigRock;
    let bigRockX;
    let coinCount;
    let lastObstaclePassed;
    let activeCoins;
    let runFrame;
    let runSpeed;
    let backgroundOffset;
    let runningSoundPlaying;
    let lastRunStep;

    function init() {
      groundY = canvas.height * 0.95;
      dino = { x: 50, y: groundY - 80, vy: 0, width: 60, height: 80, jumping: false };
      cactusGroup = { x: 600, y: groundY - 40, width: 20, height: 40, count: 1 };
      score = 20;
      gameOver = false;
      gravity = 0.8;
      jumpPower = -12;
      speed = 6;
      explosion = null;
      nightMode = false;
      autoJump = false;
      stars = [];
      houses = [];
      houseType = null;
      houseLightsInterval = null;
      trees = [];
      birds = [];
      bigRock = null;
      bigRockX = null;
      coinCount = 0;
      lastObstaclePassed = false;
      activeCoins = [];
      runFrame = 0;
      runSpeed = 0.4;
      backgroundOffset = 0;
      runningSoundPlaying = false;
      lastRunStep = 0;
    }

    // Ajustar tamaño del canvas al tamaño del contenedor
    function resizeCanvas() {
      const container = document.querySelector('.container');
      let width = Math.min(container.offsetWidth, 800);
      let height = width * 0.75; // 4:3 ratio
      canvas.width = width;
      canvas.height = height;
      init(); // Reinicializar el juego con el nuevo tamaño
    }

    window.addEventListener('resize', resizeCanvas);

    function drawDino() {
      if (gameOver && explosion && explosion.frame > 12) {
        ctx.font = '60px Arial';
        ctx.textAlign = 'left';
        ctx.textBaseline = 'top';
        ctx.fillText('💩', dino.x + 16, dino.y);
        return;
      }
      const baseX = dino.x;
      const baseY = dino.y;
      const enElAire = dino.y < groundY - dino.height;
      
      // Animación de correr
      if (!enElAire && !gameOver) {
        runFrame += runSpeed;
        if (runFrame > 8) runFrame = 0;
        
        // Sonido de correr
        let currentStep = Math.floor(runFrame / 2);
        if (currentStep !== lastRunStep) {
          lastRunStep = currentStep;
          playRunningSound();
        }
      } else {
        stopRunningSound();
      }
      
      // Calcular offset de animación para brazos y piernas
      const runOffset = Math.sin(runFrame) * 4;
      const legOffset = Math.sin(runFrame + Math.PI) * 6;
      // Cabeza (más detallada) - ligera animación de rebote
      const headBounce = enElAire ? 0 : Math.sin(runFrame * 2) * 0.5;
      ctx.fillStyle = '#a57939'; // piel
      ctx.fillRect(baseX + 16, baseY + headBounce, 28, 28);
      ctx.fillStyle = '#e0b97a'; // luz piel
      ctx.fillRect(baseX + 16, baseY + headBounce, 14, 14);
      ctx.fillStyle = '#5a3a1b'; // Cabello
      ctx.fillRect(baseX + 16, baseY + headBounce, 28, 8);
      ctx.fillStyle = '#3c2a13'; // sombra cabello
      ctx.fillRect(baseX + 16, baseY + headBounce, 14, 4);
      // Cuerpo (camiseta con detalles) - ligera animación de rebote
      const bodyBounce = enElAire ? 0 : Math.sin(runFrame * 2) * 1;
      ctx.fillStyle = '#3c9ee7';
      ctx.fillRect(baseX + 18, baseY + 28 + bodyBounce, 24, 24);
      ctx.fillStyle = '#5ec6ff'; // luz camiseta
      ctx.fillRect(baseX + 18, baseY + 28 + bodyBounce, 24, 12);
      ctx.fillStyle = '#2a6ca3'; // sombra camiseta
      ctx.fillRect(baseX + 18, baseY + 40 + bodyBounce, 24, 6);
      // Piernas (más colores y detalles)
      if (enElAire) {
        ctx.fillStyle = '#4a6c94';
        ctx.fillRect(baseX + 18, baseY + 52, 16, 10);
        ctx.fillStyle = '#6a8cb4';
        ctx.fillRect(baseX + 18, baseY + 52, 8, 5);
        ctx.fillStyle = '#2a3c54';
        ctx.fillRect(baseX + 26, baseY + 57, 8, 5);
        ctx.fillStyle = '#4a6c94';
        ctx.fillRect(baseX + 34, baseY + 62, 8, 18);
        ctx.fillStyle = '#6a8cb4';
        ctx.fillRect(baseX + 34, baseY + 62, 8, 9);
        ctx.fillStyle = '#2a3c54';
        ctx.fillRect(baseX + 34, baseY + 71, 8, 9);
        ctx.fillRect(baseX + 18, baseY + 38, 16, 14);
        ctx.fillRect(baseX + 34, baseY + 52, 8, 10);
      } else {
        // Piernas animadas corriendo - una detrás de otra
        ctx.fillStyle = '#4a6c94';
        // Pierna izquierda (se mueve hacia adelante y atrás)
        ctx.fillRect(baseX + 18 + legOffset, baseY + 52, 10, 20);
        // Pierna derecha (se mueve en oposición a la izquierda)
        ctx.fillRect(baseX + 32 - legOffset, baseY + 52, 10, 20);
        ctx.fillStyle = '#6a8cb4';
        ctx.fillRect(baseX + 18 + legOffset, baseY + 52, 10, 10);
        ctx.fillRect(baseX + 32 - legOffset, baseY + 52, 10, 10);
        ctx.fillStyle = '#2a3c54';
        ctx.fillRect(baseX + 18 + legOffset, baseY + 62, 10, 10);
        ctx.fillRect(baseX + 32 - legOffset, baseY + 62, 10, 10);
      }
      // Brazos (más colores y detalles)
      if (enElAire) {
        ctx.fillStyle = '#a57939';
        ctx.fillRect(baseX + 6, baseY + 18, 18, 8);
        ctx.fillStyle = '#e0b97a';
        ctx.fillRect(baseX + 6, baseY + 18, 9, 4);
        ctx.fillStyle = '#a57939';
        ctx.fillRect(baseX + 42, baseY + 8, 8, 24);
        ctx.fillStyle = '#e0b97a';
        ctx.fillRect(baseX + 42, baseY + 8, 8, 12);
        ctx.fillRect(baseX + 18, baseY + 18, 8, 10);
      } else {
        // Brazos animados corriendo - movimiento hacia adelante como corredor
        ctx.fillStyle = '#a57939';
        // Brazo izquierdo (se mueve hacia adelante cuando pierna derecha va hacia adelante)
        ctx.fillRect(baseX + 6 + runOffset, baseY + 28, 10, 24);
        // Brazo derecho (se mueve hacia atrás cuando pierna derecha va hacia adelante)
        ctx.fillRect(baseX + 42 - runOffset, baseY + 28, 10, 24);
        ctx.fillStyle = '#e0b97a';
        ctx.fillRect(baseX + 6 + runOffset, baseY + 28, 10, 12);
        ctx.fillRect(baseX + 42 - runOffset, baseY + 28, 10, 12);
      }
      // Ojos
      ctx.fillStyle = '#fff';
      ctx.fillRect(baseX + 22, baseY + 10 + headBounce, 5, 5);
      ctx.fillRect(baseX + 33, baseY + 10 + headBounce, 5, 5);
      ctx.fillStyle = '#3c9ee7';
      ctx.fillRect(baseX + 23, baseY + 12 + headBounce, 2, 3);
      ctx.fillRect(baseX + 35, baseY + 12 + headBounce, 2, 3);
      // Boca
      ctx.fillStyle = '#5a3a1b';
      ctx.fillRect(baseX + 27, baseY + 20 + headBounce, 8, 3);
    }

    function drawCactusGroup(group) {
      if (score >= 20) {
        // Piedra grande en movimiento
        let rockW = 80;
        let rockH = 60;
        if (bigRockX === null) bigRockX = 800;
        let rockY = groundY - rockH + 10;
        bigRock = { x: bigRockX, y: rockY, width: rockW, height: rockH };
        // Solo actualiza la posición de la moneda
        if (activeCoins.length === 0) {
          activeCoins.push({ x: bigRock.x + rockW / 2, y: rockY - 40, collected: false });
        } else {
          activeCoins[0].x = bigRock.x + rockW / 2;
          activeCoins[0].y = rockY - 40;
        }
        if (!activeCoins[0].collected) {
          drawCoin(activeCoins[0].x, activeCoins[0].y);
        }
        // Piedra
        ctx.save();
        ctx.beginPath();
        ctx.ellipse(bigRock.x + rockW / 2, rockY + rockH / 2, rockW / 2, rockH / 2, Math.random() * 0.2 - 0.1, 0, Math.PI * 2);
        ctx.fillStyle = '#888';
        ctx.fill();
        ctx.lineWidth = 3;
        ctx.strokeStyle = '#555';
        ctx.stroke();
        ctx.restore();
      } else {
        // Cactus 128 bits: más colores y detalles
        for (let i = 0; i < group.count; i++) {
          let x = group.x + i * 28;
          let y = group.y;
          ctx.fillStyle = '#228B22';
          ctx.fillRect(x, y, group.width, group.height);
          ctx.fillStyle = '#2ecc40';
          ctx.fillRect(x + 2, y + 4, group.width - 4, group.height - 8);
          ctx.fillStyle = '#145a1a';
          ctx.fillRect(x + 6, y + 10, group.width - 12, group.height - 20);
          // Espinas
          ctx.fillStyle = '#fff';
          for (let j = 0; j < 4; j++) {
            ctx.fillRect(x + 3 + j * 4, y + 6 + j * 8, 2, 4);
          }
          // Solo actualiza la posición de la moneda
          if (activeCoins[i]) {
            activeCoins[i].x = x + group.width / 2;
            activeCoins[i].y = y - 36;
          }
          if (activeCoins[i] && !activeCoins[i].collected) {
            drawCoin(activeCoins[i].x, activeCoins[i].y);
          }
        }
      }
    }

    function drawCoin(x, y) {
      ctx.save();
      ctx.beginPath();
      ctx.arc(x, y, 12, 0, Math.PI * 2);
      ctx.fillStyle = '#ffd700';
      ctx.shadowColor = '#fff';
      ctx.shadowBlur = 8;
      ctx.fill();
      ctx.shadowBlur = 0;
      ctx.strokeStyle = '#bfa100';
      ctx.lineWidth = 2;
      ctx.stroke();
      ctx.font = 'bold 14px Arial';
      ctx.fillStyle = '#bfa100';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('¢', x, y + 1);
      ctx.restore();
    }

    function drawGround() {
      ctx.strokeStyle = nightMode ? '#444' : '#888';
      ctx.beginPath();
      ctx.moveTo(0, groundY);
      ctx.lineTo(canvas.width, groundY);
      ctx.stroke();
    }

    function drawBackground() {
      if (nightMode) {
        ctx.fillStyle = '#111';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        // Dibujar estrellas
        for (let i = 0; i < stars.length; i++) {
          ctx.fillStyle = '#fff';
          ctx.beginPath();
          ctx.arc(stars[i].x, stars[i].y, stars[i].r, 0, Math.PI * 2);
          ctx.fill();
        }
        // Dibujar casas de fondo si score >= 7
        if (score >= 7) {
          for (let i = 0; i < houses.length; i++) {
            drawHouse(houses[i]);
          }
        }
        // Luna
        ctx.beginPath();
        ctx.arc(720, 70, 30, 0, Math.PI * 2);
        ctx.fillStyle = '#f5f3ce';
        ctx.fill();
        ctx.beginPath();
        ctx.arc(735, 85, 20, 0, Math.PI * 2);
        ctx.fillStyle = '#222';
        ctx.fill();
      } else {
        // Fondo amarillo reforzado
        ctx.save();
        ctx.globalAlpha = 1;
        ctx.fillStyle = '#fff9c4';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.restore();
        // Dibujar árboles si score >= 15
        if (score >= 15) {
          // Sol
          ctx.beginPath();
          ctx.arc(720, 80, 40, 0, Math.PI * 2);
          ctx.fillStyle = '#ffe066';
          ctx.fill();
          // Puntos animados
          for (let i = 0; i < birds.length; i++) {
            drawDot(birds[i]);
          }
          for (let i = 0; i < trees.length; i++) {
            drawTree(trees[i]);
          }
        }
        houses = [];
      }
    }

    function drawScore() {
      ctx.fillStyle = '#888';
      ctx.font = '20px Arial';
      ctx.fillText('Puntaje: ' + score, 480, 30);
      ctx.fillStyle = '#e6b800';
      ctx.fillText('Monedas: ' + coinCount, 650, 30);
    }

    function drawExplosion(x, y, frame) {
      // Dibuja una explosión pixelada simple
      const colors = ['#fff200', '#ff9500', '#ff0000', '#a0522d'];
      for (let i = 0; i < 12; i++) {
        const angle = (Math.PI * 2 * i) / 12;
        const radius = 18 + frame * 2 + Math.random() * 4;
        ctx.beginPath();
        ctx.arc(x + 30 + Math.cos(angle) * radius, y + 40 + Math.sin(angle) * radius, 6 + Math.random() * 3, 0, Math.PI * 2);
        ctx.fillStyle = colors[i % colors.length];
        ctx.globalAlpha = 1 - frame / 12;
        ctx.fill();
        ctx.globalAlpha = 1;
      }
    }

    function resetGame() {
      dino.y = groundY - dino.height;
      dino.vy = 0;
      dino.jumping = false;
      cactusGroup.x = 600 + Math.random() * 200;
      cactusGroup.count = 1;
      score = 20;
      coinCount = 0;
      activeCoins = [];
      gameOver = false;
      speed = 6;
      explosion = null;
      backgroundOffset = 0;
      stopRunningSound();
      loop();
    }

    function generateStars() {
      stars = [];
      for (let i = 0; i < 80; i++) {
        stars.push({
          x: Math.random() * canvas.width,
          y: Math.random() * 300,
          r: Math.random() * 1.5 + 0.5
        });
      }
    }

    function drawHouse(house) {
      let baseY = groundY - 10;
      let width = house.pisos === 1 ? 60 : 70;
      let height = house.pisos === 1 ? 40 : 70;
      // Cuerpo
      ctx.fillStyle = '#fff';
      let houseX = (house.x + backgroundOffset * 0.3) % (canvas.width + 200);
      if (houseX < -100) houseX += canvas.width + 200;
      ctx.fillRect(houseX, baseY - height, width, height);
      // Techo
      ctx.fillStyle = '#a0522d';
      ctx.beginPath();
      ctx.moveTo(houseX - 5, baseY - height);
      ctx.lineTo(houseX + width / 2, baseY - height - 20);
      ctx.lineTo(houseX + width + 5, baseY - height);
      ctx.closePath();
      ctx.fill();
      // Puerta
      ctx.fillStyle = '#7c5a3a';
      ctx.fillRect(houseX + width / 2 - 8, baseY - 24, 16, 24);
      // Ventanas con luces
      // Ventana 1 (arriba izquierda, solo si 2 pisos)
      if (house.pisos === 2) {
        ctx.fillStyle = house.luces[0] ? '#ffe066' : '#b3e6ff';
        ctx.fillRect(houseX + 10, baseY - height + 10, 16, 16);
        ctx.fillStyle = house.luces[1] ? '#ffe066' : '#b3e6ff';
        ctx.fillRect(houseX + width - 26, baseY - height + 10, 16, 16);
      }
      // Ventana 2 (abajo izquierda)
      ctx.fillStyle = house.luces[2] ? '#ffe066' : '#b3e6ff';
      ctx.fillRect(houseX + 10, baseY - 36, 16, 12);
      // Ventana 3 (abajo derecha)
      ctx.fillStyle = house.luces[3] ? '#ffe066' : '#b3e6ff';
      ctx.fillRect(houseX + width - 26, baseY - 36, 16, 12);
    }

    function generateHouses() {
      houses = [];
      for (let i = 0; i < 8; i++) {
        let pisos = Math.random() < 0.5 ? 1 : 2;
        let x = 60 + i * 90 + Math.random() * 30;
        // Luces: [ventana1, ventana2, ventana3, ventana4]
        let luces = [false, false, false, false];
        houses.push({ x: x, pisos: pisos, luces: luces });
      }
    }



    function updateHouseLights() {
      for (let i = 0; i < houses.length; i++) {
        for (let j = 0; j < houses[i].luces.length; j++) {
          houses[i].luces[j] = Math.random() < 0.5;
        }
      }
    }

    function generateTrees() {
      trees = [];
      for (let i = 0; i < 8; i++) {
        let x = 60 + i * 90 + Math.random() * 30;
        let height = 60 + Math.random() * 80;
        let width = 18 + Math.random() * 12;
        trees.push({ x: x, height: height, width: width });
      }
    }



    function generateBirds() {
      birds = [];
      for (let i = 0; i < 5; i++) {
        let centerX = 200 + Math.random() * 400;
        let centerY = 80 + Math.random() * 100;
        let radius = 30 + Math.random() * 40;
        let size = 18 + Math.random() * 10;
        let angle = Math.random() * Math.PI * 2;
        birds.push({ centerX, centerY, radius, size, angle });
      }
    }

    function updateBirds() {
      for (let i = 0; i < birds.length; i++) {
        birds[i].angle += 0.015 + i * 0.003;
        if (birds[i].angle > Math.PI * 2) birds[i].angle -= Math.PI * 2;
      }
    }

    function drawDot(dot) {
      let x = dot.centerX + Math.cos(dot.angle) * dot.radius;
      let y = dot.centerY + Math.sin(dot.angle) * dot.radius;
      // Dibujar una 'V' estilizada para simular un pájaro
      ctx.strokeStyle = '#222';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(x - 7, y);
      ctx.lineTo(x, y + 5);
      ctx.lineTo(x + 7, y);
      ctx.stroke();
      ctx.lineWidth = 1;
    }

    function drawTree(tree) {
      let baseY = groundY - 10;
      let treeX = (tree.x + backgroundOffset * 0.2) % (canvas.width + 200);
      if (treeX < -100) treeX += canvas.width + 200;
      // Tronco
      ctx.fillStyle = '#8b5a2b';
      ctx.fillRect(treeX + tree.width / 2 - 5, baseY - tree.height + 30, 10, tree.height - 30);
      // Copa
      ctx.beginPath();
      ctx.arc(treeX + tree.width / 2, baseY - tree.height + 30, tree.width * 1.2, 0, Math.PI * 2);
      ctx.fillStyle = '#2ecc40';
      ctx.fill();
      ctx.beginPath();
      ctx.arc(treeX + tree.width / 2, baseY - tree.height + 10, tree.width, 0, Math.PI * 2);
      ctx.fillStyle = '#27ae60';
      ctx.fill();
    }

    function playRunningSound() {
      if (!runningSoundPlaying) {
        var runningSound = document.getElementById('runningSound');
        runningSound.currentTime = 0;
        runningSound.play().then(() => {
          runningSoundPlaying = true;
        }).catch(e => {
          console.log('Error playing running sound:', e);
        });
      }
    }

    function stopRunningSound() {
      if (runningSoundPlaying) {
        var runningSound = document.getElementById('runningSound');
        runningSound.pause();
        runningSound.currentTime = 0;
        runningSoundPlaying = false;
      }
    }

    function loop() {
      // Cambiar a modo noche entre 5 y 14 puntos, y a día a partir de 15
      if (score >= 5 && score < 15) {
        if (!nightMode) generateStars();
        nightMode = true;
      } else {
        nightMode = false;
      }
      // Generar casas si score >= 7 y aún no hay
      if (nightMode && score >= 7 && score < 15 && houses.length === 0) {
        generateHouses();
        if (houseLightsInterval) clearInterval(houseLightsInterval);
        houseLightsInterval = setInterval(updateHouseLights, 1000);
      }
      if ((!nightMode || score < 7 || score >= 15) && houses.length > 0) {
        houses = [];
        if (houseLightsInterval) {
          clearInterval(houseLightsInterval);
          houseLightsInterval = null;
        }
      }
      // Generar árboles y pájaros si score >= 15 y aún no hay
      if (!nightMode && score >= 15 && trees.length === 0) {
        generateTrees();
        generateBirds();
      }
      if ((nightMode || score < 15) && (trees.length > 0 || birds.length > 0)) {
        trees = [];
        birds = [];
      }
      // Actualizar pájaros si es de día y score >= 15
      if (!nightMode && score >= 15) {
        updateBirds();
      }
      // Actualizar offset de fondo (movimiento contrario a los obstáculos)
      backgroundOffset -= speed * 0.5;
      

      
      // Movimiento de la piedra grande
      if (score >= 20) {
        if (bigRockX === null) bigRockX = 800;
        bigRockX -= speed;
        if (bigRockX + 80 < 0) {
          bigRockX = 800 + Math.random() * 200;
          // Reinicia la moneda de la piedra
          activeCoins = [];
        }
      } else {
        bigRockX = null;
      }
      // Detectar si Steve pasa exitosamente un obstáculo para sumar moneda
      if (score >= 20 && bigRock) {
        if (!lastObstaclePassed && bigRock.x + bigRock.width < dino.x) {
          coinCount++;
          lastObstaclePassed = true;
        }
        if (bigRock.x + bigRock.width >= dino.x) {
          lastObstaclePassed = false;
        }
      } else {
        for (let i = 0; i < cactusGroup.count; i++) {
          let obsX = cactusGroup.x + i * 28;
          if (!lastObstaclePassed && obsX + cactusGroup.width < dino.x) {
            coinCount++;
            lastObstaclePassed = true;
          }
          if (obsX + cactusGroup.width >= dino.x) {
            lastObstaclePassed = false;
          }
        }
      }
      // Colisión de Steve con monedas
      for (let i = 0; i < activeCoins.length; i++) {
        if (!activeCoins[i].collected) {
          let cx = activeCoins[i].x;
          let cy = activeCoins[i].y;
          if (
            dino.x + dino.width > cx - 12 &&
            dino.x < cx + 12 &&
            dino.y + dino.height > cy - 12 &&
            dino.y < cy + 12
          ) {
            activeCoins[i].collected = true;
            coinCount++;
          }
        }
      }
      // Mover drawBackground después de updateBirds
      drawBackground();
      drawGround();
      if (!explosion) {
        drawDino();
      }
      drawCactusGroup(cactusGroup);
      drawScore();

      // Ajustar cantidad de cactus según el puntaje
      cactusGroup.count = Math.min(1 + Math.floor(score / 5), 5); // máximo 5 juntos

      // Aumentar velocidad cuando hay 2 o más cactus
      if (cactusGroup.count >= 2) {
        speed = 10;
      } else {
        speed = 6;
      }

      // Movimiento del grupo de cactus
      cactusGroup.x -= speed;
      if (cactusGroup.x + cactusGroup.width * cactusGroup.count + 28 * (cactusGroup.count - 1) < 0) {
        cactusGroup.x = 600 + Math.random() * 200;
        score++;
        if (speed < 15) speed += 0.2;
        // Reinicia las monedas de cactus
        activeCoins = [];
      }

      // Gravedad y salto
      dino.y += dino.vy;
      if (dino.y + dino.height < groundY) {
        dino.vy += gravity;
      } else {
        dino.y = groundY - dino.height;
        dino.vy = 0;
        dino.jumping<!-- filepath: c:\Users\Dell\Desktop\Pagina google\index.html -->
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>No hay Internet</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
    }
    body {
      background: #fff;
      color: #333;
      font-family: Roboto, Arial, sans-serif;
      margin: 0;
      padding: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
    }
    .container {
      text-align: center;
      margin-top: 20px;
      width: 100%;
      max-width: 900px;
    }
    #game {
      width: 100%;
      max-width: 800px;
      height: auto;
      aspect-ratio: 4/3;
      background: #fff;
      display: block;
      margin: 0 auto 20px auto;
      border: 1px solid #eee;
      box-sizing: border-box;
    }
    .msg {
      font-size: 1.5em;
      margin-bottom: 10px;
    }
    .desc {
      color: #666;
      margin-bottom: 20px;
    }
    .error {
      color: #a0a0a0;
      font-size: 0.9em;
    }
    @media (max-width: 600px) {
      .msg {
        font-size: 1.1em;
      }
      #game {
        max-width: 100vw;
      }
    }
  </style>
</head>
<body>
  <audio id="fartSound" src="fart-6.mp3" preload="auto"></audio>
  <audio id="runningSound" src="running.mp3" preload="auto" loop></audio>
  <div class="container">
    <canvas id="game"></canvas>
    <div class="msg">No tenimos Internet</div>
    <div class="desc">Tu ordenador está desconectado.</div>
    <div class="error">ERR_INTERNET_DISCONNECTED</div>
  </div>
  <div style="position:fixed; right:24px; bottom:16px; color:#444; font-size:16px; font-family:Arial, sans-serif; opacity:0.7; z-index:10; pointer-events:none;">Powerby Benja Luc</div>
  <script>
    let canvas = document.getElementById('game');
    let ctx = canvas.getContext('2d');
    let groundY;
    let dino;
    let cactusGroup;
    let score;
    let gameOver;
    let gravity;
    let jumpPower;
    let speed;
    let explosion;
    let nightMode;
    let autoJump;
    let stars;
    let houses;
    let houseType;
    let houseLightsInterval;
    let trees;
    let birds;
    let bigRock;
    let bigRockX;
    let coinCount;
    let lastObstaclePassed;
    let activeCoins;
    let runFrame;
    let runSpeed;
    let backgroundOffset;
    let runningSoundPlaying;
    let lastRunStep;

    function init() {
      groundY = canvas.height * 0.95;
      dino = { x: 50, y: groundY - 80, vy: 0, width: 60, height: 80, jumping: false };
      cactusGroup = { x: 600, y: groundY - 40, width: 20, height: 40, count: 1 };
      score = 20;
      gameOver = false;
      gravity = 0.8;
      jumpPower = -12;
      speed = 6;
      explosion = null;
      nightMode = false;
      autoJump = false;
      stars = [];
      houses = [];
      houseType = null;
      houseLightsInterval = null;
      trees = [];
      birds = [];
      bigRock = null;
      bigRockX = null;
      coinCount = 0;
      lastObstaclePassed = false;
      activeCoins = [];
      runFrame = 0;
      runSpeed = 0.4;
      backgroundOffset = 0;
      runningSoundPlaying = false;
      lastRunStep = 0;
    }

    // Ajustar tamaño del canvas al tamaño del contenedor
    function resizeCanvas() {
      const container = document.querySelector('.container');
      let width = Math.min(container.offsetWidth, 800);
      let height = width * 0.75; // 4:3 ratio
      canvas.width = width;
      canvas.height = height;
      init(); // Reinicializar el juego con el nuevo tamaño
    }

    window.addEventListener('resize', resizeCanvas);

    function drawDino() {
      if (gameOver && explosion && explosion.frame > 12) {
        ctx.font = '60px Arial';
        ctx.textAlign = 'left';
        ctx.textBaseline = 'top';
        ctx.fillText('💩', dino.x + 16, dino.y);
        return;
      }
      const baseX = dino.x;
      const baseY = dino.y;
      const enElAire = dino.y < groundY - dino.height;
      
      // Animación de correr
      if (!enElAire && !gameOver) {
        runFrame += runSpeed;
        if (runFrame > 8) runFrame = 0;
        
        // Sonido de correr
        let currentStep = Math.floor(runFrame / 2);
        if (currentStep !== lastRunStep) {
          lastRunStep = currentStep;
          playRunningSound();
        }
      } else {
        stopRunningSound();
      }
      
      // Calcular offset de animación para brazos y piernas
      const runOffset = Math.sin(runFrame) * 4;
      const legOffset = Math.sin(runFrame + Math.PI) * 6;
      // Cabeza (más detallada) - ligera animación de rebote
      const headBounce = enElAire ? 0 : Math.sin(runFrame * 2) * 0.5;
      ctx.fillStyle = '#a57939'; // piel
      ctx.fillRect(baseX + 16, baseY + headBounce, 28, 28);
      ctx.fillStyle = '#e0b97a'; // luz piel
      ctx.fillRect(baseX + 16, baseY + headBounce, 14, 14);
      ctx.fillStyle = '#5a3a1b'; // Cabello
      ctx.fillRect(baseX + 16, baseY + headBounce, 28, 8);
      ctx.fillStyle = '#3c2a13'; // sombra cabello
      ctx.fillRect(baseX + 16, baseY + headBounce, 14, 4);
      // Cuerpo (camiseta con detalles) - ligera animación de rebote
      const bodyBounce = enElAire ? 0 : Math.sin(runFrame * 2) * 1;
      ctx.fillStyle = '#3c9ee7';
      ctx.fillRect(baseX + 18, baseY + 28 + bodyBounce, 24, 24);
      ctx.fillStyle = '#5ec6ff'; // luz camiseta
      ctx.fillRect(baseX + 18, baseY + 28 + bodyBounce, 24, 12);
      ctx.fillStyle = '#2a6ca3'; // sombra camiseta
      ctx.fillRect(baseX + 18, baseY + 40 + bodyBounce, 24, 6);
      // Piernas (más colores y detalles)
      if (enElAire) {
        ctx.fillStyle = '#4a6c94';
        ctx.fillRect(baseX + 18, baseY + 52, 16, 10);
        ctx.fillStyle = '#6a8cb4';
        ctx.fillRect(baseX + 18, baseY + 52, 8, 5);
        ctx.fillStyle = '#2a3c54';
        ctx.fillRect(baseX + 26, baseY + 57, 8, 5);
        ctx.fillStyle = '#4a6c94';
        ctx.fillRect(baseX + 34, baseY + 62, 8, 18);
        ctx.fillStyle = '#6a8cb4';
        ctx.fillRect(baseX + 34, baseY + 62, 8, 9);
        ctx.fillStyle = '#2a3c54';
        ctx.fillRect(baseX + 34, baseY + 71, 8, 9);
        ctx.fillRect(baseX + 18, baseY + 38, 16, 14);
        ctx.fillRect(baseX + 34, baseY + 52, 8, 10);
      } else {
        // Piernas animadas corriendo - una detrás de otra
        ctx.fillStyle = '#4a6c94';
        // Pierna izquierda (se mueve hacia adelante y atrás)
        ctx.fillRect(baseX + 18 + legOffset, baseY + 52, 10, 20);
        // Pierna derecha (se mueve en oposición a la izquierda)
        ctx.fillRect(baseX + 32 - legOffset, baseY + 52, 10, 20);
        ctx.fillStyle = '#6a8cb4';
        ctx.fillRect(baseX + 18 + legOffset, baseY + 52, 10, 10);
        ctx.fillRect(baseX + 32 - legOffset, baseY + 52, 10, 10);
        ctx.fillStyle = '#2a3c54';
        ctx.fillRect(baseX + 18 + legOffset, baseY + 62, 10, 10);
        ctx.fillRect(baseX + 32 - legOffset, baseY + 62, 10, 10);
      }
      // Brazos (más colores y detalles)
      if (enElAire) {
        ctx.fillStyle = '#a57939';
        ctx.fillRect(baseX + 6, baseY + 18, 18, 8);
        ctx.fillStyle = '#e0b97a';
        ctx.fillRect(baseX + 6, baseY + 18, 9, 4);
        ctx.fillStyle = '#a57939';
        ctx.fillRect(baseX + 42, baseY + 8, 8, 24);
        ctx.fillStyle = '#e0b97a';
        ctx.fillRect(baseX + 42, baseY + 8, 8, 12);
        ctx.fillRect(baseX + 18, baseY + 18, 8, 10);
      } else {
        // Brazos animados corriendo - movimiento hacia adelante como corredor
        ctx.fillStyle = '#a57939';
        // Brazo izquierdo (se mueve hacia adelante cuando pierna derecha va hacia adelante)
        ctx.fillRect(baseX + 6 + runOffset, baseY + 28, 10, 24);
        // Brazo derecho (se mueve hacia atrás cuando pierna derecha va hacia adelante)
        ctx.fillRect(baseX + 42 - runOffset, baseY + 28, 10, 24);
        ctx.fillStyle = '#e0b97a';
        ctx.fillRect(baseX + 6 + runOffset, baseY + 28, 10, 12);
        ctx.fillRect(baseX + 42 - runOffset, baseY + 28, 10, 12);
      }
      // Ojos
      ctx.fillStyle = '#fff';
      ctx.fillRect(baseX + 22, baseY + 10 + headBounce, 5, 5);
      ctx.fillRect(baseX + 33, baseY + 10 + headBounce, 5, 5);
      ctx.fillStyle = '#3c9ee7';
      ctx.fillRect(baseX + 23, baseY + 12 + headBounce, 2, 3);
      ctx.fillRect(baseX + 35, baseY + 12 + headBounce, 2, 3);
      // Boca
      ctx.fillStyle = '#5a3a1b';
      ctx.fillRect(baseX + 27, baseY + 20 + headBounce, 8, 3);
    }

    function drawCactusGroup(group) {
      if (score >= 20) {
        // Piedra grande en movimiento
        let rockW = 80;
        let rockH = 60;
        if (bigRockX === null) bigRockX = 800;
        let rockY = groundY - rockH + 10;
        bigRock = { x: bigRockX, y: rockY, width: rockW, height: rockH };
        // Solo actualiza la posición de la moneda
        if (activeCoins.length === 0) {
          activeCoins.push({ x: bigRock.x + rockW / 2, y: rockY - 40, collected: false });
        } else {
          activeCoins[0].x = bigRock.x + rockW / 2;
          activeCoins[0].y = rockY - 40;
        }
        if (!activeCoins[0].collected) {
          drawCoin(activeCoins[0].x, activeCoins[0].y);
        }
        // Piedra
        ctx.save();
        ctx.beginPath();
        ctx.ellipse(bigRock.x + rockW / 2, rockY + rockH / 2, rockW / 2, rockH / 2, Math.random() * 0.2 - 0.1, 0, Math.PI * 2);
        ctx.fillStyle = '#888';
        ctx.fill();
        ctx.lineWidth = 3;
        ctx.strokeStyle = '#555';
        ctx.stroke();
        ctx.restore();
      } else {
        // Cactus 128 bits: más colores y detalles
        for (let i = 0; i < group.count; i++) {
          let x = group.x + i * 28;
          let y = group.y;
          ctx.fillStyle = '#228B22';
          ctx.fillRect(x, y, group.width, group.height);
          ctx.fillStyle = '#2ecc40';
          ctx.fillRect(x + 2, y + 4, group.width - 4, group.height - 8);
          ctx.fillStyle = '#145a1a';
          ctx.fillRect(x + 6, y + 10, group.width - 12, group.height - 20);
          // Espinas
          ctx.fillStyle = '#fff';
          for (let j = 0; j < 4; j++) {
            ctx.fillRect(x + 3 + j * 4, y + 6 + j * 8, 2, 4);
          }
          // Solo actualiza la posición de la moneda
          if (activeCoins[i]) {
            activeCoins[i].x = x + group.width / 2;
            activeCoins[i].y = y - 36;
          }
          if (activeCoins[i] && !activeCoins[i].collected) {
            drawCoin(activeCoins[i].x, activeCoins[i].y);
          }
        }
      }
    }

    function drawCoin(x, y) {
      ctx.save();
      ctx.beginPath();
      ctx.arc(x, y, 12, 0, Math.PI * 2);
      ctx.fillStyle = '#ffd700';
      ctx.shadowColor = '#fff';
      ctx.shadowBlur = 8;
      ctx.fill();
      ctx.shadowBlur = 0;
      ctx.strokeStyle = '#bfa100';
      ctx.lineWidth = 2;
      ctx.stroke();
      ctx.font = 'bold 14px Arial';
      ctx.fillStyle = '#bfa100';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('¢', x, y + 1);
      ctx.restore();
    }

    function drawGround() {
      ctx.strokeStyle = nightMode ? '#444' : '#888';
      ctx.beginPath();
      ctx.moveTo(0, groundY);
      ctx.lineTo(canvas.width, groundY);
      ctx.stroke();
    }

    function drawBackground() {
      if (nightMode) {
        ctx.fillStyle = '#111';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        // Dibujar estrellas
        for (let i = 0; i < stars.length; i++) {
          ctx.fillStyle = '#fff';
          ctx.beginPath();
          ctx.arc(stars[i].x, stars[i].y, stars[i].r, 0, Math.PI * 2);
          ctx.fill();
        }
        // Dibujar casas de fondo si score >= 7
        if (score >= 7) {
          for (let i = 0; i < houses.length; i++) {
            drawHouse(houses[i]);
          }
        }
        // Luna
        ctx.beginPath();
        ctx.arc(720, 70, 30, 0, Math.PI * 2);
        ctx.fillStyle = '#f5f3ce';
        ctx.fill();
        ctx.beginPath();
        ctx.arc(735, 85, 20, 0, Math.PI * 2);
        ctx.fillStyle = '#222';
        ctx.fill();
      } else {
        // Fondo amarillo reforzado
        ctx.save();
        ctx.globalAlpha = 1;
        ctx.fillStyle = '#fff9c4';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.restore();
        // Dibujar árboles si score >= 15
        if (score >= 15) {
          // Sol
          ctx.beginPath();
          ctx.arc(720, 80, 40, 0, Math.PI * 2);
          ctx.fillStyle = '#ffe066';
          ctx.fill();
          // Puntos animados
          for (let i = 0; i < birds.length; i++) {
            drawDot(birds[i]);
          }
          for (let i = 0; i < trees.length; i++) {
            drawTree(trees[i]);
          }
        }
        houses = [];
      }
    }

    function drawScore() {
      ctx.fillStyle = '#888';
      ctx.font = '20px Arial';
      ctx.fillText('Puntaje: ' + score, 480, 30);
      ctx.fillStyle = '#e6b800';
      ctx.fillText('Monedas: ' + coinCount, 650, 30);
    }

    function drawExplosion(x, y, frame) {
      // Dibuja una explosión pixelada simple
      const colors = ['#fff200', '#ff9500', '#ff0000', '#a0522d'];
      for (let i = 0; i < 12; i++) {
        const angle = (Math.PI * 2 * i) / 12;
        const radius = 18 + frame * 2 + Math.random() * 4;
        ctx.beginPath();
        ctx.arc(x + 30 + Math.cos(angle) * radius, y + 40 + Math.sin(angle) * radius, 6 + Math.random() * 3, 0, Math.PI * 2);
        ctx.fillStyle = colors[i % colors.length];
        ctx.globalAlpha = 1 - frame / 12;
        ctx.fill();
        ctx.globalAlpha = 1;
      }
    }

    function resetGame() {
      dino.y = groundY - dino.height;
      dino.vy = 0;
      dino.jumping = false;
      cactusGroup.x = 600 + Math.random() * 200;
      cactusGroup.count = 1;
      score = 20;
      coinCount = 0;
      activeCoins = [];
      gameOver = false;
      speed = 6;
      explosion = null;
      backgroundOffset = 0;
      stopRunningSound();
      loop();
    }

    function generateStars() {
      stars = [];
      for (let i = 0; i < 80; i++) {
        stars.push({
          x: Math.random() * canvas.width,
          y: Math.random() * 300,
          r: Math.random() * 1.5 + 0.5
        });
      }
    }

    function drawHouse(house) {
      let baseY = groundY - 10;
      let width = house.pisos === 1 ? 60 : 70;
      let height = house.pisos === 1 ? 40 : 70;
      // Cuerpo
      ctx.fillStyle = '#fff';
      let houseX = (house.x + backgroundOffset * 0.3) % (canvas.width + 200);
      if (houseX < -100) houseX += canvas.width + 200;
      ctx.fillRect(houseX, baseY - height, width, height);
      // Techo
      ctx.fillStyle = '#a0522d';
      ctx.beginPath();
      ctx.moveTo(houseX - 5, baseY - height);
      ctx.lineTo(houseX + width / 2, baseY - height - 20);
      ctx.lineTo(houseX + width + 5, baseY - height);
      ctx.closePath();
      ctx.fill();
      // Puerta
      ctx.fillStyle = '#7c5a3a';
      ctx.fillRect(houseX + width / 2 - 8, baseY - 24, 16, 24);
      // Ventanas con luces
      // Ventana 1 (arriba izquierda, solo si 2 pisos)
      if (house.pisos === 2) {
        ctx.fillStyle = house.luces[0] ? '#ffe066' : '#b3e6ff';
        ctx.fillRect(houseX + 10, baseY - height + 10, 16, 16);
        ctx.fillStyle = house.luces[1] ? '#ffe066' : '#b3e6ff';
        ctx.fillRect(houseX + width - 26, baseY - height + 10, 16, 16);
      }
      // Ventana 2 (abajo izquierda)
      ctx.fillStyle = house.luces[2] ? '#ffe066' : '#b3e6ff';
      ctx.fillRect(houseX + 10, baseY - 36, 16, 12);
      // Ventana 3 (abajo derecha)
      ctx.fillStyle = house.luces[3] ? '#ffe066' : '#b3e6ff';
      ctx.fillRect(houseX + width - 26, baseY - 36, 16, 12);
    }

    function generateHouses() {
      houses = [];
      for (let i = 0; i < 8; i++) {
        let pisos = Math.random() < 0.5 ? 1 : 2;
        let x = 60 + i * 90 + Math.random() * 30;
        // Luces: [ventana1, ventana2, ventana3, ventana4]
        let luces = [false, false, false, false];
        houses.push({ x: x, pisos: pisos, luces: luces });
      }
    }



    function updateHouseLights() {
      for (let i = 0; i < houses.length; i++) {
        for (let j = 0; j < houses[i].luces.length; j++) {
          houses[i].luces[j] = Math.random() < 0.5;
        }
      }
    }

    function generateTrees() {
      trees = [];
      for (let i = 0; i < 8; i++) {
        let x = 60 + i * 90 + Math.random() * 30;
        let height = 60 + Math.random() * 80;
        let width = 18 + Math.random() * 12;
        trees.push({ x: x, height: height, width: width });
      }
    }



    function generateBirds() {
      birds = [];
      for (let i = 0; i < 5; i++) {
        let centerX = 200 + Math.random() * 400;
        let centerY = 80 + Math.random() * 100;
        let radius = 30 + Math.random() * 40;
        let size = 18 + Math.random() * 10;
        let angle = Math.random() * Math.PI * 2;
        birds.push({ centerX, centerY, radius, size, angle });
      }
    }

    function updateBirds() {
      for (let i = 0; i < birds.length; i++) {
        birds[i].angle += 0.015 + i * 0.003;
        if (birds[i].angle > Math.PI * 2) birds[i].angle -= Math.PI * 2;
      }
    }

    function drawDot(dot) {
      let x = dot.centerX + Math.cos(dot.angle) * dot.radius;
      let y = dot.centerY + Math.sin(dot.angle) * dot.radius;
      // Dibujar una 'V' estilizada para simular un pájaro
      ctx.strokeStyle = '#222';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(x - 7, y);
      ctx.lineTo(x, y + 5);
      ctx.lineTo(x + 7, y);
      ctx.stroke();
      ctx.lineWidth = 1;
    }

    function drawTree(tree) {
      let baseY = groundY - 10;
      let treeX = (tree.x + backgroundOffset * 0.2) % (canvas.width + 200);
      if (treeX < -100) treeX += canvas.width + 200;
      // Tronco
      ctx.fillStyle = '#8b5a2b';
      ctx.fillRect(treeX + tree.width / 2 - 5, baseY - tree.height + 30, 10, tree.height - 30);
      // Copa
      ctx.beginPath();
      ctx.arc(treeX + tree.width / 2, baseY - tree.height + 30, tree.width * 1.2, 0, Math.PI * 2);
      ctx.fillStyle = '#2ecc40';
      ctx.fill();
      ctx.beginPath();
      ctx.arc(treeX + tree.width / 2, baseY - tree.height + 10, tree.width, 0, Math.PI * 2);
      ctx.fillStyle = '#27ae60';
      ctx.fill();
    }

    function playRunningSound() {
      if (!runningSoundPlaying) {
        var runningSound = document.getElementById('runningSound');
        runningSound.currentTime = 0;
        runningSound.play().then(() => {
          runningSoundPlaying = true;
        }).catch(e => {
          console.log('Error playing running sound:', e);
        });
      }
    }

    function stopRunningSound() {
      if (runningSoundPlaying) {
        var runningSound = document.getElementById('runningSound');
        runningSound.pause();
        runningSound.currentTime = 0;
        runningSoundPlaying = false;
      }
    }

    function loop() {
      // Cambiar a modo noche entre 5 y 14 puntos, y a día a partir de 15
      if (score >= 5 && score < 15) {
        if (!nightMode) generateStars();
        nightMode = true;
      } else {
        nightMode = false;
      }
      // Generar casas si score >= 7 y aún no hay
      if (nightMode && score >= 7 && score < 15 && houses.length === 0) {
        generateHouses();
        if (houseLightsInterval) clearInterval(houseLightsInterval);
        houseLightsInterval = setInterval(updateHouseLights, 1000);
      }
      if ((!nightMode || score < 7 || score >= 15) && houses.length > 0) {
        houses = [];
        if (houseLightsInterval) {
          clearInterval(houseLightsInterval);
          houseLightsInterval = null;
        }
      }
      // Generar árboles y pájaros si score >= 15 y aún no hay
      if (!nightMode && score >= 15 && trees.length === 0) {
        generateTrees();
        generateBirds();
      }
      if ((nightMode || score < 15) && (trees.length > 0 || birds.length > 0)) {
        trees = [];
        birds = [];
      }
      // Actualizar pájaros si es de día y score >= 15
      if (!nightMode && score >= 15) {
        updateBirds();
      }
      // Actualizar offset de fondo (movimiento contrario a los obstáculos)
      backgroundOffset -= speed * 0.5;
      

      
      // Movimiento de la piedra grande
      if (score >= 20) {
        if (bigRockX === null) bigRockX = 800;
        bigRockX -= speed;
        if (bigRockX + 80 < 0) {
          bigRockX = 800 + Math.random() * 200;
          // Reinicia la moneda de la piedra
          activeCoins = [];
        }
      } else {
        bigRockX = null;
      }
      // Detectar si Steve pasa exitosamente un obstáculo para sumar moneda
      if (score >= 20 && bigRock) {
        if (!lastObstaclePassed && bigRock.x + bigRock.width < dino.x) {
          coinCount++;
          lastObstaclePassed = true;
        }
        if (bigRock.x + bigRock.width >= dino.x) {
          lastObstaclePassed = false;
        }
      } else {
        for (let i = 0; i < cactusGroup.count; i++) {
          let obsX = cactusGroup.x + i * 28;
          if (!lastObstaclePassed && obsX + cactusGroup.width < dino.x) {
            coinCount++;
            lastObstaclePassed = true;
          }
          if (obsX + cactusGroup.width >= dino.x) {
            lastObstaclePassed = false;
          }
        }
      }
      // Colisión de Steve con monedas
      for (let i = 0; i < activeCoins.length; i++) {
        if (!activeCoins[i].collected) {
          let cx = activeCoins[i].x;
          let cy = activeCoins[i].y;
          if (
            dino.x + dino.width > cx - 12 &&
            dino.x < cx + 12 &&
            dino.y + dino.height > cy - 12 &&
            dino.y < cy + 12
          ) {
            activeCoins[i].collected = true;
            coinCount++;
          }
        }
      }
      // Mover drawBackground después de updateBirds
      drawBackground();
      drawGround();
      if (!explosion) {
        drawDino();
      }
      drawCactusGroup(cactusGroup);
      drawScore();

      // Ajustar cantidad de cactus según el puntaje
      cactusGroup.count = Math.min(1 + Math.floor(score / 5), 5); // máximo 5 juntos

      // Aumentar velocidad cuando hay 2 o más cactus
      if (cactusGroup.count >= 2) {
        speed = 10;
      } else {
        speed = 6;
      }

      // Movimiento del grupo de cactus
      cactusGroup.x -= speed;
      if (cactusGroup.x + cactusGroup.width * cactusGroup.count + 28 * (cactusGroup.count - 1) < 0) {
        cactusGroup.x = 600 + Math.random() * 200;
        score++;
        if (speed < 15) speed += 0.2;
        // Reinicia las monedas de cactus
        activeCoins = [];
      }

      // Gravedad y salto
      dino.y += dino.vy;
      if (dino.y + dino.height < groundY) {
        dino.vy += gravity;
      } else {
        dino.y = groundY - dino.height;
        dino.vy = 0;
        dino.jumping