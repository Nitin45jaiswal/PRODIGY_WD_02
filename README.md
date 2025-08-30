# PRODIGY_WD_02
Stopwatch  web Application

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Creative Stopwatch & Live Clock</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
    }
    body {
      font-family: Arial, sans-serif;
      min-height: 100vh;
      display: flex; 
      align-items: center; 
      justify-content: center;
      position: relative;
      background: #ece9e6;
      overflow: hidden;
    }
    /* Animated circles background */
    .bg-circles {
      position: fixed;
      top: 0; left: 0;
      width: 100vw; height: 100vh;
      pointer-events: none;
      z-index: 0;
      overflow: hidden;
    }
    .circle {
      position: absolute;
      border-radius: 50%;
      opacity: 0.25;
      animation: float 12s infinite alternate ease-in-out;
    }
    .circle1 { width: 220px; height: 220px; background: #f39c12; left: 10vw; top: 12vh; animation-delay: 0s; }
    .circle2 { width: 110px; height: 110px; background: #009ffd; right: 13vw; top: 16vh; animation-delay: 2s;}
    .circle3 { width: 320px; height: 320px; background: #38e492; left: 60vw; top: 66vh; animation-delay: 3.5s;}
    .circle4 { width: 180px; height: 180px; background: #ff5e62; left: 77vw; top: 35vh; animation-delay: 4.5s;}
    .circle5 { width: 100px; height: 100px; background: #fff700; left: 5vw; top: 74vh; animation-delay: 7s;}
    @keyframes float {
      from { transform: translateY(0);}
      to   { transform: translateY(-40px);}
    }
    .container {
      background: rgba(255,255,255,0.93);
      padding: 2.2em 2.5em 2em 2.5em;
      border-radius: 18px;
      box-shadow: 0 4px 24px rgba(0,0,0,0.14);
      text-align: center;
      width: 350px;
      position: relative;
      z-index: 3;
    }
    #liveclock {
      font-size: 1.3em;
      margin-bottom: 1.2em;
      color: #222;
      letter-spacing: 1px;
      font-family: 'Courier New', Courier, monospace;
    }
    #display {
      font-size: 3em;
      margin-bottom: 1em;
      font-variant-numeric: tabular-nums;
    }
    .controls-row {
      display: flex;
      justify-content: space-between;
      margin-bottom: 0.5em;
    }
    .controls-row button {
      font-size: 1em;
      padding: 0.65em 2em;
      border: none;
      border-radius: 5px;
      background: #333;
      color: #fff;
      cursor: pointer;
      transition: background 0.2s;
    }
    .controls-row button:hover {
      background: #f39c12;
      color: #222;
    }
    .lap-btn-row {
      display: flex;
      justify-content: center;
      margin-bottom: 1.2em;
    }
    .lap-btn-row button {
      font-size: 1em;
      padding: 0.65em 2em;
      border: none;
      border-radius: 5px;
      background: #333;
      color: #fff;
      cursor: pointer;
      transition: background 0.2s;
    }
    .lap-btn-row button:hover {
      background: #009ffd;
      color: #222;
    }
    .laps-list {
      padding: 0;
      margin: 0.6em 0 0 0;
      list-style: none;
      max-height: 160px;
      overflow-y: auto;
      font-size: 1.08em;
    }
    .laps-list li {
      background: #f4f4f4;
      margin: 0.2em 0;
      padding: 0.4em 0.5em;
      border-radius: 3px;
      display: flex;
      justify-content: space-between;
      color: #555;
    }
    .laps-list span {
      font-family: 'Courier New', Courier, monospace;
    }
  </style>
</head>
<body>
  <!-- Animated background circles -->
  <div class="bg-circles">
    <div class="circle circle1"></div>
    <div class="circle circle2"></div>
    <div class="circle circle3"></div>
    <div class="circle circle4"></div>
    <div class="circle circle5"></div>
  </div>
  <div class="container">
    <div id="liveclock">00:00:00</div>
    <div id="display">00:00:00.00</div>
    <div class="controls-row">
      <button id="startBtn">Start</button>
      <button id="pauseBtn" disabled>Pause</button>
      <button id="resetBtn" disabled>Reset</button>
    </div>
    <div class="lap-btn-row">
      <button id="lapBtn" disabled>Lap</button>
    </div>
    <ul class="laps-list" id="laps"></ul>
  </div>
  <script>
    // Live Digital Clock
    function updateLiveClock() {
      const now = new Date();
      const h = String(now.getHours()).padStart(2, '0');
      const m = String(now.getMinutes()).padStart(2, '0');
      const s = String(now.getSeconds()).padStart(2, '0');
      document.getElementById('liveclock').textContent = `${h}:${m}:${s}`;
    }
    setInterval(updateLiveClock, 500);
    updateLiveClock();

    // Stopwatch Logic
    let startTime, elapsed = 0, timerId = null, running = false, lapCount = 0;
    const display = document.getElementById('display');
    const startBtn = document.getElementById('startBtn');
    const pauseBtn = document.getElementById('pauseBtn');
    const resetBtn = document.getElementById('resetBtn');
    const lapBtn = document.getElementById('lapBtn');
    const laps = document.getElementById('laps');

    function formatTime(ms) {
      const centiseconds = Math.floor((ms % 1000) / 10);
      const seconds = Math.floor((ms / 1000) % 60);
      const minutes = Math.floor((ms / 60000) % 60);
      const hours = Math.floor(ms / 3600000);
      return `${hours.toString().padStart(2,'0')}:${minutes.toString().padStart(2,'0')}:${seconds.toString().padStart(2,'0')}.${centiseconds.toString().padStart(2,'0')}`;
    }

    function updateDisplay() {
      const now = running ? Date.now() : startTime;
      display.textContent = formatTime(elapsed + (running ? now - startTime : 0));
      if (running) {
        timerId = requestAnimationFrame(updateDisplay);
      }
    }

    startBtn.onclick = function () {
      if (!running) {
        startTime = Date.now() - elapsed;
        running = true;
        updateDisplay();
        startBtn.disabled = true;
        pauseBtn.disabled = false;
        resetBtn.disabled = false;
        lapBtn.disabled = false;
      }
    };

    pauseBtn.onclick = function () {
      if (running) {
        running = false;
        elapsed += Date.now() - startTime;
        cancelAnimationFrame(timerId);
        startBtn.disabled = false;
        pauseBtn.disabled = true;
        lapBtn.disabled = true;
      }
    };

    resetBtn.onclick = function () {
      running = false;
      elapsed = 0;
      lapCount = 0;
      cancelAnimationFrame(timerId);
      display.textContent = '00:00:00.00';
      laps.innerHTML = '';
      startBtn.disabled = false;
      pauseBtn.disabled = true;
      resetBtn.disabled = true;
      lapBtn.disabled = true;
    };

    lapBtn.onclick = function () {
      if (running) {
        lapCount++;
        const now = Date.now();
        const time = formatTime(elapsed + (now - startTime));
        const li = document.createElement('li');
        li.innerHTML = `<span>Lap ${lapCount}</span> <span>${time}</span>`;
        laps.prepend(li);
      }
    };
  </script>
</body>
</html>
