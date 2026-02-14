<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Christmas RPG v1.0.0 - Holiday Dungeon</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <style>
    body { margin: 0; display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh; background-color: #000; overflow: hidden; touch-action: none; }
    canvas { width: 100vw; height: calc(100vh - 100px); max-width: 800px; max-height: 800px; }
    #controls { position: fixed; bottom: 10px; display: grid; grid-template-areas: ". up ." "left attack right"; gap: 15px; width: 90%; max-width: 300px; justify-items: center; align-items: center; z-index: 10; }
    .control-btn { width: 60px; height: 60px; background: #d42426; border: 2px solid #165b33; border-radius: 8px; color: #fff; font-size: 24px; display: flex; align-items: center; justify-content: center; touch-action: manipulation; cursor: pointer; }
    #up { grid-area: up; } #left { grid-area: left; } #right { grid-area: right; } #attack { grid-area: attack; background: #f8b229; color: #000; }
    #hud { position: fixed; top: 10px; left: 10px; color: #fff; font-size: 16px; font-family: Arial, sans-serif; z-index: 10; text-shadow: 1px 1px 2px #000; }
    #notification { position: fixed; top: 40px; left: 10px; color: #f8b229; font-size: 14px; font-family: Arial, sans-serif; z-index: 10; transition: opacity 2s; text-shadow: 1px 1px 2px #000; }
    #dialogue-box { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(22, 91, 51, 0.9); color: #fff; padding: 20px; border: 2px solid #d42426; border-radius: 8px; z-index: 20; display: none; max-width: 80%; text-align: center; font-family: Arial, sans-serif; }
    #dialogue-image { width: 100px; height: 100px; background: #555; margin: 0 auto 10px; background-size: contain; background-repeat: no-repeat; background-position: center; }
    #dialogue-text { margin-bottom: 10px; }
    .dialogue-btn { background: #d42426; color: #fff; border: none; padding: 10px; margin: 5px; cursor: pointer; border-radius: 5px; }
    .flash { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 0, 0, 0.5); z-index: 15; opacity: 0; pointer-events: none; }
    @media (max-width: 400px) { .control-btn { width: 50px; height: 50px; font-size: 20px; } #controls { gap: 10px; bottom: 5px; } canvas { height: calc(100vh - 80px); } }
    @media (min-width: 601px) { .control-btn { width: 70px; height: 70px; font-size: 28px; } }
  </style>
</head>
<body>
  <div id="hud">Score: 0  HP: 100  Weapon: None</div>
  <div id="notification"></div>
  <div id="dialogue-box">
    <div id="dialogue-image"></div>
    <div id="dialogue-text"></div>
    <button class="dialogue-btn" id="choice1"></button>
    <button class="dialogue-btn" id="choice2"></button>
  </div>
  <div id="flash" class="flash"></div>
  <div id="controls">
    <div class="control-btn" id="up">↑</div>
    <div class="control-btn" id="left">←</div>
    <div class="control-btn" id="attack">⚔️</div>
    <div class="control-btn" id="right">→</div>
  </div>
  
  <!-- Holiday Music -->
  <audio id="bgm" loop autoplay>
    <source src="https://arweave.net/3QaXlF77IDjwKKIsROMldfaE9XWh5cIkM_E6556BreE" type="audio/mpeg">
  </audio>

  <script>
    let scene, camera, renderer, radarLight;
    let leftHand, rightHand;
    let maze, regions, player, cols, rows, cellSize;
    let monsters = [];
    let treasures = [];
    let traps = [];
    let events = [];
    let goalMesh;
    let radarAngle = 0;
    let radarRange = 3;
    let animationState = { isAnimating: false, type: null, startTime: 0, duration: 300 };
    let handAnimation = { offset: 0, speed: 0.05 };
    let weapons = [
      { name: 'None', damage: 10 },
      { name: 'Candy Cane', damage: 20 },
      { name: 'Ornament Bomb', damage: 30 }
    ];
    let isDialogueActive = false;

    // Updated Assets for Christmas Theme
    const wallTextureUrl = 'https://arweave.net/OD1cNP8ruEeADeWPzTGeshzPBSqVLH00QRZQGWHQli8'; // Gift/Present texture for walls
    const floorTextureUrl = 'https://arweave.net/OD1cNP8ruEeADeWPzTGeshzPBSqVLH00QRZQGWHQli8'; // Gift/Present texture for floor (wrapping paper world)
    const monsterSpriteUrl = 'https://arweave.net/TGvoYCn0g9rKaNV_V2OSwsVwu4ZgxFnYNULVGeU7JdU'; // Gingerbread Man
    const treasureSpriteUrl = 'https://arweave.net/TmcSXpfuDJPXmh9F3hMvqbgyfVHANpCex7GNnpsmv2M'; // Chocolate Chip Cookie
    const trapSpriteUrl = 'https://arweave.net/AxhTRbFJ9WU7hDqSfd3gjHj9_mAGpakkXHxITYpjtAE'; // Holiday Bell
    const eventSpriteUrl = 'https://arweave.net/BufnZYf3hyWFDo6QJeV3M1YC2UyBqxE75NBl2pBr68c'; // Snowman
    const goalSpriteUrl = 'https://arweave.net/ol8b1uQnffHbCWTEfwp6A3cTKzZVH3fgQ3sj9xicQw4'; // Sleigh

    function init() {
      // Try to play audio on interaction if autoplay fails
      document.body.addEventListener('click', () => {
        const bgm = document.getElementById('bgm');
        if(bgm.paused) bgm.play();
      }, { once: true });

      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x051014); // Dark winter night
      camera = new THREE.PerspectiveCamera(90, window.innerWidth / (window.innerHeight - 100), 0.1, 1000);
      renderer = new THREE.WebGLRenderer({ antialias: true });
      let size = Math.min(window.innerWidth, window.innerHeight - 100, 800);
      renderer.setSize(size, size);
      document.body.appendChild(renderer.domElement);

      cellSize = 1;
      cols = 12;
      rows = 12;
      maze = generateMaze(cols, rows);
      regions = generateRegions();
      player = { x: 1, y: 1, score: 0, hp: 100, direction: 0, weapon: weapons[0] };

      const loader = new THREE.TextureLoader();
      const wallTex = loader.load(wallTextureUrl);
      wallTex.wrapS = wallTex.wrapT = THREE.RepeatWrapping;
      // wallTex.repeat.set(1, 1); // Adjusted for gift texture
      
      const floorTex = loader.load(floorTextureUrl);
      floorTex.wrapS = floorTex.wrapT = THREE.RepeatWrapping;
      floorTex.repeat.set(4, 4);

      createMazeGeometry(wallTex, floorTex);
      createMonsters();
      createTreasures();
      createTraps();
      createEvents();
      createGoal();
      createHands();

      const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
      scene.add(ambientLight);
      const directionalLight = new THREE.DirectionalLight(0xffd700, 0.5); // Gold light
      directionalLight.position.set(0, 2, 0);
      scene.add(directionalLight);
      radarLight = new THREE.SpotLight(0xff0000, 0.8, radarRange * cellSize * 2, Math.PI / 4); // Red radar
      scene.add(radarLight);

      updateCamera();

      document.getElementById('up').addEventListener('touchstart', (e) => { e.preventDefault(); movePlayer(38); });
      document.getElementById('left').addEventListener('touchstart', (e) => { e.preventDefault(); rotatePlayer('left'); });
      document.getElementById('right').addEventListener('touchstart', (e) => { e.preventDefault(); rotatePlayer('right'); });
      document.getElementById('attack').addEventListener('touchstart', (e) => { e.preventDefault(); attackMonster(); });
      document.addEventListener('keydown', (e) => {
        if (isDialogueActive) return;
        if (e.keyCode === 38 || e.keyCode === 87) movePlayer(38);
        if (e.keyCode === 37 || e.keyCode === 65) rotatePlayer('left');
        if (e.keyCode === 39 || e.keyCode === 68) rotatePlayer('right');
        if (e.keyCode === 32) attackMonster();
      });

      window.addEventListener('resize', onWindowResize);

      animate();
    }

    function generateMaze(cols, rows) {
      let grid = [];
      for (let y = 0; y < rows; y++) {
        grid[y] = [];
        for (let x = 0; x < cols; x++) {
          grid[y][x] = { walls: [true, true, true, true], visited: false };
        }
      }
      function carve(x, y) {
        grid[y][x].visited = true;
        let directions = shuffle([[0, -1], [1, 0], [0, 1], [-1, 0]]);
        for (let [dx, dy] of directions) {
          let nx = x + dx;
          let ny = y + dy;
          if (nx >= 0 && nx < cols && ny >= 0 && ny < rows && !grid[ny][nx].visited) {
            if (dx === 1) {
              grid[y][x].walls[1] = false;
              grid[ny][nx].walls[3] = false;
            } else if (dx === -1) {
              grid[y][x].walls[3] = false;
              grid[ny][nx].walls[1] = false;
            } else if (dy === 1) {
              grid[y][x].walls[2] = false;
              grid[ny][nx].walls[0] = false;
            } else if (dy === -1) {
              grid[y][x].walls[0] = false;
              grid[ny][nx].walls[2] = false;
            }
            carve(nx, ny);
          }
        }
      }
      carve(0, 0);
      grid[rows - 2][cols - 2].walls = [false, false, false, false];
      return grid;
    }

    function shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    }

    function generateRegions() {
      let regions = [];
      const types = [
        { type: 'Normal', wallColor: 0x00ff00, floorColor: 0x003300 },
        { type: 'Danger', wallColor: 0xff0000, floorColor: 0x330000 },
        { type: 'Treasure', wallColor: 0xffff00, floorColor: 0x333300 },
        { type: 'Safe', wallColor: 0x0000ff, floorColor: 0x000033 }
      ];
      for (let y = 0; y < rows; y++) {
        regions[y] = [];
        for (let x = 0; x < cols; x++) {
          regions[y][x] = types[Math.floor(Math.random() * 4)];
        }
      }
      // Smoothing regions (simplified)
      regions[1][1] = types[0];
      regions[rows - 2][cols - 2] = types[3];
      return regions;
    }

    function createMazeGeometry(wallTex, floorTex) {
      const wallHeight = 1.2;
      const wallMat = new THREE.MeshLambertMaterial({ map: wallTex });
      // Darken the floor texture to distinguish it from walls
      const floorMat = new THREE.MeshLambertMaterial({ map: floorTex, color: 0x888888 }); 
      
      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
          let floor = new THREE.Mesh(new THREE.PlaneGeometry(cellSize, cellSize), floorMat);
          floor.rotation.x = -Math.PI / 2;
          floor.position.set(x * cellSize, 0, y * cellSize);
          scene.add(floor);
          
          const wallGeo = new THREE.BoxGeometry(cellSize, wallHeight, 0.1);
          if (maze[y][x].walls[0]) { let wall = new THREE.Mesh(wallGeo, wallMat); wall.position.set(x * cellSize, wallHeight / 2, y * cellSize - cellSize / 2); scene.add(wall); }
          if (maze[y][x].walls[1]) { let wall = new THREE.Mesh(wallGeo, wallMat); wall.rotation.y = Math.PI / 2; wall.position.set(x * cellSize + cellSize / 2, wallHeight / 2, y * cellSize); scene.add(wall); }
          if (maze[y][x].walls[2]) { let wall = new THREE.Mesh(wallGeo, wallMat); wall.position.set(x * cellSize, wallHeight / 2, y * cellSize + cellSize / 2); scene.add(wall); }
          if (maze[y][x].walls[3]) { let wall = new THREE.Mesh(wallGeo, wallMat); wall.rotation.y = Math.PI / 2; wall.position.set(x * cellSize - cellSize / 2, wallHeight / 2, y * cellSize); scene.add(wall); }
        }
      }
    }

    function createSprite(url, scale = 0.8, height = 0.4) {
      const loader = new THREE.TextureLoader();
      const tex = loader.load(url);
      const mat = new THREE.MeshBasicMaterial({ map: tex, transparent: true, side: THREE.DoubleSide });
      const mesh = new THREE.Mesh(new THREE.PlaneGeometry(1, 1), mat);
      mesh.scale.set(scale, scale, scale);
      mesh.position.y = height;
      return mesh;
    }

    function createMonsters() {
      const monsterCount = 5;
      for (let i = 0; i < monsterCount; i++) {
        let x, y;
        do { x = Math.floor(Math.random() * cols); y = Math.floor(Math.random() * rows); } while ((x === 1 && y === 1) || (x === cols - 2 && y === rows - 2));
        let mesh = createSprite(monsterSpriteUrl, 0.7, 0.35);
        mesh.position.set(x * cellSize, 0, y * cellSize);
        scene.add(mesh);
        monsters.push({ x, y, hp: 50, mesh });
      }
    }

    function createTreasures() {
      const treasureCount = 3;
      for (let i = 0; i < treasureCount; i++) {
        let x, y;
        do { x = Math.floor(Math.random() * cols); y = Math.floor(Math.random() * rows); } while ((x === 1 && y === 1) || (x === cols - 2 && y === rows - 2) || monsters.some(m => m.x === x && m.y === y));
        let mesh = createSprite(treasureSpriteUrl, 0.6, 0.3);
        mesh.position.set(x * cellSize, 0, y * cellSize);
        scene.add(mesh);
        treasures.push({ x, y, mesh });
      }
    }

    function createTraps() {
      const trapCount = 3;
      for (let i = 0; i < trapCount; i++) {
        let x, y;
        do { x = Math.floor(Math.random() * cols); y = Math.floor(Math.random() * rows); } while ((x === 1 && y === 1) || (x === cols - 2 && y === rows - 2));
        let mesh = createSprite(trapSpriteUrl, 0.8, 0.05);
        mesh.rotation.x = -Math.PI / 2;
        mesh.position.set(x * cellSize, 0.01, y * cellSize);
        scene.add(mesh);
        traps.push({ x, y, mesh });
      }
    }

    function createEvents() {
      const eventCount = 2;
      for (let i = 0; i < eventCount; i++) {
        let x, y;
        do { x = Math.floor(Math.random() * cols); y = Math.floor(Math.random() * rows); } while ((x === 1 && y === 1) || (x === cols - 2 && y === rows - 2));
        let mesh = createSprite(eventSpriteUrl, 0.7, 0.45);
        mesh.position.set(x * cellSize, 0, y * cellSize);
        scene.add(mesh);
        events.push({ x, y, mesh });
      }
    }

    function createGoal() {
      goalMesh = createSprite(goalSpriteUrl, 0.8, 0.4);
      goalMesh.position.set((cols - 2) * cellSize, 0, (rows - 2) * cellSize);
      scene.add(goalMesh);
    }

    function createHands() {
      // Simple colored hands, changing color on weapon upgrade
      const handGeo = new THREE.BoxGeometry(0.1, 0.05, 0.2);
      let handMat = new THREE.MeshLambertMaterial({ color: 0xcccccc });
      leftHand = new THREE.Mesh(handGeo, handMat);
      rightHand = new THREE.Mesh(handGeo, handMat);
      leftHand.position.set(-0.2, -0.2, -0.5);
      rightHand.position.set(0.2, -0.2, -0.5);
      camera.add(leftHand);
      camera.add(rightHand);
      scene.add(camera);
    }

    function updateHands() {
      const colors = [0xcccccc, 0xff0000, 0x00ff00]; // Grey, Red, Green
      leftHand.material.color.setHex(colors[weapons.indexOf(player.weapon)]);
      rightHand.material.color.setHex(colors[weapons.indexOf(player.weapon)]);
      const bob = Math.sin(handAnimation.offset) * 0.02;
      leftHand.position.y = -0.2 + bob;
      rightHand.position.y = -0.2 + bob;
      handAnimation.offset += handAnimation.speed;
    }

    function updateBillboards() {
      const cameraDirection = new THREE.Vector3();
      camera.getWorldDirection(cameraDirection);
      monsters.forEach(m => { m.mesh.lookAt(camera.position); });
      treasures.forEach(t => { t.mesh.lookAt(camera.position); });
      events.forEach(e => { e.mesh.lookAt(camera.position); });
      if (goalMesh) goalMesh.lookAt(camera.position);
    }

    function animate() {
      requestAnimationFrame(animate);
      radarLight.position.set(player.x * cellSize, 1, player.y * cellSize);
      radarLight.target.position.set(
        player.x * cellSize + Math.cos(radarAngle) * radarRange * cellSize,
        0,
        player.y * cellSize + Math.sin(radarAngle) * radarRange * cellSize
      );
      radarLight.target.updateMatrixWorld();
      radarAngle += 0.05;
      updateBillboards();
      updateHands();
      updateCamera();
      renderer.render(scene, camera);
    }

    function triggerTreasure(x, y) {
      const treasureIndex = treasures.findIndex(t => t.x === x && t.y === y);
      if (treasureIndex === -1) return;
      const treasure = treasures[treasureIndex];
      scene.remove(treasure.mesh);
      treasures.splice(treasureIndex, 1);
      const hpGain = Math.floor(Math.random() * 20) + 10;
      const scoreGain = Math.floor(Math.random() * 20) + 10;
      player.hp = Math.min(player.hp + hpGain, 100);
      player.score += scoreGain;
      const weaponRoll = Math.random();
      let newWeapon = player.weapon;
      if (weaponRoll < 0.3) newWeapon = weapons[1]; // Candy Cane
      else if (weaponRoll < 0.5) newWeapon = weapons[2]; // Ornament Bomb
      player.weapon = newWeapon;
      showNotification(`Found Cookie! +${scoreGain} Score, +${hpGain} HP, Got ${newWeapon.name}`);
      updateHUD();
    }

    function triggerTrap(x, y) {
      const trapIndex = traps.findIndex(t => t.x === x && t.y === y);
      if (trapIndex === -1) return;
      traps.splice(trapIndex, 1);
      const hpLoss = Math.floor(Math.random() * 20) + 10;
      const scoreLoss = Math.floor(Math.random() * 10) + 5;
      player.hp -= hpLoss;
      player.score = Math.max(0, player.score - scoreLoss);
      showNotification(`Hit Bell Trap! -${hpLoss} HP, -${scoreLoss} Score`);
      triggerFlash();
      updateHUD();
      if (player.hp <= 0) {
        showNotification(`Game Over! HP reached 0. Final Score: ${player.score}`);
        resetGame();
      }
    }

    function triggerEvent(x, y) {
      const eventIndex = events.findIndex(e => e.x === x && e.y === y);
      if (eventIndex === -1) return;
      events.splice(eventIndex, 1);
      showDialogue();
    }

    function showDialogue() {
      isDialogueActive = true;
      const dialogueBox = document.getElementById('dialogue-box');
      const dialogueText = document.getElementById('dialogue-text');
      const choice1 = document.getElementById('choice1');
      const choice2 = document.getElementById('choice2');
      dialogueText.innerText = "A Snowman appears: 'Do you want a gift?'";
      choice1.innerText = "Yes, please!";
      choice2.innerText = "No, it's a trick!";
      dialogueBox.style.display = 'block';
      choice1.onclick = () => {
        const scoreChange = Math.random() > 0.5 ? 20 : -20;
        player.score = Math.max(0, player.score + scoreChange);
        showNotification(`Opened gift! Score ${scoreChange > 0 ? '+' : ''}${scoreChange}`);
        updateHUD();
        dialogueBox.style.display = 'none';
        isDialogueActive = false;
      };
      choice2.onclick = () => {
        const hpChange = Math.random() > 0.5 ? 10 : -10;
        player.hp = Math.max(0, Math.min(100, player.hp + hpChange));
        showNotification(`Ignored Snowman! HP ${hpChange > 0 ? '+' : ''}${hpChange}`);
        updateHUD();
        if (player.hp <= 0) {
          showNotification(`Game Over! HP reached 0. Final Score: ${player.score}`);
          resetGame();
        }
        dialogueBox.style.display = 'none';
        isDialogueActive = false;
      };
    }

    function showNotification(message) {
      const notification = document.getElementById('notification');
      notification.innerText = message;
      notification.style.opacity = 1;
      setTimeout(() => { notification.style.opacity = 0; }, 2000);
    }

    function triggerFlash() {
      const flash = document.getElementById('flash');
      flash.style.opacity = 0.5;
      setTimeout(() => { flash.style.opacity = 0; }, 200);
    }

    function attackMonster() {
      if (animationState.isAnimating || isDialogueActive) return;
      const directions = [
        { dx: 0, dy: -1 }, // North
        { dx: 1, dy: 0 }, // East
        { dx: 0, dy: 1 }, // South
        { dx: -1, dy: 0 } // West
      ];
      const dir = directions[player.direction];
      const targetX = player.x + dir.dx;
      const targetY = player.y + dir.dy;
      if (targetX < 0 || targetX >= cols || targetY < 0 || targetY >= rows) return;
      const monsterIndex = monsters.findIndex(m => m.x === targetX && m.y === targetY && m.hp > 0);
      if (monsterIndex === -1) {
        showNotification(`No monster to attack!`);
        return;
      }
      animationState = {
        isAnimating: true,
        type: 'attack',
        startTime: performance.now(),
        duration: 300
      };
      const monster = monsters[monsterIndex];
      monster.hp -= player.weapon.damage;
      showNotification(`Attacked Gingerbread Man! HP: ${monster.hp}`);
      triggerFlash();
      if (monster.hp <= 0) {
        scene.remove(monster.mesh);
        monsters.splice(monsterIndex, 1);
        showNotification(`Gingerbread Man crumbled!`);
      } else {
        player.hp -= 20;
        showNotification(`Gingerbread Man bites back! -20 HP`);
        triggerFlash();
        if (player.hp <= 0) {
          showNotification(`Game Over! HP reached 0. Final Score: ${player.score}`);
          resetGame();
        }
      }
      updateHUD();
    }

    function movePlayer(keyCode) {
      if (animationState.isAnimating || isDialogueActive) return;
      let newX = player.x;
      let newY = player.y;
      let wallIndex;
      if (keyCode === 38) {
        if (player.direction === 0) { // North
          newY--;
          wallIndex = 0;
        } else if (player.direction === 1) { // East
          newX++;
          wallIndex = 1;
        } else if (player.direction === 2) { // South
          newY++;
          wallIndex = 2;
        } else if (player.direction === 3) { // West
          newX--;
          wallIndex = 3;
        }
        if (newX >= 0 && newX < cols && newY >= 0 && newY < rows && !maze[player.y][player.x].walls[wallIndex]) {
          if (monsters.some(m => m.x === newX && m.y === newY && m.hp > 0)) {
            showNotification(`Cannot move! Gingerbread Man blocks the path.`);
            return;
          }
          animationState = {
            isAnimating: true,
            type: 'move',
            startTime: performance.now(),
            duration: 300,
            startPos: { x: player.x * cellSize, z: player.y * cellSize },
            endPos: { x: newX * cellSize, z: newY * cellSize }
          };
          player.x = newX;
          player.y = newY;
          triggerTreasure(newX, newY);
          triggerTrap(newX, newY);
          triggerEvent(newX, newY);
          if (player.x === cols - 2 && player.y === rows - 2) {
            showNotification(`Merry Christmas! You reached the Sleigh! Final Score: ${player.score}, HP: ${player.hp}`);
            resetGame();
          }
        }
      }
    }

    function rotatePlayer(direction) {
      if (animationState.isAnimating || isDialogueActive) return;
      let newDirection = player.direction;
      let deltaYaw;
      if (direction === 'right') {
        newDirection = (player.direction + 1) % 4;
        deltaYaw = -Math.PI / 2;
      } else if (direction === 'left') {
        newDirection = (player.direction - 1 + 4) % 4;
        deltaYaw = Math.PI / 2;
      }
      animationState = {
        isAnimating: true,
        type: 'rotate',
        startTime: performance.now(),
        duration: 300,
        startYaw: getCurrentYaw(),
        deltaYaw: deltaYaw
      };
      player.direction = newDirection;
    }

    function getCurrentYaw() {
      if (player.direction === 0) return 0;
      else if (player.direction === 1) return -Math.PI / 2;
      else if (player.direction === 2) return Math.PI;
      else return Math.PI / 2;
    }

    function resetGame() {
      scene.clear();
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
      scene.add(ambientLight);
      const directionalLight = new THREE.DirectionalLight(0xffd700, 0.5);
      directionalLight.position.set(0, 2, 0);
      scene.add(directionalLight);
      radarLight = new THREE.SpotLight(0xff0000, 0.8, radarRange * cellSize * 2, Math.PI / 4);
      scene.add(radarLight);
      maze = generateMaze(cols, rows);
      regions = generateRegions();
      player = { x: 1, y: 1, score: 0, hp: 100, direction: 0, weapon: weapons[0] };
      monsters = [];
      treasures = [];
      traps = [];
      events = [];
      createMazeGeometry(new THREE.TextureLoader().load(wallTextureUrl), new THREE.TextureLoader().load(floorTextureUrl));
      createMonsters();
      createTreasures();
      createTraps();
      createEvents();
      createGoal();
      createHands();
      updateHUD();
      updateCamera();
      animationState.isAnimating = false;
      isDialogueActive = false;
      document.getElementById('dialogue-box').style.display = 'none';
    }

    function updateCamera() {
      if (animationState.isAnimating) {
        let elapsed = performance.now() - animationState.startTime;
        let t = Math.min(elapsed / animationState.duration, 1);
        if (animationState.type === 'move') {
          let x = animationState.startPos.x + (animationState.endPos.x - animationState.startPos.x) * t;
          let z = animationState.startPos.z + (animationState.endPos.z - animationState.startPos.z) * t;
          camera.position.set(x, cellSize * 0.5, z);
        } else if (animationState.type === 'rotate') {
          let yaw = animationState.startYaw + animationState.deltaYaw * t;
          camera.rotation.set(0, yaw, 0);
        } else if (animationState.type === 'attack') {
          // No camera movement during attack
        }
        if (t >= 1) {
          animationState.isAnimating = false;
          camera.position.set(player.x * cellSize, cellSize * 0.5, player.y * cellSize);
          camera.rotation.set(0, getCurrentYaw(), 0);
        }
      } else {
        camera.position.set(player.x * cellSize, cellSize * 0.5, player.y * cellSize);
        camera.rotation.set(0, getCurrentYaw(), 0);
      }
      camera.rotation.order = 'YXZ';
    }

    function updateHUD() {
      document.getElementById('hud').innerText = `Score: ${player.score}  HP: ${player.hp}  Weapon: ${player.weapon.name}`;
    }

    function onWindowResize() {
      let size = Math.min(window.innerWidth, window.innerHeight - 100, 800);
      renderer.setSize(size, size);
      camera.aspect = size / size;
      camera.updateProjectionMatrix();
    }

    init();
  </script>
</body>
</html>
