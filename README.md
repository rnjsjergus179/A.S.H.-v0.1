<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <style>
        body { margin: 0; overflow: hidden; background: black; }
        canvas { display: block; }
        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            width: 320px;
            color: #00ffcc;
            font-family: monospace;
            font-size: 13px;
            background: rgba(0, 0, 0, 0.7);
            padding: 10px;
            border-radius: 5px;
            z-index: 10;
            text-align: center;
            opacity: 0; /* 초기 숨김 */
            transition: opacity 1s ease-in-out;
        }
        #hud-version {
            font-size: 14px;
            font-weight: bold;
            margin-bottom: 5px;
        }
        #hud-video {
            margin-top: 10px;
            width: 240px;
            height: 135px;
            border: 1px solid #00ffcc;
            object-fit: cover;
        }
        #hud-answer {
            margin-top: 5px;
            padding: 5px;
            border: 1px solid #00ffcc;
            min-height: 90px;
        }
        #hud-orbit {
            margin-top: 5px;
            padding: 5px;
            border: 1px solid #00ffcc;
        }
        #chat {
            position: absolute;
            top: 100px;
            right: 20px;
            width: 340px;
            height: 500px;
            background: rgba(0, 0, 0, 0.85);
            color: #00ffcc;
            font-family: monospace;
            font-size: 14px;
            padding: 15px;
            border-radius: 8px;
            z-index: 10;
            display: flex;
            flex-direction: column;
            transition: width 0.5s ease, height 0.5s ease, transform 0.5s ease, opacity 0.5s ease; /* 크기 전환 애니메이션 */
            box-shadow: 0 0 10px rgba(0, 255, 204, 0.3);
        }
        #chat.expanded {
            width: 500px; /* 확대 시 너비 */
            height: 700px; /* 확대 시 높이 */
        }
        .alert {
            transform: translateY(0);
            opacity: 1;
            animation: ring 0.5s ease;
        }
        .minimized {
            transform: translateY(-50px);
            opacity: 0.5;
        }
        @keyframes ring {
            0% { transform: translateY(0); }
            25% { transform: translateY(-5px); }
            50% { transform: translateY(0); }
            75% { transform: translateY(-5px); }
            100% { transform: translateY(0); }
        }
        #messages {
            flex: 1;
            overflow-y: auto;
            margin-bottom: 10px;
        }
        #messages div {
            text-shadow: 0 0 3px rgba(0, 255, 204, 0.5);
            margin-bottom: 5px;
        }
        #chat-input {
            width: 100%;
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #00ffcc;
            color: #00ffcc;
            padding: 8px;
            font-family: monospace;
            font-size: 14px;
            box-shadow: inset 0 0 5px rgba(0, 255, 204, 0.2);
            margin-top: 10px;
        }
        #command-guide {
            margin-top: 10px;
            font-size: 12px;
            color: #66ff99;
            border-top: 1px solid #00ffcc;
            padding-top: 5px;
        }
        #intro-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, 100%);
            color: #00ffcc;
            font-family: monospace;
            font-size: 48px;
            font-weight: bold;
            text-align: center;
            z-index: 20;
            animation: riseUp 2s ease-out forwards;
        }
        @keyframes riseUp {
            0% {
                transform: translate(-50%, 100%);
                opacity: 0;
            }
            50% {
                transform: translate(-50%, -50%);
                opacity: 1;
            }
            100% {
                transform: translate(-50%, -50%) scale(0.3);
                opacity: 0;
            }
        }
    </style>
</head>
<body>
<div id="hud">
    <div id="hud-version"></div>
    ☀️ 정보:<br>
    <video id="hud-video" autoplay loop muted>
        <source id="hud-source" src="https://videos.pexels.com/video-files/5921369/5921369-hd_1920_1080_30fps.mp4" type="video/mp4">
    </video>
    <div id="hud-answer">명령 또는 질문을 입력하세요.</div>
    <div id="hud-orbit">궤도 반지름: <span id="orbit-value"></span> AU | 속도: <span id="speed-value"></span> km/s</div>
</div>

<div id="chat" class="minimized">
    <div id="messages"></div>
    <input type="text" id="chat-input" placeholder="Enter로 메시지 입력">
    <div id="command-guide">
        📝 명령어 입력 방법:<br>
        - "태양으로 가게": 태양으로 이동<br>
        - "지구로 가게" 또는 "지구로 귀환": 지구로 이동<br>
        - "태양 알려줘": 태양 정보 확인<br>
        - "지구 알려줘": 지구 정보 확인<br>
        - "확대 해줘": 채팅창을 확대합니다 (500px x 700px)<br>
        - "축소 해줘": 채팅창을 원래 크기로 축소합니다 (340px x 500px)<br>
        <br>
        📖 시스템 상세 설명:<br>
        - **제작 방식**: 이 시스템은 Three.js 라이브러리를 기반으로 3D 객체(태양, 지구, 인공위성)를 렌더링하며, JavaScript로 동적 애니메이션과 상호작용을 구현했습니다. HTML5 Video 요소를 활용해 비디오 텍스처를 적용했습니다.<br>
        - **UI 기반 여부**: UI는 완전한 UI 프레임워크(예: React, Vue) 없이 순수 HTML/CSS/JavaScript로 제작되었으며, HUD와 채팅창은 정적 레이아웃으로 구성되어 있습니다. 3D 시각화는 UI와 분리되어 Three.js에서 관리됩니다.<br>
        - **목적**: 실시간 위성 시뮬레이션을 통해 태양계 탐사를 시각화하고, 사용자가 명령어를 통해 상호작용할 수 있도록 설계되었습니다.<br>
        - **확대 기능 설명**: "확대 해줘" 명령어로 채팅창 크기를 늘려 더 많은 메시지를 확인할 수 있으며, "축소 해줘"로 원래 크기로 되돌릴 수 있습니다. 크기 전환은 부드러운 애니메이션으로 진행됩니다.
    </div>
</div>

<!-- 알림 소리 -->
<audio id="notificationSound" src="https://www.soundjay.com/buttons/beep-01a.mp3" preload="auto"></audio>

<div id="intro-text">A.S.H. v0.1</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script>
    // Three.js 기본 설정
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const loader = new THREE.TextureLoader();

    // 태양 구체
    const sunTexture = loader.load('https://images.pexels.com/photos/87611/sun-fireball-solar-flare-sunlight-87611.jpeg');
    const sunGeo = new THREE.SphereGeometry(1.5, 64, 64);
    const sunMat = new THREE.MeshStandardMaterial({ map: sunTexture });
    const sun = new THREE.Mesh(sunGeo, sunMat);
    scene.add(sun);

    // 지구(MP4 비디오)
    const earthVideo = document.createElement('video');
    earthVideo.src = 'https://videos.pexels.com/video-files/5921369/5921369-hd_1920_1080_30fps.mp4';
    earthVideo.crossOrigin = 'anonymous';
    earthVideo.loop = true;
    earthVideo.muted = true;
    earthVideo.play();
    const earthVideoTexture = new THREE.VideoTexture(earthVideo);
    const earthGeo = new THREE.SphereGeometry(0.5, 64, 64);
    const earthMat = new THREE.MeshStandardMaterial({ map: earthVideoTexture });
    const earth = new THREE.Mesh(earthGeo, earthMat);
    scene.add(earth);

    // 인공위성(MP4 비디오)
    const satelliteVideo = document.createElement('video');
    satelliteVideo.src = 'https://videos.pexels.com/video-files/854277/854277-hd_1280_720_30fps.mp4';
    satelliteVideo.crossOrigin = 'anonymous';
    satelliteVideo.loop = true;
    satelliteVideo.muted = true;
    satelliteVideo.play().catch(error => {
        console.error('Satellite video playback failed:', error);
        alert('인공위성 비디오 로드에 실패했습니다. CORS 문제일 가능성이 있습니다. 로컬 파일로 대체해 주세요.');
    });
    const satelliteVideoTexture = new THREE.VideoTexture(satelliteVideo);
    satelliteVideoTexture.minFilter = THREE.LinearFilter;
    satelliteVideoTexture.magFilter = THREE.LinearFilter;
    satelliteVideoTexture.format = THREE.RGBFormat;

    const satelliteGeo = new THREE.BoxGeometry(0.4, 0.4, 0.4);
    const satelliteMat = new THREE.MeshStandardMaterial({ map: satelliteVideoTexture });
    const satellite = new THREE.Group();
    const body = new THREE.Mesh(satelliteGeo, satelliteMat);
    satellite.add(body);

    const panelL = new THREE.Mesh(new THREE.PlaneGeometry(0.8, 0.3), new THREE.MeshStandardMaterial({ color: 0x666699, metalness: 0.2 }));
    panelL.position.x = -0.6;
    panelL.rotation.y = Math.PI / 15;
    const panelR = panelL.clone();
    panelR.position.x = 0.6;
    panelR.rotation.y = -Math.PI / 15;
    satellite.add(panelL);
    satellite.add(panelR);
    scene.add(satellite);

    const fallbackTexture = loader.load('https://images.pexels.com/photos/23789/pexels-photo.jpg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2');
    satelliteVideo.addEventListener('error', () => {
        console.warn('인공위성 비디오 로드 실패, 대체 텍스처로 전환');
        satelliteMat.map = fallbackTexture;
        satelliteMat.needsUpdate = true;
    });

    const light = new THREE.PointLight(0xffffff, 2);
    light.position.set(10, 10, 10);
    scene.add(light);

    camera.position.z = 12;

    let satelliteMode = "orbit";
    let angleEarth = 0;
    let angleSatellite = 0;
    const earthOrbitRadius = 5;
    let satelliteOrbitRadius = 1.2;
    const satelliteSpeed = 0.05;
    let sensorRotation = 0;
    let sensorRotating = false;
    let lastTimeUpdate = 0;
    const timeUpdateInterval = 60000;

    function toggleChatAlert(message) {
        const chatBox = document.getElementById('chat');
        const notificationSound = document.getElementById('notificationSound');
        chatBox.classList.add('alert');
        chatBox.classList.remove('minimized');
        notificationSound.play();
        const messages = document.getElementById('messages');
        messages.innerHTML += `<div style="color: #ff0;">📢 ${message || '채팅창 입력하세요!'}</div>`;
        messages.scrollTop = messages.scrollHeight;
        setTimeout(() => {
            chatBox.classList.remove('alert');
            chatBox.classList.add('minimized');
            setTimeout(() => {
                const alertMsg = messages.getElementsByTagName('div')[messages.getElementsByTagName('div').length - 1];
                if (alertMsg && alertMsg.textContent.includes('채팅창 입력하세요') || alertMsg && alertMsg.textContent.includes('현재 시간') || alertMsg && alertMsg.textContent.includes('인공위성 상태')) {
                    alertMsg.remove();
                }
            }, 2000);
        }, 2000);
    }

    function checkSatelliteStatus() {
        const randomFactor = Math.random();
        const isStableOrbit = satelliteOrbitRadius > 1.0 && satelliteOrbitRadius < 1.5;
        const isNormalSpeed = satelliteSpeed === 0.05;
        let statusMessage;

        if (randomFactor < 0.3 || !isStableOrbit || !isNormalSpeed) {
            statusMessage = `⚠️ 인공위성 상태: 안양호입니다! 궤도나 속도에 이상이 있을 수 있어요. 점검을 권장합니다.`;
        } else {
            statusMessage = `✅ 인공위성 상태: 양호입니다! 현재 궤도와 속도가 정상이에요.`;
        }

        return statusMessage;
    }

    function updateSatelliteTime() {
        const currentTime = performance.now();
        if (currentTime - lastTimeUpdate >= timeUpdateInterval) {
            const now = new Date();
            const hours = String(now.getHours()).padStart(2, '0');
            const minutes = String(now.getMinutes()).padStart(2, '0');
            const seconds = String(now.getSeconds()).padStart(2, '0');
            const timeMessage = `🕒 인공위성 현재 시간: ${hours}시 ${minutes}분 ${seconds}초 (2025-03-19 기준)`;
            const statusMessage = checkSatelliteStatus();

            sendChat(timeMessage);
            sendChat(statusMessage);
            toggleChatAlert(timeMessage + "\n" + statusMessage);
            lastTimeUpdate = currentTime;
        }
    }

    document.getElementById('chat-input').addEventListener('keypress', function(event) {
        if (event.key === 'Enter') {
            const input = document.getElementById('chat-input');
            const msg = input.value.trim();
            if (msg) {
                sendChat(`👨‍🚀 ${msg}`);
                toggleChatAlert(`👨‍🚀 ${msg}`);

                if (msg.includes("태양으로 가게")) {
                    satelliteMode = "toSun";
                    sendChat(`🛰 위성이 태양으로 이동합니다!`);
                    toggleChatAlert(`🛰 위성이 태양으로 이동합니다!`);
                } else if (msg.includes("지구로 가게") || msg.includes("지구로 귀환")) {
                    satelliteMode = "toEarth";
                    sendChat(`🛰 위성이 지구로 이동합니다!`);
                    toggleChatAlert(`🛰 위성이 지구로 이동합니다!`);
                } else if (msg.includes("태양 알려줘")) {
                    autoSunInfo();
                } else if (msg.includes("지구 알려줘")) {
                    autoEarthInfo();
                } else if (msg.includes("확대 해줘")) {
                    toggleChatSize(true);
                    sendChat(`📏 채팅창이 확대되었습니다!`);
                    toggleChatAlert(`📏 채팅창이 확대되었습니다!`);
                } else if (msg.includes("축소 해줘")) {
                    toggleChatSize(false);
                    sendChat(`📏 채팅창이 축소되었습니다!`);
                    toggleChatAlert(`📏 채팅창이 축소되었습니다!`);
                } else {
                    sendChat(`🤖 명령어를 인식하지 못했습니다.`);
                    toggleChatAlert(`🤖 명령어를 인식하지 못했습니다.`);
                }
                input.value = "";
            }
        }
    });

    function toggleChatSize(expand) {
        const chat = document.getElementById('chat');
        if (expand) {
            chat.classList.add('expanded');
        } else {
            chat.classList.remove('expanded');
        }
    }

    function animate() {
        requestAnimationFrame(animate);
        sun.rotation.y += 0.0002;
        angleEarth += 0.002;
        earth.position.x = earthOrbitRadius * Math.cos(angleEarth);
        earth.position.z = earthOrbitRadius * Math.sin(angleEarth);

        if (satelliteMode === "orbit") {
            angleSatellite += satelliteSpeed;
            satelliteOrbitRadius = 1.2 + 0.1 * Math.sin(angleSatellite);
            satellite.position.x = earth.position.x + satelliteOrbitRadius * Math.cos(angleSatellite);
            satellite.position.z = earth.position.z + satelliteOrbitRadius * Math.sin(angleSatellite);
        } else if (satelliteMode === "toSun") {
            if (satellite.position.distanceTo(sun.position) > 2.5) {
                satellite.position.lerp(sun.position, 0.01);
            } else {
                if (satelliteMode !== "sun-station") {
                    satelliteMode = "sun-station";
                    autoSunInfo();
                    sensorRotating = true;
                    sendChat("📡 태양 근접 완료 - 촬영 중...");
                    const sunRotationInfo = `
☀️ 태양 자전 정보:
- 적도 자전 주기: 약 25일
- 극지방 자전 주기: 약 35일
- 현재 시뮬레이션 속도: 자전은 시각적으로 0.0002 rad/프레임으로 표현`;
                    sendChat(sunRotationInfo);
                    toggleChatAlert(sunRotationInfo);
                }
            }
        } else if (satelliteMode === "toEarth") {
            if (satellite.position.distanceTo(earth.position) > 1) {
                satellite.position.lerp(earth.position, 0.01);
            } else {
                if (satelliteMode !== "orbit") {
                    satelliteMode = "orbit";
                    autoEarthInfo();
                    sensorRotating = false;
                    sendChat("📡 지구 근접 완료 - 촬영 중...");
                    const orbitProgress = ((angleEarth % (2 * Math.PI)) / (2 * Math.PI) * 100).toFixed(1);
                    const earthSunDistance = earth.position.distanceTo(sun.position).toFixed(2);
                    let orbitState = "";
                    const angleDeg = (angleEarth % (2 * Math.PI)) * 180 / Math.PI;
                    if (angleDeg < 10 || angleDeg > 350) orbitState = "근일점 근처";
                    else if (angleDeg > 170 && angleDeg < 190) orbitState = "원일점 근처";
                    else orbitState = "공전 중";
                    const earthOrbitInfo = `
🌍 지구 공전 정보:
- 공전 주기: 약 365.25일
- 현재 공전 진행률: ${orbitProgress}%
- 공전 상태: ${orbitState}
- 태양 거리: ${earthSunDistance} AU`;
                    sendChat(earthOrbitInfo);
                    toggleChatAlert(earthOrbitInfo);
                }
            }
        }

        if (sensorRotating) {
            sensorRotation += 0.05;
            panelL.rotation.z = sensorRotation;
            panelR.rotation.z = sensorRotation;
        } else {
            panelL.rotation.z = 0;
            panelR.rotation.z = 0;
        }

        updateSatelliteTime();

        document.getElementById('orbit-value').innerText = satelliteOrbitRadius.toFixed(2);
        document.getElementById('speed-value').innerText = (satelliteSpeed * 50).toFixed(2);

        renderer.render(scene, camera);
    }

    const introText = document.getElementById('intro-text');
    introText.addEventListener('animationend', () => {
        introText.style.display = 'none';
        const hudVersion = document.getElementById('hud-version');
        hudVersion.textContent = 'A.S.H. v0.1';
        document.getElementById('hud').style.opacity = 1;
    });

    animate();

    function sendChat(text) {
        const messages = document.getElementById('messages');
        messages.innerHTML += `<div>${text}</div>`;
        messages.scrollTop = messages.scrollHeight;
    }

    function autoSunInfo() {
        const sunInfo = `
☀️ 태양 정보:
- 11년 자기장 주기 및 플레어 주기 순환
- 자전 주기: 적도 25일 / 극지방 35일
- 질량: 태양계 질량의 99.86%
- 핵융합: 수소를 헬륨으로 융합하여 에너지 발생
- 광구 온도: 약 5,500°C
- 플레어 & CME: 지구 자기장에 영향 가능
- 지구 거리: 약 1억 4,960만 km
- 수명: 약 100억년 중 절반 경과`;
        sendChat(sunInfo);
        document.getElementById('hud-answer').innerText = sunInfo;
        switchVideo("sun");
    }

    function autoEarthInfo() {
        const earthInfo = `
🌍 지구 정보:
- 반지름: 약 6,371 km
- 자전 주기: 약 24시간
- 공전 주기: 약 365일
- 위성: 1개 (달)
- 평균 온도: 약 15°C
- 대기: 질소 78%, 산소 21%
- 🌱 생명체 존재: 다양한 동식물 및 인간 서식
- 대기 환경: 인간과 생명체가 호흡 가능한 산소 대기`;
        sendChat(earthInfo);
        document.getElementById('hud-answer').innerText = earthInfo;
        switchVideo("earth");
    }

    function switchVideo(target) {
        const video = document.getElementById('hud-video');
        const source = document.getElementById('hud-source');
        if (target === "sun") {
            source.src = "https://vp.nyt.com/video/2020/09/15/88634_1_15SCI-SOLARCYCLE_wg_1080p.mp4";
        } else if (target === "earth") {
            source.src = "https://videos.pexels.com/video-files/5921369/5921369-hd_1920_1080_30fps.mp4";
        }
        video.load();
        video.play();
    }
</script>
</body>
</html>
