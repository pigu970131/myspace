<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Space Odyssey | 3D Interactive Earth Overview Project</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Three.js & OrbitControls & GSAP -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
  <!-- Font Awesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
    
    body {
      margin: 0;
      overflow: hidden;
      background-color: #020617;
      font-family: 'Noto Sans TC', sans-serif;
      color: #f8fafc;
    }
    
    .font-orbitron {
      font-family: 'Orbitron', sans-serif;
    }

    /* 玻璃擬態 UI Glassmorphism */
    .glass-panel {
      background: rgba(15, 23, 42, 0.75);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      border: 1px solid rgba(255, 255, 255, 0.15);
      box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.7);
    }

    .glass-card {
      background: rgba(30, 41, 59, 0.5);
      backdrop-filter: blur(8px);
      border: 1px solid rgba(255, 255, 255, 0.1);
    }

    /* 自訂 UI 捲軸 */
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

    /* 雷達掃描動畫 */
    .radar-sweep {
      background: conic-gradient(from 0deg at 50% 50%, rgba(56, 189, 248, 0.3) 0deg, transparent 60deg, transparent 360deg);
      animation: sweep 4s linear infinite;
    }
    @keyframes sweep {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }
  </style>
</head>
<body class="relative h-screen w-screen select-none overflow-hidden">

  <!-- 3D Canvas 容器 -->
  <div id="canvas-container" class="absolute inset-0 z-0"></div>

  <!-- Loading 載入畫面 -->
  <div id="loading-screen" class="absolute inset-0 z-50 flex flex-col items-center justify-center bg-slate-950 transition-opacity duration-700">
    <div class="relative flex items-center justify-center">
      <div class="h-28 w-28 rounded-full border-t-2 border-b-2 border-cyan-400 animate-spin"></div>
      <div class="absolute h-20 w-20 rounded-full radar-sweep"></div>
      <i class="fa-solid fa-earth-americas text-cyan-400 text-2xl absolute"></i>
    </div>
    <h2 class="mt-6 text-xl font-bold tracking-widest text-slate-100 font-orbitron">INITIALIZING HIGH-RES EARTH...</h2>
    <p class="text-xs text-slate-400 mt-2 font-mono">載入 8K 衛星地圖、夜景燈火與 3D ISS 太空站軌道中...</p>
  </div>

  <!-- 頂部 HUD 狀態標題欄 -->
  <header class="absolute top-6 left-6 z-10 pointer-events-none">
    <div class="flex items-center gap-3">
      <div class="h-3 w-3 rounded-full bg-cyan-400 animate-ping"></div>
      <h1 class="text-2xl font-black tracking-wider font-orbitron text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-sky-300 to-indigo-400">
        SPACE ODYSSEY v2.0
      </h1>
      <span class="px-2 py-0.5 text-[10px] font-orbitron font-semibold bg-cyan-500/20 text-cyan-300 border border-cyan-500/30 rounded-full pointer-events-auto">MIDTERM EDITION</span>
    </div>
    <p class="text-xs text-slate-400 mt-1 tracking-widest uppercase font-mono">Interactive Overview Effect Visualization</p>
  </header>

  <!-- 左側控制面板 (三頁面選單：控制 / 景點飛躍 / 太空站追蹤) -->
  <aside class="absolute top-24 left-6 bottom-6 z-10 w-88 glass-panel rounded-2xl p-5 flex flex-col justify-between overflow-y-auto no-scrollbar pointer-events-auto">
    <div class="space-y-5">
      
      <!-- 系統數據遙測面板 -->
      <div>
        <div class="flex items-center justify-between text-xs text-slate-400 mb-2 font-orbitron">
          <span>SYSTEM TELEMETRY</span>
          <span class="flex items-center gap-1 text-emerald-400"><i class="fa-solid fa-circle text-[8px]"></i> ONLINE</span>
        </div>
        <div class="grid grid-cols-2 gap-2 text-xs font-mono">
          <div class="glass-card p-2.5 rounded-lg border-l-2 border-cyan-400">
            <span class="text-slate-400 block text-[10px]">ALTITUDE (高度)</span>
            <span id="telemetry-alt" class="font-orbitron font-semibold text-cyan-300 text-sm">400 km</span>
          </div>
          <div class="glass-card p-2.5 rounded-lg border-l-2 border-indigo-400">
            <span class="text-slate-400 block text-[10px]">ISS SPEED (速度)</span>
            <span class="font-orbitron font-semibold text-indigo-300 text-sm">27,600 km/h</span>
          </div>
        </div>
      </div>

      <hr class="border-slate-800">

      <!-- ISS 太空站即時追蹤按鈕 -->
      <div>
        <label class="text-xs font-semibold text-slate-300 tracking-wider flex items-center gap-2 mb-2 font-orbitron">
          <i class="fa-solid fa-satellite text-cyan-400"></i> ISS TRACKING SYSTEM
        </label>
        <button id="btn-lock-iss" onclick="toggleLockISS()" class="w-full glass-card hover:bg-cyan-500/20 text-cyan-300 border border-cyan-500/40 font-semibold py-2.5 px-4 rounded-xl text-xs transition flex items-center justify-center gap-2">
          <i class="fa-solid fa-crosshairs text-sm"></i> <span id="iss-btn-text">鎖定 ISS 太空站視角</span>
        </button>
      </div>

      <hr class="border-slate-800">

      <!-- 景點快速巡航 Fly-To -->
      <div class="space-y-2">
        <label class="text-xs font-semibold text-slate-300 tracking-wider flex items-center gap-2 font-orbitron">
          <i class="fa-solid fa-location-dot text-rose-400"></i> HOTSPOT FLY-TO (地標飛躍)
        </label>
        <div class="grid grid-cols-2 gap-2 text-xs">
          <button onclick="flyToLocation(23.5, 121, 2.2)" class="glass-card hover:bg-slate-700/60 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-mountain text-emerald-400 text-[10px]"></i> 台灣 (Taiwan)
          </button>
          <button onclick="flyToLocation(27.98, 86.92, 2.3)" class="glass-card hover:bg-slate-700/60 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-mountain-sun text-amber-400 text-[10px]"></i> 喜馬拉雅山
          </button>
          <button onclick="flyToLocation(29.97, 31.13, 2.3)" class="glass-card hover:bg-slate-700/60 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-pyramid text-yellow-500 text-[10px]"></i> 埃及金字塔
          </button>
          <button onclick="flyToLocation(19.89, -155.5, 2.4)" class="glass-card hover:bg-slate-700/60 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-volcano text-rose-500 text-[10px]"></i> 夏威夷火山
          </button>
        </div>
      </div>

      <hr class="border-slate-800">

      <!-- 太陽時間與控制滑塊 -->
      <div class="space-y-3">
        <label class="text-xs font-semibold text-slate-300 tracking-wider flex items-center justify-between font-orbitron">
          <span class="flex items-center gap-2"><i class="fa-solid fa-sun text-amber-400"></i> 晝夜光照時間</span>
          <span id="sun-time-text" class="text-amber-400 font-mono text-[11px]">12:00 PM</span>
        </label>
        <input type="range" id="solar-time" min="0" max="24" step="0.1" value="12" class="w-full h-1.5 bg-slate-700 rounded-lg appearance-none cursor-pointer accent-amber-400">

        <!-- 自轉速度 -->
        <div class="space-y-1 pt-1">
          <div class="flex justify-between text-xs text-slate-400 font-mono">
            <span>地球自轉速度</span>
            <span id="speed-val" class="text-cyan-400">1.0x</span>
          </div>
          <input type="range" id="rotation-speed" min="0" max="5" step="0.1" value="1" class="w-full h-1 bg-slate-700 rounded-lg appearance-none cursor-pointer accent-cyan-400">
        </div>

        <!-- 雲層與夜景開關 -->
        <div class="grid grid-cols-2 gap-2 pt-1 text-xs text-slate-300">
          <label class="flex items-center gap-2 cursor-pointer glass-card p-2 rounded-lg">
            <input type="checkbox" id="toggle-clouds" checked class="accent-cyan-400"> 3D 雲層渲染
          </label>
          <label class="flex items-center gap-2 cursor-pointer glass-card p-2 rounded-lg">
            <input type="checkbox" id="toggle-atmosphere" checked class="accent-cyan-400"> 大氣光暈
          </label>
        </div>
      </div>
    </div>

    <!-- 底部操作提示 -->
    <div class="mt-4 pt-3 border-t border-slate-800 text-[11px] text-slate-400 space-y-1 font-mono">
      <p><i class="fa-solid fa-rotate text-cyan-400"></i> 按住左鍵拖曳：360度旋轉視野</p>
      <p><i class="fa-solid fa-magnifying-glass text-cyan-400"></i> 滾輪滾動：拉近與縮放軌道</p>
    </div>
  </aside>

  <!-- 右下角太空思考與名言卡片 -->
  <main class="absolute bottom-6 right-6 z-10 max-w-sm glass-panel rounded-2xl p-5 pointer-events-auto transition-all">
    <div class="flex items-center justify-between text-xs font-orbitron text-cyan-400 mb-2">
      <span class="flex items-center gap-1.5"><i class="fa-solid fa-quote-left"></i> OVERVIEW EFFECT</span>
      <span class="text-[10px] text-slate-500 font-mono">CARD 1/3</span>
    </div>
    <p id="quote-text" class="text-xs text-slate-200 leading-relaxed font-light">
      「從外太空看地球，你看不到國界與衝突。你只會看到一顆懸浮於虛空之中、極其脆弱卻又無比美麗的藍色家園。」
    </p>
    <div class="mt-3 pt-3 border-t border-slate-800/80 flex items-center justify-between text-xs text-slate-400 font-mono">
      <span id="quote-author" class="italic text-[11px]">—— Apollo Astronaut</span>
      <button onclick="nextQuote()" class="text-cyan-400 hover:text-cyan-300 flex items-center gap-1 transition text-[11px]">
        下一張 <i class="fa-solid fa-chevron-right text-[10px]"></i>
      </button>
    </div>
  </main>

  <!-- 核心 3D Engine 腳本 -->
  <script>
    let scene, camera, renderer, controls;
    let earth, nightEarth, clouds, atmosphere, issGroup, sunLight, ambientLight;
    let isLockedISS = false;
    let issOrbitAngle = 0;

    const quotes = [
      { text: "「從外太空看地球，你看不到國界與衝突。你只會看到一顆懸浮於虛空之中、極其脆弱卻又無比美麗的藍色家園。」", author: "—— 總觀效應 (Overview Effect)" },
      { text: "「人類所有的驕傲、紛爭與夢想，其實都發生在那顆懸浮在太陽光束中的『淡藍小點』上。」", author: "—— 卡爾·薩根 (Carl Sagan)" },
      { text: "「在太空裡，大氣層看起來就像一層薄薄的藍色面紗，提醒著我們這顆星球上的生命有多麼依賴它。」", author: "—— ISS 國際太空站太空人" }
    ];
    let currentQuoteIndex = 0;

    function init() {
      const container = document.getElementById('canvas-container');

      // 1. Scene & Camera
      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(0, 0, 4.5);

      // 2. Renderer
      renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      container.appendChild(renderer.domElement);

      // 3. Orbit Controls
      controls = new THREE.OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;
      controls.minDistance = 1.6;
      controls.maxDistance = 10.0;

      // 4. Lighting
      ambientLight = new THREE.AmbientLight(0xffffff, 0.15);
      scene.add(ambientLight);

      sunLight = new THREE.DirectionalLight(0xffffff, 2.0);
      sunLight.position.set(5, 0, 3);
      scene.add(sunLight);

      // 5. Textures Loading
      const textureLoader = new THREE.TextureLoader();
      
      const dayTexture = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg', () => {
        document.getElementById('loading-screen').classList.add('opacity-0');
        setTimeout(() => document.getElementById('loading-screen').remove(), 700);
      });
      const nightTexture = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-night-lights.png');
      const bumpTexture = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-topology.png');

      // 6. Earth Mesh (白天與地形)
      const earthGeo = new THREE.SphereGeometry(1, 64, 64);
      const earthMat = new THREE.MeshStandardMaterial({
        map: dayTexture,
        bumpMap: bumpTexture,
        bumpScale: 0.04,
        roughness: 0.7,
        metalness: 0.1
      });
      earth = new THREE.Mesh(earthGeo, earthMat);
      scene.add(earth);

      // 7. Night Lights Mesh (黑夜夜景發光層)
      const nightMat = new THREE.MeshBasicMaterial({
        map: nightTexture,
        blending: THREE.AdditiveBlending,
        transparent: true,
        opacity: 0.85
      });
      nightEarth = new THREE.Mesh(earthGeo, nightMat);
      earth.add(nightEarth);

      // 8. 3D Clouds (動態雲層)
      const cloudsGeo = new THREE.SphereGeometry(1.02, 64, 64);
      const cloudsMat = new THREE.MeshStandardMaterial({
        map: textureLoader.load('https://unpkg.com/three-globe/example/img/earth-clouds.png'),
        transparent: true,
        opacity: 0.45,
        blending: THREE.AdditiveBlending
      });
      clouds = new THREE.Mesh(cloudsGeo, cloudsMat);
      scene.add(clouds);

      // 9. Custom Atmosphere Rayleigh Glow (大氣 Shader)
      const atmosphereGeo = new THREE.SphereGeometry(1.22, 64, 64);
      const atmosphereMat = new THREE.ShaderMaterial({
        vertexShader: `
          varying vec3 vNormal;
          void main() {
            vNormal = normalize(normalMatrix * normal);
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
          }
        `,
        fragmentShader: `
          varying vec3 vNormal;
          void main() {
            float intensity = pow(0.65 - dot(vNormal, vec3(0, 0, 1.0)), 2.2);
            gl_FragColor = vec4(0.25, 0.6, 1.0, 1.0) * intensity;
          }
        `,
        blending: THREE.AdditiveBlending,
        side: THREE.BackSide,
        transparent: true
      });
      atmosphere = new THREE.Mesh(atmosphereGeo, atmosphereMat);
      scene.add(atmosphere);

      // 10. ISS 軌道與 3D 太空站 Mesh 創建
      createISSAndOrbit();

      // 11. 星空背景
      createStarfield();

      // 事件綁定
      window.addEventListener('resize', onWindowResize);
      setupUIEvents();

      // 動態渲染迴圈
      animate();
    }

    // 建立 3D ISS 太空站模型與軌道線
    function createISSAndOrbit() {
      // 軌道圓環線
      const orbitGeo = new THREE.RingGeometry(1.44, 1.45, 128);
      const orbitMat = new THREE.MeshBasicMaterial({
        color: 0x38bdf8,
        side: THREE.DoubleSide,
        transparent: true,
        opacity: 0.35
      });
      const orbitLine = new THREE.Mesh(orbitGeo, orbitMat);
      orbitLine.rotation.x = Math.PI / 3; // 軌道傾角
      scene.add(orbitLine);

      // ISS 組合群組
      issGroup = new THREE.Group();
      
      // ISS 主體 (銀色金屬)
      const bodyGeo = new THREE.CylinderGeometry(0.012, 0.012, 0.08, 16);
      const bodyMat = new THREE.MeshStandardMaterial({ color: 0xe2e8f0, metalness: 0.9, roughness: 0.2 });
      const body = new THREE.Mesh(bodyGeo, bodyMat);
      body.rotation.z = Math.PI / 2;
      issGroup.add(body);

      // 太陽能板 (深藍發光材質)
      const panelGeo = new THREE.BoxGeometry(0.18, 0.002, 0.04);
      const panelMat = new THREE.MeshStandardMaterial({ color: 0x1e3a8a, metalness: 0.5, roughness: 0.1 });
      const panels = new THREE.Mesh(panelGeo, panelMat);
      issGroup.add(panels);

      scene.add(issGroup);
    }

    // 建立千顆深空星雲背景
    function createStarfield() {
      const starsGeo = new THREE.BufferGeometry();
      const count = 3500;
      const positions = new Float32Array(count * 3);

      for(let i = 0; i < count * 3; i++) {
        positions[i] = (Math.random() - 0.5) * 250;
      }

      starsGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      const starsMat = new THREE.PointsMaterial({ color: 0xffffff, size: 0.18, transparent: true, opacity: 0.85 });
      scene.add(new THREE.Points(starsGeo, starsMat));
    }

    // UI 控制與滑塊事件
    function setupUIEvents() {
      // 地球自轉速度
      document.getElementById('rotation-speed').addEventListener('input', (e) => {
        document.getElementById('speed-val').innerText = `${parseFloat(e.target.value).toFixed(1)}x`;
      });

      // 太陽時間 / 晝夜移動
      document.getElementById('solar-time').addEventListener('input', (e) => {
        const hour = parseFloat(e.target.value);
        const angle = (hour / 24) * Math.PI * 2 - Math.PI / 2;
        sunLight.position.x = Math.cos(angle) * 6;
        sunLight.position.z = Math.sin(angle) * 6;

        const displayHour = Math.floor(hour).toString().padStart(2, '0');
        const displayMin = Math.floor((hour % 1) * 60).toString().padStart(2, '0');
        document.getElementById('sun-time-text').innerText = `${displayHour}:${displayMin}`;
      });

      // 開關控制
      document.getElementById('toggle-clouds').addEventListener('change', (e) => clouds.visible = e.target.checked);
      document.getElementById('toggle-atmosphere').addEventListener('change', (e) => atmosphere.visible = e.target.checked);
    }

    // 鎖定 / 解鎖 ISS 太空站視角
    function toggleLockISS() {
      isLockedISS = !isLockedISS;
      const btnText = document.getElementById('iss-btn-text');
      const btn = document.getElementById('btn-lock-iss');

      if (isLockedISS) {
        btnText.innerText = "解鎖自由視野";
        btn.classList.add('bg-cyan-500/30', 'border-cyan-400');
      } else {
        btnText.innerText = "鎖定 ISS 太空站視角";
        btn.classList.remove('bg-cyan-500/30', 'border-cyan-400');
        gsap.to(camera.position, { x: 0, y: 0, z: 4.5, duration: 1.5 });
        controls.target.set(0, 0, 0);
      }
    }

    // 地標飛躍 (Fly-To GPS Location with GSAP)
    function flyToLocation(lat, lon, altitude = 2.2) {
      if (isLockedISS) toggleLockISS(); // 如果鎖定 ISS，先自動解鎖

      const phi = (90 - lat) * (Math.PI / 180);
      const theta = (lon + 180) * (Math.PI / 180);

      const targetX = -(altitude * Math.sin(phi) * Math.cos(theta));
      const targetY = altitude * Math.cos(phi);
      const targetZ = altitude * Math.sin(phi) * Math.sin(theta);

      // 用 GSAP 做出流暢動畫鏡頭飛躍
      gsap.to(camera.position, {
        x: targetX,
        y: targetY,
        z: targetZ,
        duration: 2.2,
        ease: "power2.inOut",
        onUpdate: () => controls.update()
      });
    }

    // 切換思考卡片
    function nextQuote() {
      currentQuoteIndex = (currentQuoteIndex + 1) % quotes.length;
      document.getElementById('quote-text').innerText = quotes[currentQuoteIndex].text;
      document.getElementById('quote-author').innerText = quotes[currentQuoteIndex].author;
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    // 動態繪製與計時器迴圈
    function animate() {
      requestAnimationFrame(animate);

      const speedFactor = parseFloat(document.getElementById('rotation-speed').value);

      // 地球與雲層自轉
      if(earth) earth.rotation.y += 0.0008 * speedFactor;
      if(clouds) clouds.rotation.y += 0.0012 * speedFactor;

      // ISS 3D 太空站軌道計算 (沿著傾斜圓環旋轉)
      issOrbitAngle += 0.005;
      const radius = 1.44;
      const issX = Math.cos(issOrbitAngle) * radius;
      const issY = Math.sin(issOrbitAngle) * radius * Math.sin(Math.PI / 3);
      const issZ = Math.sin(issOrbitAngle) * radius * Math.cos(Math.PI / 3);

      if (issGroup) {
        issGroup.position.set(issX, issY, issZ);
        issGroup.rotation.y += 0.01;

        // 如果開啟「鎖定 ISS 視角」，鏡頭自動緊跟太空站位置
        if (isLockedISS) {
          camera.position.set(issX * 1.5, issY * 1.5 + 0.2, issZ * 1.5);
          controls.target.set(issX, issY, issZ);
        }
      }

      controls.update();

      // 即時軌道高度計算與更新
      const dist = camera.position.distanceTo(scene.position);
      document.getElementById('telemetry-alt').innerText = `${Math.round(dist * 220 + 100)} km`;

      renderer.render(scene, camera);
    }

    window.onload = init;
  </script>
</body>
</html>
