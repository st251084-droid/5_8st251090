<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>シンプルリズムゲーム</title>
    <style>
        body {
            font-family: 'Hiragino Maru Gothic ProN', 'Zen Maru Gothic', 'Rounded M+ 1c', 'Meiryo', sans-serif;
            background-color: #fff0f5; /* 桜色 */
            color: #5c4033; /* 焦げ茶色 */
            text-align: center;
            margin: 0;
            padding: 20px;
        }

        h1 { margin-bottom: 10px; color: #d7003a; } /* 紅色 */

        /* スコアゲージ */
        #score-gauge-container {
            width: 800px;
            max-width: 90%;
            height: 25px;
            background-color: #fffae8;
            border: 3px solid #d4a3b3;
            border-radius: 15px;
            margin: 0 auto 15px auto;
            overflow: hidden;
            box-shadow: inset 0 2px 5px rgba(92, 64, 51, 0.1);
        }

        #score-gauge-bar {
            width: 0%;
            height: 100%;
            background: linear-gradient(90deg, #f08300, #e95464);
            box-shadow: inset 0 3px 5px rgba(255, 255, 255, 0.4); /* ぷっくり感 */
            transition: width 0.3s ease-out;
        }

        #game-area {
            position: relative;
            width: 800px;
            max-width: 100%;
            height: 150px;
            background-color: #fffae8; /* 象牙色（薄いクリーム） */
            margin: 20px auto;
            border: 4px solid #d4a3b3; /* 和風な薄いピンク紫 */
            border-radius: 15px; /* 少し丸みを強く */
            overflow: hidden;
            box-shadow: 0 5px 15px rgba(92, 64, 51, 0.1);
        }

        /* 判定枠 */
        #target {
            position: absolute;
            top: 40px; /* 150/2 - 70/2 */
            left: 100px;
            width: 60px;
            height: 60px;
            border: 6px double rgba(215, 0, 58, 0.4); /* 紅色の二重線 */
            border-radius: 50%;
            box-sizing: border-box;
            z-index: 10;
        }

        /* 音符（ノーツ） */
        .note {
            position: absolute;
            top: 45px;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            box-sizing: border-box;
            box-shadow: 2px 2px 0px rgba(92, 64, 51, 0.2); /* ポップな影 */
        }

        /* ノーツの内側に和風の模様（太鼓風の丸）をつける */
        .note::after {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 20px;
            height: 20px;
            border-radius: 50%;
            border: 3px solid #fff;
            opacity: 0.9;
        }

        .red {
            background-color: #e95464; /* 紅色 */
            border: 3px solid #fff;
        }

        .blue {
            background-color: #00a3af; /* 浅葱色 */
            border: 3px solid #fff;
        }

        /* スコアとコンボ表示 */
        #stats {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 20px;
            color: #5c4033;
        }

        #combo-text { color: #d7003a; }

        /* 判定テキスト（良、可、不可） */
        #feedback {
            position: absolute;
            top: 10px;
            left: 100px;
            font-size: 28px;
            font-weight: bold;
            opacity: 0;
            transition: opacity 0.2s;
            z-index: 20;
            /* 明るい背景に合わせて白枠をつける */
            text-shadow: 2px 2px 0 #fff, -2px -2px 0 #fff, 2px -2px 0 #fff, -2px 2px 0 #fff; 
        }

        .controls-info {
            background: #e8ecc9; /* 薄い抹茶色 */
            color: #3f5127;      /* 濃い抹茶色 */
            display: inline-block;
            padding: 15px 30px;
            border-radius: 12px;
            margin-top: 20px;
            font-size: 18px;
            border: 2px dashed #b5caa0; /* 和風の点線 */
        }

        button {
            padding: 10px 30px;
            font-size: 20px;
            font-weight: bold;
            background-color: #f08300; /* 山吹色 */
            color: white;
            border: 2px solid #fff;
            border-radius: 25px; /* 可愛く丸っこく */
            cursor: pointer;
            margin-top: 10px;
            box-shadow: 0 4px 0px rgba(215, 0, 58, 0.2);
            transition: all 0.1s;
        }
        button:hover { 
            background-color: #d7003a; /* ホバーで紅色に */
            transform: translateY(2px); 
            box-shadow: 0 2px 0px rgba(215, 0, 58, 0.2); 
        }
        button:disabled { 
            background-color: #d3c0b5; 
            color: #fff;
            cursor: not-allowed; 
            transform: none; 
            box-shadow: none;
            border-color: transparent;
        }

        /* リザルト画面 */
        #result-screen {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(255, 240, 245, 0.95); /* 半透明の桜色 */
            z-index: 100;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #5c4033;
        }
        #result-screen h2 {
            font-size: 40px;
            color: #d7003a;
            margin-bottom: 10px;
        }
        #result-screen p {
            font-size: 24px;
            margin: 10px 0;
        }
    </style>
</head>
<body>

    <h1>🌸 和風ぽっぷ リズムゲーム 🍵</h1>
    
    <div id="score-gauge-container">
        <div id="score-gauge-bar"></div>
    </div>

    <div id="stats">
        残り時間: <span id="time">60</span>秒 | スコア: <span id="score">0</span> | コンボ: <span id="combo-text">0</span>
    </div>

    <div id="game-area">
        <div id="target"></div>
        <div id="feedback">良!</div>
        <div id="notes-container"></div>
    </div>

    <button id="start-btn" onclick="startGame()">ゲームスタート</button>

    <div class="controls-info">
        <p>🔴 <b>どん (赤)</b> : 「 <b>F</b> 」キー</p>
        <p>🔵 <b>かつ (青)</b> : 「 <b>J</b> 」キー</p>
    </div>

    <!-- リザルト画面 -->
    <div id="result-screen">
        <h2>ゲーム終了！</h2>
        <p>最終スコア: <span id="final-score">0</span></p>
        <button onclick="closeResult()">閉じる</button>
    </div>

    <script>
        let score = 0;
        let combo = 0;
        let notes = [];
        let isPlaying = false;
        let gameLoopId;
        let lastSpawnTime = 0;
        let currentSpawnDelay = 1000; // 次のノーツが出るまでの待ち時間
        let burstCountRemaining = 0;  // 連打の残り回数
        
        // 制限時間と難易度設定
        const GAME_DURATION = 60; // 60秒
        let timeLeft = GAME_DURATION;
        let timerInterval;
        
        const MAX_SCORE = 60000; // ゲージが満タンになる目安のスコア

        const BASE_SPEED = 5;      // 初期のスピード
        const MAX_SPEED = 14;      // 終盤のスピード
        const BASE_SPAWN_RATE = 1000; // 初期の出現間隔(ms)
        const MIN_SPAWN_RATE = 250;   // 終盤の出現間隔(ms)

        const TARGET_X = 100;    // 判定枠の左端位置

        const timeEl = document.getElementById('time');
        const scoreEl = document.getElementById('score');
        const comboEl = document.getElementById('combo-text');
        const feedbackEl = document.getElementById('feedback');
        const container = document.getElementById('notes-container');
        const startBtn = document.getElementById('start-btn');
        const gameArea = document.getElementById('game-area');
        const gaugeBar = document.getElementById('score-gauge-bar');
        
        // 画面幅に合わせて出現位置を調整
        let spawnX = gameArea.offsetWidth;

        // ウィンドウリサイズ時に出現位置を更新
        window.addEventListener('resize', () => {
             spawnX = gameArea.offsetWidth;
        });

        function startGame() {
            if (isPlaying) return;
            
            // フォーカスを当ててキーボード入力を受け付けやすくする
            window.focus();
            
            // 初期化
            isPlaying = true;
            score = 0;
            combo = 0;
            timeLeft = GAME_DURATION;
            updateStats();
            timeEl.innerText = timeLeft;
            container.innerHTML = '';
            notes = [];
            startBtn.disabled = true;
            startBtn.innerText = "プレイ中...";
            currentSpawnDelay = 1000;
            burstCountRemaining = 0;
            gaugeBar.style.width = '0%'; // ゲージをリセット

            // タイマー開始
            timerInterval = setInterval(() => {
                timeLeft--;
                timeEl.innerText = timeLeft;
                if (timeLeft <= 0) {
                    endGame();
                }
            }, 1000);

            // ゲームループ開始
            lastSpawnTime = performance.now(); // 最初の出現タイミングをリセット
            requestAnimationFrame(gameLoop);
        }

        function endGame() {
            isPlaying = false;
            clearInterval(timerInterval);
            cancelAnimationFrame(gameLoopId);
            
            startBtn.disabled = false;
            startBtn.innerText = "もう一度プレイ";
            
            // リザルト画面を表示
            document.getElementById('final-score').innerText = score;
            document.getElementById('result-screen').style.display = 'flex';
        }

        function closeResult() {
            document.getElementById('result-screen').style.display = 'none';
        }

        function gameLoop(timestamp) {
            if (!isPlaying) return;

            // 進行度合い (0.0 〜 1.0) を計算
            let progress = (GAME_DURATION - timeLeft) / GAME_DURATION;
            progress = Math.min(progress, 1); // 1を超えないようにする

            // 現在の難易度（スピード）を計算
            let currentSpeed = BASE_SPEED + (MAX_SPEED - BASE_SPEED) * Math.pow(progress, 1.2);

            // 音符を生成する
            if (timestamp - lastSpawnTime > currentSpawnDelay) {
                spawnNote(currentSpeed);
                lastSpawnTime = timestamp;

                // --- 次のノーツが出現するまでの時間を計算（不規則＆連打要素） ---
                if (burstCountRemaining > 0) {
                    // 連打中：短い間隔で出現（進行度に応じて 250ms 〜 120ms に変化）
                    currentSpawnDelay = 250 - (progress * 130);
                    burstCountRemaining--;
                } else {
                    // 通常の不規則な間隔をベースレートから計算
                    let baseRate = BASE_SPAWN_RATE - (BASE_SPAWN_RATE - MIN_SPAWN_RATE) * Math.pow(progress, 1.5);
                    
                    // ベースの間隔から ±40% ほどランダムにずらして不規則にする
                    currentSpawnDelay = baseRate * (0.6 + Math.random() * 0.8);

                    // 10%の確率で少し長めの「休符」を入れる
                    if (Math.random() < 0.1) {
                        currentSpawnDelay *= 1.8;
                    }

                    // 確率で連打（バースト）を予約する（進行度に応じて 10% 〜 60% の確率）
                    let burstProb = 0.1 + (progress * 0.5);
                    if (Math.random() < burstProb) {
                        // 連打の追加回数を決定（進行度に応じて 2連打 〜 最大5連打 になる）
                        let maxBurst = 1 + Math.floor(progress * 3);
                        burstCountRemaining = 1 + Math.floor(Math.random() * maxBurst);
                    }
                }
            }

            // 音符を動かす
            for (let i = 0; i < notes.length; i++) {
                let note = notes[i];
                note.x -= note.speed; // 難易度に応じた個別のスピードで移動
                note.element.style.left = note.x + 'px';

                // 判定枠を通り過ぎたらミス
                if (note.x < 30) {
                    note.element.remove();
                    notes.splice(i, 1);
                    i--; // 配列がずれるのでインデックスを戻す
                    
                    combo = 0;
                    showFeedback("残念...", "#89916b"); // ミスの色を海松色(くすんだ緑)に
                    updateStats();
                }
            }

            gameLoopId = requestAnimationFrame(gameLoop);
        }

        function spawnNote(speed) {
            // 赤か青をランダムで決定
            const isRed = Math.random() > 0.4; // 赤の方が少し出やすい
            const type = isRed ? 'red' : 'blue';
            
            const el = document.createElement('div');
            el.className = `note ${type}`;
            el.style.left = spawnX + 'px';
            container.appendChild(el);

            notes.push({
                x: spawnX,
                type: type,
                speed: speed, // 難易度に応じたスピードを記録
                element: el
            });
        }

        function showFeedback(text, color) {
            feedbackEl.innerText = text;
            feedbackEl.style.color = color;
            feedbackEl.style.opacity = 1;
            
            // 少ししたら文字を消す
            setTimeout(() => {
                feedbackEl.style.opacity = 0;
            }, 400);
        }

        function updateStats() {
            scoreEl.innerText = score;
            comboEl.innerText = combo;

            // ゲージの幅を更新
            let percentage = (score / MAX_SCORE) * 100;
            if (percentage > 100) percentage = 100; // 100%を超えないようにする
            gaugeBar.style.width = percentage + '%';
        }

        // キーボード入力の処理
        document.addEventListener('keydown', (e) => {
            if (!isPlaying) return;

            let keyType = null;
            if (e.key === 'f' || e.key === 'F') keyType = 'red';
            if (e.key === 'j' || e.key === 'J') keyType = 'blue';

            if (keyType && notes.length > 0) {
                // 一番左にある音符を取得
                let targetNote = notes[0];
                
                // 判定枠との距離を計算
                let distance = Math.abs(targetNote.x - TARGET_X);

                // 判定枠に近い場合（ヒット）
                if (distance < 50) {
                    if (targetNote.type === keyType) {
                        // 正しい色を押した
                        if (distance < 20) {
                            score += 300;
                            combo++;
                            showFeedback("あっぱれ!", "#e95464"); // 良を紅色に
                        } else {
                            score += 100;
                            combo++;
                            showFeedback("よい", "#86a373"); // 可を抹茶色に
                        }
                    } else {
                        // 間違った色を押した
                        combo = 0;
                        showFeedback("残念...", "#89916b"); // ミスを海松色に
                    }

                    // 叩いた音符を消す
                    targetNote.element.remove();
                    notes.shift(); // 配列の先頭を削除
                    updateStats();
                }
            }
        });
    </script>
</body>
</html>
