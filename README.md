<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة 3D الاحترافية - مع قائمة رئيسية ورسومات HD</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; user-select: none; background-color: #0a0a16; }
        canvas { display: block; }
        
        /* واجهة القائمة الرئيسية (Start Menu) */
        #startMenu {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(10, 10, 25, 0.7);
            backdrop-filter: blur(15px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 30;
            transition: opacity 0.5s ease, pointer-events 0.5s;
        }
        #startMenu.hidden { opacity: 0; pointer-events: none; }
        #startMenu h1 { font-size: 60px; color: #00ffcc; margin-bottom: 5px; text-shadow: 0 0 25px rgba(0,255,204,0.6); text-align: center; }
        #startMenu p { font-size: 20px; color: #aaa; margin-bottom: 35px; text-align: center; max-width: 500px; line-height: 1.6; }
        .controls-guide { display: flex; gap: 20px; margin-bottom: 35px; background: rgba(255,255,255,0.05); padding: 15px 25px; border-radius: 12px; border: 1px solid rgba(255,255,255,0.1); direction: rtl; }
        .control-item { font-size: 16px; color: #eee; }
        .control-item strong { color: #ffd700; }
        
        #startBtn {
            padding: 16px 50px;
            font-size: 24px;
            font-weight: bold;
            color: #0a0a16;
            background: linear-gradient(45deg, #00ffcc, #00adb5);
            border: none;
            border-radius: 35px;
            cursor: pointer;
            box-shadow: 0 10px 30px rgba(0, 255, 204, 0.4);
            transition: all 0.2s ease;
        }
        #startBtn:hover { transform: translateY(-4px) scale(1.03); box-shadow: 0 15px 35px rgba(0, 255, 204, 0.6); }
        #startBtn:active { transform: translateY(1px) scale(0.98); }

        /* واجهة النتيجة العلوية أثناء اللعب */
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            background: rgba(10, 10, 25, 0.85);
            backdrop-filter: blur(10px);
            padding: 15px 25px;
            border-radius: 16px;
            direction: rtl;
            border: 1px solid rgba(255, 215, 0, 0.3);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            z-index: 10;
            display: none; /* تظهر فقط عند بدء اللعب */
        }
        #ui.visible { display: block; }
        .info-text { font-size: 20px; margin: 5px 0; font-weight: 500; }
        #levelText { color: #00ffcc; font-weight: bold; text-shadow: 0 0 8px rgba(0,255,204,0.4); }
        #scoreText { color: #ffd700; font-weight: bold; text-shadow: 0 0 8px rgba(255,215,0,0.4); }

        /* شاشة الخسارة وإعادة التشغيل */
        #gameOverScreen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(5, 5, 15, 0.9);
            backdrop-filter: blur(15px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 20;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.4s ease;
        }
        #gameOverScreen.active { opacity: 1; pointer-events: auto; }
        #gameOverScreen h1 { font-size: 55px; color: #ff2e63; margin-bottom: 10px; text-shadow: 0 0 20px rgba(255,46,99,0.6); }
        #gameOverScreen p { font-size: 24px; margin-bottom: 30px; color: #ccc; }
        #restartBtn {
            padding: 14px 40px;
            font-size: 22px;
            font-weight: bold;
            color: #0a0a16;
            background: linear-gradient(45deg, #ffd700, #ffaa00);
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 8px 25px rgba(255, 215, 0, 0.4);
            transition: all 0.2s ease;
        }
        #restartBtn:hover { transform: translateY(-3px) scale(1.03); box-shadow: 0 12px 30px rgba(255, 215, 0, 0.6); }
        #restartBtn:active { transform: translateY(1px) scale(0.98); }
    </style>
    <!-- استدعاء مكتبة Three.js -->
    <script src="https://cloudflare.com"></script>
</head>
<body>

    <!-- القائمة الرئيسية -->
    <div id="startMenu">
        <h1>الهروب السيبراني 3D</h1>
        <p>اجمع العملات الذهبية وتفادى العقبات الهرمية الحادة. تقدم في المستويات لتزيد السرعة والإثارة!</p>
        <div class="controls-guide">
            <div class="control-item">الحركة: <strong>W, A, S, D</strong></div>
            <div class="control-item">القفز: <strong>Spacebar (المسافة)</strong></div>
        </div>
        <button id="startBtn" onclick="startGame()">بدء اللعبة 🎮</button>
    </div>

    <!-- واجهة اللعب -->
    <div id="ui">
        <div class="info-text">المستوى: <span id="levelText">1</span></div>
        <div class="info-text">النقاط: <span id="scoreText">0</span></div>
    </div>

    <!-- شاشة الخسارة -->
    <div id="gameOverScreen">
        <h1>لقد خسرت!</h1>
        <p>النتيجة النهائية: <span id="finalScore">0</span> نُقطة</p>
        <button id="restartBtn" onclick="resetGame()">إعادة المحاولة ↻</button>
    </div>

    <script>
        // 1. نظام هندسة الصوت التفاعلي
        let audioCtx;
        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        function playSound(type) {
            if (!audioCtx) return;
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain); gain.connect(audioCtx.destination);

            if (type === 'jump') {
                osc.type = 'triangle'; osc.frequency.setValueAtTime(180, audioCtx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(450, audioCtx.currentTime + 0.12);
                gain.gain.setValueAtTime(0.12, audioCtx.currentTime); gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.12);
                osc.start(); osc.stop(audioCtx.currentTime + 0.12);
            } else if (type === 'coin') {
                osc.type = 'sine'; osc.frequency.setValueAtTime(659.25, audioCtx.currentTime);
                osc.frequency.setValueAtTime(987.77, audioCtx.currentTime + 0.07);
                gain.gain.setValueAtTime(0.08, audioCtx.currentTime); gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.2);
                osc.start(); osc.stop(audioCtx.currentTime + 0.2);
            } else if (type === 'fail') {
                osc.type = 'sawtooth'; osc.frequency.setValueAtTime(250, audioCtx.currentTime);
                osc.frequency.linearRampToValueAtTime(80, audioCtx.currentTime + 0.4);
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime); gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.4);
                osc.start(); osc.stop(audioCtx.currentTime + 0.4);
            }
        }

        // 2. إعداد مشهد جرافيكس عالي الدقة (HD)
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x0a0a1a);
        scene.fog = new THREE.FogExp2(0x0a0a1a, 0.025);

        const camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 5.5, 8.5);

        const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);

        // الإضاءة
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0xffffff, 1.2);
        dirLight.position.set(10, 20, 15);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 2048;
        dirLight.shadow.mapSize.height = 2048;
        scene.add(dirLight);

        // الأرضية وشبكة النيون
        const floorGroup = new THREE.Group();
        const floorGeo = new THREE.PlaneGeometry(16, 250);
        const floorMat = new THREE.MeshStandardMaterial({ color: 0x111318, roughness: 0.6, metalness: 0.2 });
        const floor = new THREE.Mesh(floorGeo, floorMat);
        floor.rotation.x = -Math.PI / 2;
        floor.position.z = -100;
        floor.receiveShadow = true;
        floorGroup.add(floor);

        const gridHelper = new THREE.GridHelper(250, 50, 0x00adb5, 0x22252c);
        gridHelper.position.set(0, 0.01, -100);
        floorGroup.add(gridHelper);
        scene.add(floorGroup);

        // بناء مجسم اللاعب الاحترافي المطور
        const playerGroup = new THREE.Group();
        const bodyGeo = new THREE.CylinderGeometry(0.45, 0.55, 1, 32);
        const bodyMat = new THREE.MeshStandardMaterial({ color: 0x00adb5, metalness: 0.7, roughness: 0.15 });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.position.y = 0.5; body.castShadow = true; body.receiveShadow = true;
        playerGroup.add(body);

        const headGeo = new THREE.SphereGeometry(0.38, 32, 32);
        const headMat = new THREE.MeshStandardMaterial({ color: 0xdddddd, metalness: 0.9, roughness: 0.05 });
        const head = new THREE.Mesh(headGeo, headMat);
        head.position.y = 1.08; head.castShadow = true;
        playerGroup.add(head);

