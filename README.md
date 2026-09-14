<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HTML 遊戲完整範例</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: Arial, sans-serif;
    }

    body {
      background-color: #1a1a2e;
      color: #ffffff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      overflow: hidden;
    }

    .card {
      background-color: #16213e;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 8px 16px rgba(0,0,0,0.4);
      text-align: center;
      width: 340px;
    }

    h2, h1 {
      margin-bottom: 20px;
      color: #e94560;
    }

    input[type="text"] {
      width: 100%;
      padding: 10px;
      margin-bottom: 15px;
      border: 1px solid #0f3460;
      border-radius: 6px;
      outline: none;
      background-color: #0f3460;
      color: #fff;
      font-size: 16px;
    }

    button {
      width: 100%;
      padding: 10px;
      background-color: #e94560;
      border: none;
      border-radius: 6px;
      color: white;
      font-size: 16px;
      cursor: pointer;
      transition: background 0.2s;
    }

    button:hover {
      background-color: #0f3460;
    }

    /* 畫面顯示控制 */
    #lobbyScreen, #gameScreen {
      display: none;
    }

    /* 遊戲畫面樣式 */
    #gameScreen {
      text-align: center;
    }

    canvas {
      background-color: #0f3460;
      border: 2px solid #e94560;
      border-radius: 8px;
      margin-top: 15px;
      display: block;
    }
  </style>
</head>
<body>

  <!-- 1. 輸入名字畫面 -->
  <div id="loginScreen" class="card">
    <h2>玩家登入</h2>
    <form id="nameForm">
      <input type="text" id="usernameInput" placeholder="請輸入名字..." required autocomplete="off">
      <button type="submit">進入遊戲大廳</button>
    </form>
  </div>

  <!-- 2. 遊戲大廳畫面 -->
  <div id="lobbyScreen" class="card">
    <h1 id="welcomeText">歡迎進入大廳</h1>
    <p style="margin-bottom: 20px; color: #a2a2a2;">準備好開始對戰了嗎？</p>
    <button id="startGameBtn" style="margin-bottom: 10px;">開始遊戲</button>
    <button id="logoutBtn" style="background-color: #53565a;">切換名字</button>
  </div>

  <!-- 3. 遊戲進行畫面 -->
  <div id="gameScreen">
    <h2 id="playerGameName">遊戲進行中</h2>
    <canvas id="gameCanvas" width="400" height="300"></canvas>
    <button id="backToLobbyBtn" style="margin-top: 15px; background-color: #53565a;">返回大廳</button>
  </div>

  <script>
    // --- DOM 元素取得 ---
    const loginScreen = document.getElementById('loginScreen');
    const lobbyScreen = document.getElementById('lobbyScreen');
    const gameScreen = document.getElementById('gameScreen');

    const nameForm = document.getElementById('nameForm');
    const usernameInput = document.getElementById('usernameInput');
    const welcomeText = document.getElementById('welcomeText');
    const playerGameName = document.getElementById('playerGameName');

    const startGameBtn = document.getElementById('startGameBtn');
    const logoutBtn = document.getElementById('logoutBtn');
    const backToLobbyBtn = document.getElementById('backToLobbyBtn');

    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    let currentPlayerName = "";
    let gameAnimationId = null;

    // --- 流程控制函數 ---

    // 進入大廳
    function enterLobby(name) {
      currentPlayerName = name;
      welcomeText.innerText = `歡迎，${name}！`;
      
      loginScreen.style.display = 'none';
      gameScreen.style.display = 'none';
      lobbyScreen.style.display = 'block';

      if (gameAnimationId) cancelAnimationFrame(gameAnimationId);
    }

    // 開始遊戲 (核心修復：綁定 startGameBtn 並觸發遊戲邏輯)
    function startGame() {
      lobbyScreen.style.display = 'none';
      gameScreen.style.display = 'block';
      playerGameName.innerText = `玩家：${currentPlayerName}`;

      initAndRunGame();
    }

    // --- 簡易遊戲動畫邏輯 (Demo) ---
    let posX = 50;
    let speedX = 2;

    function initAndRunGame() {
      posX = 50; // 重置位置
      
      function gameLoop() {
        // 清除畫布
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 繪製背景文字
        ctx.fillStyle = "#ffffff";
        ctx.font = "16px Arial";
        ctx.fillText(`正在遊戲中：${currentPlayerName}`, 20, 30);

        // 繪製移動方塊
        ctx.fillStyle = "#e94560";
        ctx.fillRect(posX, 120, 50, 50);

        // 移動碰撞邏輯
        posX += speedX;
        if (posX + 50 > canvas.width || posX < 0) {
          speedX = -speedX;
        }

        gameAnimationId = requestAnimationFrame(gameLoop);
      }

      gameLoop();
    }

    // --- 事件監聽 (EventListener) ---

    // 1. 提交名字 -> 進入大廳
    nameForm.addEventListener('submit', function(e) {
      e.preventDefault();
      const name = usernameInput.value.trim();
      if (name) {
        localStorage.setItem('gamePlayerName', name);
        enterLobby(name);
      } else {
        alert('請輸入有效的名字！');
      }
    });

    // 2. 點擊開始遊戲 -> 進入遊戲畫面
    startGameBtn.addEventListener('click', function() {
      startGame();
    });

    // 3. 返回大廳
    backToLobbyBtn.addEventListener('click', function() {
      enterLobby(currentPlayerName);
    });

    // 4. 切換名字/登出
    logoutBtn.addEventListener('click', function() {
      localStorage.removeItem('gamePlayerName');
      usernameInput.value = '';
      lobbyScreen.style.display = 'none';
      gameScreen.style.display = 'none';
      loginScreen.style.display = 'block';
    });

    // 5. 自動載入名字
    window.addEventListener('DOMContentLoaded', function() {
      const savedName = localStorage.getItem('gamePlayerName');
      if (savedName) {
        enterLobby(savedName);
      }
    });
  </script>

</body>
</html>
