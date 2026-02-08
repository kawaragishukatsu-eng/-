<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>区民ホール舞台照明シミュレーション (メツブシ照射角/強度変更済み)</title>
    <style>
        /* --- CSS スタイル --- */
        body { margin: 0; overflow: hidden; font-family: sans-serif; }
        #app-container { display: flex; height: 100vh; }
        #ui-panel { width: 300px; padding: 15px; background: #f4f4f4; overflow-y: auto; box-shadow: 2px 0 5px rgba(0,0,0,0.1); }
        #scene-container { flex-grow: 1; position: relative; }
        .control-group { margin-bottom: 20px; padding: 10px; border: 1px solid #ccc; border-radius: 5px; background: #fff; }
        .control-group h4 { margin-top: 0; cursor: pointer; /* クリック可能にする */ }
        .control-group h4::before { content: '▶ '; font-size: 0.8em; }
        .control-group.expanded h4::before { content: '▼ '; }
        
        .control-group label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 0.9em; }
        .slider-wrapper { display: flex; align-items: center; }
        .slider-wrapper input[type="range"] { flex-grow: 1; margin-right: 10px; }
        .slider-wrapper span { width: 30px; text-align: right; font-weight: bold; }
        .coord-input { width: 45px; margin-right: 5px; }
        .name-input { width: 95%; margin-top: 5px; margin-bottom: 10px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; }
        button { padding: 8px 12px; background-color: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; margin-bottom: 5px; }
        button.side-spot-btn { background-color: #ff9800; }
        button.foot-light-btn { background-color: #00bcd4; }
        button.pin-spot-btn { background-color: #ffc107; }
        button.center-pin-btn { background-color: #ff5722; }
        button.metubushi-btn { background-color: #673ab7; } /* メツブシボタンの色 */
        button.side-spot-btn:hover { background-color: #e68900; }
        button.foot-light-btn:hover { background-color: #00acc1; }
        button.pin-spot-btn:hover { background-color: #e0a800; }
        button.center-pin-btn:hover { background-color: #e64a19; }
        button.metubushi-btn:hover { background-color: #512da8; }
        button:hover { background-color: #0056b3; }
        .delete-btn { background-color: #dc3545; margin-top: 10px; }
        .delete-btn:hover { background-color: #c82333; }
        
        /* 展開/折りたたみ可能な詳細設定のスタイル */
        .light-details { display: none; }
        .control-group.expanded .light-details { display: block; }

        /* 初期表示項目は常に見えるようにする */
        .always-visible { display: block !important; }

        #help-overlay {
            position: absolute;
            top: 10px;
            right: 10px;
            /* 初期状態では表示 */
            z-index: 10;
        }

        #help-content {
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            border-radius: 5px;
            font-size: 0.9em;
            margin-top: 5px;
        }

        /* ヘルプ表示/非表示ボタンのスタイル */
        #help-toggle-btn {
            background: #555;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
        }
        #help-toggle-btn:hover {
            background: #777;
        }
    </style>
</head>
<body>
    <div id="app-container">
        <div id="ui-panel">
            <h2>🎭 照明操作パネル</h2>

            <div class="control-group">
                <h3>機材追加</h3>
                <button onclick="addLight('フレネル(単)', 0, 18, 5)">フレネル（単体）を追加 (センター)</button>
                <button onclick="addLightPair('フレネル(対称)', 5, 18, 5)">フレネル（対称ペア）を追加</button>
                <button class="side-spot-btn" onclick="addSideSpots()">サイドスポットライトを追加 (4灯)</button>
                <button class="foot-light-btn" onclick="addFootLights()">フットライト（転がし）を追加 (4灯)</button>
                <button class="pin-spot-btn" onclick="addPinSpots()">ピンスポットライトを追加 (6灯)</button>
                <button class="center-pin-btn" onclick="addCenterPinSpot()">センターピンスポットを追加</button>
                <button class="metubushi-btn" onclick="addMetubushiLights()">メツブシを追加 (4灯)</button>
                <div id="light-list">
                </div>
            </div>

            <div class="control-group">
                <h3>地明かり (Stage Wash)</h3>
                <label>地明かり強度</label>
                <div class="slider-wrapper">
                    <input type="range" id="wash-intensity" min="0" max="100" value="0" oninput="updateWashLight(this.value)">
                    <span id="wash-value">0</span>
                </div>
            </div>
            
            <div class="control-group">


                <h3>背景 LEDウォール</h3>


                <label>上部色</label>


                <input type="color" id="led-top-color" value="#0000ff" onchange="updateLEDWall()">


                <label>下部色</label>


                <input type="color" id="led-bottom-color" value="#ff0000" onchange="updateLEDWall()">


            </div>


                        <div class="control-group">








                <h3>サーチライト機能 (LED Wall)</h3>








                <label for="searchlight-intensity">サーチライト強度 (1-100)</label>








                <div class="slider-wrapper">








                    <input type="range" id="searchlight-intensity" min="0" max="100" value="0" oninput="updateSearchlightIntensity(this.value)">








                    <span id="searchlight-value">0</span>








                </div>








                <label for="searchlight-color">混色色 (初期: 白)</label>








                <input type="color" id="searchlight-color" value="#ffffff" onchange="updateLEDWall()">








                <label for="searchlight-angle">頂点角度 (度)</label>








                <div class="slider-wrapper">








                    <input type="range" id="searchlight-angle" min="5" max="90" value="15" oninput="updateSearchlightAngle(this.value)">








                    <span id="searchlight-angle-value">15°</span>








                </div>








            </div>













            <div class="control-group">
                <h3>全体操作 (マスターフェーダー)</h3>
                <div class="slider-wrapper">
                    <label for="master-intensity">マスター強度</label>
                    <input type="range" id="master-intensity" min="0" max="100" value="50" oninput="updateMasterIntensity(this.value)">
                    <span id="master-value">50</span>
                </div>
            </div>

            <div id="fader-groups">
            </div>
        </div>

        <div id="scene-container">
            <div id="help-overlay">
                <button id="help-toggle-btn" onclick="toggleHelpOverlay()">ヘルプ (H)</button>
                <div id="help-content">
                    <h4>操作モード切替 / ショートカット</h4>


                    <ul>


                        <li> クリックで選択しドラッグ操作でフレネルの位置/向きを変更できます</li>


                        <li>Shiftキー + マウス操作で視点回転・移動ができます</li>


                        <li>P + 矢印↑↓キーでパフォーマーを奥/手前へ移動できます</li>


                        <li>Sキー + 矢印↑↓キーでピンスポットライトを奥/手前へ移動できます</li>


                        <li>下矢印↓キーで選択中のフレネルを真下に向けます</li>


                        <li>左右矢印キー←→で選択中のフレネルを真横に向けます</li>


                    </ul>


                </div>


            </div>


        </div>


    </div>




    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/controls/TransformControls.js"></script>

    <script>
        // --- JavaScript / Three.js コード ---

        let scene, camera, renderer, controls, transformControls;
        let ledWall;
        let washAmbientLight, washDirectionalLight;
        let performerGroup; 
        const lights = []; 
        const lightListEl = document.getElementById('light-list');
        const masterIntensityEl = document.getElementById('master-intensity');
        const masterValueEl = document.getElementById('master-value');
        const helpContentEl = document.getElementById('help-content');
        const helpToggleBtnEl = document.getElementById('help-toggle-btn');

        // 🎨 【新規追加】サーチライト関連の状態


        let searchlightIntensity = 0; // 0-100


        let searchlightColor = new THREE.Color('#ffffff');


        let searchlightAngleRad = 15 * (Math.PI / 180); // 15度をラジアンに変換


        // --------------------------------



        // 定数
        const STAGE_WIDTH = 30;
        const STAGE_DEPTH = 30;
        const STAGE_BACKDROP_HEIGHT = 18;
        const MAX_INTENSITY_FACTOR = 4.0;
        const STAGE_FLOOR_Y = 0; 
        const METUBUSHI_INTENSITY_MULTIPLIER = 2.0; 
        

        let isShiftPressed = false; 
        let isPPressed = false; 
        let isSPressed = false; 

        // シーンの初期化
        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x111111);

            const width = window.innerWidth - 300;
            const height = window.innerHeight;
            camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
            camera.position.set(0, 10, 30); 

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(width, height);
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap; 
            document.getElementById('scene-container').appendChild(renderer.domElement);

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.target.set(0, 4, 0); 
            controls.enableDamping = true;
            controls.update();
            
            controls.enabled = false; 

            transformControls = new THREE.TransformControls(camera, renderer.domElement);
            scene.add(transformControls);

            transformControls.addEventListener('dragging-changed', function (event) {
                controls.enabled = !event.value;
                if (!event.value) {
                    controls.enabled = isShiftPressed;
                }
            });
            transformControls.addEventListener('objectChange', onLightTransformChange);

            createStage();
            setupWashLight();
            
            performerGroup = new THREE.Group();
            scene.add(performerGroup);

            // パフォーマーの初期配置
            const performerCount = 6;
            const spacing = 4;
            const startX = -((performerCount - 1) * spacing) / 2;
            for (let i = 0; i < performerCount; i++) {
                addPerformer(startX + i * spacing, 0, 0);
            }
            
            updateLEDWall();

            window.addEventListener('resize', onWindowResize, false);
            window.addEventListener('mousedown', onMouseDown, false); 
            
            document.addEventListener('keydown', onKeyDown);
            document.addEventListener('keyup', onKeyUp);

            animate();
        }

        // --- 【新規追加】ヘルプ表示/非表示機能 ---
        function toggleHelpOverlay() {
            const isVisible = helpContentEl.style.display !== 'none';
            if (isVisible) {
                helpContentEl.style.display = 'none';
                helpToggleBtnEl.textContent = 'ヘルプ (H)';
            } else {
                helpContentEl.style.display = 'block';
                helpToggleBtnEl.textContent = 'ヘルプを閉じる (H)';
            }
        }
        // --- ------------------------------- ---


        // --- キーボードイベントハンドラ ---
        function onKeyDown(event) {
            if (event.key === 'Shift') {
                isShiftPressed = true;
                if (!transformControls.dragging) {
                    controls.enabled = true;
                }
            } else if (event.key === 'p' || event.key === 'P') {
                isPPressed = true;
            } else if (event.key === 's' || event.key === 'S') {
                isSPressed = true;
            } else if (event.key === 'h' || event.key === 'H') {
                 // Hキーでヘルプ表示をトグル
                toggleHelpOverlay();
            }
            
            const moveStep = 0.5;

            // パフォーマー移動 (Pキー + 上下矢印)
            if (isPPressed) {
                if (event.key === 'ArrowUp') {
                    performerGroup.position.z -= moveStep; 
                    event.preventDefault(); 
                } else if (event.key === 'ArrowDown') {
                    performerGroup.position.z += moveStep; 
                    event.preventDefault();
                }
            }

            // ピンスポットライトZ座標移動 (Sキー + 上下矢印)
            if (isSPressed && (event.key === 'ArrowUp' || event.key === 'ArrowDown')) {
                const pinSpotLights = lights.filter(l => l.isPinSpot);
                const centerPinSpotLights = lights.filter(l => l.isCenterPinSpot);
                
                const allPinLights = [...pinSpotLights, ...centerPinSpotLights];

                if (allPinLights.length > 0) {
                    const direction = (event.key === 'ArrowUp') ? -1 : 1; // $\uparrow$ = 奥 (Z減少)
                    
                    allPinLights.forEach(lightData => {
                        const newZ = lightData.z + direction * moveStep;
                        
                        lightData.z = newZ;
                        lightData.object.forEach(light => { light.position.z = newZ; });
                        
                        // Target座標の再計算 (Target Z = Position Z - 4)
                        const newTargetZ = newZ - 4;
                        lightData.targetZ = newTargetZ;
                        lightData.target.forEach(target => { 
                            target.position.z = newTargetZ;
                            target.updateMatrixWorld();
                        });
                    });

                    // UIのPosition Z, Target Zを更新 (代表値としてグループのIDを使用)
                    if (pinSpotLights.length > 0) {
                        const groupId = 'pin-spot-group';
                        const posZ_El = document.getElementById(`posZ-${groupId}`);
                        const targetZ_El = document.getElementById(`targetZ-${groupId}`);

                        if (posZ_El) posZ_El.value = pinSpotLights[0].z.toFixed(1);
                        if (targetZ_El) targetZ_El.value = pinSpotLights[0].targetZ.toFixed(1);
                    }
                    if (centerPinSpotLights.length > 0) {
                        const groupId = centerPinSpotLights[0].id;
                        const posZ_El = document.getElementById(`posZ-${groupId}`);
                        const targetZ_El = document.getElementById(`targetZ-${groupId}`);

                        if (posZ_El) posZ_El.value = centerPinSpotLights[0].z.toFixed(1);
                        if (targetZ_El) targetZ_El.value = centerPinSpotLights[0].targetZ.toFixed(1);
                    }
                    
                    event.preventDefault();
                }
            }

            // 矢印キーでのターゲット設定 (P, Sキーが押されていないときのみ)
            if (!isPPressed && !isSPressed) {
                const activeLightData = getActiveLightData();
                if (activeLightData && !activeLightData.isSideSpot && !activeLightData.isFootLight && !activeLightData.isPinSpot && !activeLightData.isCenterPinSpot && !activeLightData.isMetubushi) {
                    let newTargetX, newTargetY, newTargetZ;
                    const lightPos = activeLightData.object[0].position;
                    
                    if (event.key === 'ArrowDown') { // $\downarrow$キー: 真下 (舞台床のY座標)
                        newTargetX = lightPos.x;
                        newTargetY = STAGE_FLOOR_Y; 
                        newTargetZ = lightPos.z;
                        event.preventDefault(); 
                    } else if (event.key === 'ArrowLeft' || event.key === 'ArrowRight') { // $\leftarrow$/$\rightarrow$キー: 真横
                        newTargetX = lightPos.x;
                        newTargetY = lightPos.y; 
                        newTargetZ = lightPos.z > 0 ? -100 : 100;
                        event.preventDefault();
                    }

                    if (event.key === 'ArrowDown' || event.key === 'ArrowLeft' || event.key === 'ArrowRight') {
                        updateLightTarget(activeLightData.id, newTargetX, newTargetY, newTargetZ);
                    }
                }
            }
        }

        function onKeyUp(event) {
            if (event.key === 'Shift') {
                isShiftPressed = false;
                controls.enabled = false;
            } else if (event.key === 'p' || event.key === 'P') {
                isPPressed = false;
            } else if (event.key === 's' || event.key === 'S') {
                isSPressed = false;
            }
        }


        // --- ヘルパー関数 ---

        function getActiveLightData() {
            const transformedObject = transformControls.object;
            if (!transformedObject || transformedObject.name !== "BulbModel" && transformedObject.name !== "LightTarget") return null;

            let lightData = lights.find(ld => {
                const isLightBulb = transformedObject.name === "BulbModel";
                if (isLightBulb && ld.object.some(o => o === transformedObject.parent)) {
                    return true;
                }
                if (transformedObject.name === "LightTarget" && ld.target.includes(transformedObject)) {
                    return true;
                }
                return false;
            });
            return lightData;
        }

        // --- 地明かり機能 ---
        function setupWashLight() {
            washAmbientLight = new THREE.AmbientLight(0xffffff, 0);
            scene.add(washAmbientLight);
            washDirectionalLight = new THREE.DirectionalLight(0xffffff, 0);
            washDirectionalLight.position.set(0, 10, 10);
            washDirectionalLight.target.position.set(0, 0, 0);
            washDirectionalLight.castShadow = true;
            washDirectionalLight.shadow.mapSize.width = 1024; // 解像度を落とす
            washDirectionalLight.shadow.mapSize.height = 1024;
            washDirectionalLight.shadow.camera.left = -20;
            washDirectionalLight.shadow.camera.right = 20;
            washDirectionalLight.shadow.camera.top = 20;
            washDirectionalLight.shadow.camera.bottom = -20;
            washDirectionalLight.shadow.camera.near = 1;
            washDirectionalLight.shadow.camera.far = 40; 
            scene.add(washDirectionalLight);
            scene.add(washDirectionalLight.target);
        }

        function updateWashLight(value) {
            const uiValue = parseInt(value);
            const intensity = uiValue / 100;
            washDirectionalLight.intensity = intensity * 1.5; 
            washAmbientLight.intensity = intensity * 0.2; 
            document.getElementById('wash-value').textContent = uiValue;
        }

        // --- 舞台/パフォーマー/LEDウォール ---








        function createStage() {








            const stageGeometry = new THREE.BoxGeometry(STAGE_WIDTH, 0.1, STAGE_DEPTH);








            const stageMaterial = new THREE.MeshPhongMaterial({ color: 0x444444 });








            const stage = new THREE.Mesh(stageGeometry, stageMaterial);








            stage.position.set(0, 0, 0);








            stage.receiveShadow = true;








            scene.add(stage);

















            const backGeometry = new THREE.PlaneGeometry(STAGE_WIDTH, STAGE_BACKDROP_HEIGHT, 100, 100); // 頂点数を増やしてグラデーションとサーチライトの品質を向上








            const backMaterial = new THREE.MeshBasicMaterial({ vertexColors: true, side: THREE.DoubleSide });








            ledWall = new THREE.Mesh(backGeometry, backMaterial);








            ledWall.position.set(0, STAGE_BACKDROP_HEIGHT / 2, -STAGE_DEPTH / 2);








            scene.add(ledWall);








        }








        








        // 🎨 【更新】サーチライト機能を組み込み


// サーチライト強度スライダーの値を更新し、LEDウォールを更新する








function updateSearchlightIntensity(value) {








    // 1. グローバル変数の更新








    searchlightIntensity = parseInt(value);








    








    // 2. 画面上の表示を更新








    document.getElementById('searchlight-value').textContent = searchlightIntensity;








    








    // 3. LEDウォールの描画を更新








    updateLEDWall();








}

















// 頂点角度スライダーの値を更新し、LEDウォールを更新する








function updateSearchlightAngle(value) {








    // 1. グローバル変数の更新








    // 角度（度）をラジアンに変換して格納








    searchlightAngleRad = parseFloat(value) * (Math.PI / 180);








    








    // 2. 画面上の表示を更新








    document.getElementById('searchlight-angle-value').textContent = `${value}°`;








    








    // 3. LEDウォールの描画を更新








    updateLEDWall();








}





        function updateLEDWall() {








            const topColor = new THREE.Color(document.getElementById('led-top-color').value);








            const bottomColor = new THREE.Color(document.getElementById('led-bottom-color').value);








            








            // 🎨 【新規追加】サーチライト関連の変数








            searchlightColor = new THREE.Color(document.getElementById('searchlight-color').value);








            const lightIntensityFactor = searchlightIntensity / 100; // 0.0 to 1.0








            const halfWidth = STAGE_WIDTH / 2; // 15








            const maxLightY = 12; // Y座標 12で消滅

















            const position = ledWall.geometry.attributes.position;








            const colors = [];








            const colorAttribute = new THREE.BufferAttribute(new Float32Array(position.count * 3), 3);








            








            const angleTan = Math.tan(searchlightAngleRad / 2); // 頂角の半分のタンジェント











for (let i = 0; i < position.count; i++) {











                // 1. 基本のグラデーション色を計算








                const localY = position.getY(i); // ローカル座標でのY (-9 to 9)








                const wallY = localY + STAGE_BACKDROP_HEIGHT / 2; // 壁下端からのY座標 (0 to 18)








                








                const interpolation = (localY + STAGE_BACKDROP_HEIGHT / 2) / STAGE_BACKDROP_HEIGHT; // 0 (下) から 1 (上)








                const baseColor = bottomColor.clone().lerp(topColor, interpolation);








                








                // 2. サーチライト効果の計算








    let lightMixFactor = 0; // サーチライトの混色比率

















    if (lightIntensityFactor > 0 && wallY > 0) { // 強度がゼロでなく、かつY>0 のときのみ計算








        








        // a. Y座標による減衰 (0で最大、maxLightY (12) で 0)








        const fadeFactor = THREE.MathUtils.clamp(1.0 - (wallY / maxLightY), 0.0, 1.0);








        








        // b. 左右の三角形領域の判定 (新しいロジック)








        const wallX = position.getX(i); // ローカル座標でのX (-15 to 15)








        








        // **修正後の判定ロジック**








        // wallY > 0 の時のみ実行されるため、分母がゼロになる心配はない








        const requiredX = wallY * angleTan;








        








        // X座標の絶対値が requiredX より小さい場合、光の円錐内にある








        if (Math.abs(wallX) < requiredX) {








            








            // 3. 混色係数を決定








            lightMixFactor = fadeFactor * lightIntensityFactor;








        }








    }








                








                // 3. 最終的な色を計算 (ベース色にサーチライト色を線形補間)








                // lerp(a, b, t) = a * (1 - t) + b * t








                const finalColor = baseColor.clone().lerp(searchlightColor, lightMixFactor);

















                colors.push(finalColor.r, finalColor.g, finalColor.b);








            }








            colorAttribute.set(new Float32Array(colors));








            ledWall.geometry.setAttribute('color', colorAttribute);








            ledWall.geometry.attributes.color.needsUpdate = true;








        }


















        

        function addPerformer(x, y, z) {
            const bodyHeight = 1.5;
            const bodyRadius = 0.5;
            const headRadius = 0.5;
            const material = new THREE.MeshPhongMaterial({ color: 0x00ccff });
            const bodyGeometry = new THREE.CylinderGeometry(bodyRadius, bodyRadius, bodyHeight, 16);
            const body = new THREE.Mesh(bodyGeometry, material);
            body.position.set(x, y + bodyHeight / 2, z);
            body.castShadow = true;
            performerGroup.add(body); 
            const headGeometry = new THREE.SphereGeometry(headRadius, 16, 16);
            const head = new THREE.Mesh(headGeometry, material);
            head.position.set(x, y + bodyHeight + headRadius, z);
            head.castShadow = true;
            performerGroup.add(head); 
        }


        // --- 照明機材の生成/削除/UI ---

        function createSpotLight(intensity, isPinSpot = false, initialAngle = null, disableShadow = false) {
            let angle, penumbra, decay;
            
            if (isPinSpot) {
                // ピンスポット専用の設定 (R=1.8m, D=4m の照射角の1/3)
                angle = 0.1409; 
                penumbra = 0.1; 
                decay = 2;
            } else {
                // 通常のフレネルの設定 (従来の SpotLight の設定)
                angle = Math.PI / 8; // 0.3927 rad (22.5度)
                penumbra = 0.5;
                decay = 2;
            }

            // センターピンスポット/メツブシ用に初期角が指定された場合はそれを使用
            if (initialAngle !== null) {
                angle = initialAngle;
            }

            const light = new THREE.SpotLight(0xffffff, intensity, 60, angle, penumbra, decay); 
            
            // シャドウが不要なライトは無効にする
            light.castShadow = !disableShadow;
            
            if (!disableShadow) {
                 light.shadow.mapSize.width = 512; // 解像度をさらに落とし、テクスチャ消費を抑える
                 light.shadow.mapSize.height = 512;
                 light.shadow.camera.far = 60; 
            }


            const bulbGeometry = new THREE.SphereGeometry(0.1, 8, 8);
            const bulbMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
            const bulb = new THREE.Mesh(bulbGeometry, bulbMaterial);
            bulb.name = "BulbModel";
            light.add(bulb); 
            return light;
        }

        function createLightTarget(x, y, z) {
            const targetObject = new THREE.Object3D(); 
            targetObject.position.set(x, y, z); 
            targetObject.name = "LightTarget";
            scene.add(targetObject);
            return targetObject;
        }
        
        // 照射角を度からラジアンに変換して更新
        function updateLightAngle(lightId, degrees) {
            // グループIDから所属する全てのライトデータを取得
            let lightsToUpdate = [];
            if (lightId === 'metubushi-group') {
                lightsToUpdate = lights.filter(l => l.isMetubushi);
            } else {
                const lightData = lights.find(l => l.id === lightId);
                if (lightData) lightsToUpdate.push(lightData);
            }

            if (lightsToUpdate.length === 0) return;

            const radians = degrees * (Math.PI / 180);
            
            lightsToUpdate.forEach(lightData => {
                lightData.angle = radians; 
                
                lightData.object.forEach(light => {
                    light.angle = radians;
                    light.updateMatrixWorld();
                });
            });

            const angleValueEl = document.getElementById(`angle-value-${lightId}`);
            if (angleValueEl) {
                angleValueEl.textContent = degrees + '°';
            }
        }


        function addLight(type, x, y, z) {
            const initialIntensityThreeJS = 1.0; 
            const initialIntensityUI = 100;
            // シャドウは有効 (disableShadow = false)
            const light = createSpotLight(initialIntensityThreeJS, false); 
            light.position.set(x, y, z);
            const targetX = x;
            const targetY = STAGE_FLOOR_Y;
            const targetZ = z;
            const targetObject = createLightTarget(targetX, targetY, targetZ);
            light.target = targetObject;
            scene.add(light);
            
            const lightData = {
                id: THREE.MathUtils.generateUUID(),
                type: type,
                name: type,
                object: [light],
                target: [targetObject],
                isPair: false,
                isSideSpot: false,
                isFootLight: false,
                isPinSpot: false,
                isCenterPinSpot: false,
                isMetubushi: false, // メツブシではない
                intensity: initialIntensityUI, 
                color: '#ffffff',
                x: x, y: y, z: z,
                targetX: targetX, targetY: targetY, targetZ: targetZ,
                angle: light.angle
            };
            lights.push(lightData);
            createLightControlUI(lightData);
            updateLightIntensity(lightData.id, initialIntensityUI);
        }

        function addLightPair(type, x, y, z, targetX, targetY, targetZ, isSideSpot = false, isFootLight = false, isPinSpot = false, isMetubushi = false) {
            const initialIntensityThreeJS = 1.0; 
            const initialIntensityUI = 100;

            const finalTargetX = targetX !== undefined ? targetX : x;
            const finalTargetY = targetY !== undefined ? targetY : STAGE_FLOOR_Y;
            const finalTargetZ = targetZ !== undefined ? targetZ : z;
            
            // isSideSpot, isFootLight, isMetubushi の場合はシャドウを無効化
            const disableShadow = isSideSpot || isFootLight || isMetubushi;
            
            const lightL = createSpotLight(initialIntensityThreeJS, isPinSpot, null, disableShadow);
            lightL.position.set(x, y, z);
            const targetL = createLightTarget(finalTargetX, finalTargetY, finalTargetZ);
            lightL.target = targetL;
            scene.add(lightL);

            const lightR = createSpotLight(initialIntensityThreeJS, isPinSpot, null, disableShadow);
            lightR.position.set(-x, y, z); 
            const targetR = createLightTarget(-finalTargetX, finalTargetY, finalTargetZ); 
            lightR.target = targetR;
            scene.add(lightR);

            const lightData = {
                id: THREE.MathUtils.generateUUID(),
                type: type,
                name: type,
                object: [lightL, lightR], 
                target: [targetL, targetR], 
                isPair: true,
                isSideSpot: isSideSpot,
                isFootLight: isFootLight,
                isPinSpot: isPinSpot,
                isCenterPinSpot: false,
                isMetubushi: isMetubushi,
                intensity: initialIntensityUI, 
                color: '#ffffff',
                x: x, y: y, z: z,
                targetX: finalTargetX, targetY: finalTargetY, finalTargetZ: finalTargetZ,
                angle: lightL.angle
            };
            lights.push(lightData);
            
            if (!isSideSpot && !isFootLight && !isPinSpot && !isMetubushi) {
                createLightControlUI(lightData);
            }
            updateLightIntensity(lightData.id, initialIntensityUI);
        }

        function addSideSpots() {
            if (lights.some(l => l.isSideSpot)) {
                alert("サイドスポットライトは既に追加されています。");
                return;
            }

            const positions = [
                { x: 16, y: 2.5, z: 2.5 },
                { x: 16, y: 2.5, z: 10 }
            ];
            
            let firstLightData = null; 

            positions.forEach((pos, index) => {
                const targetX = pos.x * (-1);
                const targetY = STAGE_FLOOR_Y; 
                const targetZ = pos.z - 2;

                const initialIntensityThreeJS = 1.0; 
                const initialIntensityUI = 100;
                
                // サイドスポットライトのシャドウを無効化 (disableShadow = true)
                const lightL = createSpotLight(initialIntensityThreeJS, false, null, true); 
                lightL.position.set(pos.x, pos.y, pos.z);
                const targetL = createLightTarget(targetX, targetY, targetZ);
                lightL.target = targetL;
                scene.add(lightL);

                // サイドスポットライトのシャドウを無効化 (disableShadow = true)
                const lightR = createSpotLight(initialIntensityThreeJS, false, null, true); 
                lightR.position.set(-pos.x, pos.y, pos.z); 
                const targetR = createLightTarget(-targetX, targetY, targetZ); 
                lightR.target = targetR;
                scene.add(lightR);
                
                const lightData = {
                    id: THREE.MathUtils.generateUUID(),
                    type: 'サイドスポット',
                    name: `サイドスポット (Z=${pos.z.toFixed(1)})`,
                    object: [lightL, lightR], 
                    target: [targetL, targetR], 
                    isPair: true,
                    isSideSpot: true, 
                    isFootLight: false,
                    isPinSpot: false,
                    isCenterPinSpot: false,
                    isMetubushi: false,
                    intensity: initialIntensityUI, 
                    color: '#ffffff',
                    x: pos.x, y: pos.y, z: pos.z,
                    targetX: targetX, targetY: targetY, targetZ: targetZ,
                    angle: lightL.angle
                };
                lights.push(lightData);
                
                if (index === 0) {
                    firstLightData = lightData;
                }
                updateLightIntensity(lightData.id, initialIntensityUI);
            });

            if (firstLightData) {
                const sideSpotGroupData = {
                    id: 'side-spot-group', 
                    type: 'サイドスポット',
                    name: 'サイドスポット (全4灯)',
                    isSideSpot: true,
                    isFootLight: false,
                    isPinSpot: false,
                    isCenterPinSpot: false,
                    isMetubushi: false,
                    intensity: 100,
                    color: '#ffffff',
                    x: firstLightData.x, y: firstLightData.y, z: firstLightData.z,
                    targetX: firstLightData.targetX, targetY: firstLightData.targetY, targetZ: firstLightData.targetZ,
                    angle: firstLightData.angle
                };
                createLightControlUI(sideSpotGroupData);
                updateLightIntensity('side-spot-group', 100);
            }
        }
        
        function addFootLights() {
            if (lights.some(l => l.isFootLight)) {
                alert("フットライトは既に追加されています。");
                return;
            }

            const xPositions = [14, 7, -7, -14];
            const y = -2.5;
            const z = 13;
            
            let firstLightData = null; 

            xPositions.forEach((xPos, index) => {
                const targetX = Math.sign(xPos) * (Math.abs(xPos) - 4);
                const targetY = STAGE_FLOOR_Y; 
                const targetZ = z - 3;

                const initialIntensityThreeJS = 1.0; 
                const initialIntensityUI = 100;

                // フットライトのシャドウを無効化 (disableShadow = true)
                const light = createSpotLight(initialIntensityThreeJS, false, null, true); 
                light.position.set(xPos, y, z);
                const targetObject = createLightTarget(targetX, targetY, targetZ);
                light.target = targetObject;
                scene.add(light);
                
                const lightData = {
                    id: THREE.MathUtils.generateUUID(),
                    type: 'フットライト',
                    name: `フットライト (X=${xPos})`,
                    object: [light], 
                    target: [targetObject], 
                    isPair: false,
                    isSideSpot: false, 
                    isFootLight: true, 
                    isPinSpot: false,
                    isCenterPinSpot: false,
                    isMetubushi: false,
                    intensity: initialIntensityUI, 
                    color: '#ffffff',
                    x: xPos, y: y, z: z,
                    targetX: targetX, targetY: targetY, targetZ: targetZ,
                    angle: light.angle
                };
                lights.push(lightData);
                
                if (index === 0) {
                    firstLightData = lightData;
                }
                updateLightIntensity(lightData.id, initialIntensityUI);
            });

            if (firstLightData) {
                const footLightGroupData = {
                    id: 'foot-light-group', 
                    type: 'フットライト',
                    name: 'フットライト (全4灯)',
                    isSideSpot: false,
                    isFootLight: true,
                    isPinSpot: false,
                    isCenterPinSpot: false,
                    isMetubushi: false,
                    intensity: 100,
                    color: '#ffffff',
                    x: firstLightData.x, y: firstLightData.y, z: firstLightData.z,
                    targetX: firstLightData.targetX, targetY: firstLightData.targetY, targetZ: firstLightData.targetZ,
                    angle: firstLightData.angle
                };
                createLightControlUI(footLightGroupData);
                updateLightIntensity('foot-light-group', 100);
            }
        }

        function addPinSpots() {
            if (lights.some(l => l.isPinSpot)) {
                alert("ピンスポットライトは既に追加されています。");
                return;
            }

            // センターピンスポットを先に追加
            addCenterPinSpot();

            const performerPositions = [];
            performerGroup.children.forEach(child => {
                if (child.geometry && child.geometry.type === 'CylinderGeometry') {
                    const globalPos = new THREE.Vector3();
                    child.getWorldPosition(globalPos);
                    // X=0のセンターは除く
                    if (Math.abs(globalPos.x) > 0.01) {
                         performerPositions.push(globalPos.x);
                    }
                }
            });

            const y = 18;
            let z = 14; 
            
            let firstLightData = null; 

            performerPositions.forEach((performerX, index) => {
                const xPos = performerX;
                
                const targetX = xPos;
                const targetY = STAGE_FLOOR_Y; 
                const targetZ = z - 4; 

                const initialIntensityThreeJS = 1.0; 
                const initialIntensityUI = 100;

                // シャドウは有効 (disableShadow = false)
                const light = createSpotLight(initialIntensityThreeJS, true); // ピンスポット用フレネル (照射角: 1/3)
                light.position.set(xPos, y, z);
                const targetObject = createLightTarget(targetX, targetY, targetZ);
                light.target = targetObject;
                scene.add(light);
                
                const lightData = {
                    id: THREE.MathUtils.generateUUID(),
                    type: 'ピンスポット',
                    name: `ピンスポット (X=${xPos.toFixed(1)})`,
                    object: [light], 
                    target: [targetObject], 
                    isPair: false,
                    isSideSpot: false, 
                    isFootLight: false,
                    isPinSpot: true, 
                    isCenterPinSpot: false,
                    isMetubushi: false,
                    intensity: initialIntensityUI, 
                    color: '#ffffff',
                    x: xPos, y: y, z: z,
                    targetX: targetX, targetY: targetY, targetZ: targetZ,
                    angle: light.angle
                };
                lights.push(lightData);
                
                if (index === 0) {
                    firstLightData = lightData;
                }
                updateLightIntensity(lightData.id, initialIntensityUI);
            });

            if (firstLightData) {
                const pinSpotGroupData = {
                    id: 'pin-spot-group', 
                    type: 'ピンスポット',
                    name: 'ピンスポット (全5灯)', 
                    isSideSpot: false,
                    isFootLight: false,
                    isPinSpot: true,
                    isCenterPinSpot: false,
                    isMetubushi: false,
                    intensity: 100,
                    color: '#ffffff',
                    x: firstLightData.x, y: firstLightData.y, z: firstLightData.z,
                    targetX: firstLightData.targetX, targetY: firstLightData.targetY, targetZ: firstLightData.targetZ,
                    angle: firstLightData.angle
                };
                createLightControlUI(pinSpotGroupData);
                updateLightIntensity('pin-spot-group', 100);
            }
        }
        
        function addCenterPinSpot() {
             if (lights.some(l => l.isCenterPinSpot)) {
                alert("センターピンスポットライトは既に追加されています。");
                return;
            }

            const x = 0;
            const y = 18;
            const z = 14; 
            
            const targetX = x;
            const targetY = STAGE_FLOOR_Y; 
            const targetZ = z - 4; // 10

            const initialIntensityThreeJS = 1.0; 
            const initialIntensityUI = 100;
            
            // センターピンスポットの初期照射角は、他のピンスポットと同じ $0.1409$ rad (約 8度)にする
            const initialAngle = 0.1409; 
            // シャドウは有効 (disableShadow = false)
            const light = createSpotLight(initialIntensityThreeJS, false, initialAngle); 
            light.position.set(x, y, z);
            const targetObject = createLightTarget(targetX, targetY, targetZ);
            light.target = targetObject;
            scene.add(light);
            
            const lightData = {
                id: THREE.MathUtils.generateUUID(),
                type: 'センターピンスポット',
                name: `センターピンスポット`,
                object: [light], 
                target: [targetObject], 
                isPair: false,
                isSideSpot: false, 
                isFootLight: false,
                isPinSpot: false, 
                isCenterPinSpot: true,
                isMetubushi: false,
                intensity: initialIntensityUI, 
                color: '#ffffff',
                x: x, y: y, z: z,
                targetX: targetX, targetY: targetY, targetZ: targetZ,
                angle: initialAngle
            };
            lights.push(lightData);
            
            // センターピンスポットは個別UIを持つ
            createLightControlUI(lightData);
            updateLightIntensity(lightData.id, initialIntensityUI);
        }

        // --- メツブシ機能の実装 ---
        function addMetubushiLights() {
            if (lights.some(l => l.isMetubushi)) {
                alert("メツブシライトは既に追加されています。");
                return;
            }

            const xPositions = [6, 2, -2, -6];
            const y = 3;
            const z = -10;
            const metubushiLightsData = [];

            // 照射角はピンスポットと同じ $0.1409$ rad (約 8度)を初期値とする
            const fixedAngleDegrees = 40;
            const fixedAngleRadians = fixedAngleDegrees * (Math.PI / 180); // 0.6981 rad
            const initialIntensityThreeJS = 1.0; 
            const initialIntensityUI = 100;
            const disableShadow = true; // シャドウは無効

            xPositions.forEach(xPos => {
                const targetX = xPos;
                const targetY = y;
                const targetZ = 10;

                // 照射角を指定してライトを作成
                const light = createSpotLight(initialIntensityThreeJS, false, fixedAngleRadians, disableShadow); 

                light.position.set(xPos, y, z);

                const targetObject = createLightTarget(targetX, targetY, targetZ);

                light.target = targetObject;

                scene.add(light);


                
                const lightData = {
                    id: THREE.MathUtils.generateUUID(),
                    type: 'メツブシ',
                    name: `メツブシ (X=${xPos})`,
                    object: [light], 
                    target: [targetObject], 
                    isPair: false,
                    isSideSpot: false, 
                    isFootLight: false,
                    isPinSpot: false, 
                    isCenterPinSpot: false,
                    isMetubushi: true,
                    intensity: initialIntensityUI, 
                    color: '#ffffff',
                    x: xPos, y: y, z: z,
                    targetX: targetX, targetY: targetY, targetZ: targetZ,
                    angle: fixedAngleRadians // 💡 固定角度を保存

                };
                lights.push(lightData);
                metubushiLightsData.push(lightData);
            });

            // 共通フェーダー用のグループデータを作成
            if (metubushiLightsData.length > 0) {
                const metubushiGroupData = {
                    id: 'metubushi-group', 
                    type: 'メツブシ',
                    name: 'メツブシ (4灯)',
                    isSideSpot: false,
                    isFootLight: false,
                    isPinSpot: false,
                    isCenterPinSpot: false,
                    isMetubushi: true,
                    intensity: 100,
                    color: '#ffffff',
                    x: metubushiLightsData[0].x, y: metubushiLightsData[0].y, z: metubushiLightsData[0].z,
                    targetX: metubushiLightsData[0].targetX, targetY: metubushiLightsData[0].targetY, targetZ: metubushiLightsData[0].targetZ,
                    angle: fixedAngleRadians // 💡 固定角度をグループデータにも保存

                };
                createLightControlUI(metubushiGroupData);
                // グループデータでまとめて強度更新 (強度2倍のロジックが適用される)
                updateLightIntensity('metubushi-group', 100); 
            }
        }
        // --- メツブシ機能の実装 終了 ---


        function deleteLight(lightId) {
            let lightsToRemove = [];

            if (lightId === 'side-spot-group' || lightId === 'foot-light-group' || lightId === 'pin-spot-group' || lightId === 'metubushi-group') {
                const isSide = lightId === 'side-spot-group';
                const isFoot = lightId === 'foot-light-group';
                const isPin = lightId === 'pin-spot-group';
                const isMetubushi = lightId === 'metubushi-group';
                
                lightsToRemove = lights.filter(l => (isSide && l.isSideSpot) || (isFoot && l.isFootLight) || (isPin && l.isPinSpot) || (isMetubushi && l.isMetubushi));
                
                lightsToRemove.forEach(lightData => {
                    lightData.object.forEach(light => { scene.remove(light); });
                    lightData.target.forEach(target => { scene.remove(target); });
                });
                
                lightsToRemove.forEach(lightData => {
                    const idx = lights.indexOf(lightData);
                    if (idx > -1) lights.splice(idx, 1);
                });

                const uiElement = document.getElementById(`light-control-${lightId}`);
                if (uiElement) { uiElement.remove(); }

            } else {
                const index = lights.findIndex(l => l.id === lightId);
                if (index === -1) return;
                const lightData = lights[index];
                
                lightData.object.forEach(light => { scene.remove(light); });
                lightData.target.forEach(target => { scene.remove(target); });
                
                if (transformControls.object && (lightData.object.includes(transformControls.object.parent) || lightData.target.includes(transformControls.object))) {
                     transformControls.detach();
                }

                const uiElement = document.getElementById(`light-control-${lightId}`);
                if (uiElement) { uiElement.remove(); }
                lights.splice(index, 1);
            }
            
            transformControls.detach();
        }

        // UI表示/非表示の切り替え
        function toggleDetails(id) {
            const group = document.getElementById(`light-control-${id}`);
            group.classList.toggle('expanded');
        }

        function createLightControlUI(lightData) {
            const group = document.createElement('div');
            group.className = 'control-group';
            group.id = `light-control-${lightData.id}`;
            
            const isGroup = lightData.isSideSpot || lightData.isFootLight || lightData.isPinSpot;
            const isCenterPin = lightData.isCenterPinSpot;
            const isMetubushi = lightData.isMetubushi;
            
            // 角度を度で計算
            const currentAngleDeg = lightData.angle * (180 / Math.PI);
            
            // --- 【変更点2】センターピンのみ角度可変UIを表示 ---
            const angleControlHtml = (isCenterPin) ? `
                <h4>照射角 (Angle) [可変]</h4>
                <div class="slider-wrapper">
                    <input type="range" id="angle-${lightData.id}" min="5" max="90" step="1" value="${currentAngleDeg.toFixed(0)}" 
                           oninput="updateLightAngle('${lightData.id}', this.value)">
                    <span id="angle-value-${lightData.id}">${currentAngleDeg.toFixed(0)}°</span>
                </div>
            ` : `
                <h4>照射角 (Angle) [固定 ${currentAngleDeg.toFixed(1)}°]</h4>
            `;
            // -------------------------------------------------------------

            
            group.innerHTML = `
                <h4 id="name-display-${lightData.id}" onclick="toggleDetails('${lightData.id}')">${lightData.name}</h4>
                <div class="always-visible">
                    <label>フェーダー名:</label>
                    <input type="text" id="name-input-${lightData.id}" class="name-input" 
                           value="${lightData.name}" onchange="updateLightName('${lightData.id}', this.value)">
                    
                    <label>強度 (Fader)</label>
                    <div class="slider-wrapper">
                        <input type="range" id="intensity-${lightData.id}" min="0" max="100" value="${lightData.intensity}" 
                               oninput="updateLightIntensity('${lightData.id}', this.value)">
                        <span id="value-${lightData.id}">${lightData.intensity}</span>
                    </div>
                </div>

                <div class="light-details">
                    <label>色 (Color)</label>
                    <input type="color" id="color-${lightData.id}" value="${lightData.color}"
                               onchange="updateLightColor('${lightData.id}', this.value)">
                    
                    ${angleControlHtml}

                    <h4>位置 (Position) [L側]</h4>
                    <label>上手/下手 (X):</label><input type="number" id="posX-${lightData.id}" class="coord-input" step="0.5" value="${lightData.x.toFixed(1)}" onchange="updateLightPosition('${lightData.id}')" ${isGroup || isCenterPin || isMetubushi ? 'disabled' : ''}>
                    <label>高さ (Y):</label><input type="number" id="posY-${lightData.id}" class="coord-input" step="0.5" value="${lightData.y.toFixed(1)}" onchange="updateLightPosition('${lightData.id}')" ${isGroup || isCenterPin || isMetubushi ? 'disabled' : ''}>
                    <label>手前/奥 (Z):</label><input type="number" id="posZ-${lightData.id}" class="coord-input" step="0.5" value="${lightData.z.toFixed(1)}" onchange="updateLightPosition('${lightData.id}')" ${isGroup || isCenterPin || isMetubushi ? 'disabled' : ''}>

                    <h4>向き (Target) [L側]</h4>
                    <label>上手/下手 (X):</label><input type="number" id="targetX-${lightData.id}" class="coord-input" step="0.5" value="${lightData.targetX.toFixed(1)}" onchange="updateLightTarget('${lightData.id}')" ${isGroup || isCenterPin || isMetubushi ? 'disabled' : ''}>
                    <label>高さ (Y):</label><input type="number" id="targetY-${lightData.id}" class="coord-input" step="0.5" value="${lightData.targetY.toFixed(1)}" onchange="updateLightTarget('${lightData.id}')" ${isGroup || isCenterPin || isMetubushi ? 'disabled' : ''}>
                    <label>手前/奥 (Z):</label><input type="number" id="targetZ-${lightData.id}" class="coord-input" step="0.5" value="${lightData.targetZ.toFixed(1)}" onchange="updateLightTarget('${lightData.id}')" ${isGroup || isCenterPin || isMetubushi ? 'disabled' : ''}>
                    
                    <button class="delete-btn" onclick="deleteLight('${lightData.id}')">フレネルを削除</button>
                </div>
            `;

            document.getElementById('light-list').appendChild(group);
            
            // UI上の座標が変更される可能性があるライトは初期値設定
            if (!isGroup && !isCenterPin && !isMetubushi) {
                document.getElementById(`targetX-${lightData.id}`).value = lightData.targetX.toFixed(1);
                document.getElementById(`targetY-${lightData.id}`).value = lightData.targetY.toFixed(1);
                document.getElementById(`targetZ-${lightData.id}`).value = lightData.targetZ.toFixed(1);
            }
        }

        // --- ドラッグ操作の処理 ---
        
        function onLightTransformChange() {
            const lightData = getActiveLightData();
            if (!lightData) return;
            
            // グループ化、またはセンターピン、メツブシライトは個別のドラッグによる座標変更をUIに反映させない
            if (lightData.isSideSpot || lightData.isFootLight || lightData.isPinSpot || lightData.isCenterPinSpot || lightData.isMetubushi) return;

            const transformedObject = transformControls.object;
            const isLightBulb = transformedObject.name === "BulbModel";
            
            const newPos = transformedObject.parent ? transformedObject.parent.position : transformedObject.position;

            if (isLightBulb) {
                lightData.x = newPos.x; lightData.y = newPos.y; lightData.z = newPos.z;
                
                document.getElementById(`posX-${lightData.id}`).value = newPos.x.toFixed(1);
                document.getElementById(`posY-${lightData.id}`).value = newPos.y.toFixed(1);
                document.getElementById(`posZ-${lightData.id}`).value = newPos.z.toFixed(1);

                if (lightData.isPair) { lightData.object[1].position.set(-newPos.x, newPos.y, newPos.z); }
                
                updateLightTarget(lightData.id, newPos.x, STAGE_FLOOR_Y, newPos.z);
                
            } else {
                lightData.targetX = newPos.x; lightData.targetY = newPos.y; lightData.targetZ = newPos.z;
                document.getElementById(`targetX-${lightData.id}`).value = newPos.x.toFixed(1);
                document.getElementById(`targetY-${lightData.id}`).value = newPos.y.toFixed(1);
                document.getElementById(`targetZ-${lightData.id}`).value = newPos.z.toFixed(1);
                if (lightData.isPair) { 
                    lightData.target[1].position.set(-newPos.x, newPos.y, newPos.z);
                    lightData.target[1].updateMatrixWorld();
                }
            }
        }


        function onMouseDown(event) {
            if (event.target.closest('#ui-panel')) return;
            if (event.shiftKey) {
                transformControls.detach(); 
                return;
            }

            const rect = renderer.domElement.getBoundingClientRect();
            const mouse = new THREE.Vector2();
            mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
            mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(mouse, camera);

            const interactableObjects = [];
            lights.forEach(ld => {
                ld.object.forEach(light => {
                    if (light.children.length > 0) interactableObjects.push(light.children[0]);
                });
                ld.target.forEach(target => {
                    interactableObjects.push(target);
                });
            });

            const intersects = raycaster.intersectObjects(interactableObjects, true);

            if (intersects.length > 0) {
                const selectedObject = intersects[0].object;
                
                let objectToAttach;
                let mode = "translate";
                if (selectedObject.name === "LightTarget") {
                    objectToAttach = selectedObject;
                } 
                else if (selectedObject.name === "BulbModel") {
                    objectToAttach = selectedObject.parent;
                } else {
                    return;
                }
                
                const lightData = lights.find(ld => ld.object.some(o => o === objectToAttach) || ld.target.includes(objectToAttach));
                // グループ化されたライト、センターピン、メツブシライトはドラッグを無効化
                if (lightData && (lightData.isSideSpot || lightData.isFootLight || lightData.isPinSpot || lightData.isCenterPinSpot || lightData.isMetubushi)) {
                    transformControls.detach();
                    return; 
                }

                if (transformControls.object === objectToAttach) {
                    transformControls.detach();
                } else {
                    transformControls.attach(objectToAttach);
                    transformControls.setMode(mode);
                }
            } else {
                transformControls.detach();
            }
        }
        
        function updateLightTarget(lightId, x, y, z) {
            const lightData = lights.find(l => l.id === lightId);
            if (!lightData) return;

            if (lightData.isSideSpot || lightData.isFootLight || lightData.isPinSpot || lightData.isCenterPinSpot || lightData.isMetubushi) return;

            if (arguments.length === 4) {
                lightData.targetX = x;
                lightData.targetY = y;
                lightData.targetZ = z;
            } else {
                lightData.targetX = parseFloat(document.getElementById(`targetX-${lightId}`).value);
                lightData.targetY = parseFloat(document.getElementById(`targetY-${lightId}`).value);
                lightData.targetZ = parseFloat(document.getElementById(`targetZ-${lightId}`).value);
            }

            lightData.target[0].position.set(lightData.targetX, lightData.targetY, lightData.targetZ);
            lightData.target[0].updateMatrixWorld();
            
            if (lightData.isPair) {
                lightData.target[1].position.set(-lightData.targetX, lightData.targetY, lightData.targetZ);
                lightData.target[1].updateMatrixWorld();
            }

            document.getElementById(`targetX-${lightData.id}`).value = lightData.targetX.toFixed(1);
            document.getElementById(`targetY-${lightData.id}`).value = lightData.targetY.toFixed(1);
            document.getElementById(`targetZ-${lightData.id}`).value = lightData.targetZ.toFixed(1);
        }

        // --- UI制御関数 ---
        
        function updateLightName(lightId, name) {
            if (lightId === 'side-spot-group') {
                lights.filter(l => l.isSideSpot).forEach(l => l.name = name);
            } else if (lightId === 'foot-light-group') {
                lights.filter(l => l.isFootLight).forEach(l => l.name = name);
            } else if (lightId === 'pin-spot-group') {
                lights.filter(l => l.isPinSpot).forEach(l => l.name = name);
            } else if (lightId === 'metubushi-group') {
                lights.filter(l => l.isMetubushi).forEach(l => l.name = name);
            }
            const lightData = lights.find(l => l.id === lightId);
            if (lightData) {
                lightData.name = name;
                document.getElementById(`name-display-${lightData.id}`).textContent = name;
            }
        }
        
        function updateLightIntensity(lightId, value) {
            const uiIntensity = parseInt(value);
            const masterFactor = parseInt(masterIntensityEl.value) / 100;
            
            let lightsToUpdate = [];
            let isMetubushiGroup = false;

            if (lightId === 'side-spot-group') {
                lightsToUpdate = lights.filter(l => l.isSideSpot);
                lightsToUpdate.forEach(l => l.intensity = uiIntensity); 
            } else if (lightId === 'foot-light-group') {
                lightsToUpdate = lights.filter(l => l.isFootLight);
                lightsToUpdate.forEach(l => l.intensity = uiIntensity); 
            } else if (lightId === 'pin-spot-group') {
                lightsToUpdate = lights.filter(l => l.isPinSpot);
                lightsToUpdate.forEach(l => l.intensity = uiIntensity); 
            } else if (lightId === 'metubushi-group') {
                lightsToUpdate = lights.filter(l => l.isMetubushi);
                lightsToUpdate.forEach(l => l.intensity = uiIntensity); 
                isMetubushiGroup = true;
            } else {
                const lightData = lights.find(l => l.id === lightId);
                if (lightData) {
                    lightData.intensity = uiIntensity;
                    lightsToUpdate.push(lightData);
                    isMetubushiGroup = lightData.isMetubushi;
                }
            }
            
            // --- 【変更点3】メツブシの強度を2倍に補正するロジック ---
            lightsToUpdate.forEach(lightData => {
                let intensityFactor = 1.0;
                if (lightData.isMetubushi) {
                    intensityFactor = METUBUSHI_INTENSITY_MULTIPLIER;
                }
                
                const finalIntensity = (lightData.intensity / 100) * masterFactor * MAX_INTENSITY_FACTOR * intensityFactor;
                lightData.object.forEach(light => { light.intensity = finalIntensity; });
            });
            // -------------------------------------------------------------
            
            const valueEl = document.getElementById(`value-${lightId}`);
            if (valueEl) {
                valueEl.textContent = uiIntensity;
            }
        }
        
        function updateLightColor(lightId, hexColor) {
            let lightsToUpdate = [];

            if (lightId === 'side-spot-group') {
                lightsToUpdate = lights.filter(l => l.isSideSpot);
                lightsToUpdate.forEach(l => l.color = hexColor);
            } else if (lightId === 'foot-light-group') {
                lightsToUpdate = lights.filter(l => l.isFootLight);
                lightsToUpdate.forEach(l => l.color = hexColor);
            } else if (lightId === 'pin-spot-group') {
                lightsToUpdate = lights.filter(l => l.isPinSpot);
                lightsToUpdate.forEach(l => l.color = hexColor);
            } else if (lightId === 'metubushi-group') {
                lightsToUpdate = lights.filter(l => l.isMetubushi);
                lightsToUpdate.forEach(l => l.color = hexColor);
            } else {
                const lightData = lights.find(l => l.id === lightId);
                if (lightData) {
                    lightData.color = hexColor;
                    lightsToUpdate.push(lightData);
                }
            }

            lightsToUpdate.forEach(lightData => {
                lightData.object.forEach(light => {
                    light.color.set(hexColor);
                    if (light.children.length > 0) { light.children[0].material.color.set(hexColor); }
                });
            });
        }
        
        function updateMasterIntensity(value) {
            const masterFactor = value / 100;
            lights.forEach(lightData => {
                let intensityFactor = 1.0;
                // --- 【変更点4】マスターフェーダーでもメツブシの強度2倍を維持 ---
                if (lightData.isMetubushi) {
                    intensityFactor = METUBUSHI_INTENSITY_MULTIPLIER;
                }
                const finalIntensity = (lightData.intensity / 100) * masterFactor * MAX_INTENSITY_FACTOR * intensityFactor;
                // -------------------------------------------------------------
                lightData.object.forEach(light => { light.intensity = finalIntensity; });
            });
            masterValueEl.textContent = value;
        }

        function updateLightPosition(lightId) {
            const lightData = lights.find(l => l.id === lightId);
            if (!lightData || lightData.isSideSpot || lightData.isFootLight || lightData.isPinSpot || lightData.isCenterPinSpot || lightData.isMetubushi) return;

            const x = parseFloat(document.getElementById(`posX-${lightId}`).value);
            const y = parseFloat(document.getElementById(`posY-${lightId}`).value);
            const z = parseFloat(document.getElementById(`posZ-${lightId}`).value);
            
            lightData.x = x; lightData.y = y; lightData.z = z;
            lightData.object[0].position.set(x, y, z);
            if (lightData.isPair) { lightData.object[1].position.set(-x, y, z); }
            
            updateLightTarget(lightId, x, STAGE_FLOOR_Y, z);
        }

        // --- アニメーションループとレンダリング ---
        function animate() {
            requestAnimationFrame(animate);
            controls.update(); 
            renderer.render(scene, camera); 
        }

        function onWindowResize() {
            const width = window.innerWidth - 300;
            const height = window.innerHeight;
            camera.aspect = width / height;
            camera.updateProjectionMatrix();
            renderer.setSize(width, height);
        }

        // アプリケーション開始
        init();

    </script>
</body>
</html>


