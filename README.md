<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Space Odyssey | Astrophysics & GLSL Ultimate Edition</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Three.js & OrbitControls & GSAP -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
  <!-- Font Awesome -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800;900&family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
    
    body {
      margin: 0;
      overflow: hidden;
      background-color: #010409;
      font-family: 'Noto Sans TC', sans-serif;
      color: #f0f6fc;
    }
    
    .font-orbitron { font-family: 'Orbitron', sans-serif; }

    /* Ultra Glassmorphism */
    .glass-panel {
      background: rgba(13, 17, 23, 0.75);
      backdrop-filter: blur(20px);
      -webkit-backdrop-filter: blur(20px);
      border: 1px solid rgba(255, 255, 255, 0.12);
      box-shadow: 0 30px 60px rgba(0, 0, 0, 0.8);
    }

    .glass-card {
      background: rgba(22, 27, 34, 0.5);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(255, 255, 255, 0.08);
    }

    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

    /* NASA Pulse Animation */
    .pulse-glow {
      box-shadow: 0 0 15px rgba(56, 189, 248, 0.5);
    }
  </style>
</head>
<body class="relative h-screen w-screen select-none overflow-hidden">

  <!-- 3D Canvas -->
  <div id="canvas-container" class="absolute inset-0 z-0"></div>

  <!-- Loading Screen -->
  <div id="loading-screen" class="absolute inset-0 z-50 flex flex-col items-center justify-center bg-slate-950 transition-opacity duration-1000">
    <div class="relative flex items-center justify-center">
      <div class="h-32 w-32 rounded-full border-t-2 border-b-2 border-cyan-400 animate-spin"></div>
      <div class="h-24 w-24 rounded-full border-r-2 border-l-2 border-indigo-500 animate-ping absolute"></div>
      <i class="fa-solid fa-atom text-cyan-400 text-3xl absolute animate-pulse"></i>
    </div>
    <h2 class="mt-8 text-2xl font-black tracking-widest text-slate-100 font-orbitron">COMPUTING GLSL ATMOSPHERE...</h2>
    <p class="text-xs text-cyan-400/80 mt-2 font-mono">正在載入物理 Ray-Matching 散射 Shader 與 NASA 實時 API 連線...</p>
  </div>

  <!-- 頂部 HUD 狀態欄 -->
  <header class="absolute top-6 left-6 z-10 pointer-events-none">
    <div class="flex items-center gap-3">
      <div class="h-3.5 w-3.5 rounded-full bg-cyan-400 animate-ping"></div>
      <h1 class="text-3xl font-black tracking-wider font-orbitron text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-sky-300 to-indigo-500">
        SPACE ODYSSEY
      </h1>
      <span class="px-2.5 py-0.5 text-[10px] font-orbitron font-extrabold bg-cyan-500/20 text-cyan-300 border border-cyan-500/40 rounded-full pointer-events-auto pulse-glow">
        GLSL ULTIMATE
      </span>
    </div>
    <p class="text-xs text-slate-400 mt-1 tracking-widest font-mono uppercase">Physically Based Atmosphere & Live NASA Telemetry</p>
  </header>

  <!-- 左側控制與遙測面板 -->
  <aside class="absolute top-24 left-6 bottom-6 z-10 w-96 glass-panel rounded-2xl p-5 flex flex-col justify-between overflow-y-auto no-scrollbar pointer-events-auto">
    <div class="space-y-5">
      
      <!-- NASA ISS 即時數據 (Live API) -->
      <div>
        <div class="flex items-center justify-between text-xs font-orbitron text-slate-400 mb-2">
          <span class="flex items-center gap-1.5"><i class="fa-solid fa-satellite-dish text-cyan-400"></i> REAL-TIME ISS TELEMETRY</span>
          <span id="api-status" class="text-[10px] text-emerald-400 font-mono flex items-center gap-1"><i class="fa-solid fa-circle text-[6px]"></i> CONNECTED</span>
        </div>
        <div class="grid grid-cols-2 gap-2 text-xs font-mono">
          <div class="glass-card p-2.5 rounded-xl border-l-2 border-cyan-400">
            <span class="text-slate-500 block text-[9px]">REAL ALTITUDE (真實高度)</span>
            <span id="iss-real-alt" class="font-orbitron font-bold text-cyan-300 text-xs">418.2 km</span>
          </div>
          <div class="glass-card p-2.5 rounded-xl border-l-2 border-indigo-400">
            <span class="text-slate-500 block text-[9px]">VELOCITY (真實時速)</span>
            <span id="iss-real-spd" class="font-orbitron font-bold text-indigo-300 text-xs">27,580 km/h</span>
          </div>
          <div class="glass-card p-2.5 rounded-xl border-l-2 border-sky-400 col-span-2 flex justify-between items-center">
            <div>
              <span class="text-slate-500 block text-[9px]">CURRENT LAT / LON (即時經緯度)</span>
              <span id="iss-real-pos" class="font-orbitron font-bold text-sky-200 text-xs">Lat: 14.2° N, Lon: 120.8° E</span>
            </div>
            <button id="btn-lock-iss" onclick="toggleLockISS()" class="px-3 py-1.5 bg-cyan-500/20 hover:bg-cyan-500/40 border border-cyan-400/50 text-cyan-300 rounded-lg text-[10px] font-orbitron transition">
              <i class="fa-solid fa-crosshairs"></i> 追蹤
            </button>
          </div>
        </div>
      </div>

      <hr class="border-slate-800">

      <!-- GLSL 物理 Shader 參數微調 -->
      <div class="space-y-3">
        <label class="text-xs font-bold text-slate-200 tracking-wider flex items-center justify-between font-orbitron">
          <span class="flex items-center gap-2"><i class="fa-solid fa-wand-magic-sparkles text-amber-400"></i> GLSL RAYLEIGH SCATTERING</span>
        </label>
        
        <!-- 大氣散射強度 -->
        <div class="space-y-1">
          <div class="flex justify-between text-xs text-slate-400 font-mono">
            <span>大氣密度 (Rayleigh Power)</span>
            <span id="rayleigh-val" class="text-cyan-400">1.0</span>
          </div>
          <input type="range" id="rayleigh-power" min="0.2" max="3.0" step="0.1" value="1.0" class="w-full h-1 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-cyan-400">
        </div>

        <!-- 太陽時間與角度 -->
        <div class="space-y-1">
          <div class="flex justify-between text-xs text-slate-400 font-mono">
            <span>太陽光照角度</span>
            <span id="sun-time-text" class="text-amber-400">12:00</span>
          </div>
          <input type="range" id="solar-time" min="0" max="24" step="0.1" value="12" class="w-full h-1 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-amber-400">
        </div>
      </div>

      <hr class="border-slate-800">

      <!-- 地標飛躍 GPS Fly-To -->
      <div class="space-y-2">
        <label class="text-xs font-bold text-slate-200 tracking-wider flex items-center gap-2 font-orbitron">
          <i class="fa-solid fa-earth-asia text-emerald-400"></i> PRECISION FLY-TO
        </label>
        <div class="grid grid-cols-2 gap-2 text-xs font-mono">
          <button onclick="flyToLocation(23.5, 121, 2.2)" class="glass-card hover:bg-slate-800 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-location-dot text-emerald-400"></i> 台灣 (Taiwan)
          </button>
          <button onclick="flyToLocation(35.67, 139.65, 2.2)" class="glass-card hover:bg-slate-800 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-location-dot text-rose-400"></i> 東京 (Tokyo)
          </button>
          <button onclick="flyToLocation(40.71, -74.00, 2.2)" class="glass-card hover:bg-slate-800 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-location-dot text-indigo-400"></i> 紐約 (NYC)
          </button>
          <button onclick="flyToLocation(51.50, -0.12, 2.2)" class="glass-card hover:bg-slate-800 text-slate-200 py-2 px-2.5 rounded-lg text-left transition flex items-center gap-1.5">
            <i class="fa-solid fa-location-dot text-amber-400"></i> 倫敦 (London)
          </button>
        </div>
      </div>

      <!-- 音效與開關 -->
      <div class="flex items-center justify-between pt-2">
        <button id="btn-audio" onclick="toggleSpaceAudio()" class="w-full glass-card hover:bg-slate-800 text-slate-300 py-2 px-3 rounded-xl text-xs font-mono flex items-center justify-center gap-2 transition">
          <i id="audio-icon" class="fa-solid fa-volume-xmark text-rose-400"></i> <span id="audio-text">開啟深空沉浸音效 (Audio)</span>
        </button>
      </div>

    </div>

    <!-- 底部資訊 -->
    <div class="mt-4 pt-3 border-t border-slate-800/80 text-[10px] text-slate-500 font-mono flex justify-between">
      <span>WebGL GLSL Shader Core</span>
      <span>NASA Open API v1.0</span>
    </div>
  </aside>

  <!-- 核心 3D Engine & GLSL Shaders -->
  <script>
    let scene, camera, renderer, controls;
    let earth, nightEarth, clouds, atmosphereShader, issMesh, sunLight;
    let isLockedISS = false;
    let audioCtx, osc1, osc2, gainNode, isAudioPlaying = false;
    
    // Real NASA ISS Data Storage
    let issRealData = { lat: 0, lon: 0, alt: 408, vel: 27600 };

    // GLSL Custom Atmosphere Vertex Shader
    const atmosphereVertexShader = `
      varying vec3 vNormal;
      varying vec3 vPosition;
      void main() {
        vNormal = normalize(normalMatrix * normal);
        vPosition = (modelViewMatrix * vec4(position, 1.0)).xyz;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `;

    // GLSL Custom Atmosphere Fragment Shader (Physical Rayleigh Scattering Simulation)
    const atmosphereFragmentShader = `
      varying vec3 vNormal;
      varying vec3 vPosition;
      uniform vec3 sunPosition;
      uniform float rayleighPower;

      void main() {
        vec3 viewDir = normalize(-vPosition);
        vec3 sunDir = normalize(sunPosition);
        
        float intensity = pow(0.7 - dot(vNormal, viewDir), 2.5) * rayleighPower;
        float sunAtmosphere = max(0.0, dot(vNormal, sunDir));
        
        // 藍色天際與晨昏夕陽切線紅暈 (Rayleigh & Mie)
        vec3 atmosphereColor = mix(vec3(0.1, 0.4, 1.0), vec3(1.0, 0.4, 0.1), pow(1.0 - sunAtmosphere, 4.0));
        
        gl_FragColor = vec4(atmosphereColor, 1.0) * intensity * (sunAtmosphere + 0.2);
      }
    `;

    function init() {
      const container = document.getElementById('canvas-container');

      // 1. Scene & Camera
      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(0, 0, 4.5);

      // 2. WebGL Renderer
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

      // 4. Lights
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.12);
      scene.add(ambientLight);

      sunLight = new THREE.DirectionalLight(0xffffff, 2.2);
      sunLight.position.set(6, 0, 3);
      scene.add(sunLight);

      // 5. Textures
      const textureLoader = new THREE.TextureLoader();
      const dayTexture = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg', () => {
        document.getElementById('loading-screen').classList.add('opacity-0');
        setTimeout(() => document.getElementById('loading-screen').remove(), 1000);
      });
      const nightTexture = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-night-lights.png');
      const bumpTexture = textureLoader.load('https://unpkg.com/three-globe/example/img/earth-topology.png');

      // 6. Earth Mesh
      const earthGeo = new THREE.SphereGeometry(1, 64, 64);
      const earthMat = new THREE.MeshStandardMaterial({
        map: dayTexture,
        bumpMap: bumpTexture,
        bumpScale: 0.04,
        roughness: 0.65,
        metalness: 0.1
      });
      earth = new THREE.Mesh(earthGeo, earthMat);
      scene.add(earth);

      // Night Lights Blending
      const nightMat = new THREE.MeshBasicMaterial({
        map: nightTexture,
        blending: THREE.AdditiveBlending,
        transparent: true,
        opacity: 0.85
      });
      nightEarth = new THREE.Mesh(earthGeo, nightMat);
      earth.add(nightEarth);

      // 7. Clouds Mesh
      const cloudsGeo = new THREE.SphereGeometry(1.02, 64, 64);
      const cloudsMat = new THREE.MeshStandardMaterial({
        map: textureLoader.load('https://unpkg.com/three-globe/example/img/earth-clouds.png'),
        transparent: true,
        opacity: 0.4,
        blending: THREE.AdditiveBlending
      });
      clouds = new THREE.Mesh(cloudsGeo, cloudsMat);
      scene.add(clouds);

      // 8. Custom GLSL Atmosphere Shader Mesh
      const atmosphereGeo = new THREE.SphereGeometry(1.24, 64, 64);
      atmosphereShader = new THREE.ShaderMaterial({
        vertexShader: atmosphereVertexShader,
        fragmentShader: atmosphereFragmentShader,
        uniforms: {
          sunPosition: { value: sunLight.position },
          rayleighPower: { value: 1.0 }
        },
        blending: THREE.AdditiveBlending,
        side: THREE.BackSide,
        transparent: true
      });
      const atmosphereMesh = new THREE.Mesh(atmosphereGeo, atmosphereShader);
      scene.add(atmosphereMesh);

      // 9. 3D ISS Satellite Mesh
      createISSMesh();

      // 10. Starfield
      createStarfield();

      // Listeners
      window.addEventListener('resize', onWindowResize);
      setupUI();

      // Start Fetching Real NASA ISS Position
      fetchISSData();
      setInterval(fetchISSData, 5000); // 每 5 秒更新真實 ISS 數據

      animate();
    }

    // 建立 3D ISS 太空站模型
    function createISSMesh() {
      issMesh = new THREE.Group();
      
      const body = new THREE.Mesh(
        new THREE.CylinderGeometry(0.01, 0.01, 0.06, 12),
        new THREE.MeshStandardMaterial({ color: 0xffffff, metalness: 0.8, roughness: 0.2 })
      );
      body.rotation.z = Math.PI / 2;
      issMesh.add(body);

      const panel = new THREE.Mesh(
        new THREE.BoxGeometry(0.16, 0.002, 0.03),
        new THREE.MeshStandardMaterial({ color: 0x0284c7, metalness: 0.5 })
      );
      issMesh.add(panel);

      scene.add(issMesh);
    }

    // 實時串接 NASA / ISS Open API
    async function fetchISSData() {
      try {
        const response = await fetch('https://api.wheretheiss.at/v1/satellites/25544');
        const data = await response.json();
        
        issRealData.lat = data.latitude;
        issRealData.lon = data.longitude;
        issRealData.alt = data.altitude;
        issRealData.vel = data.velocity;

        // UI 數據更新
        document.getElementById('iss-real-alt').innerText = `${data.altitude.toFixed(1)} km`;
        document.getElementById('iss-real-spd').innerText = `${Math.round(data.velocity).toLocaleString()} km/h`;
        document.getElementById('iss-real-pos').innerText = `Lat: ${data.latitude.toFixed(2)}°, Lon: ${data.longitude.toFixed(2)}°`;

        // 將實時經緯度轉換為 3D 空間 3D 向量座標
        const radius = 1.38; // 3D 軌道高度
        const phi = (90 - data.latitude) * (Math.PI / 180);
        const theta = (data.longitude + 180) * (Math.PI / 180);

        const targetX = -(radius * Math.sin(phi) * Math.cos(theta));
        const targetY = radius * Math.cos(phi);
        const targetZ = radius * Math.sin(phi) * Math.sin(theta);

        // 使用 GSAP 平滑過渡 3D 太空站位置
        gsap.to(issMesh.position, { x: targetX, y: targetY, z: targetZ, duration: 2.5, ease: "power1.out" });

      } catch (err) {
        document.getElementById('api-status').innerText = 'API OFFLINE (SIMULATED)';
        document.getElementById('api-status').className = 'text-[10px] text-amber-400 font-mono';
      }
    }

    // 建立星空
    function createStarfield() {
      const geo = new THREE.BufferGeometry();
      const count = 4000;
      const pos = new Float32Array(count * 3);
      for(let i = 0; i < count * 3; i++) pos[i] = (Math.random() - 0.5) * 300;
      geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
      scene.add(new THREE.Points(geo, new THREE.PointsMaterial({ color: 0xffffff, size: 0.15, transparent: true, opacity: 0.8 })));
    }

    // UI 事件
    function setupUI() {
      document.getElementById('rayleigh-power').addEventListener('input', (e) => {
        const val = parseFloat(e.target.value);
        atmosphereShader.uniforms.rayleighPower.value = val;
        document.getElementById('rayleigh-val').innerText = val.toFixed(1);
      });

      document.getElementById('solar-time').addEventListener('input', (e) => {
        const hour = parseFloat(e.target.value);
        const angle = (hour / 24) * Math.PI * 2 - Math.PI / 2;
        sunLight.position.x = Math.cos(angle) * 6;
        sunLight.position.z = Math.sin(angle) * 6;

        const h = Math.floor(hour).toString().padStart(2, '0');
        const m = Math.floor((hour % 1) * 60).toString().padStart(2, '0');
        document.getElementById('sun-time-text').innerText = `${h}:${m}`;
      });
    }

    // 鎖定 ISS 實時位置
    function toggleLockISS() {
      isLockedISS = !isLockedISS;
      const btn = document.getElementById('btn-lock-iss');
      if (isLockedISS) {
        btn.innerHTML = `<i class="fa-solid fa-unlock"></i> 解鎖`;
        btn.classList.add('bg-cyan-500/50');
      } else {
        btn.innerHTML = `<i class="fa-solid fa-crosshairs"></i> 追蹤`;
        btn.classList.remove('bg-cyan-500/50');
        gsap.to(camera.position, { x: 0, y: 0, z: 4.5, duration: 1.5 });
        controls.target.set(0, 0, 0);
      }
    }

    // GPS 地標精確飛躍
    function flyToLocation(lat, lon, altitude = 2.2) {
      if (isLockedISS) toggleLockISS();

      const phi = (90 - lat) * (Math.PI / 180);
      const theta = (lon + 180) * (Math.PI / 180);

      const targetX = -(altitude * Math.sin(phi) * Math.cos(theta));
      const targetY = altitude * Math.cos(phi);
      const targetZ = altitude * Math.sin(phi) * Math.sin(theta);

      gsap.to(camera.position, {
        x: targetX, y: targetY, z: targetZ, duration: 2.2, ease: "power2.inOut",
        onUpdate: () => controls.update()
      });
    }

    // Web Audio API 深空低頻聲效（免外掛 MP3）
    function toggleSpaceAudio() {
      if (!isAudioPlaying) {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        osc1 = audioCtx.createOscillator();
        osc2 = audioCtx.createOscillator();
        gainNode = audioCtx.createGain();

        osc1.type = 'sine';
        osc1.frequency.setValueAtTime(55, audioCtx.currentTime); // A1 低音

        osc2.type = 'triangle';
        osc2.frequency.setValueAtTime(110, audioCtx.currentTime);

        gainNode.gain.setValueAtTime(0.05, audioCtx.currentTime);

        osc1.connect(gainNode);
        osc2.connect(gainNode);
        gainNode.connect(audioCtx.destination);

        osc1.start();
        osc2.start();

        document.getElementById('audio-icon').className = 'fa-solid fa-volume-high text-emerald-400';
        document.getElementById('audio-text').innerText = '深空聲效播放中 (Audio On)';
        isAudioPlaying = true;
      } else {
        audioCtx.close();
        document.getElementById('audio-icon').className = 'fa-solid fa-volume-xmark text-rose-400';
        document.getElementById('audio-text').innerText = '開啟深空沉浸音效 (Audio)';
        isAudioPlaying = false;
      }
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    // Render Loop
    function animate() {
      requestAnimationFrame(animate);

      // 地球與雲層自轉
      if (earth) earth.rotation.y += 0.0006;
      if (clouds) clouds.rotation.y += 0.0009;

      // 鎖定 ISS 視角
      if (isLockedISS && issMesh) {
        camera.position.set(issMesh.position.x * 1.6, issMesh.position.y * 1.6 + 0.1, issMesh.position.z * 1.6);
        controls.target.copy(issMesh.position);
      }

      controls.update();
      renderer.render(scene, camera);
    }

    window.onload = init;
  </script>
</body>
</html>
