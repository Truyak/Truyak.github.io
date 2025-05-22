# AI ile olan konuşmaları içeren dosya

## Birinci Sorgu
-ilk bölümde oyunu anlatan bir yazı olmasını istiyorum. "Yeşil olan çantalar yem'dir. Onları yiyip büyürsün ve ateş edebilirsin. Basınç plakasına gelip ateş etmeyi dene ve sihri gör" tarzında bir yazıyı ilk bölümde yazdırabilir miyiz? Kontrolleri ise sadece ilk 2 bölümde gösterelim. diğer bölümlerde sadece level sayısı yazsın.
kodum:
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Getting GOOpy</title>
  <style>
    canvas { border: 1px solid black; position: absolute; top: 0; left: 0; }
    body { margin: 0; background: #f0f0f0; overflow: hidden; }
    button {
      background: #fbca1f;
      font-family: inherit;
      padding: 0.6em 1.3em;
      font-weight: 900;
      font-size: 18px;
      border: 3px solid black;
      border-radius: 0.4em;
      box-shadow: 0.1em 0.1em;
      cursor: pointer;
      position: absolute;
      display: none; /* Varsayılan olarak gizli */
    }
    button:hover {
      transform: translate(-0.05em, -0.05em);
      box-shadow: 0.15em 0.15em;
    }
    button:active {
      transform: translate(0.05em, 0.05em);
      box-shadow: 0.05em 0.05em;
    }
    button.visible {
      display: block; /* Ana menüde ve zafer ekranında görünür */
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas"></canvas>
  <button id="playButton">Play</button>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const playButtonElement = document.getElementById('playButton');
    // Ölçekleme faktörleri (800x600 → 1920x1080)
    const scaleFactorX = 2.4; // 1920 / 800
    const scaleFactorY = 1.8; // 1080 / 600
    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      scaleX = canvas.width / 1920;
      scaleY = canvas.height / 1080;
      playButtonElement.style.left = `${860 * scaleX}px`;
      playButtonElement.style.top = `${490 * scaleY}px`;
      playButtonElement.style.width = `${200 * scaleX}px`;
      playButtonElement.style.height = `${100 * scaleY}px`;
      playButtonElement.style.fontSize = `${18 * Math.min(scaleX, scaleY)}px`;
    }
    let scaleX = 1;
    let scaleY = 1;
    resizeCanvas();
    window.addEventListener('resize', resizeCanvas);
    let currentLevel = 1;
    let gameState = {
      screen: 'menu',
      levelText: `Level ${currentLevel}`,
      controlsText: 'Arrow Keys: Move, Space: Jump, Mouse: Fire Goo, R: Restart',
      controlsAlpha: 1,
      controlsTimer: 0,
      audioContext: null,
      assetsLoaded: false
    };
    let player = {
      x: 240,
      y: 900,
      w: 72,
      h: 54,
      vx: 0,
      vy: 0,
      speed: 4.8,
      airSpeed: 4.8,
      jumpForce: -10.8,
      grounded: false,
      color: 'gray',
      sizeMultiplier: 1,
      recoil: 9.6,
      foodCount: 0,
      animation: {
        state: 'idle',
        fireEndTime: 0,
        frame: 0,
        frameTimer: 0,
        frameDurations: {
          idle: 1200,
          right: 800,
          left: 800,
          jump: 600,
          fire: 400
        }
      }
    };
    let platforms = [];
    let movingPlatforms = [];
    let walls = [];
    let hazardWalls = [];
    let goos = [];
    let pressurePlates = [];
    let coloredWalls = [];
    let foods = [];
    const gravity = 0.18;
    const gooGravity = 0.18;
    const gooSpeed = 12;
    const friction = 0.4;
    const gooImmuneTime = 100;
    const epsilon = 0.24;
    const brickWidth = 80;
    const brickHeight = 50;
    const sprites = { animations: {} };
    const sounds = {};
    async function loadAssets() {
      const imageFiles = [
        { key: 'player', src: 'assets/images/player.png' },
        { key: 'player_green', src: 'assets/images/player_green.png' },
        { key: 'food', src: 'assets/images/food.png' },
        { key: 'goo', src: 'assets/images/goo.png' },
        { key: 'platform', src: 'assets/images/platform.png' },
        { key: 'moving_platform', src: 'assets/images/moving_platform.png' },
        { key: 'brick', src: 'assets/images/brick.png' },
        { key: 'hazard_wall', src: 'assets/images/hazard_wall.png' },
        { key: 'pressure_plate_red', src: 'assets/images/pressure_plate_red.png' },
        { key: 'pressure_plate_yellow', src: 'assets/images/pressure_plate_yellow.png' },
        { key: 'pressure_plate_blue', src: 'assets/images/pressure_plate_blue.png' },
        { key: 'pressure_plate_brown', src: 'assets/images/pressure_plate_brown.png' },
        { key: 'colored_wall_red', src: 'assets/images/colored_wall_red.png' },
        { key: 'colored_wall_yellow', src: 'assets/images/colored_wall_yellow.png' },
        { key: 'colored_wall_blue', src: 'assets/images/colored_wall_blue.png' },
        { key: 'colored_wall_brown', src: 'assets/images/colored_wall_brown.png' },
        { key: 'colored_wall_active', src: 'assets/images/colored_wall_active.png' },
        { key: 'menu_background', src: 'assets/images/menu_background.png' },
        { key: 'background1', src: 'assets/images/background1.png' },
        { key: 'background2', src: 'assets/images/background2.png' },
        { key: 'background3', src: 'assets/images/background3.png' },
        { key: 'background4', src: 'assets/images/background4.png' },
        { key: 'background5', src: 'assets/images/background5.png' },
        { key: 'player_right', src: 'assets/images/animations/RightWalkingAnimation.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_right_green', src: 'assets/images/animations/RightWalkingAnimationGreen.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_left', src: 'assets/images/animations/LeftWalkingAnimation.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_left_green', src: 'assets/images/animations/LeftWalkingAnimationGreen.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_jump', src: 'assets/images/animations/JumpStart.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_jump_green', src: 'assets/images/animations/JumpGreen.png', animation: true, columns: 1, rows: 2 },
      ];
      const soundFiles = [
        { key: 'bgm', src: 'assets/sounds/bgm.mp3' },
        { key: 'jump', src: 'assets/sounds/jump.mp3' },
        { key: 'fire', src: 'assets/sounds/fire.mp3' },
        { key: 'level_complete', src: 'assets/sounds/level_complete.mp3' },
        { key: 'death', src: 'assets/sounds/death.mp3' },
        { key: 'finish_effect', src: 'assets/sounds/FinishEffect.mp3' },
        { key: 'eat', src: 'assets/sounds/eat.mp3' }
      ];
      const imagePromises = imageFiles.map(({ key, src, animation, columns, rows }) => {
        return new Promise((resolve) => {
          const img = new Image();
          img.src = src;
          img.onload = () => {
            if (animation) {
              const frameColumns = columns || 1;
              const frameRows = rows || 2;
              sprites.animations[key] = {
                image: img,
                frameCount: frameColumns * frameRows,
                frameWidth: img.width / frameColumns,
                frameHeight: img.height / frameRows,
                columns: frameColumns,
                rows: frameRows,
                scaledFrameWidth: (img.width / frameColumns) * scaleFactorX,
                scaledFrameHeight: (img.height / frameRows) * scaleFactorY
              };
            } else {
              sprites[key] = img;
            }
            resolve();
          };
          img.onerror = () => {
            console.warn(`Failed to load image: ${src}. Using fallback.`);
            if (animation) {
              sprites.animations[key] = null;
            } else {
              sprites[key] = null;
            }
            resolve();
          };
        });
      });
      const soundPromises = soundFiles.map(({ key, src }) => {
        return new Promise((resolve) => {
          const audio = new Audio(src);
          audio.onloadeddata = () => {
            sounds[key] = audio;
            resolve();
          };
          audio.onerror = () => {
            console.warn(`Failed to load sound: ${src}. Skipping.`);
            sounds[key] = null;
            resolve();
          };
        });
      });
      await Promise.all([...imagePromises, ...soundPromises]);
      if (sounds.bgm) {
        sounds.bgm.loop = true;
        sounds.bgm.volume = 0.3;
      }
      if (sounds.finish_effect) sounds.finish_effect.volume = 0.5;
      if (sounds.jump) sounds.jump.volume = 0.5;
      if (sounds.fire) sounds.fire.volume = 0;
      if (sounds.level_complete) sounds.level_complete.volume = 0.5;
      if (sounds.death) sounds.death.volume = 0.7;
      if (sounds.eat) sounds.eat.volume = 0.5;
      gameState.assetsLoaded = true;
      console.log('Assets loaded successfully.');
    }
    function playSound(key) {
      if (!gameState.audioContext) {
        gameState.audioContext = new (window.AudioContext || window.webkitAudioContext)();
      }
      if (gameState.audioContext.state === 'suspended') {
        gameState.audioContext.resume();
      }
      if (sounds[key]) {
        sounds[key].currentTime = 0;
        sounds[key].play().catch((e) => console.warn(`Failed to play sound ${key}: ${e}`));
      }
    }
    const levels = {
      1: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 240, y: 810, w: 360, h: 36 },
          { x: 840, y: 614, w: 480, h: 36 }
        ],
        walls: [
          { x: 1320, y: 790, w: 640, h: 200 },
          { x: 1320, y: 0, w: 640, h: 650 }
        ],
        pressurePlates: [
          { x: 1027.2, y: 602, w: 120, h: 12.6, color: 'red', targetWall: 0 }
        ],
        coloredWalls: [
          { x: 1440, y: 650, w: 48, h: 144, color: 'red', active: false }
        ],
        foods: [
          { x: 396, y: 774, w: 48, h: 36, eaten: false }
        ]
      },
      2: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 720, y: 810, w: 48, h: 1.8 },
          { x: 1200, y: 666, w: 48, h: 1.8 }
        ],
        walls: [
          { x: 0, y: 0, w: 240, h: 846 },
          { x: 1680, y: 0, w: 240, h: 846 },
          { x: 720, y: 790, w: 80, h: 200 },
          { x: 1200, y: 450, w: 80, h: 550 }
        ],
        foods: [
          { x: 840, y: 774, w: 48, h: 36, eaten: false }
        ]
      },
      3: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 480, y: 720, w: 360, h: 36 },
          { x: 1656, y: 810, w: 96, h: 36 },
          { x: 1460, y: 540, w: 96, h: 36 },
          { x: 1656, y: 270, w: 96, h: 36 }
        ],
        walls: [
          { x: 0, y: 0, w: 1920, h: 150 },
          { x: 0, y: 150, w: 160, h: 750 },
          { x: 960, y: 250, w: 80, h: 750 },
          { x: 1752, y: 270, w: 160, h: 730 },
          { x: 1300, y: 150, w: 160, h: 650 }
        ],
        pressurePlates: [
          { x: 1100, y: 977.4, w: 120, h: 12.6, color: 'brown', targetWall: 0 }
        ],
        coloredWalls: [
          { x: 1368, y: 800, w: 48, h: 190, color: 'brown', active: false }
        ],
        foods: [
          { x: 528, y: 684, w: 48, h: 36, eaten: false },
          { x: 720, y: 684, w: 48, h: 36, eaten: false }
        ]
      },
      4: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 240, y: 720, w: 480, h: 36 },
          { x: 480, y: 550, w: 720, h: 36 }
        ],
        walls: [
          { x: 1440, y: 650, w: 80, h: 350 },
          { x: 0, y: 550, w: 240, h: 350 },
          { x: 0, y: 250, w: 240, h: 200 },
          { x: 0, y: 0, w: 240, h: 300 },
          { x: 240, y: 0, w: 1760, h: 300 },
          { x: 480, y: 300, w: 800, h: 50 },
          { x: 480, y: 350, w: 240, h: 100 },
          { x: 1040, y: 350, w: 240, h: 250 },
          { x: 480, y: 550, w: 800, h: 50 },
          { x: 1680, y: 300, w: 800, h: 550 }
        ],
        pressurePlates: [
          { x: 60, y: 537.4, w: 120, h: 12.6, color: 'yellow', targetWall: 0 },
          { x: 1272, y: 977.4, w: 120, h: 12.6, color: 'blue', targetWall: 1 }
        ],
        coloredWalls: [
          { x: 576, y: 450, w: 48, h: 100, color: 'yellow', active: false },
          { x: 1456, y: 300, w: 48, h: 350, color: 'blue', active: false }
        ],
        foods: [
          { x: 288, y: 684, w: 48, h: 36, eaten: false },
          { x: 840, y: 504, w: 48, h: 36, eaten: false }
        ]
      },
      5: {
        platforms: [
          { x: 0, y: 990, w: 840, h: 90 },
          { x: 1320, y: 990, w: 720, h: 90 },
          { x: 360, y: 810, w: 120, h: 18 },
          { x: 240, y: 504, w: 120, h: 18 },
          { x: 360, y: 198, w: 120, h: 18 },
          { x: 560, y: 198, w: 120, h: 18 },
          { x: 795.2, y: 286.2, w: 169.6, h: 3.6 }
        ],
        walls: [
          { x: 0, y: 0, w: 240, h: 850 },
          { x: 800, y: 0, w: 1120, h: 150 },
          { x: 480, y: 90, w: 80, h: 900 },
          { x: 800, y: 288, w: 160, h: 600 },
          { x: 960, y: 400, w: 720, h: 488 },
          { x: 1680, y: 288, w: 240, h: 600 },
          { x: 1840, y: 150, w: 80, h: 150 }
        ],
        hazardWalls: [
          { x: 840, y: 1026, w: 485, h: 50 },
          { x: 960, y: 310, w: 720, h: 100 }
        ],
        pressurePlates: [
          { x: 1720, y: 275.4, w: 120, h: 12.6, color: 'blue', targetWall: 0 }
        ],
        coloredWalls: [
          { x: 840, y: 990, w: 480, h: 36, color: 'blue', active: false }
        ],
        foods: [
          { x: 1756, y: 180, w: 48, h: 36, eaten: false }
        ],
        movingPlatforms: [
          { x1: 960, y1: 288, x2: 1520, y2: 288, w: 240, h: 18, speed: 1.2, currentX: 960, currentY: 288, direction: 1 }
        ]
      }
    };
    const playButton = {
      x: 860,
      y: 490,
      w: 200,
      h: 100
    };
    playButtonElement.addEventListener('click', () => {
      if (gameState.screen === 'menu' || gameState.screen === 'win') {
        gameState.screen = 'game';
        currentLevel = 1;
        loadLevel(currentLevel);
        playSound('bgm');
      }
    });
    canvas.addEventListener('click', (e) => {
      if (gameState.screen === 'menu') {
        const rect = canvas.getBoundingClientRect();
        const clickX = (e.clientX - rect.left) / scaleX;
        const clickY = (e.clientY - rect.top) / scaleY;
        if (
          clickX >= playButton.x &&
          clickX <= playButton.x + playButton.w &&
          clickY >= playButton.y &&
          clickY <= playButton.y + playButton.h
        ) {
          gameState.screen = 'game';
          loadLevel(currentLevel);
          playSound('bgm');
        }
      } else {
        fireGoo(e);
      }
    });
    function loadLevel(level) {
      const l = levels[level];
      platforms = l.platforms ? l.platforms.map(p => ({ ...p })) : [];
      movingPlatforms = l.movingPlatforms ? l.movingPlatforms.map(p => ({ ...p, currentX: p.x1, currentY: p.y1, direction: 1 })) : [];
      walls = l.walls ? l.walls.map(w => ({ ...w })) : [];
      hazardWalls = l.hazardWalls ? l.hazardWalls.map(h => ({ ...h })) : [];
      pressurePlates = l.pressurePlates ? l.pressurePlates.map(p => ({ ...p })) : [];
      coloredWalls = l.coloredWalls ? l.coloredWalls.map(w => ({ ...w })) : [];
      foods = l.foods ? l.foods.map(f => ({ ...f, eaten: false, vx: 0, vy: 0, grounded: false })) : [];
      goos = [];
      if (level === 5) sounds.fire.volume = 0.8;
      player.x = 240;
      player.y = 900;
      player.vx = 0;
      player.vy = 0;
      player.color = 'gray';
      player.sizeMultiplier = 1;
      player.w = 72;
      player.h = 54;
      player.foodCount = 0;
      player.animation = {
        state: 'idle',
        fireEndTime: 0,
        frame: 0,
        frameTimer: 0,
        frameDurations: {
          idle: 1200,
          right: 800,
          left: 800,
          jump: 600,
          fire: 400
        }
      };
      keys = {};
      gameState.levelText = `Level ${level}`;
      gameState.controlsAlpha = 1;
      gameState.controlsTimer = 0;
      let pressureStates = pressurePlates.map(() => false);
      for (let i = 0; i < pressurePlates.length; i++) {
        let plate = pressurePlates[i];
        for (let food of foods) {
          if (
            !food.eaten &&
            food.x + food.w > plate.x &&
            food.x < plate.x + plate.w &&
            food.y + food.h > plate.y &&
            food.y < plate.y + plate.h
          ) {
            pressureStates[i] = true;
          }
        }
      }
      for (let i = 0; i < pressurePlates.length; i++) {
        coloredWalls[pressurePlates[i].targetWall].active = pressureStates[i];
      }
      console.log(`Seviye ${level} yüklendi, yem sayısı: ${foods.length}, tehlike duvarları: ${hazardWalls.length || 0}, hareketli platformlar: ${movingPlatforms.length || 0}`);
    }
    let keys = {};
    window.addEventListener('keydown', (e) => {
      if (gameState.screen !== 'game') return;
      keys[e.key] = true;
      if (!gameState.audioContext && gameState.assetsLoaded) {
        playSound('bgm');
      }
      if (e.key === 'r' || e.key === 'R') {
        loadLevel(currentLevel);
      }
      if (e.key === 'p' || e.key === 'P') {
        if (currentLevel > 1) {
          currentLevel--;
          loadLevel(currentLevel);
        }
      }
      if (e.key === 'm' || e.key === 'M') {
        if (levels[currentLevel + 1]) {
          currentLevel++;
          loadLevel(currentLevel);
        }
      }
      if ((e.key === ' ' || e.key === 'ArrowUp') && player.grounded) {
        player.vy = player.jumpForce;
        player.grounded = false;
        playSound('jump');
        console.log('Zıplama: vy =', player.vy);
      }
    });
    window.addEventListener('keyup', (e) => {
      if (gameState.screen !== 'game') return;
      keys[e.key] = false;
    });
    function fireGoo(e) {
      if (gameState.screen !== 'game') return;
      if (player.color !== 'green' || player.foodCount <= 0) {
        console.log('Ateş edemezsiniz: Yeşil değil veya küp yok.');
        return;
      }
      const rect = canvas.getBoundingClientRect();
      const mouseX = (e.clientX - rect.left) / scaleX;
      const mouseY = (e.clientY - rect.top) / scaleY;
      const dx = mouseX - (player.x + player.w / 2);
      const dy = mouseY - (player.y + player.h / 2);
      const mag = Math.sqrt(dx * dx + dy * dy);
      let willCollideWithWall = false;
      for (let wall of walls) {
        if (
          (dx > 0 && player.x + player.w > wall.x && player.x < wall.x + wall.w) ||
          (dx < 0 && player.x < wall.x + wall.w && player.x + player.w > wall.x)
        ) {
          if (
            (dy > 0 && player.y + player.h > wall.y && player.y < wall.y + wall.h) ||
            (dy < 0 && player.y < wall.y + wall.h && player.y + player.h > wall.y)
          ) {
            willCollideWithWall = true;
            break;
          }
        }
      }
      let adjustedDx = dx;
      let adjustedDy = dy;
      if (willCollideWithWall) {
        const angleAdjustment = (Math.random() * 0.5 - 0.25) * Math.PI;
        const newAngle = Math.atan2(dy, dx) + angleAdjustment;
        const newMag = mag;
        adjustedDx = Math.cos(newAngle) * newMag;
        adjustedDy = Math.sin(newAngle) * newMag;
      }
      const vx = (adjustedDx / mag) * gooSpeed;
      const vy = (adjustedDy / mag) * gooSpeed;
      let offset = Math.max(player.w, player.h) / 2 + foods[0].w / 2;
      let startX = player.x + player.w / 2 + (adjustedDx / mag) * offset;
      let startY = player.y + player.h / 2 + (adjustedDy / mag) * offset;
      for (let wall of walls) {
        if (
          startX + foods[0].w > wall.x &&
          startX < wall.x + wall.w &&
          startY + foods[0].h > wall.y &&
          startY < wall.y + wall.h
        ) {
          if (adjustedDx > 0) {
            startX = wall.x - foods[0].w - 1;
          } else {
            startX = wall.x + wall.w + 1;
          }
        }
      }
      let immuneTime = gooImmuneTime;
      let recoilY = player.recoil;
      if (player.grounded && dy > 0) {
        startY = player.y - foods[0].h;
        immuneTime = 300;
        recoilY *= 0.5;
      }
      goos.push({
        x: startX,
        y: startY,
        vx: vx,
        vy: vy,
        w: foods[0].w,
        h: foods[0].h,
        grounded: false,
        spawnTime: performance.now(),
        immuneTime: immuneTime
      });
      player.vx -= (adjustedDx / mag) * player.recoil * 1.5;
      player.vy -= (adjustedDy / mag) * recoilY;
      player.foodCount--;
      if (player.foodCount <= 0) {
        player.color = 'gray';
        player.sizeMultiplier = 1;
        player.w = 72;
        player.h = 54;
      }
      player.animation.state = 'fire';
      player.animation.fireEndTime = performance.now() + 400;
      player.animation.frame = 0;
      player.animation.frameTimer = 0;
      playSound('fire');
      console.log(`Ateş edildi: vx = ${vx}, vy = ${vy}, startY = ${startY}`);
    }
    function update() {
      if (gameState.screen !== 'game' || !gameState.assetsLoaded) return;
      const moveSpeed = player.grounded ? player.speed : player.airSpeed;
      if (keys['ArrowLeft'] || keys['a'] || keys['A']) player.vx = -moveSpeed;
      else if (keys['ArrowRight'] || keys['d'] || keys['D']) player.vx = moveSpeed;
      else player.vx *= 0.9;
      player.vy += gravity;
      let newX = player.x + player.vx;
      let newY = player.y + player.vy;
      if (gameState.controlsAlpha > 0) {
        gameState.controlsTimer += 1 / 60;
        if (gameState.controlsTimer >= 5) {
          gameState.controlsAlpha = Math.max(0, gameState.controlsAlpha - 1 / 60);
        }
      }
      const currentTime = performance.now();
      let newAnimationState = player.animation.state;
      if (player.animation.fireEndTime > currentTime) {
        newAnimationState = 'fire';
      } else if (!player.grounded) {
        newAnimationState = 'jump';
      } else if (keys['ArrowRight'] || keys['d'] || keys['D']) {
        newAnimationState = 'right';
      } else if (keys['ArrowLeft'] || keys['a'] || keys['A']) {
        newAnimationState = 'left';
      } else {
        newAnimationState = 'idle';
      }
      if (newAnimationState !== player.animation.state) {
        player.animation.state = newAnimationState;
        player.animation.frame = 0;
        player.animation.frameTimer = 0;
      }
      player.animation.frameTimer += 1000 / 60;
      const currentFrameDuration = player.animation.frameDurations[player.animation.state] || 800;
      if (player.animation.frameTimer >= currentFrameDuration) {
        const anim = sprites.animations[`player_${player.animation.state}${player.color === 'green' ? '_green' : ''}`];
        player.animation.frame = (player.animation.frame + 1) % (anim ? anim.frameCount : 2);
        player.animation.frameTimer = 0;
      }
      for (let platform of movingPlatforms) {
        const targetX = platform.direction === 1 ? platform.x2 : platform.x1;
        const targetY = platform.direction === 1 ? platform.y2 : platform.y1;
        const dx = targetX - platform.currentX;
        const dy = targetY - platform.currentY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        if (distance <= platform.speed) {
          platform.currentX = targetX;
          platform.currentY = targetY;
          platform.direction *= -1;
        } else {
          const moveX = (dx / distance) * platform.speed;
          const moveY = (dy / distance) * platform.speed;
          platform.currentX += moveX;
          platform.currentY += moveY;
        }
      }
      let pressureStates = pressurePlates.map(() => false);
      for (let food of foods) {
        if (!food.eaten) {
          food.vy += gravity;
          let newFoodX = food.x + food.vx;
          let newFoodY = food.y + food.vy;
          food.grounded = false;
          for (let platform of platforms) {
            if (
              newFoodX + food.w > platform.x &&
              newFoodX < platform.x + platform.w &&
              newFoodY + food.h > platform.y &&
              food.y + food.h <= platform.y &&
              food.vy >= 0
            ) {
              newFoodY = platform.y - food.h;
              food.vy = 0;
              food.vx *= friction;
              food.grounded = true;
            }
          }
          for (let platform of movingPlatforms) {
            if (
              newFoodX + food.w > platform.currentX &&
              newFoodX < platform.currentX + platform.w &&
              newFoodY + food.h > platform.currentY &&
              food.y + food.h <= platform.currentY &&
              food.vy >= 0
            ) {
              newFoodY = platform.currentY - food.h;
              food.vy = 0;
              food.vx *= friction;
              food.grounded = true;
            }
          }
          for (let wall of walls) {
            if (
              newFoodX + food.w > wall.x &&
              newFoodX < wall.x + wall.w &&
              newFoodY + food.h > wall.y &&
              newFoodY < wall.y + wall.h
            ) {
              if (food.y + food.h <= wall.y && newFoodY + food.h > wall.y && food.vy >= 0) {
                newFoodY = wall.y - food.h;
                food.vy = 0;
                food.vx *= friction;
                food.grounded = true;
              } else if (food.y >= wall.y + wall.h && newFoodY < wall.y + wall.h && food.vy < 0) {
                newFoodY = wall.y + wall.h;
                food.vy = 0;
              } else if (food.x + food.w <= wall.x && newFoodX + food.w > wall.x) {
                newFoodX = wall.x - food.w;
                food.vx = 0;
              } else if (food.x >= wall.x + wall.w && newFoodX < wall.x + wall.w) {
                newFoodX = wall.x + wall.w;
                food.vx = 0;
              }
            }
          }
          for (let wall of coloredWalls) {
            if (
              !wall.active &&
              newFoodX + food.w > wall.x &&
              newFoodX < wall.x + wall.w &&
              newFoodY + food.h > wall.y &&
              newFoodY < wall.y + wall.h
            ) {
              if (food.y + food.h <= wall.y && newFoodY + food.h > wall.y && food.vy >= 0) {
                newFoodY = wall.y - food.h;
                food.vy = 0;
                food.vx *= friction;
                food.grounded = true;
              } else if (food.y >= wall.y + wall.h && newFoodY < wall.y + wall.h && food.vy < 0) {
                newFoodY = wall.y + wall.h;
                food.vy = 0;
              } else if (food.x + food.w <= wall.x && newFoodX + food.w > wall.x) {
                newFoodX = wall.x - food.w;
                food.vx = 0;
              } else if (food.x >= wall.x + wall.w && newFoodX < wall.x + wall.w) {
                newFoodX = wall.x + wall.w;
                food.vx = 0;
              }
            }
          }
          for (let i = 0; i < pressurePlates.length; i++) {
            let plate = pressurePlates[i];
            if (
              newFoodX + food.w > plate.x &&
              newFoodX < plate.x + plate.w &&
              newFoodY + food.h > plate.y &&
              newFoodY < plate.y + plate.h
            ) {
              pressureStates[i] = true;
              newFoodY = plate.y - food.h;
              food.vy = 0;
              food.vx = 0;
              food.grounded = true;
            }
          }
          if (newFoodX < 0) newFoodX = 0;
          if (newFoodX + food.w > 1920) newFoodX = 1920 - food.w;
          if (newFoodY + food.h > 1080) {
            newFoodY = 1080 - food.h;
            food.vy = 0;
            food.vx *= friction;
            food.grounded = true;
          }
          food.x = newFoodX;
          food.y = newFoodY;
        }
      }
      player.grounded = false;
      let playerOnMovingPlatform = null;
      for (let platform of platforms) {
        if (newX + player.w > platform.x && newX < platform.x + platform.w) {
          if (
            newY + player.h > platform.y &&
            player.y + player.h <= platform.y + epsilon &&
            player.vy >= 0
          ) {
            newY = platform.y - player.h;
            player.vy = 0;
            player.grounded = true;
          } else if (
            newY < platform.y + platform.h &&
            player.y >= platform.y + platform.h &&
            player.vy < 0
          ) {
            newY = platform.y + platform.h;
            player.vy = 0;
          }
        }
      }
      for (let platform of movingPlatforms) {
        if (newX + player.w > platform.currentX && newX < platform.currentX + platform.w) {
          if (
            newY + player.h > platform.currentY &&
            player.y + player.h <= platform.currentY + epsilon &&
            player.vy >= 0
          ) {
            newY = platform.currentY - player.h;
            player.vy = 0;
            player.grounded = true;
            playerOnMovingPlatform = platform;
          } else if (
            newY < platform.currentY + platform.h &&
            player.y >= platform.currentY + platform.h &&
            player.vy < 0
          ) {
            newY = platform.currentY + platform.h;
            player.vy = 0;
          }
        }
      }
      for (let wall of walls) {
        if (
          newX + player.w > wall.x &&
          newX < wall.x + wall.w &&
          newY + player.h > wall.y &&
          newY < wall.y + wall.h
        ) {
          if (
            player.y + player.h <= wall.y + epsilon &&
            newY + player.h > wall.y &&
            player.vy >= 0
          ) {
            newY = wall.y - player.h;
            player.vy = 0;
            player.grounded = true;
          } else if (
            player.y >= wall.y + wall.h &&
            newY < wall.y + wall.h &&
            player.vy < 0
          ) {
            newY = wall.y + wall.h;
            player.vy = 0;
          } else if (
            player.x + player.w <= wall.x &&
            newX + player.w > wall.x
          ) {
            newX = wall.x - player.w;
            player.vx = 0;
          } else if (
            player.x >= wall.x + wall.w &&
            newX < wall.x + wall.w
          ) {
            newX = wall.x + wall.w;
            player.vx = 0;
          } else {
            let overlapX = Math.min(
              newX + player.w - wall.x,
              wall.x + wall.w - newX
            );
            let overlapY = Math.min(
              newY + player.h - wall.y,
              wall.y + wall.h - newY
            );
            if (overlapX < overlapY) {
              if (newX < wall.x) newX = wall.x - player.w;
              else newX = wall.x + wall.w;
              player.vx = 0;
            } else {
              if (
                Math.abs(newY + player.h - wall.y) <
                Math.abs(newY - wall.y - wall.h)
              ) {
                newY = wall.y - player.h;
                player.vy = 0;
                player.grounded = true;
              } else {
                newY = wall.y + wall.h;
                player.vy = 0;
              }
            }
          }
        }
      }
      for (let wall of coloredWalls) {
        if (
          !wall.active &&
          newX + player.w > wall.x &&
          newX < wall.x + wall.w &&
          newY + player.h > wall.y &&
          newY < wall.y + wall.h
        ) {
          if (player.x + player.w <= wall.x) newX = wall.x - player.w;
          else if (player.x >= wall.x + wall.w) newX = wall.x + wall.w;
          else if (player.y + player.h <= wall.y) newY = wall.y - player.h;
          else if (player.y >= wall.y + wall.h) newY = wall.y + wall.h;
          player.vx = 0;
          player.vy = 0;
        }
      }
      for (let hazard of hazardWalls) {
        if (
          newX + player.w > hazard.x &&
          newX < hazard.x + hazard.w &&
          newY + player.h > hazard.y &&
          newY < hazard.y + hazard.h
        ) {
          playSound('death');
          console.log(`Tehlike duvarına çarpıldı, seviye ${currentLevel} yeniden başlatılıyor.`);
          loadLevel(currentLevel);
          return;
        }
      }
      if (newX < 0) newX = 0;
      if (newX + player.w > 1920) {
        currentLevel++;
        if (levels[currentLevel]) {
          playSound('level_complete');
          loadLevel(currentLevel);
        } else {
          gameState.screen = 'win';
          currentLevel = 1;
          playSound('finish_effect');
        }
        return;
      }
      if (newY > 1080) {
        newY = 1080 - player.h;
        player.vy = 0;
        player.grounded = true;
      }
      player.x = newX;
      player.y = newY;
      if (playerOnMovingPlatform) {
        const targetX =
          playerOnMovingPlatform.direction === 1
            ? playerOnMovingPlatform.x2
            : playerOnMovingPlatform.x1;
        const targetY =
          playerOnMovingPlatform.direction === 1
            ? playerOnMovingPlatform.y2
            : playerOnMovingPlatform.y1;
        const dx = targetX - playerOnMovingPlatform.currentX;
        const dy = targetY - playerOnMovingPlatform.currentY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        if (distance > playerOnMovingPlatform.speed) {
          const moveX = (dx / distance) * playerOnMovingPlatform.speed;
          const moveY = (dy / distance) * playerOnMovingPlatform.speed;
          player.x += moveX;
          player.y += moveY;
        }
      }
      for (let i = 0; i < pressurePlates.length; i++) {
        let plate = pressurePlates[i];
        if (
          player.x + player.w > plate.x &&
          player.x < plate.x + plate.w &&
          player.y + player.h > plate.y &&
          player.y < plate.y + plate.h
        ) {
          pressureStates[i] = true;
        }
      }
      for (let i = 0; i < foods.length; i++) {
        let food = foods[i];
        if (
          !food.eaten &&
          player.x + player.w > food.x &&
          player.x < food.x + food.w &&
          player.y + player.h > food.y &&
          player.y < food.y + food.h
        ) {
          food.eaten = true;
          playSound('eat');
          player.color = 'green';
          let oldHeight = player.h;
          player.sizeMultiplier = 1.2;
          player.w = 72 * player.sizeMultiplier;
          player.h = 54 * player.sizeMultiplier;
          if (player.grounded) {
            player.y -= player.h - oldHeight;
          }
          player.foodCount++;
        }
      }
      for (let i = goos.length - 1; i >= 0; i--) {
        let goo = goos[i];
        goo.vy += gooGravity;
        let newGooX = goo.x + goo.vx;
        let newGooY = goo.y + goo.vy;
        goo.grounded = false;
        for (let platform of platforms) {
          if (
            newGooX + goo.w > platform.x &&
            newGooX < platform.x + platform.w &&
            newGooY + goo.h > platform.y &&
            goo.y < platform.y + platform.h &&
            goo.vy >= 0
          ) {
            newGooY = platform.y - goo.h;
            goo.vy = 0;
            goo.vx *= friction;
            goo.grounded = true;
          }
        }
        for (let platform of movingPlatforms) {
          if (
            newGooX + goo.w > platform.currentX &&
            newGooX < platform.currentX + platform.w &&
            newGooY + goo.h > platform.currentY &&
            goo.y < platform.y + platform.h &&
            goo.vy >= 0
          ) {
            newGooY = platform.currentY - goo.h;
            goo.vy = 0;
            goo.vx *= friction;
            goo.grounded = true;
          }
        }
        for (let wall of walls) {
          if (
            newGooX + goo.w > wall.x &&
            newGooX < wall.x + wall.w &&
            newGooY + goo.h > wall.y &&
            newGooY < wall.y + wall.h
          ) {
            newGooX = goo.x;
            newGooY = goo.y;
            goo.vx = 0;
            goo.vy = 0;
            goo.grounded = true;
          }
        }
        for (let wall of coloredWalls) {
          if (
            !wall.active &&
            newGooX + goo.w > wall.x &&
            newGooX < wall.x + wall.w &&
            newGooY + goo.h > wall.y &&
            newGooY < wall.y + wall.h
          ) {
            newGooX = goo.x;
            newGooY = goo.y;
            goo.vx = 0;
            goo.vy = 0;
            goo.grounded = true;
          }
        }
        for (let j = 0; j < pressurePlates.length; j++) {
          let plate = pressurePlates[j];
          if (
            newGooX + goo.w > plate.x &&
            newGooX < plate.x + plate.w &&
            newGooY + goo.h > plate.y &&
            goo.y < plate.y + plate.h
          ) {
            pressureStates[j] = true;
          }
        }
        if (newGooX < 0) newGooX = 0;
        if (newGooX + goo.w > 1920) newGooX = 1920 - goo.w;
        if (newGooY > 1080) newGooY = 1080 - goo.h;
        goo.x = newGooX;
        goo.y = newGooY;
        if (
          goo.grounded &&
          player.x + player.w > goo.x &&
          player.x < goo.x + goo.w &&
          player.y + player.h > goo.y &&
          player.y < goo.y + goo.h &&
          currentTime - goo.spawnTime > (goo.immuneTime || gooImmuneTime)
        ) {
          goos.splice(i, 1);
          player.foodCount++;
          player.color = 'green';
          let oldHeight = player.h;
          player.sizeMultiplier = 1.2;
          player.w = 72 * player.sizeMultiplier;
          player.h = 54 * player.sizeMultiplier;
          if (player.grounded) {
            player.y -= player.h - oldHeight;
          }
        }
      }
      for (let i = 0; i < pressurePlates.length; i++) {
        coloredWalls[pressurePlates[i].targetWall].active = pressureStates[i];
      }
    }
    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.save();
      ctx.scale(scaleX, scaleY);
      if (gameState.screen === 'menu' || gameState.screen === 'win') {
        playButtonElement.classList.add('visible');
      } else {
        playButtonElement.classList.remove('visible');
      }
      if (gameState.screen === 'menu') {
        if (sprites.menu_background) {
          ctx.drawImage(sprites.menu_background, 0, 0, 1920, 1080);
        } else {
          ctx.fillStyle = '#333';
          ctx.fillRect(0, 0, 1920, 1080);
        }
      } else if (gameState.screen === 'win') {
        if (sprites.menu_background) {
          ctx.drawImage(sprites.menu_background, 0, 0, 1920, 1080);
        } else {
          ctx.fillStyle = '#333';
          ctx.fillRect(0, 0, 1920, 1080);
        }
        ctx.font = '80px Arial';
        ctx.fillStyle = 'white';
        ctx.textAlign = 'center';
        ctx.shadowColor = 'black';
        ctx.shadowBlur = 10;
        ctx.fillText('Tebrikler! Tüm seviyeleri tamamladınız.', 960, 400);
        ctx.shadowBlur = 0;
      } else {
        const backgroundKey = `background${currentLevel}`;
        if (sprites[backgroundKey]) {
          ctx.drawImage(sprites[backgroundKey], 0, 0, 1920, 1080);
        } else {
          ctx.fillStyle = '#87CEEB';
          ctx.fillRect(0, 0, 1920, 1080);
        }
        for (let platform of platforms) {
          if (sprites.platform) {
            ctx.drawImage(
              sprites.platform,
              0,
              0,
              sprites.platform.width,
              sprites.platform.height,
              platform.x,
              platform.y,
              platform.w,
              platform.h
            );
          } else {
            ctx.fillStyle = 'gray';
            ctx.fillRect(platform.x, platform.y, platform.w, platform.h);
          }
        }
        for (let platform of movingPlatforms) {
          if (sprites.moving_platform) {
            ctx.drawImage(
              sprites.moving_platform,
              0,
              0,
              sprites.moving_platform.width,
              sprites.moving_platform.height,
              platform.currentX,
              platform.currentY,
              platform.w,
              platform.h
            );
          } else {
            ctx.fillStyle = '#800080';
            ctx.fillRect(platform.currentX, platform.currentY, platform.w, platform.h);
          }
        }
        for (let wall of walls) {
          if (sprites.brick) {
            const numBricksX = Math.ceil(wall.w / brickWidth);
            const numBricksY = Math.ceil(wall.h / brickHeight);
            for (let i = 0; i < numBricksX; i++) {
              for (let j = 0; j < numBricksY; j++) {
                const brickX = wall.x + i * brickWidth;
                const brickY = wall.y + j * brickHeight;
                const drawWidth = Math.min(brickWidth, wall.x + wall.w - brickX);
                const drawHeight = Math.min(brickHeight, wall.y + wall.h - brickY);
                if (drawWidth > 0 && drawHeight > 0) {
                  ctx.drawImage(
                    sprites.brick,
                    0,
                    0,
                    sprites.brick.width,
                    sprites.brick.height,
                    brickX,
                    brickY,
                    drawWidth,
                    drawHeight
                  );
                }
              }
            }
          } else {
            ctx.fillStyle = 'brown';
            ctx.fillRect(wall.x, wall.y, wall.w, wall.h);
          }
        }
        for (let hazard of hazardWalls) {
          if (sprites.hazard_wall) {
            const numTilesX = Math.ceil(hazard.w / brickWidth);
            const numTilesY = Math.ceil(hazard.h / brickHeight);
            for (let i = 0; i < numTilesX; i++) {
              for (let j = 0; j < numTilesY; j++) {
                const tileX = hazard.x + i * brickWidth;
                const tileY = hazard.y + j * brickHeight;
                const drawWidth = Math.min(brickWidth, hazard.x + hazard.w - tileX);
                const drawHeight = Math.min(brickHeight, hazard.y + hazard.h - tileY);
                if (drawWidth > 0 && drawHeight > 0) {
                  ctx.drawImage(
                    sprites.hazard_wall,
                    0,
                    0,
                    sprites.hazard_wall.width,
                    sprites.hazard_wall.height,
                    tileX,
                    tileY,
                    drawWidth,
                    drawHeight
                  );
                }
              }
            }
          } else {
            ctx.fillStyle = '#FF0000';
            ctx.shadowColor = '#FF0000';
            ctx.shadowBlur = 10;
            ctx.fillRect(hazard.x, hazard.y, hazard.w, hazard.h);
            ctx.shadowBlur = 0;
            ctx.shadowColor = 'transparent';
          }
        }
        for (let wall of coloredWalls) {
          const spriteKey = wall.active ? 'colored_wall_active' : `colored_wall_${wall.color}`;
          if (sprites[spriteKey]) {
            const numCellsX = Math.ceil(wall.w / brickWidth);
            const numCellsY = Math.ceil(wall.h / brickHeight);
            for (let i = 0; i < numCellsX; i++) {
              for (let j = 0; j < numCellsY; j++) {
                const cellX = wall.x + i * brickWidth;
                const cellY = wall.y + j * brickHeight;
                const drawWidth = Math.min(brickWidth, wall.x + wall.w - cellX);
                const drawHeight = Math.min(brickHeight, wall.y + wall.h - cellY);
                if (drawWidth > 0 && drawHeight > 0) {
                  ctx.drawImage(
                    sprites[spriteKey],
                    0,
                    0,
                    sprites[spriteKey].width,
                    sprites[spriteKey].height,
                    cellX,
                    cellY,
                    drawWidth,
                    drawHeight
                  );
                }
              }
            }
          } else {
            ctx.fillStyle = wall.active
              ? 'rgba(0, 255, 0, 0.5)'
              : wall.color === 'yellow'
              ? '#FFFF00'
              : wall.color === 'brown'
              ? '#8B4513'
              : wall.color === 'blue'
              ? '#0000FF'
              : 'red';
            ctx.fillRect(wall.x, wall.y, wall.w, wall.h);
          }
        }
        for (let plate of pressurePlates) {
          const spriteKey = `pressure_plate_${plate.color}`;
          if (sprites[spriteKey]) {
            ctx.drawImage(
              sprites[spriteKey],
              0,
              0,
              sprites[spriteKey].width,
              sprites[spriteKey].height,
              plate.x,
              plate.y,
              plate.w,
              plate.h
            );
          } else {
            ctx.fillStyle =
              plate.color === 'yellow'
              ? '#FFFF00'
              : plate.color === 'brown'
              ? '#8B4513'
              : plate.color === 'blue'
              ? '#0000FF'
              : 'red';
            ctx.fillRect(plate.x, plate.y, plate.w, plate.h);
          }
        }
        for (let food of foods) {
          if (!food.eaten) {
            if (sprites.food) {
              ctx.drawImage(
                sprites.food,
                0,
                0,
                sprites.food.width,
                sprites.food.height,
                food.x,
                food.y,
                food.w,
                food.h
              );
            } else {
              ctx.fillStyle = 'green';
              ctx.fillRect(food.x, food.y, food.w, food.h);
            }
          }
        }
        const animKey = `player_${player.animation.state}${player.color === 'green' ? '_green' : ''}`;
        const anim = sprites.animations[animKey];
        const playerSprite = player.color === 'green' ? sprites.player_green : sprites.player;
        if (player.animation.state === 'idle' && playerSprite) {
          const offsetY = Math.sin(performance.now() / 500) * 4.8;
          ctx.drawImage(
            playerSprite,
            0,
            0,
            playerSprite.width,
            playerSprite.height,
            player.x,
            player.y + offsetY,
            player.w,
            player.h
          );
        } else if (anim) {
          const frameIndex = player.animation.frame;
          const frameX = (frameIndex % anim.columns) * anim.frameWidth;
          const frameY = Math.floor(frameIndex / anim.columns) * anim.frameHeight;
          ctx.drawImage(
            anim.image,
            frameX,
            frameY,
            anim.frameWidth,
            anim.frameHeight,
            player.x,
            player.y,
            player.w,
            player.h
          );
        } else if (playerSprite) {
          ctx.drawImage(
            playerSprite,
            0,
            0,
            playerSprite.width,
            playerSprite.height,
            player.x,
            player.y,
            player.w,
            player.h
          );
        } else {
          ctx.fillStyle = player.color;
          ctx.fillRect(player.x, player.y, player.w, player.h);
        }
        for (let goo of goos) {
          if (sprites.goo) {
            ctx.drawImage(
              sprites.goo,
              0,
              0,
              sprites.goo.width,
              sprites.goo.height,
              goo.x,
              goo.y,
              goo.w,
              goo.h
            );
          } else {
            ctx.fillStyle = 'green';
            ctx.fillRect(goo.x, goo.y, goo.w, goo.h);
          }
        }
        ctx.font = '57.6px Arial';
        ctx.fillStyle = 'white';
        ctx.textAlign = 'center';
        ctx.shadowColor = 'black';
        ctx.shadowBlur = 9.6;
        ctx.fillText(gameState.levelText, 960, 72);
        if (gameState.controlsAlpha > 0) {
          ctx.globalAlpha = gameState.controlsAlpha;
          ctx.font = '38.4px Arial';
          ctx.fillText(gameState.controlsText, 960, 720);
          ctx.globalAlpha = 1;
        }
        ctx.shadowBlur = 0;
        ctx.textAlign = 'left';
      }
      ctx.restore();
    }
    function gameLoop() {
      if (!gameState.assetsLoaded) return;
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }
    loadAssets().then(() => {
      gameLoop();
    });
  </script>
</body>
</html>


-İşte oyunun ilk bölümünde oyun mekaniklerini anlatan bir yazı eklediğim kodunuz:

html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Getting GOOpy</title>
  <style>
    canvas { border: 1px solid black; position: absolute; top: 0; left: 0; }
    body { margin: 0; background: #f0f0f0; overflow: hidden; }
    button {
      background: #fbca1f;
      font-family: inherit;
      padding: 0.6em 1.3em;
      font-weight: 900;
      font-size: 18px;
      border: 3px solid black;
      border-radius: 0.4em;
      box-shadow: 0.1em 0.1em;
      cursor: pointer;
      position: absolute;
      display: none; /* Varsayılan olarak gizli */
    }
    button:hover {
      transform: translate(-0.05em, -0.05em);
      box-shadow: 0.15em 0.15em;
    }
    button:active {
      transform: translate(0.05em, 0.05em);
      box-shadow: 0.05em 0.05em;
    }
    button.visible {
      display: block; /* Ana menüde ve zafer ekranında görünür */
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas"></canvas>
  <button id="playButton">Play</button>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const playButtonElement = document.getElementById('playButton');
    // Ölçekleme faktörleri (800x600 → 1920x1080)
    const scaleFactorX = 2.4; // 1920 / 800
    const scaleFactorY = 1.8; // 1080 / 600
    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      scaleX = canvas.width / 1920;
      scaleY = canvas.height / 1080;
      playButtonElement.style.left = `${860 * scaleX}px`;
      playButtonElement.style.top = `${490 * scaleY}px`;
      playButtonElement.style.width = `${200 * scaleX}px`;
      playButtonElement.style.height = `${100 * scaleY}px`;
      playButtonElement.style.fontSize = `${18 * Math.min(scaleX, scaleY)}px`;
    }
    let scaleX = 1;
    let scaleY = 1;
    resizeCanvas();
    window.addEventListener('resize', resizeCanvas);
    let currentLevel = 1;
    let gameState = {
      screen: 'menu',
      levelText: `Level ${currentLevel}`,
      controlsText: 'Arrow Keys: Move, Space: Jump, Mouse: Fire Goo, R: Restart',
      tutorialText: 'Yeşil olan çantalar yem\'dir. Onları yiyip büyürsün ve ateş edebilirsin. Basınç plakasına gelip ateş etmeyi dene ve sihri gör!',
      controlsAlpha: 1,
      controlsTimer: 0,
      tutorialAlpha: 1,
      tutorialTimer: 0,
      audioContext: null,
      assetsLoaded: false
    };
    let player = {
      x: 240,
      y: 900,
      w: 72,
      h: 54,
      vx: 0,
      vy: 0,
      speed: 4.8,
      airSpeed: 4.8,
      jumpForce: -10.8,
      grounded: false,
      color: 'gray',
      sizeMultiplier: 1,
      recoil: 9.6,
      foodCount: 0,
      animation: {
        state: 'idle',
        fireEndTime: 0,
        frame: 0,
        frameTimer: 0,
        frameDurations: {
          idle: 1200,
          right: 800,
          left: 800,
          jump: 600,
          fire: 400
        }
      }
    };
    let platforms = [];
    let movingPlatforms = [];
    let walls = [];
    let hazardWalls = [];
    let goos = [];
    let pressurePlates = [];
    let coloredWalls = [];
    let foods = [];
    const gravity = 0.18;
    const gooGravity = 0.18;
    const gooSpeed = 12;
    const friction = 0.4;
    const gooImmuneTime = 100;
    const epsilon = 0.24;
    const brickWidth = 80;
    const brickHeight = 50;
    const sprites = { animations: {} };
    const sounds = {};
    async function loadAssets() {
      const imageFiles = [
        { key: 'player', src: 'assets/images/player.png' },
        { key: 'player_green', src: 'assets/images/player_green.png' },
        { key: 'food', src: 'assets/images/food.png' },
        { key: 'goo', src: 'assets/images/goo.png' },
        { key: 'platform', src: 'assets/images/platform.png' },
        { key: 'moving_platform', src: 'assets/images/moving_platform.png' },
        { key: 'brick', src: 'assets/images/brick.png' },
        { key: 'hazard_wall', src: 'assets/images/hazard_wall.png' },
        { key: 'pressure_plate_red', src: 'assets/images/pressure_plate_red.png' },
        { key: 'pressure_plate_yellow', src: 'assets/images/pressure_plate_yellow.png' },
        { key: 'pressure_plate_blue', src: 'assets/images/pressure_plate_blue.png' },
        { key: 'pressure_plate_brown', src: 'assets/images/pressure_plate_brown.png' },
        { key: 'colored_wall_red', src: 'assets/images/colored_wall_red.png' },
        { key: 'colored_wall_yellow', src: 'assets/images/colored_wall_yellow.png' },
        { key: 'colored_wall_blue', src: 'assets/images/colored_wall_blue.png' },
        { key: 'colored_wall_brown', src: 'assets/images/colored_wall_brown.png' },
        { key: 'colored_wall_active', src: 'assets/images/colored_wall_active.png' },
        { key: 'menu_background', src: 'assets/images/menu_background.png' },
        { key: 'background1', src: 'assets/images/background1.png' },
        { key: 'background2', src: 'assets/images/background2.png' },
        { key: 'background3', src: 'assets/images/background3.png' },
        { key: 'background4', src: 'assets/images/background4.png' },
        { key: 'background5', src: 'assets/images/background5.png' },
        { key: 'player_right', src: 'assets/images/animations/RightWalkingAnimation.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_right_green', src: 'assets/images/animations/RightWalkingAnimationGreen.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_left', src: 'assets/images/animations/LeftWalkingAnimation.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_left_green', src: 'assets/images/animations/LeftWalkingAnimationGreen.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_jump', src: 'assets/images/animations/JumpStart.png', animation: true, columns: 1, rows: 2 },
        { key: 'player_jump_green', src: 'assets/images/animations/JumpGreen.png', animation: true, columns: 1, rows: 2 },
      ];
      const soundFiles = [
        { key: 'bgm', src: 'assets/sounds/bgm.mp3' },
        { key: 'jump', src: 'assets/sounds/jump.mp3' },
        { key: 'fire', src: 'assets/sounds/fire.mp3' },
        { key: 'level_complete', src: 'assets/sounds/level_complete.mp3' },
        { key: 'death', src: 'assets/sounds/death.mp3' },
        { key: 'finish_effect', src: 'assets/sounds/FinishEffect.mp3' },
        { key: 'eat', src: 'assets/sounds/eat.mp3' }
      ];
      const imagePromises = imageFiles.map(({ key, src, animation, columns, rows }) => {
        return new Promise((resolve) => {
          const img = new Image();
          img.src = src;
          img.onload = () => {
            if (animation) {
              const frameColumns = columns || 1;
              const frameRows = rows || 2;
              sprites.animations[key] = {
                image: img,
                frameCount: frameColumns * frameRows,
                frameWidth: img.width / frameColumns,
                frameHeight: img.height / frameRows,
                columns: frameColumns,
                rows: frameRows,
                scaledFrameWidth: (img.width / frameColumns) * scaleFactorX,
                scaledFrameHeight: (img.height / frameRows) * scaleFactorY
              };
            } else {
              sprites[key] = img;
            }
            resolve();
          };
          img.onerror = () => {
            console.warn(`Failed to load image: ${src}. Using fallback.`);
            if (animation) {
              sprites.animations[key] = null;
            } else {
              sprites[key] = null;
            }
            resolve();
          };
        });
      });
      const soundPromises = soundFiles.map(({ key, src }) => {
        return new Promise((resolve) => {
          const audio = new Audio(src);
          audio.onloadeddata = () => {
            sounds[key] = audio;
            resolve();
          };
          audio.onerror = () => {
            console.warn(`Failed to load sound: ${src}. Skipping.`);
            sounds[key] = null;
            resolve();
          };
        });
      });
      await Promise.all([...imagePromises, ...soundPromises]);
      if (sounds.bgm) {
        sounds.bgm.loop = true;
        sounds.bgm.volume = 0.3;
      }
      if (sounds.finish_effect) sounds.finish_effect.volume = 0.5;
      if (sounds.jump) sounds.jump.volume = 0.5;
      if (sounds.fire) sounds.fire.volume = 0;
      if (sounds.level_complete) sounds.level_complete.volume = 0.5;
      if (sounds.death) sounds.death.volume = 0.7;
      if (sounds.eat) sounds.eat.volume = 0.5;
      gameState.assetsLoaded = true;
      console.log('Assets loaded successfully.');
    }
    function playSound(key) {
      if (!gameState.audioContext) {
        gameState.audioContext = new (window.AudioContext || window.webkitAudioContext)();
      }
      if (gameState.audioContext.state === 'suspended') {
        gameState.audioContext.resume();
      }
      if (sounds[key]) {
        sounds[key].currentTime = 0;
        sounds[key].play().catch((e) => console.warn(`Failed to play sound ${key}: ${e}`));
      }
    }
    const levels = {
      1: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 240, y: 810, w: 360, h: 36 },
          { x: 840, y: 614, w: 480, h: 36 }
        ],
        walls: [
          { x: 1320, y: 790, w: 640, h: 200 },
          { x: 1320, y: 0, w: 640, h: 650 }
        ],
        pressurePlates: [
          { x: 1027.2, y: 602, w: 120, h: 12.6, color: 'red', targetWall: 0 }
        ],
        coloredWalls: [
          { x: 1440, y: 650, w: 48, h: 144, color: 'red', active: false }
        ],
        foods: [
          { x: 396, y: 774, w: 48, h: 36, eaten: false }
        ]
      },
      2: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 720, y: 810, w: 48, h: 1.8 },
          { x: 1200, y: 666, w: 48, h: 1.8 }
        ],
        walls: [
          { x: 0, y: 0, w: 240, h: 846 },
          { x: 1680, y: 0, w: 240, h: 846 },
          { x: 720, y: 790, w: 80, h: 200 },
          { x: 1200, y: 450, w: 80, h: 550 }
        ],
        foods: [
          { x: 840, y: 774, w: 48, h: 36, eaten: false }
        ]
      },
      3: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 480, y: 720, w: 360, h: 36 },
          { x: 1656, y: 810, w: 96, h: 36 },
          { x: 1460, y: 540, w: 96, h: 36 },
          { x: 1656, y: 270, w: 96, h: 36 }
        ],
        walls: [
          { x: 0, y: 0, w: 1920, h: 150 },
          { x: 0, y: 150, w: 160, h: 750 },
          { x: 960, y: 250, w: 80, h: 750 },
          { x: 1752, y: 270, w: 160, h: 730 },
          { x: 1300, y: 150, w: 160, h: 650 }
        ],
        pressurePlates: [
          { x: 1100, y: 977.4, w: 120, h: 12.6, color: 'brown', targetWall: 0 }
        ],
        coloredWalls: [
          { x: 1368, y: 800, w: 48, h: 190, color: 'brown', active: false }
        ],
        foods: [
          { x: 528, y: 684, w: 48, h: 36, eaten: false },
          { x: 720, y: 684, w: 48, h: 36, eaten: false }
        ]
      },
      4: {
        platforms: [
          { x: 0, y: 990, w: 1920, h: 90 },
          { x: 240, y: 720, w: 480, h: 36 },
          { x: 480, y: 550, w: 720, h: 36 }
        ],
        walls: [
          { x: 1440, y: 650, w: 80, h: 350 },
          { x: 0, y: 550, w: 240, h: 350 },
          { x: 0, y: 250, w: 240, h: 200 },
          { x: 0, y: 0, w: 240, h: 300 },
          { x: 240, y: 0, w: 1760, h: 300 },
          { x: 480, y: 300, w: 800, h: 50 },
          { x: 480, y: 350, w: 240, h: 100 },
          { x: 1040, y: 350, w: 240, h: 250 },
          { x: 480, y: 550, w: 800, h: 50 },
          { x: 1680, y: 300, w: 800, h: 550 }
        ],
        pressurePlates: [
          { x: 60, y: 537.4, w: 120, h: 12.6, color: 'yellow', targetWall: 0 },
          { x: 1272, y: 977.4, w: 120, h: 12.6, color: 'blue', targetWall: 1 }
        ],
        coloredWalls: [
          { x: 576, y: 450, w: 48, h: 100, color: 'yellow', active: false },
          { x: 1456, y: 300, w: 48, h: 350, color: 'blue', active: false }
        ],
        foods: [
          { x: 288, y: 684, w: 48, h: 36, eaten: false },
          { x: 840, y: 504, w: 48, h: 36, eaten: false }
        ]
      },
      5: {
        platforms: [
          { x: 0, y: 990, w: 840, h: 90 },
          { x: 1320, y: 990, w: 720, h: 90 },
          { x: 360, y: 810, w: 120, h: 18 },
          { x: 240, y: 504, w: 120, h: 18 },
          { x: 360, y: 198, w: 120, h: 18 },
          { x: 560, y: 198, w: 120, h: 18 },
          { x: 795.2, y: 286.2, w: 169.6, h: 3.6 }
        ],
        walls: [
          { x: 0, y: 0, w: 240, h: 850 },
          { x: 800, y: 0, w: 1120, h: 150 },
          { x: 480, y: 90, w: 80, h: 900 },
          { x: 800, y: 288, w: 160, h: 600 },
          { x: 960, y: 400, w: 720, h: 488 },
          { x: 1680, y: 288, w: 240, h: 600 },
          { x: 1840, y: 150, w: 80, h: 150 }
        ],
        hazardWalls: [
          { x: 840, y: 1026, w: 485, h: 50 },
          { x: 960, y: 310, w: 720, h: 100 }
        ],
        pressurePlates: [
          { x: 1720, y: 275.4, w: 120, h: 12.6, color: 'blue', targetWall: 0 }
        ],
        coloredWalls: [
          { x: 840, y: 990, w: 480, h: 36, color: 'blue', active: false }
        ],
        foods: [
          { x: 1756, y: 180, w: 48, h: 36, eaten: false }
        ],
        movingPlatforms: [
          { x1: 960, y1: 288, x2: 1520, y2: 288, w: 240, h: 18, speed: 1.2, currentX: 960, currentY: 288, direction: 1 }
        ]
      }
    };
    const playButton = {
      x: 860,
      y: 490,
      w: 200,
      h: 100
    };
    playButtonElement.addEventListener('click', () => {
      if (gameState.screen === 'menu' || gameState.screen === 'win') {
        gameState.screen = 'game';
        currentLevel = 1;
        loadLevel(currentLevel);
        playSound('bgm');
      }
    });
    canvas.addEventListener('click', (e) => {
      if (gameState.screen === 'menu') {
        const rect = canvas.getBoundingClientRect();
        const clickX = (e.clientX - rect.left) / scaleX;
        const clickY = (e.clientY - rect.top) / scaleY;
        if (
          clickX >= playButton.x &&
          clickX <= playButton.x + playButton.w &&
          clickY >= playButton.y &&
          clickY <= playButton.y + playButton.h
        ) {
          gameState.screen = 'game';
          loadLevel(currentLevel);
          playSound('bgm');
        }
      } else {
        fireGoo(e);
      }
    });
    function loadLevel(level) {
      const l = levels[level];
      platforms = l.platforms ? l.platforms.map(p => ({ ...p })) : [];
      movingPlatforms = l.movingPlatforms ? l.movingPlatforms.map(p => ({ ...p, currentX: p.x1, currentY: p.y1, direction: 1 })) : [];
      walls = l.walls ? l.walls.map(w => ({ ...w })) : [];
      hazardWalls = l.hazardWalls ? l.hazardWalls.map(h => ({ ...h })) : [];
      pressurePlates = l.pressurePlates ? l.pressurePlates.map(p => ({ ...p })) : [];
      coloredWalls = l.coloredWalls ? l.coloredWalls.map(w => ({ ...w })) : [];
      foods = l.foods ? l.foods.map(f => ({ ...f, eaten: false, vx: 0, vy: 0, grounded: false })) : [];
      goos = [];
      if (level === 5) sounds.fire.volume = 0.8;
      player.x = 240;
      player.y = 900;
      player.vx = 0;
      player.vy = 0;
      player.color = 'gray';
      player.sizeMultiplier = 1;
      player.w = 72;
      player.h = 54;
      player.foodCount = 0;
      player.animation = {
        state: 'idle',
        fireEndTime: 0,
        frame: 0,
        frameTimer: 0,
        frameDurations: {
          idle: 1200,
          right: 800,
          left: 800,
          jump: 600,
          fire: 400
        }
      };
      keys = {};
      gameState.levelText = `Level ${level}`;
      gameState.controlsAlpha = level <= 2 ? 1 : 0; // Sadece ilk 2 bölümde kontrolleri göster
      gameState.controlsTimer = 0;
      gameState.tutorialAlpha = level === 1 ? 1 : 0; // Sadece ilk bölümde eğitim metnini göster
      gameState.tutorialTimer = 0;
      let pressureStates = pressurePlates.map(() => false);
      for (let i = 0; i < pressurePlates.length; i++) {
        let plate = pressurePlates[i];
        for (let food of foods) {
          if (
            !food.eaten &&
            food.x + food.w > plate.x &&
            food.x < plate.x + plate.w &&
            food.y + food.h > plate.y &&
            food.y < plate.y + plate.h
          ) {
            pressureStates[i] = true;
          }
        }
      }
      for (let i = 0; i < pressurePlates.length; i++) {
        coloredWalls[pressurePlates[i].targetWall].active = pressureStates[i];
      }
      console.log(`Seviye ${level} yüklendi, yem sayısı: ${foods.length}, tehlike duvarları: ${hazardWalls.length || 0}, hareketli platformlar: ${movingPlatforms.length || 0}`);
    }
    let keys = {};
    window.addEventListener('keydown', (e) => {
      if (gameState.screen !== 'game') return;
      keys[e.key] = true;
      if (!gameState.audioContext && gameState.assetsLoaded) {
        playSound('bgm');
      }
      if (e.key === 'r' || e.key === 'R') {
        loadLevel(currentLevel);
      }
      if (e.key === 'p' || e.key === 'P') {
        if (currentLevel > 1) {
          currentLevel--;
          loadLevel(currentLevel);
        }
      }
      if (e.key === 'm' || e.key === 'M') {
        if (levels[currentLevel + 1]) {
          currentLevel++;
          loadLevel(currentLevel);
        }
      }
      if ((e.key === ' ' || e.key === 'ArrowUp') && player.grounded) {
        player.vy = player.jumpForce;
        player.grounded = false;
        playSound('jump');
        console.log('Zıplama: vy =', player.vy);
      }
    });
    window.addEventListener('keyup', (e) => {
      if (gameState.screen !== 'game') return;
      keys[e.key] = false;
    });
    function fireGoo(e) {
      if (gameState.screen !== 'game') return;
      if (player.color !== 'green' || player.foodCount <= 0) {
        console.log('Ateş edemezsiniz: Yeşil değil veya küp yok.');
        return;
      }
      const rect = canvas.getBoundingClientRect();
      const mouseX = (e.clientX - rect.left) / scaleX;
      const mouseY = (e.clientY - rect.top) / scaleY;
      const dx = mouseX - (player.x + player.w / 2);
      const dy = mouseY - (player.y + player.h / 2);
      const mag = Math.sqrt(dx * dx + dy * dy);
      let willCollideWithWall = false;
      for (let wall of walls) {
        if (
          (dx > 0 && player.x + player.w > wall.x && player.x < wall.x + wall.w) ||
          (dx < 0 && player.x < wall.x + wall.w && player.x + player.w > wall.x)
        ) {
          if (
            (dy > 0 && player.y + player.h > wall.y && player.y < wall.y + wall.h) ||
            (dy < 0 && player.y < wall.y + wall.h && player.y + player.h > wall.y)
          ) {
            willCollideWithWall = true;
            break;
          }
        }
      }
      let adjustedDx = dx;
      let adjustedDy = dy;
      if (willCollideWithWall) {
        const angleAdjustment = (Math.random() * 0.5 - 0.25) * Math.PI;
        const newAngle = Math.atan2(dy, dx) + angleAdjustment;
        const newMag = mag;
        adjustedDx = Math.cos(newAngle) * newMag;
        adjustedDy = Math.sin(newAngle) * newMag;
      }
      const vx = (adjustedDx / mag) * gooSpeed;
      const vy = (adjustedDy / mag) * gooSpeed;
      let offset = Math.max(player.w, player.h) / 2 + foods[0].w / 2;
      let startX = player.x + player.w / 2 + (adjustedDx / mag) * offset;
      let startY = player.y + player.h / 2 + (adjustedDy / mag) * offset;
      for (let wall of walls) {
        if (
          startX + foods[0].w > wall.x &&
          startX < wall.x + wall.w &&
          startY + foods[0].h > wall.y &&
          startY < wall.y + wall.h
        ) {
          if (adjustedDx > 0) {
            startX = wall.x - foods[0].w - 1;
          } else {
            startX = wall.x + wall.w + 1;
          }
        }
      }
      let immuneTime = gooImmuneTime;
      let recoilY = player.recoil;
      if (player.grounded && dy > 0) {
        startY = player.y - foods[0].h;
        immuneTime = 300;
        recoilY *= 0.5;
      }
      goos.push({
        x: startX,
        y: startY,
        vx: vx,
        vy: vy,
        w: foods[0].w,
        h: foods[0].h,
        grounded: false,
        spawnTime: performance.now(),
        immuneTime: immuneTime
      });
      player.vx -= (adjustedDx / mag) * player.recoil * 1.5;
      player.vy -= (adjustedDy / mag) * recoilY;
      player.foodCount--;
      if (player.foodCount <= 0) {
        player.color = 'gray';
        player.sizeMultiplier = 1;
        player.w = 72;
        player.h = 54;
      }
      player.animation.state = 'fire';
      player.animation.fireEndTime = performance.now() + 400;
      player.animation.frame = 0;
      player.animation.frameTimer = 0;
      playSound('fire');
      console.log(`Ateş edildi: vx = ${vx}, vy = ${vy}, startY = ${startY}`);
    }
    function update() {
      if (gameState.screen !== 'game' || !gameState.assetsLoaded) return;
      const moveSpeed = player.grounded ? player.speed : player.airSpeed;
      if (keys['ArrowLeft'] || keys['a'] || keys['A']) player.vx = -moveSpeed;
      else if (keys['ArrowRight'] || keys['d'] || keys['D']) player.vx = moveSpeed;
      else player.vx *= 0.9;
      player.vy += gravity;
      let newX = player.x + player.vx;
      let newY = player.y + player.vy;
      if (gameState.controlsAlpha > 0) {
        gameState.controlsTimer += 1 / 60;
        if (gameState.controlsTimer >= 5) {
          gameState.controlsAlpha = Math.max(0, gameState.controlsAlpha - 1 / 60);
        }
      }
      if (gameState.tutorialAlpha > 0) {
        gameState.tutorialTimer += 1 / 60;
        if (gameState.tutorialTimer >= 10) {
          gameState.tutorialAlpha = Math.max(0, gameState.tutorialAlpha - 1 / 60);
        }
      }
      const currentTime = performance.now();
      let newAnimationState = player.animation.state;
      if (player.animation.fireEndTime > currentTime) {
        newAnimationState = 'fire';
      } else if (!player.grounded) {
        newAnimationState = 'jump';
      } else if (keys['ArrowRight'] || keys['d'] || keys['D']) {
        newAnimationState = 'right';
      } else if (keys['ArrowLeft'] || keys['a'] || keys['A']) {
        newAnimationState = 'left';
      } else {
        newAnimationState = 'idle';
      }
      if (newAnimationState !== player.animation.state) {
        player.animation.state = newAnimationState;
        player.animation.frame = 0;
        player.animation.frameTimer = 0;
      }
      player.animation.frameTimer += 1000 / 60;
      const currentFrameDuration = player.animation.frameDurations[player.animation.state] || 800;
      if (player.animation.frameTimer >= currentFrameDuration) {
        const anim = sprites.animations[`player_${player.animation.state}${player.color === 'green' ? '_green' : ''}`];
        player.animation.frame = (player.animation.frame + 1) % (anim ? anim.frameCount : 2);
        player.animation.frameTimer = 0;
      }
      for (let platform of movingPlatforms) {
        const targetX = platform.direction === 1 ? platform.x2 : platform.x1;
        const targetY = platform.direction === 1 ? platform.y2 : platform.y1;
        const dx = targetX - platform.currentX;
        const dy = targetY - platform.currentY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        if (distance <= platform.speed) {
          platform.currentX = targetX;
          platform.currentY = targetY;
          platform.direction *= -1;
        } else {
          const moveX = (dx / distance) * platform.speed;
          const moveY = (dy / distance) * platform.speed;
          platform.currentX += moveX;
          platform.currentY += moveY;
        }
      }
      let pressureStates = pressurePlates.map(() => false);
      for (let food of foods) {
        if (!food.eaten) {
          food.vy += gravity;
          let newFoodX = food.x + food.vx;
          let newFoodY = food.y + food.vy;
          food.grounded = false;
          for (let platform of platforms) {
            if (
              newFoodX + food.w > platform.x &&
              newFoodX < platform.x + platform.w &&
              newFoodY + food.h > platform.y &&
              food.y + food.h <= platform.y &&
              food.vy >= 0
            ) {
              newFoodY = platform.y - food.h;
              food.vy = 0;
              food.vx *= friction;
              food.grounded = true;
            }
          }
          for (let platform of movingPlatforms) {
            if (
              newFoodX + food.w > platform.currentX &&
              newFoodX < platform.currentX + platform.w &&
              newFoodY + food.h > platform.currentY &&
              food.y + food.h <= platform.currentY &&
              food.vy >= 0
            ) {
              newFoodY = platform.currentY - food.h;
              food.vy = 0;
              food.vx *= friction;
              food.grounded = true;
            }
          }
          for (let wall of walls) {
            if (
              newFoodX + food.w > wall.x &&
              newFoodX < wall.x + wall.w &&
              newFoodY + food.h > wall.y &&
              newFoodY < wall.y + wall.h
            ) {
              if (food.y + food.h <= wall.y && newFoodY + food.h > wall.y && food.vy >= 0) {
                newFoodY = wall.y - food.h;
                food.vy = 0;
                food.vx *= friction;
                food.grounded = true;
              } else if (food.y >= wall.y + wall.h && newFoodY < wall.y + wall.h && food.vy < 0) {
                newFoodY = wall.y + wall.h;
                food.vy = 0;
              } else if (food.x + food.w <= wall.x && newFoodX + food.w > wall.x) {
                newFoodX = wall.x - food.w;
                food.vx = 0;
              } else if (food.x >= wall.x + wall.w && newFoodX < wall.x + wall.w) {
                newFoodX = wall.x + wall.w;
                food.vx = 0;
              }
            }
          }
          for (let wall of coloredWalls) {
            if (
              !wall.active &&
              newFoodX + food.w > wall.x &&
              newFoodX < wall.x + wall.w &&
              newFoodY + food.h > wall.y &&
              newFoodY < wall.y + wall.h
            ) {
              if (food.y + food.h <= wall.y && newFoodY + food.h > wall.y && food.vy >= 0) {
                newFoodY = wall.y - food.h;
                food.vy = 0;
                food.vx *= friction;
               
## İkinci Sorgu
-1 saniye kodun beklemesini istiyorum o gecikmeyi nasıl yaşatabilirim

-JavaScript'te 1 Saniye Bekleme (Gecikme) Ekleme
JavaScript'te kodunuzun 1 saniye beklemesini sağlamak için birkaç farklı yöntem kullanabilirsiniz. İşte en yaygın ve uygun çözümler:

1. setTimeout Kullanarak
javascript
console.log("Bekleme başlıyor...");
setTimeout(() => {
  console.log("1 saniye sonra bu mesaj görünecek");
  // Burada bekledikten sonra yapılacak işlemler
}, 1000); // 1000 milisaniye = 1 saniye
2. async/await ile Promise Kullanarak (Modern Yöntem)
javascript
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function myFunction() {
  console.log("Bekleme başlıyor...");
  await delay(1000); // 1 saniye bekle
  console.log("1 saniye sonra bu mesaj görünecek");
  // Burada bekledikten sonra yapılacak işlemler
}

myFunction();
3. Oyun Döngüsüne Uyarlanmış Örnek
Oyun döngünüzde 1 saniye bekleme eklemek için:

javascript
let waitStartTime = 0;
let isWaiting = false;

function gameLoop(timestamp) {
  if (!isWaiting) {
    // Normal oyun güncellemeleri
    update();
    draw();
  } else {
    // Bekleme süresini kontrol et
    if (timestamp - waitStartTime >= 1000) {
      isWaiting = false;
    }
  }
  requestAnimationFrame(gameLoop);
}

// Bekleme başlatma fonksiyonu
function startWaiting() {
  isWaiting = true;
  waitStartTime = performance.now();
}

// Kullanım örneği:
startWaiting(); // 1 saniye beklemeyi başlat
4. Özel Durum: Oyununuzda Belirli Bir Animasyon İçin Bekleme
Eğer belirli bir animasyon için 1 saniye beklemek istiyorsanız:

javascript
let animationPauseTime = 0;

function update() {
  const currentTime = performance.now();
  
  // Bekleme durumunu kontrol et
  if (animationPauseTime > 0 && currentTime < animationPauseTime) {
    return; // Bekleme süresince güncelleme yapma
  }
  
  // Normal güncellemeler
  // ...
}

// 1 saniye beklemeyi başlat
function pauseAnimation() {
  animationPauseTime = performance.now() + 1000;
}
Önemli Notlar:
JavaScript'te sleep() gibi direkt bekleme fonksiyonları yoktur, bu yüzden yukarıdaki yöntemler kullanılır.

setTimeout callback tabanlı olduğu için async/await ile kullanım daha modern ve okunabilirdir.

Oyun döngüsünde direkt bekleme yapmak performansı etkileyebilir, bu yüzden zaman damgası (timestamp) karşılaştırması yapmak daha iyidir.

Bekleme süresince kullanıcı etkileşimlerini engellememeye dikkat edin.

Hangi yöntemi seçeceğiniz, kodunuzun yapısına ve ne yapmaya çalıştığınıza bağlıdır. Oyun geliştirme bağlamında, zaman damgası karşılaştırması yapmak genellikle en iyi seçenektir.

## Üçüncü Sorgu
-böyle bir oyun kodum var sol tıkladığımda ateş ediyorum ve karakterim ateş ettiğim yönün tersine doğru geri tepiyor. Bu tepme miktarını arttırmak istiyorum biraz daha hissedilebilir yapar mısın
Karakterin geri tepme (recoil) miktarını artırmak için player.recoil değerini artırabiliriz. Şu anda bu değer 3 olarak ayarlanmış. Daha fazla tepki hissetmek için bu değeri 5 veya 6 gibi daha yüksek bir değere çıkarabiliriz.

-İşte değiştirilmiş kod parçası:

javascript
let player = {
  // ... diğer özellikler aynı
  recoil: 6, // 3 yerine 6 yaptık (2 kat daha güçlü tepki)
  foodCount: 0
};
Bu değişiklik, karakterin ateş ettiğinde daha fazla geri tepmesini sağlayacaktır. Eğer bu hala yeterli gelmezse, değeri 8 veya 10 gibi daha yüksek bir sayıya çıkarabilirsiniz.

Ayrıca, geri tepmenin daha dramatik olmasını isterseniz, hem yatay hem de dikey tepkiyi artırabiliriz. Bunun için fireGoo fonksiyonundaki geri tepme kuvvetini değiştirebiliriz:

javascript
function fireGoo(e) {
  // ... önceki kod aynı
  player.vx -= (dx / mag) * player.recoil * 1.5; // 1.5 kat daha fazla yatay tepki
  player.vy -= (dy / mag) * player.recoil * 1.5; // 1.5 kat daha fazla dikey tepki
  // ... sonraki kod aynı
}
Bu değişikliklerle birlikte karakteriniz ateş ettiğinde çok daha belirgin bir şekilde geri tepecektir. Değerleri deneyerek istediğiniz hissi yakalayana kadar ayarlayabilirsiniz.

Recoil değerini değiştirmek tam da beklediğim değişiklikti teşekkür ederim fakat anlamadığım bir nokta var player.vx için ne yaparsam yapayım bir türlü yatay geri tepmeyi kazanamıyorum yatay olarak hep aynı yerinde kalmaya devam ediyor
Yatay geri tepme (recoil) sorununuzun nedeni muhtemelen update() fonksiyonunda her frame'de player.vx değerini sıfırlamanızdan kaynaklanıyor. Şu anda kodunuzda:

javascript
function update() {
  // Hareket
  player.vx = 0; // <-- Bu satır her frame'de vx'i sıfırlıyor
  const moveSpeed = player.grounded ? player.speed : player.airSpeed;
  if (keys['ArrowLeft'] || keys['a'] || keys['A']) player.vx = -moveSpeed;
  if (keys['ArrowRight'] || keys['d'] || keys['D']) player.vx = moveSpeed;
  
  // ... diğer kodlar
}
Bu, ateş ettiğinizde uygulanan geri tepme kuvvetinin hemen sıfırlanmasına neden oluyor. Çözüm olarak şu değişiklikleri yapabiliriz:

player.vx = 0 satırını kaldırın veya değiştirin

Hareket girdilerini player.vx'e ekleyin (üzerine yazmak yerine)

İşte düzeltilmiş kod:

javascript
function update() {
  // Hareket - vx'i sıfırlamak yerine hareket inputlarını ekliyoruz
  const moveSpeed = player.grounded ? player.speed : player.airSpeed;
  player.vx = 0; // Önce sıfırla (veya bu satırı tamamen kaldırın)
  
  if (keys['ArrowLeft'] || keys['a'] || keys['A']) player.vx -= moveSpeed;
  if (keys['ArrowRight'] || keys['d'] || keys['D']) player.vx += moveSpeed;
  
  // ... diğer kodlar aynı
}
Veya daha iyi bir alternatif - geri tepmeyi korumak için:

javascript
function update() {
  // Hareket - vx'i sıfırlamıyoruz, sadece inputları uyguluyoruz
  const moveSpeed = player.grounded ? player.speed : player.airSpeed;
  
  // Önceki vx değerini koru, sadece inputları ekle
  if (keys['ArrowLeft'] || keys['a'] || keys['A']) player.vx = -moveSpeed;
  else if (keys['ArrowRight'] || keys['d'] || keys['D']) player.vx = moveSpeed;
  else player.vx *= 0.9; // Yavaşça durması için (isteğe bağlı)
  
  // ... diğer kodlar aynı
}
Ayrıca, fireGoo() fonksiyonundaki geri tepme kuvvetini de artırabilirsiniz:

javascript
function fireGoo(e) {
  // ... önceki kodlar
  player.vx -= (dx / mag) * player.recoil * 2; // 2 kat daha güçlü yatay tepki
  player.vy -= (dy / mag) * player.recoil;
  // ... sonraki kodlar
}
Bu değişikliklerle birlikte:

Geri tepme kuvveti artık frame başında sıfırlanmayacak

Yatay hareket daha belirgin olacak

Karakteriniz ateş ettiğinde gerçekten geri tepecek

Eğer hala yeterli gelmezse, player.recoil değerini daha da artırabilir (örneğin 8-10 gibi) veya hava direncini azaltabilirsiniz (şu anda friction değeri 0.4).

- yine aynı oyunla ilgili soru soracağım bugün biraz gelişti oyun, şimdi şey yapmak istiyorum yemlerde fizik var yani yerçekimiyle düşüyorlar fakat level yüklendiğinde mesela düştükleri konum eğer basınç plakası ise kapının ona göre aktifleşmesini istiyorum fakat ben ateş etmeden basınç plakası yemi tanımıyor.

-Yemlerin basınç plakalarını tetiklemesi için loadLevel fonksiyonunda ve update fonksiyonunda bazı düzenlemeler yapmamız gerekiyor. Probleminiz, yemlerin başlangıç pozisyonlarında basınç plakalarını kontrol etmemesi ve fizik hesaplarının doğru çalışmamasından kaynaklanıyor.

İşte düzeltilmiş kod parçaları:

loadLevel fonksiyonunda yemlerin başlangıç pozisyonlarını kontrol eden kısmı güncelleyin:

javascript
function loadLevel(level) {
  // ... diğer kodlar aynı

  // Check initial food-pressure plate collisions
  let pressureStates = pressurePlates ? pressurePlates.map(() => false) : [];
  if (foods && pressurePlates) {
    for (let i = 0; i < pressurePlates.length; i++) {
      let plate = pressurePlates[i];
      for (let food of foods) {
        if (!food.eaten &&
            food.x + food.w > plate.x && food.x < plate.x + plate.w &&
            food.y + food.h > plate.y + plate.h && // Bu satırı değiştirdik
            food.y < plate.y + plate.h) {          // Bu satırı ekledik
          pressureStates[i] = true;
        }
      }
    }
  }
  
  // ... diğer kodlar aynı
}
update fonksiyonunda yemlerin basınç plakalarıyla etkileşimini düzelten kısmı güncelleyin:

javascript
// Yem fiziksel güncelleme ve pressure plate kontrolü
if (foods) {
  for (let food of foods) {
    if (!food.eaten) {
      // ... diğer fizik hesaplamaları aynı

      // Pressure plate çarpışmaları
      if (pressurePlates) {
        for (let i = 0; i < pressurePlates.length; i++) {
          let plate = pressurePlates[i];
          if (newFoodX + food.w > plate.x && newFoodX < plate.x + plate.w &&
              newFoodY + food.h > plate.y && newFoodY < plate.y + plate.h) { // Tam çarpışma kontrolü
            pressureStates[i] = true;
            // Yemi plakanın üzerine sabitle
            newFoodY = plate.y - food.h;
            food.vy = 0;
            food.vx = 0;
            food.grounded = true;
          }
        }
      }

      // ... diğer kodlar aynı
    }
  }
}
Ayrıca, 5. seviyenin tanımında basınç plakasının konumunu biraz daha ayarlayabiliriz:

javascript
5: {
  // ... diğer tanımlar aynı
  pressurePlates: [
    { x: 730, y: 543, w: 50, h: 7, color: 'brown', targetWall: 0 } // y değerini 543 yaptık
  ],
  // ... diğer tanımlar aynı
}
Bu değişikliklerle:

Yemler başlangıçta basınç plakalarının üzerindeyse plakayı aktif hale getirecek

Yemler düşerken basınç plakalarına daha doğru bir şekilde oturacak

Yemler plakaların üzerine düştüğünde sabit duracak ve plakayı sürekli aktif tutacak

seviyede yem (yeşil küp) başlangıçta plakanın üzerine düşecek ve plakayı tetikleyecek, böylece kahverengi duvar (hazard wall) kaybolacak. Eğer yem plakanın üzerine düşmüyorsa, plakanın y pozisyonunu biraz daha ayarlamanız gerekebilir.

-yemleri herhangi bir duvarın tam üstünde dururken ateş edersem (tam altıma, duvara doğru olacak şekilde) yemler duvarın içine giriyor hata oluyor.

-Yemlerin duvarların içine girmesini engellemek için fireGoo fonksiyonunu ve goo çarpışma kontrolünü güncelleyeceğim. İşte düzeltilmiş kod:

javascript
function fireGoo(e) {
  if (player.color !== 'green' || player.foodCount <= 0) {
    return;
  }
  const rect = canvas.getBoundingClientRect();
  const mouseX = (e.clientX - rect.left) / scaleX;
  const mouseY = (e.clientY - rect.top) / scaleY;
  const dx = mouseX - (player.x + player.w / 2);
  const dy = mouseY - (player.y + player.h / 2);
  const mag = Math.sqrt(dx * dx + dy * dy);
  
  // Duvara çarpma kontrolü - eğer ateş edilen yön duvara doğruysa
  let willCollideWithWall = false;
  if (walls) {
    for (let wall of walls) {
      // Ateş edilen yönde bir duvar var mı kontrol et
      if ((dx > 0 && player.x + player.w > wall.x && player.x < wall.x + wall.w) ||
          (dx < 0 && player.x < wall.x + wall.w && player.x + player.w > wall.x)) {
        if ((dy > 0 && player.y + player.h > wall.y && player.y < wall.y + wall.h) ||
            (dy < 0 && player.y < wall.y + wall.h && player.y + player.h > wall.y)) {
          willCollideWithWall = true;
          break;
        }
      }
    }
  }
  
  // Eğer duvara doğru ateş ediliyorsa, yönü biraz değiştir
  let adjustedDx = dx;
  let adjustedDy = dy;
  if (willCollideWithWall) {
    // Duvara çarpmamak için rastgele küçük bir açı ekle
    const angleAdjustment = (Math.random() * 0.5 - 0.25) * Math.PI; // -22.5° ile +22.5° arası
    const newAngle = Math.atan2(dy, dx) + angleAdjustment;
    const newMag = mag;
    adjustedDx = Math.cos(newAngle) * newMag;
    adjustedDy = Math.sin(newAngle) * newMag;
  }

  const vx = (adjustedDx / mag) * gooSpeed;
  const vy = (adjustedDy / mag) * gooSpeed;

  let offset = Math.max(player.w, player.h) / 2 + foods[0].w / 2;
  let startX = player.x + player.w / 2 + (adjustedDx / mag) * offset;
  let startY = player.y + player.h / 2 + (adjustedDy / mag) * offset;
  
  // Başlangıç pozisyonunun duvar içinde olup olmadığını kontrol et
  if (walls) {
    for (let wall of walls) {
      if (startX + foods[0].w > wall.x && startX < wall.x + wall.w &&
          startY + foods[0].h > wall.y && startY < wall.y + wall.h) {
        // Duvara çakışıyorsa pozisyonu ayarla
        if (adjustedDx > 0) {
          startX = wall.x - foods[0].w - 1;
        } else {
          startX = wall.x + wall.w + 1;
        }
      }
    }
  }

  let immuneTime = gooImmuneTime;
  let recoilY = player.recoil;

  if (player.grounded && dy > 0) {
    startY = player.y - foods[0].h;
    immuneTime = 300;
    recoilY *= 0.5; 
  }

  goos.push({
    x: startX,
    y: startY,
    vx: vx,
    vy: vy,
    w: foods[0].w,
    h: foods[0].h,
    grounded: false,
    spawnTime: performance.now(),
    immuneTime: immuneTime
  });
  player.vx -= (adjustedDx / mag) * player.recoil * 1.5;
  player.vy -= (adjustedDy / mag) * recoilY;
  player.foodCount--;
  if (player.foodCount <= 0) {
    player.color = 'gray';
    player.sizeMultiplier = 1;
    player.w = 30;
    player.h = 30;
  }
}
Ayrıca, goo'ların güncellenmesi sırasında duvar çarpışma kontrolünü de iyileştirmeliyiz:

javascript
// Goo hareketi ve çarpışma kontrolü
for (let i = goos.length - 1; i >= 0; i--) {
  let goo = goos[i];
  goo.vy += gooGravity;
  let newGooX = goo.x + goo.vx;
  let newGooY = goo.y + goo.vy;

  goo.grounded = false;
  
  // Duvarlarla çarpışma kontrolü
  if (walls) {
    for (let wall of walls) {
      if (newGooX + goo.w > wall.x && newGooX < wall.x + wall.w &&
          newGooY + goo.h > wall.y && newGooY < wall.y + wall.h) {
        
        // Çarpışma yönünü belirle
        let overlapX = Math.min(
          newGooX + goo.w - wall.x,
          wall.x + wall.w - newGooX
        );
        let overlapY = Math.min(
          newGooY + goo.h - wall.y,
          wall.y + wall.h - newGooY
        );
        
        // En küçük overlap hangi yöndeyse o yönde çarpışma var
        if (overlapX < overlapY) {
          // Yatay çarpışma
          if (newGooX < wall.x) {
            newGooX = wall.x - goo.w;
          } else {
            newGooX = wall.x + wall.w;
          }
          goo.vx = 0;
        } else {
          // Dikey çarpışma
          if (newGooY < wall.y) {
            newGooY = wall.y - goo.h;
          } else {
            newGooY = wall.y + wall.h;
            goo.grounded = true;
          }
          goo.vy = 0;
        }
      }
    }
  }

  // ... diğer çarpışma kontrolleri (platformlar vs.)
}
Bu değişikliklerle:

Yemler duvara doğru ateş edildiğinde, rastgele küçük bir açıyla yön değiştirecek

Yemlerin başlangıç pozisyonları duvarlarla çakışmıyor olacak

Yemler hareket ederken duvarlara daha sağlam bir şekilde çarpacak ve içine girmeyecek

Çarpışma sonrası yemler doğru şekilde sekerek hareketine devam edecek

Bu çözüm, yemlerin duvarların içine girmesini engelleyecek ve daha doğal bir fizik simülasyonu sağlayacaktır.
