<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Space Odyssey | 總觀效應與地球探索</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Three.js & OrbitControls -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
  <!-- Font Awesome -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
    
    body {
      margin: 0;
      overflow: hidden;
      background-color: #030712;
      font-family: 'Noto Sans TC', sans-serif;
      color: #f3f4f6;
    }
    
    .font-orbitron {
      font-family: 'Orbitron', sans-serif;
    }

    /* 玻璃擬態 (Glassmorphism) */
    .glass-panel {
      background: rgba(15, 23, 42, 0.65);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      border: 1px solid rgba(255, 255, 255, 0.12);
      box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
    }

    .glass-card {
      background: rgba(30, 41, 59, 0.4);
      backdrop-filter: blur(8px);
      border: 1px solid rgba(255, 255, 255, 0.08);
    }

    /* 自訂滑塊樣式 */
    input[type=range] {
      accent-color: #38bdf8;
    }

    /* 隱藏捲軸 */
    .no-scrollbar::-webkit-scrollbar {
      display: none;
    }
    .no-scrollbar {
      -ms-overflow-style: none;
      scrollbar-width: none;
    }
  </style>
</head>
<body class="relative h-screen w-screen select-none">

  <!-- 3D Canvas 容器 -->
  <div id="canvas-container" class="absolute inset-0 z-0"></div>

  <!-- Loading 載入畫面 -->
  <div id="loading-screen" class="absolute inset-0 z-50 flex flex-col items-center justify-center bg-slate-950 transition-opacity duration-700">
    <div class="relative flex items-center justify-center">
      <div class="h-24 w-24 rounded-full border-t-2 border-b-2 border-cyan-400 animate-spin"></div>
      <i class="fa-solid font-orbitron text-cyan-400 text-xl absolute">EARTH</i>
    </div>
    <h2 class="mt-6 text-xl font-medium tracking-widest text-slate-200 font-orbitron">INITIALIZING ODYSSEY...</h2>
    <p class="text-sm text-slate-400 mt-2">載入高解析度衛星地圖與大氣 Shader 渲染中...</p>
  </div>

  <!-- 頂部標題與狀態 -->
  <header class="absolute top-6 left-6 z-10 pointer-events-none">
    <div class="flex items-center gap-3">
      <div class="h-3 w-3 rounded-full bg-cyan-400 animate-ping"></div>
      <h1 class="text-2xl font-extrabold tracking-wider font-orbitron text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-sky-300 to-indigo-400">
        SPACE ODYSSEY
      </h1>
    </div>
    <p class="text-xs text-slate-400 mt-1 tracking-widest uppercase">Project Overview Effect / 期中成果展現</p>
  </header>

  <!-- 左側互動控制儀表板 -->
  <aside class="absolute top-24 left-6 bottom-6 z-10 w-80 glass-panel rounded-2xl p-5 flex flex-col justify-between overflow-y-auto no-scrollbar pointer-events-auto">
    
    <div class="space-y-6">
      <!-- 狀態面板 -->
      <div>
        <div class="flex items-center justify-between text-xs text-slate-400 mb-2 font-orbitron">
          <span>ORBITAL TELEMETRY</span>
          <span class="text-cyan-400">LIVE</span>
        </div>
        <div class="grid grid-cols-2 gap-2 text-xs">
          <div class="glass-card p-2.5 rounded-lg">
            <span class="text-slate-400 block text-[10px]">ALTITUDE</span>
            <span id="telemetry-alt" class="font-orbitron font-semibold text-cyan-300 text-sm">400 km</span>
          </div>
          <div class="glass-card p-2.5 rounded-lg">
            <span class="text-slate-400 block text-[10px]">ROTATION SPEED</span>
            <span id="telemetry-speed" class="font-orbitron font-semibold text-cyan-300 text-sm">1,670 km/h</span>
          </div>
        </div>
      </div>

      <hr class="border-slate-800">

      <!-- 光照與場景預設 -->
      <div class="space-y-3">
        <label class="text-xs font-semibold text-slate-300 tracking-wider flex items-center gap-2">
          <i class="fa-solid fa-sun text-amber-400"></i> 光影與氣氛情境
        </label>
        <div class="grid grid-cols-2 gap-2">
          <button onclick="setLighting('day')" class="glass-card hover:bg-slate-700/50 text-slate-200 py-2 px-3 rounded-lg text-xs text-left transition flex items-center gap-2">
            <span class="w-2 h-2 rounded-full bg-amber-400"></span> 標準晝半球
          </button>
          <button onclick="setLighting('sunset')" class="glass-card hover:bg-slate-700/50 text-slate-200 py-2 px-3 rounded-lg text-xs text-left transition flex items-center gap-2">
            <span class="w-2 h-2 rounded-full bg-orange-500"></span> 晨昏夕陽切線
          </button>
          <button onclick="setLighting('night')" class="glass-card hover:bg-slate-700/50 text-slate-200 py-2 px-3 rounded-lg text-xs text-left transition flex items-center gap-2">
            <span class="w-2 h-2 rounded-full bg-indigo-400"></span> 深夜城市光點
          </button>
          <button onclick="setLighting('amber')" class="glass-card hover:bg-slate-700/50 text-slate-200 py-2 px-3 rounded-lg text-xs text-left transition flex items-center gap-2">
            <span class="w-2 h-2 rounded-full bg-amber-700"></span> 暗褐色月影折射
          </button>
        </div>
      </div>

      <!-- 細節微調 -->
      <div class="space-y-4">
        <label class="text-xs font-semibold text-slate-300 tracking-wider flex items-center gap-2">
          <i class="fa-solid fa-sliders text-cyan-400"></i> 視覺參數控制
        </label>
        
        <!-- 地球自轉速度 -->
        <div class="space-y-1">
          <div class="flex justify-between text-xs text-slate-400">
            <span>地球自轉速度</span>
            <span id="speed-val" class="font-orbitron text-cyan-400">1.0x</span>
          </div>
          <input type="range" id="rotation-speed" min="0" max="5" step="0.1" value="1" class="w-full h-1 bg-slate-700 rounded-lg appearance-none cursor-pointer">
        </div>

        <!-- 雲層開關 -->
        <div class="flex items-center justify-between text-xs text-slate-300 py-1">
          <span>動態雲層渲染</span>
          <input type="checkbox" id="toggle-clouds" checked class="w-4 h-4 accent-cyan-400 cursor-pointer">
        </div>

        <!-- 大氣光暈開關 -->
        <div class="flex items-center justify-between text-xs text-slate-300 py-1">
          <span>大氣層 Rayleigh 光暈</span>
          <input type="checkbox" id="toggle-atmosphere" checked class="w-4 h-4 accent-cyan-400 cursor-pointer">
        </div>
      </div>
    </div>

    <!-- 底部說明 -->
    <div class="mt-6 pt-4 border-t border-slate-800 text-[11px] text-slate-400 space-y-1">
      <p><i class="fa-solid fa-mouse text-slate-500"></i> 滑鼠左鍵：360度旋轉視野</p>
      <p><i class="fa-solid fa-arrows-up-down text-slate-500"></i> 滾輪：拉近/拉遠軌道距離</p>
    </div>
  </aside>

  <!-- 右下角太空人感言卡片 -->
  <main class="absolute bottom-6 right-6 z-10 max-w-md glass-panel rounded-2xl p-6 pointer-events-auto transition-all">
    <div class="flex items-center gap-2 text-xs font-orbitron text-cyan-400 mb-2">
      <i class="fa-solid fa-quote-left"></i> OVERVIEW EFFECT REFLECTION
    </div>
    <p id="quote-text" class="text-sm text-slate-200 leading-relaxed font-light">
      「從外太空看地球，你看不到國界、看不出衝突。你只會看到一顆懸浮於虛空之中、極其脆弱卻又無比美麗的藍色家園。」
    </p>
    <div class="mt-4 flex items-center justify-between text-xs text-slate-400">
      <span id="quote-author" class="italic">—— Apollo Astronaut</span>
      <button onclick="nextQuote()" class="text-cyan-400 hover:text-cyan-300 flex items-center gap-1 transition">
        切換思考卡片 <i class="fa-solid fa-arrow-right"></i>
      </button>
    </div>
  </main>

  <!-- Three.js 核心腳本 -->
  <script>
    let scene, camera, renderer, controls;
    let earth, clouds, atmosphere, dirLight, ambientLight;

    // 思考卡片庫
    const quotes = [
      { text: "「從外太空看地球，你看不到國界、看不出衝突。你只會看到一顆懸浮於虛空之中、極其脆弱卻又無比美麗的藍色家園。」", author: "—— 總觀效應 (Overview Effect) 核心感言" },
      { text: "「人類所有的驕傲、紛爭與夢想，其實都發生在那顆懸浮在太陽光束中的『淡藍小點』上。」", author: "—— 卡爾·薩根 (Carl Sagan)" },
      { text: "「在太空裡，大氣層看起來就像一層薄薄的藍色面紗，提醒著我們這顆星球上的生命有多麼依賴它。」", author: "—— 國際太空站 (ISS) 太空人" }
    ];
    let currentQuoteIndex = 0;

    function init() {
      const container = document.getElementById('canvas-container');

      // 1. Scene
      scene = new THREE.Scene();

      // 2. Camera
      camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(0, 0, 4.2);

      // 3. Renderer
      renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      container.appendChild(renderer.domElement);

      // 4. Orbit Controls
      controls = new THREE.OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;
      controls.minDistance = 1.8;
      controls.maxDistance = 8.0;

      // 5. Lighting
      ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
      scene.add(ambientLight);

      dirLight = new THREE.DirectionalLight(0xffffff, 1.8);
      dirLight.position.set(5, 3, 5);
      scene.add(dirLight);

      // 6. Texture Loader
      const textureLoader = new THREE.TextureLoader();
      
      // 使用三方高解析度靜態圖（包含備用 Base64 / Unsplash 處理）
      const earthMap = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg', () => {
        document.getElementById('loading-screen').classList.add('opacity-0');
        setTimeout(() => document.getElementById('loading-screen').remove(), 700);
      });
      const bumpMap = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-topology.png');

      // 7. Earth Mesh
      const earthGeo = new THREE.SphereGeometry(1, 64, 64);
      const earthMat = new THREE.MeshStandardMaterial({
        map: earthMap,
        bumpMap: bumpMap,
        bumpScale: 0.04,
        roughness: 0.6,
        metalness: 0.1
      });
      earth = new THREE.Mesh(earthGeo, earthMat);
      scene.add(earth);

      // 8. Clouds Mesh
      const cloudsGeo = new THREE.SphereGeometry(1.015, 64, 64);
      const cloudsMat = new THREE.MeshStandardMaterial({
        map: textureLoader.load('https://unpkg.com/three-globe/example/img/earth-clouds.png'),
        transparent: true,
        opacity: 0.4,
        blending: THREE.AdditiveBlending
      });
      clouds = new THREE.Mesh(cloudsGeo, cloudsMat);
      scene.add(clouds);

      // 9. Custom Atmosphere Shader Glow
      const atmosphereGeo = new THREE.SphereGeometry(1.2, 64, 64);
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
            float intensity = pow(0.6 - dot(vNormal, vec3(0, 0, 1.0)), 2.0);
            gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
          }
        `,
        blending: THREE.AdditiveBlending,
        side: THREE.BackSide,
        transparent: true
      });
      atmosphere = new THREE.Mesh(atmosphereGeo, atmosphereMat);
      scene.add(atmosphere);

      // 10. Starfield Background
      createStarfield();

      // Listeners
      window.addEventListener('resize', onWindowResize);
      setupControls();

      // Animate Loop
      animate();
    }

    // 創建星空背景
    function createStarfield() {
      const starsGeo = new THREE.BufferGeometry();
      const count = 3000;
      const positions = new Float32Array(count * 3);

      for(let i = 0; i < count * 3; i++) {
        positions[i] = (Math.random() - 0.5) * 200;
      }

      starsGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      const starsMat = new THREE.PointsMaterial({
        color: 0xffffff,
        size: 0.15,
        transparent: true,
        opacity: 0.8
      });
      const starField = new THREE.Points(starsGeo, starsMat);
      scene.add(starField);
    }

    // UI 事件綁定
    function setupControls() {
      const speedInput = document.getElementById('rotation-speed');
      const speedVal = document.getElementById('speed-val');
      speedInput.addEventListener('input', (e) => {
        speedVal.innerText = `${parseFloat(e.target.value).toFixed(1)}x`;
      });

      document.getElementById('toggle-clouds').addEventListener('change', (e) => {
        clouds.visible = e.target.checked;
      });

      document.getElementById('toggle-atmosphere').addEventListener('change', (e) => {
        atmosphere.visible = e.target.checked;
      });
    }

    // 光影預設切換
    function setLighting(type) {
      if(type === 'day') {
        dirLight.position.set(5, 3, 5);
        dirLight.color.setHex(0xffffff);
        ambientLight.intensity = 0.35;
      } else if(type === 'sunset') {
        dirLight.position.set(-5, 0.5, 2);
        dirLight.color.setHex(0xffaa55);
        ambientLight.intensity = 0.15;
      } else if(type === 'night') {
        dirLight.position.set(-5, -3, -5);
        dirLight.color.setHex(0x6688ff);
        ambientLight.intensity = 0.05;
      } else if(type === 'amber') {
        dirLight.position.set(3, 1, 4);
        dirLight.color.setHex(0xb85d19);
        ambientLight.intensity = 0.1;
      }
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

    // 動態渲染迴圈
    function animate() {
      requestAnimationFrame(animate);

      const speedFactor = parseFloat(document.getElementById('rotation-speed').value);
      
      // 地球與雲層旋轉
      if(earth) earth.rotation.y += 0.001 * speedFactor;
      if(clouds) clouds.rotation.y += 0.0014 * speedFactor;

      controls.update();

      // 即時數據更新 (動態模擬軌道高度)
      const dist = camera.position.distanceTo(scene.position);
      document.getElementById('telemetry-alt').innerText = `${Math.round(dist * 180 + 200)} km`;

      renderer.render(scene, camera);
    }

    window.onload = init;
  </script>
</body>
</html>
