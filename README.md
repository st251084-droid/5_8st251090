<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>シンプルリズムゲーム</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #222;
            color: #fff;
            text-align: center;
            margin: 0;
            padding: 20px;
        }

        h1 { margin-bottom: 10px; }

        #game-area {
            position: relative;
            width: 800px;
            height: 150px;
            background-color: #444;
            margin: 20px auto;
            border: 4px solid #888;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 5px 15px rgba(0,0,0,0.5);
        }

        /* 判定枠 */
        #target {
            position: absolute;
            top: 40px; /* 150/2 - 70/2 */
            left: 100px;
            width: 60px;
            height: 60px;
            border: 5px solid rgba(255, 255, 255, 0.6);
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
            box-shadow: 2px 2px 5px rgba(0,0,0,0.5);
        }

        .red {
            background-color: #ff4d4d;
            border: 3px solid #fff;
        }

        .blue {
            background-color: #4da6ff;
            border: 3px solid #fff;
        }

        /* スコアとコンボ表示 */
        #stats {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 20px;
        }

        #combo-text { color: #ffeb3b; }

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
            text-shadow: 2px 2px 2px #000;
        }

        .controls-info {
            background: #333;
            display: inline-block;
            padding: 15px 30px;
            border-radius: 8px;
            margin-top: 20px;
            font-size: 18px;
        }

        button {
            padding: 10px 30px;
            font-size: 20px;
            font-weight: bold;
            background-color: #ff9800;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }
        button:hover { background-color: #e68a00; }
        button:disabled { background-color: #777; cursor: not-allowed; }
    </style>
</head>
<body>

    <h1>🥁 太鼓風 シンプルリズムゲーム</h1>
    
    <div id="stats">
        スコア: <span id="score">0</span> | コンボ: <span id="combo-text">0</span>
    </div>

    <div id="game-area">
        <div id="target"></div>
        <div id="feedback">良!</div>
        <div id="notes-container"></div>
    </div>

    <button id="start-btn" onclick="startGame()">ゲームスタート</button>

    <div class="controls-info">
        <p>🔴 <b>ドン (赤)</b> : 「 <b>F</b> 」キー</p>
        <p>🔵 <b>カツ (青)</b> : 「 <b>J</b> 」キー</p>
    </div>

    <script>
        let score = 0;
        let combo = 0;
        let notes = [];
        let isPlaying = false;
        let gameLoopId;
        let lastSpawnTime = 0;
        
        // 設定
        const TARGET_X = 100;    // 判定枠の左端位置
        const NOTE_SPEED = 6;    // 音符が流れるスピード
        const SPAWN_RATE = 800;  // 音符が出る間隔 (ミリ秒)

        const scoreEl = document.getElementById('score');
        const comboEl = document.getElementById('combo-text');
        const feedbackEl = document.getElementById('feedback');
        const container = document.getElementById('notes-container');
        const startBtn = document.getElementById('start-btn');

        function startGame() {
            if (isPlaying) return;
            
            // 初期化
            isPlaying = true;
            score = 0;
            combo = 0;
            updateStats();
            container.innerHTML = '';
            notes = [];
            startBtn.disabled = true;
            startBtn.innerText = "プレイ中...";

            // ゲームループ開始
            requestAnimationFrame(gameLoop);
        }

        function gameLoop(timestamp) {
            if (!isPlaying) return;

            // 音符を生成する
            if (timestamp - lastSpawnTime > SPAWN_RATE) {
                spawnNote();
                lastSpawnTime = timestamp;
            }

            // 音符を動かす
            for (let i = 0; i < notes.length; i++) {
                let note = notes[i];
                note.x -= NOTE_SPEED;
                note.element.style.left = note.x + 'px';

                // 判定枠を通り過ぎたらミス
                if (note.x < 30) {
                    note.element.remove();
                    notes.splice(i, 1);
                    i--; // 配列がずれるのでインデックスを戻す
                    
                    combo = 0;
                    showFeedback("ミス...", "#ff4d4d");
                    updateStats();
                }
            }

            gameLoopId = requestAnimationFrame(gameLoop);
        }

        function spawnNote() {
            // 赤か青をランダムで決定
            const isRed = Math.random() > 0.4; // 赤の方が少し出やすい
            const type = isRed ? 'red' : 'blue';
            
            const el = document.createElement('div');
            el.className = `note ${type}`;
            el.style.left = '800px';
            container.appendChild(el);

            notes.push({
                x: 800,
                type: type,
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
                            showFeedback("良!", "#ffeb3b");
                        } else {
                            score += 100;
                            combo++;
                            showFeedback("可", "#ffffff");
                        }
                    } else {
                        // 間違った色を押した
                        combo = 0;
                        showFeedback("ミス...", "#ff4d4d");
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
