<!DOCTYPE html>
<html lang="zh-TW" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Globe & Space Explorer | 總觀效應模擬器</title>
    
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        space: {
                            900: '#05070B',
                            800: '#0B0F19',
                            700: '#111827',
                            glass: 'rgba(15, 23, 42, 0.65)'
                        },
                        accent: {
                            cyan: '#38bdf8',
                            blue: '#3b82f6',
                            gold: '#f59e0b',
                            sunset: '#f97316'
                        }
                    },
                    fontFamily: {
                        sans: ['Inter', '-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'Roboto', 'sans-serif']
                    }
                }
            }
        }
    </script>

    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Three.js for 3D Earth rendering -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- OrbitControls -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <!-- Tone.js for relaxing cosmic ambient sound generator -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>

    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #030508;
            font-family: 'Inter', sans-serif;
            user-select: none;
        }

        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        /* Glassmorphism Styling */
        .glass-panel {
            background: rgba(11, 15, 25, 0.7);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.08);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.5);
        }

        .glass-button {
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            transition: all 0.3s ease;
        }
        .glass-button:hover {
            background: rgba(56, 189, 248, 0.15);
            border-color: rgba(56, 189, 248, 0.4);
            transform: translateY(-1px);
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 4px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.2);
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(56, 189, 248, 0.3);
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(56, 189, 248, 0.6);
        }

        /* Ambient Glow animation */
        .glow-cyan {
            box-shadow: 0 0 25px rgba(56, 189, 248, 0.25);
        }
        
        .glow-text {
            text-shadow: 0 0 10px rgba(56, 189, 248, 0.5);
        }

        /* Hide elements gracefully */
        .fade-exit {
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.5s ease;
        }
    </style>
</head>
<body class="text-slate-200 antialiased h-screen w-screen overflow-hidden">

    <!-- 3D WebGL Canvas Container -->
    <div id="canvas-container"></div>

    <!-- UI Overlay Layer -->
    <header class="absolute top-0 left-0 w-full z-10 p-4 md:p-6 flex justify-between items-center pointer-events-none">
        <div class="flex items-center space-x-3 pointer-events-auto">
            <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-sky-500 to-indigo-600 flex items-center justify-center shadow-lg shadow-sky-500/20 border border-white/20">
                <i class="fa-solid font-bold fa-globe text-white text-lg"></i>
            </div>
            <div>
                <h1 class="font-bold text-lg md:text-xl tracking-wider text-white glow-text flex items-center gap-2">
                    OVERVIEW <span class="text-xs px-2 py-0.5 rounded-full bg-sky-500/20 text-sky-400 border border-sky-500/30">3D Explorer</span>
                </h1>
                <p class="text-xs text-slate-400">總觀效應 · 俯瞰藍色星球</p>
            </div>
        </div>

        <!-- Header Actions -->
        <div class="flex items-center space-x-3 pointer-events-auto">
            <!-- Ambient Music Toggle -->
            <button id="btn-audio" class="glass-button px-4 py-2 rounded-xl text-xs md:text-sm flex items-center gap-2 text-slate-300 hover:text-white">
                <i class="fa-solid fa-volume-xmark text-slate-400" id="audio-icon"></i>
                <span id="audio-text" class="hidden md:inline">太空冥想音效</span>
            </button>
            
            <!-- Information Modal Toggle -->
            <button id="btn-info" class="glass-button w-10 h-10 rounded-xl flex items-center justify-center text-slate-300 hover:text-white">
                <i class="fa-solid fa-circle-info text-base"></i>
            </button>
        </div>
    </header>

    <!-- Floating HUD Left Panel: View Controls -->
    <aside id="left-panel" class="absolute top-24 left-4 md:left-6 z-10 w-72 md:w-80 glass-panel rounded-2xl p-5 space-y-5 transition-all duration-300">
        <div class="flex justify-between items-center border-b border-white/10 pb-3">
            <h2 class="text-sm font-semibold tracking-wide text-sky-400 flex items-center gap-2">
                <i class="fa-solid fa-sliders"></i> 光照與視覺設定
            </h2>
            <button id="toggle-panel-btn" class="text-slate-400 hover:text-white text-xs">
                <i class="fa-solid fa-chevron-left"></i>
            </button>
        </div>

        <!-- Presets: Sunset, Day, Night, Blood Moon -->
        <div class="space-y-2">
            <label class="text-xs text-slate-400 font-medium">光影情境預設</label>
            <div class="grid grid-cols-2 gap-2">
                <button data-preset="day" class="preset-btn active glass-button p-2.5 rounded-xl text-left flex items-center gap-2 border-sky-500/50 bg-sky-500/10">
                    <i class="fa-solid fa-sun text-amber-400 text-sm"></i>
                    <span class="text-xs font-medium">清晰晝半球</span>
                </button>
                <button data-preset="sunset" class="preset-btn glass-button p-2.5 rounded-xl text-left flex items-center gap-2">
                    <i class="fa-solid fa-cloud-sun text-orange-400 text-sm"></i>
                    <span class="text-xs font-medium">暮光夕陽</span>
                </button>
                <button data-preset="night" class="preset-btn glass-button p-2.5 rounded-xl text-left flex items-center gap-2">
                    <i class="fa-solid fa-moon text-indigo-400 text-sm"></i>
                    <span class="text-xs font-medium">深夜城市光點</span>
                </button>
                <button data-preset="bloodmoon" class="preset-btn glass-button p-2.5 rounded-xl text-left flex items-center gap-2">
                    <i class="fa-solid fa-circle text-rose-500 text-sm"></i>
                    <span class="text-xs font-medium">奇幻暗褐光線</span>
                </button>
            </div>
        </div>

        <!-- Sliders -->
        <div class="space-y-4 pt-1">
            <!-- Atmosphere Glow Intensity -->
            <div class="space-y-1.5">
                <div class="flex justify-between text-xs">
                    <span class="text-slate-300">大氣層光暈強度</span>
                    <span id="glow-val" class="text-sky-400 font-mono">1.2</span>
                </div>
                <input type="range" id="slider-glow" min="0" max="3" step="0.1" value="1.2" 
                       class="w-full h-1.5 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-sky-400">
            </div>

            <!-- Rotation Speed -->
            <div class="space-y-1.5">
                <div class="flex justify-between text-xs">
                    <span class="text-slate-300">地球自轉速度</span>
                    <span id="speed-val" class="text-sky-400 font-mono">1.0</span>
                </div>
                <input type="range" id="slider-speed" min="0" max="3" step="0.1" value="1.0" 
                       class="w-full h-1.5 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-sky-400">
            </div>

            <!-- Cloud Layer Visibility -->
            <div class="flex items-center justify-between text-xs pt-1">
                <span class="text-slate-300">雲層大氣流動</span>
                <label class="relative inline-flex items-center cursor-pointer">
                    <input type="checkbox" id="toggle-clouds" checked class="sr-only peer">
                    <div class="w-9 h-5 bg-slate-800 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-sky-500"></div>
                </label>
            </div>
        </div>

        <!-- Camera Angle Presets -->
        <div class="pt-2 border-t border-white/10 space-y-2">
            <label class="text-xs text-slate-400 font-medium">觀測軌道視角</label>
            <div class="flex gap-2">
                <button id="cam-orbit" class="flex-1 glass-button py-2 rounded-xl text-xs flex items-center justify-center gap-1.5">
                    <i class="fa-solid fa-satellite text-sky-400"></i> 近地軌道
                </button>
                <button id="cam-deep" class="flex-1 glass-button py-2 rounded-xl text-xs flex items-center justify-center gap-1.5">
                    <i class="fa-solid fa-globe text-indigo-400"></i> 深空全景
                </button>
            </div>
        </div>
    </aside>

    <!-- Overview Effect Quote/Reflection Card Bottom Bar -->
    <div class="absolute bottom-6 left-1/2 -translate-x-1/2 z-10 w-[90%] max-w-xl glass-panel rounded-2xl p-4 md:p-5 text-center pointer-events-auto">
        <div class="flex justify-between items-center mb-2">
            <span class="text-[10px] tracking-widest text-sky-400 font-mono uppercase"><i class="fa-solid fa-brain mr-1"></i> Space Reflection · 總觀心境</span>
            <button id="btn-next-quote" class="text-xs text-slate-400 hover:text-sky-400 transition-colors flex items-center gap-1">
                換一張卡片 <i class="fa-solid fa-rotate-right"></i>
            </button>
        </div>
        <p id="quote-text" class="text-xs md:text-sm text-slate-200 leading-relaxed font-light italic">
            「當你從外太空俯瞰地球，所有的界線與國界都不復存在。你所關心的一切、所有愛與紛爭，都發生在這顆懸浮於漆黑宇宙中的藍色微塵之上。」
        </p>
        <span id="quote-author" class="block text-[11px] text-slate-400 mt-2 font-mono">— 艾德加·米切爾 (Edgar Mitchell), 阿波羅14號太空人</span>
    </div>

    <!-- Info Modal -->
    <div id="info-modal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/70 backdrop-blur-md opacity-0 pointer-events-none transition-opacity duration-300">
        <div class="glass-panel max-w-lg w-full rounded-2xl p-6 md:p-8 space-y-5 border border-sky-500/30">
            <div class="flex justify-between items-start">
                <div class="flex items-center gap-3">
                    <div class="p-3 bg-sky-500/20 text-sky-400 rounded-xl">
                        <i class="fa-solid fa-user-astronaut text-2xl"></i>
                    </div>
                    <div>
                        <h3 class="text-lg font-bold text-white">關於總觀效應 (Overview Effect)</h3>
                        <p class="text-xs text-slate-400">認知轉變與心靈震撼</p>
                    </div>
                </div>
                <button id="close-modal" class="text-slate-400 hover:text-white p-1">
                    <i class="fa-solid fa-xmark text-xl"></i>
                </button>
            </div>

            <div class="text-xs md:text-sm text-slate-300 space-y-3 leading-relaxed">
                <p>
                    <strong>「總觀效應」</strong> 是指太空人在從太空親眼俯瞰地球時所經歷的一種認知與心理轉變。
                </p>
                <p>
                    當人們從數百公里外的軌道遠眺，看到保護著所有生命的薄薄大氣層，以及無邊無際的黑寂宇宙時，許多人會感受到極大的震撼、謙卑與超脫感——原本生活中的紛擾、焦慮與國界，在宏觀的宇宙面前都顯得無比微小。
                </p>
                <div class="p-3 rounded-xl bg-sky-500/10 border border-sky-500/20 text-sky-300 text-xs">
                    <i class="fa-solid fa-lightbulb mr-1"></i> 提示：您可以按住滑鼠左 your 鍵旋轉角度，滑鼠滾輪縮放距離，並在左側選單嘗試切換不同的光束與大氣氛圍！
                </div>
            </div>

            <button id="modal-ok-btn" class="w-full py-2.5 rounded-xl bg-sky-500 hover:bg-sky-400 text-slate-950 font-semibold text-sm transition-colors">
                開始探索太空體驗
            </button>
        </div>
    </div>

    <!-- JavaScript Logic Section -->
    <script>
        // --- 1. Quotes Collection ---
        const quotes = [
            {
                text: "「當你從外太空俯瞰地球，所有的界線與國界都不復存在。你所關心的一切、所有愛與紛爭，都發生在這顆懸浮於漆黑宇宙中的藍色微塵之上。」",
                author: "— 艾德加·米切爾 (Edgar Mitchell), 阿波羅14號太空人"
            },
            {
                text: "「站在月球上看地球，它就像一顆光芒四射的藍色寶石，孤零零地懸掛在無邊無際的黑夜中。那一刻你才會明白，我們是一個整體。」",
                author: "— 詹姆斯·歐文 (James Irwin), 阿波羅15號太空人"
            },
            {
                text: "「宇宙是一片無垠的海洋，而地球只是這片汪洋中的一艘小船。保護它，是我們唯一的選擇。」",
                author: "— 卡爾·薩根 (Carl Sagan), 天文學家"
            },
            {
                text: "「從外太空看，地球沒有政治邊界。你只會看到一個生生不息、由大氣層溫柔包裹著的生命共同體。」",
                author: "— 阿努什·安薩里 (Anousheh Ansari), 首位女性太空遊客"
            },
            {
                text: "「那種宏大與渺小的強烈對比，會徹底刷新你的世界觀。那些困擾你許久的日常生活煩惱，瞬間都變得微不足道。」",
                author: "— 總觀效應心理學研究總結"
            }
        ];

        let currentQuoteIdx = 0;

        // --- 2. Three.js Engine Setup ---
        let scene, camera, renderer, controls;
        let earthMesh, cloudsMesh, atmosphereMesh, starsParticles;
        let dirLight, ambientLight;

        // Interactive parameters
        const params = {
            rotationSpeed: 0.001,
            glowIntensity: 1.2,
            preset: 'day'
        };

        function initThree() {
            const container = document.getElementById('canvas-container');

            // Scene
            scene = new THREE.Scene();

            // Camera
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 0, 4.2);

            // Renderer
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            renderer.toneMappingExposure = 1.2;
            container.appendChild(renderer.domElement);

            // Controls
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            controls.minDistance = 2.2;
            controls.maxDistance = 12.0;
            controls.rotateSpeed = 0.8;

            // Lights
            ambientLight = new THREE.AmbientLight(0x333333, 0.8);
            scene.add(ambientLight);

            dirLight = new THREE.DirectionalLight(0xffffff, 2.0);
            dirLight.position.set(5, 3, 5);
            scene.add(dirLight);

            // Create Procedural Earth & Stars
            createStarfield();
            createEarth();
            createAtmosphere();

            // Window Resize Listener
            window.addEventListener('resize', onWindowResize);

            // Animation Loop
            animate();
        }

        // Procedural Starfield Particle System
        function createStarfield() {
            const starsCount = 2500;
            const geometry = new THREE.BufferGeometry();
            const positions = new Float32Array(starsCount * 3);
            const colors = new Float32Array(starsCount * 3);

            for (let i = 0; i < starsCount * 3; i += 3) {
                // Distribute stars on a wide sphere
                const u = Math.random();
                const v = Math.random();
                const theta = u * 2.0 * Math.PI;
                const phi = Math.acos(2.0 * v - 1.0);
                const r = 80 + Math.random() * 40;

                positions[i] = r * Math.sin(phi) * Math.cos(theta);
                positions[i + 1] = r * Math.sin(phi) * Math.sin(theta);
                positions[i + 2] = r * Math.cos(phi);

                // Slight color variations (white, bluish, yellow)
                const tint = Math.random();
                if (tint > 0.8) {
                    colors[i] = 0.7; colors[i+1] = 0.8; colors[i+2] = 1.0;
                } else if (tint > 0.6) {
                    colors[i] = 1.0; colors[i+1] = 0.9; colors[i+2] = 0.7;
                } else {
                    colors[i] = 1.0; colors[i+1] = 1.0; colors[i+2] = 1.0;
                }
            }

            geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

            const material = new THREE.PointsMaterial({
                size: 0.8,
                vertexColors: true,
                transparent: true,
                opacity: 0.85
            });

            starsParticles = new THREE.Points(geometry, material);
            scene.add(starsParticles);
        }

        // Generate Dynamic Procedural Canvas Textures for Earth
        function generateEarthTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 1024;
            canvas.height = 512;
            const ctx = canvas.getContext('2d');

            // Deep ocean background
            ctx.fillStyle = '#081c38';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw noise landmasses
            ctx.fillStyle = '#1e3d29';
            for (let i = 0; i < 1200; i++) {
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height;
                const radius = 10 + Math.random() * 50;
                ctx.beginPath();
                ctx.arc(x, y, radius, 0, Math.PI * 2);
                ctx.fill();
            }

            // Deserts / Terrain details
            ctx.fillStyle = '#4a422d';
            for (let i = 0; i < 400; i++) {
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height * 0.6 + canvas.height * 0.2;
                const radius = 5 + Math.random() * 25;
                ctx.beginPath();
                ctx.arc(x, y, radius, 0, Math.PI * 2);
                ctx.fill();
            }

            return new THREE.CanvasTexture(canvas);
        }

        function generateCloudsTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 1024;
            canvas.height = 512;
            const ctx = canvas.getContext('2d');

            ctx.fillStyle = 'rgba(0, 0, 0, 0)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
            for (let i = 0; i < 800; i++) {
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height;
                const radius = 15 + Math.random() * 45;
                ctx.beginPath();
                ctx.arc(x, y, radius, 0, Math.PI * 2);
                ctx.fill();
            }

            return new THREE.CanvasTexture(canvas);
        }

        // Create Earth Sphere & Cloud Sphere
        function createEarth() {
            const geometry = new THREE.SphereGeometry(1.5, 64, 64);

            // Earth Material
            const earthTexture = generateEarthTexture();
            const earthMaterial = new THREE.MeshStandardMaterial({
                map: earthTexture,
                roughness: 0.6,
                metalness: 0.1
            });

            earthMesh = new THREE.Mesh(geometry, earthMaterial);
            scene.add(earthMesh);

            // Clouds Mesh Layer
            const cloudGeometry = new THREE.SphereGeometry(1.52, 64, 64);
            const cloudTexture = generateCloudsTexture();
            const cloudMaterial = new THREE.MeshStandardMaterial({
                map: cloudTexture,
                transparent: true,
                opacity: 0.4,
                blending: THREE.AdditiveBlending
            });

            cloudsMesh = new THREE.Mesh(cloudGeometry, cloudMaterial);
            scene.add(cloudsMesh);
        }

        // Custom Atmospheric Outer Glow Shader
        function createAtmosphere() {
            const atmosphereGeometry = new THREE.SphereGeometry(1.63, 64, 64);

            const customShader = {
                vertexShader: `
                    varying vec3 vNormal;
                    void main() {
                        vNormal = normalize(normalMatrix * normal);
                        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                    }
                `,
                fragmentShader: `
                    varying vec3 vNormal;
                    uniform vec3 color;
                    uniform float intensity;
                    void main() {
                        float atmosphere = pow(0.65 - dot(vNormal, vec3(0, 0, 1.0)), 2.0);
                        gl_FragColor = vec4(color, atmosphere * intensity);
                    }
                `
            };

            const atmosphereMaterial = new THREE.ShaderMaterial({
                vertexShader: customShader.vertexShader,
                fragmentShader: customShader.fragmentShader,
                uniforms: {
                    color: { value: new THREE.Color(0x38bdf8) },
                    intensity: { value: params.glowIntensity }
                },
                blending: THREE.AdditiveBlending,
                side: THREE.BackSide,
                transparent: true
            });

            atmosphereMesh = new THREE.Mesh(atmosphereGeometry, atmosphereMaterial);
            scene.add(atmosphereMesh);
        }

        // Resize Event Handler
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Main Render Loop
        function animate() {
            requestAnimationFrame(animate);

            // Earth Self-Rotation
            if (earthMesh) earthMesh.rotation.y += 0.001 * params.rotationSpeed;
            if (cloudsMesh) cloudsMesh.rotation.y += 0.0013 * params.rotationSpeed;
            if (starsParticles) starsParticles.rotation.y -= 0.0001;

            controls.update();
            renderer.render(scene, camera);
        }

        // --- 3. Ambient Cosmic Audio Generator (Tone.js) ---
        let audioSynth = null;
        let isAudioPlaying = false;

        function initAudio() {
            if (audioSynth) return;

            // Soft cosmic synth pad sound
            audioSynth = new Tone.PolySynth(Tone.Synth, {
                oscillator: { type: "sine" },
                envelope: { attack: 3, decay: 2, sustain: 0.8, release: 5 }
            }).toDestination();

            audioSynth.volume.value = -18; // Low ambient background volume

            // Ambient Chord Loop (Space Ambient Chords)
            const chords = [
                ["C3", "G3", "B3", "E4"],
                ["A2", "E3", "G3", "C4"],
                ["F2", "C3", "E3", "A3"],
                ["G2", "D3", "F#3", "B3"]
            ];

            let chordIndex = 0;
            Tone.Transport.scheduleRepeat((time) => {
                audioSynth.releaseAll(time);
                audioSynth.triggerAttack(chords[chordIndex], time);
                chordIndex = (chordIndex + 1) % chords.length;
            }, "8n");
        }

        function toggleAudio() {
            if (!isAudioPlaying) {
                Tone.start();
                initAudio();
                Tone.Transport.start();
                isAudioPlaying = true;
                document.getElementById('audio-icon').className = 'fa-solid fa-volume-high text-sky-400';
                document.getElementById('audio-text').textContent = '聲音播放中';
            } else {
                Tone.Transport.stop();
                if (audioSynth) audioSynth.releaseAll();
                isAudioPlaying = false;
                document.getElementById('audio-icon').className = 'fa-solid fa-volume-xmark text-slate-400';
                document.getElementById('audio-text').textContent = '太空冥想音效';
            }
        }

        // --- 4. Interactivity & Presets Event Listeners ---
        window.onload = function() {
            initThree();

            // Preset Buttons Logic
            document.querySelectorAll('.preset-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    document.querySelectorAll('.preset-btn').forEach(b => {
                        b.classList.remove('active', 'border-sky-500/50', 'bg-sky-500/10');
                    });
                    const currentBtn = e.currentTarget;
                    currentBtn.classList.add('active', 'border-sky-500/50', 'bg-sky-500/10');

                    const preset = currentBtn.dataset.preset;
                    applyPreset(preset);
                });
            });

            // Sliders
            const glowSlider = document.getElementById('slider-glow');
            glowSlider.addEventListener('input', (e) => {
                const val = parseFloat(e.target.value);
                document.getElementById('glow-val').textContent = val.toFixed(1);
                if (atmosphereMesh) {
                    atmosphereMesh.material.uniforms.intensity.value = val;
                }
            });

            const speedSlider = document.getElementById('slider-speed');
            speedSlider.addEventListener('input', (e) => {
                const val = parseFloat(e.target.value);
                document.getElementById('speed-val').textContent = val.toFixed(1);
                params.rotationSpeed = val;
            });

            // Toggle Clouds
            document.getElementById('toggle-clouds').addEventListener('change', (e) => {
                if (cloudsMesh) {
                    cloudsMesh.visible = e.target.checked;
                }
            });

            // Camera View Switchers
            document.getElementById('cam-orbit').addEventListener('click', () => {
                gsapCameraMove(2.8);
            });
            document.getElementById('cam-deep').addEventListener('click', () => {
                gsapCameraMove(6.5);
            });

            // Quote Cycle Button
            document.getElementById('btn-next-quote').addEventListener('click', () => {
                currentQuoteIdx = (currentQuoteIdx + 1) % quotes.length;
                const quoteText = document.getElementById('quote-text');
                const quoteAuthor = document.getElementById('quote-author');
                
                quoteText.style.opacity = 0;
                quoteAuthor.style.opacity = 0;
                
                setTimeout(() => {
                    quoteText.textContent = quotes[currentQuoteIdx].text;
                    quoteAuthor.textContent = quotes[currentQuoteIdx].author;
                    quoteText.style.opacity = 1;
                    quoteAuthor.style.opacity = 1;
                }, 300);
            });

            // Audio Toggle Button
            document.getElementById('btn-audio').addEventListener('click', toggleAudio);

            // Info Modal Logic
            const modal = document.getElementById('info-modal');
            document.getElementById('btn-info').addEventListener('click', () => {
                modal.classList.remove('opacity-0', 'pointer-events-none');
            });
            document.getElementById('close-modal').addEventListener('click', () => {
                modal.classList.add('opacity-0', 'pointer-events-none');
            });
            document.getElementById('modal-ok-btn').addEventListener('click', () => {
                modal.classList.add('opacity-0', 'pointer-events-none');
            });

            // Toggle Left Panel
            const panel = document.getElementById('left-panel');
            const togglePanelBtn = document.getElementById('toggle-panel-btn');
            let isPanelOpen = true;

            togglePanelBtn.addEventListener('click', () => {
                isPanelOpen = !isPanelOpen;
                if (!isPanelOpen) {
                    panel.style.transform = 'translateX(-110%)';
                    togglePanelBtn.innerHTML = '<i class="fa-solid fa-chevron-right"></i>';
                } else {
                    panel.style.transform = 'translateX(0)';
                    togglePanelBtn.innerHTML = '<i class="fa-solid fa-chevron-left"></i>';
                }
            });
        };

        // Smooth Light & Preset Transitions
        function applyPreset(preset) {
            if (!dirLight || !atmosphereMesh) return;

            if (preset === 'day') {
                dirLight.color.setHex(0xffffff);
                dirLight.intensity = 2.0;
                dirLight.position.set(5, 3, 5);
                atmosphereMesh.material.uniforms.color.value.setHex(0x38bdf8);
            } else if (preset === 'sunset') {
                dirLight.color.setHex(0xff7733);
                dirLight.intensity = 2.5;
                dirLight.position.set(-5, 1, 2);
                atmosphereMesh.material.uniforms.color.value.setHex(0xf97316);
            } else if (preset === 'night') {
                dirLight.color.setHex(0x223366);
                dirLight.intensity = 0.5;
                dirLight.position.set(0, -5, -5);
                atmosphereMesh.material.uniforms.color.value.setHex(0x3b82f6);
            } else if (preset === 'bloodmoon') {
                dirLight.color.setHex(0xb91c1c);
                dirLight.intensity = 1.8;
                dirLight.position.set(2, -2, 4);
                atmosphereMesh.material.uniforms.color.value.setHex(0xef4444);
            }
        }

        // Camera Smooth Distance Interpolation
        function gsapCameraMove(targetZ) {
            const startZ = camera.position.z;
            const startTime = performance.now();
            const duration = 1000;

            function step(currentTime) {
                const elapsed = currentTime - startTime;
                const progress = Math.min(elapsed / duration, 1);
                // Ease out cubic
                const ease = 1 - Math.pow(1 - progress, 3);
                
                camera.position.z = startZ + (targetZ - startZ) * ease;

                if (progress < 1) {
                    requestAnimationFrame(step);
                }
            }
            requestAnimationFrame(step);
        }
    </script>
</body>
</html>
