<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SUT 戰術口令辨識生成器</title>
    <style>
        :root {
            --bg-color: #121417;
            --card-bg: #1e2229;
            --accent-green: #2ecc71;
            --accent-red: #e74c3c;
            --text-main: #f1f2f6;
            --text-muted: #a4b0be;
            --border-color: #2f3542;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            width: 100%;
            max-width: 600px;
            background-color: var(--card-bg);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 30px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.5);
        }

        h2 {
            margin-top: 0;
            color: var(--accent-green);
            text-align: center;
            letter-spacing: 1px;
            text-transform: uppercase;
            font-size: 1.4rem;
            border-bottom: 2px solid var(--border-color);
            padding-bottom: 15px;
        }

        .section-desc {
            font-size: 0.9rem;
            color: var(--text-muted);
            margin-bottom: 25px;
            text-align: center;
        }

        .config-group {
            margin-bottom: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(255,255,255,0.03);
            padding: 12px 15px;
            border-radius: 6px;
        }

        .config-group label {
            font-weight: 600;
            font-size: 0.95rem;
        }

        select {
            background-color: var(--bg-color);
            color: var(--text-main);
            border: 1px solid var(--border-color);
            padding: 8px 12px;
            border-radius: 4px;
            font-size: 0.95rem;
            outline: none;
        }

        button.generate-btn {
            width: 100%;
            background-color: var(--accent-green);
            color: #fff;
            border: none;
            padding: 14px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            transition: background 0.2s;
            text-transform: uppercase;
            letter-spacing: 1px;
            margin-top: 10px;
        }

        button.generate-btn:hover {
            background-color: #27ae60;
        }

        .result-box {
            margin-top: 30px;
            background-color: var(--bg-color);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            padding: 20px;
            display: none;
        }

        .result-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 12px;
            padding-bottom: 8px;
            border-bottom: 1px dashed var(--border-color);
        }

        .result-row:last-child {
            margin-bottom: 0;
            padding-bottom: 0;
            border-bottom: none;
        }

        .label-title {
            color: var(--text-muted);
            font-size: 0.9rem;
        }

        .label-value {
            font-weight: bold;
            font-size: 1.1rem;
            color: var(--accent-green);
            letter-spacing: 1px;
        }

        .time-badge {
            display: inline-block;
            background: rgba(231, 76, 60, 0.2);
            color: var(--accent-red);
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 0.8rem;
            font-weight: bold;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>SUT 戰術口令辨識生成器</h2>
    <div class="section-desc">自動化產生互不聯想之口令（Challenge & Password），確保夜間敵我識別安全。</div>

    <div class="config-group">
        <label for="timeSlot">時效設定：</label>
        <select id="timeSlot">
            <option value="current">當前時段 (立即生效)</option>
            <option value="h1">第一階段 (00:00 - 06:00)</option>
            <option value="h2">第二階段 (06:00 - 12:00)</option>
            <option value="h3">第三階段 (12:00 - 18:00)</option>
            <option value="h4">第四階段 (18:00 - 24:00)</option>
        </select>
    </div>

    <button class="generate-btn" onclick="generatePassword()">生成安全口令</button>

    <div class="result-box" id="resultBox">
        <div class="result-row">
            <span class="label-title">有效時段</span>
            <span id="resTime" class="time-badge">當前時段</span>
        </div>
        <div class="result-row">
            <span class="label-title">口令 (Challenge - 哨兵喊)</span>
            <span id="resChallenge" class="label-value">-</span>
        </div>
        <div class="result-row">
            <span class="label-title">密語 (Password - 來者答)</span>
            <span id="resPassword" class="label-value">-</span>
        </div>
    </div>
</div>

<script>
    // 獨立詞庫設計（確保字數短、發音清晰、無直接語意關聯）
    const poolA = [
        "鐵錘", "閃電", "獵鷹", "奔狼", "黑熊", 
        "荒野", "風暴", "利刃", "磐石", "鐵騎", 
        "銀狐", "冰川", "赤焰", "叢林", "戰斧"
    ];

    const poolB = [
        "地平線", "三角洲", "信號旗", "北極星", "指南針", 
        "深邃谷", "回音壁", "防禦線", "集結點", "九號樁", 
        "孤立樹", "岩石區", "撤退道", "老鷹巢", "界線標"
    ];

    function getRandomItem(arr) {
        return arr[Math.floor(Math.random() * arr.length)];
    }

    function generatePassword() {
        // 隨機抽樣
        let challenge = getRandomItem(poolA);
        let password = getRandomItem(poolB);

        // 確保前後詞彙不會因為隨機重複（雖然機率低）
        while (challenge === password) {
            password = getRandomItem(poolB);
        }

        // 取得時段名稱
        const timeSelect = document.getElementById("timeSlot");
        const timeText = timeSelect.options[timeSelect.selectedIndex].text;

        // 顯示結果
        document.getElementById("resTime").innerText = timeText;
        document.getElementById("resChallenge").innerText = challenge;
        document.getElementById("resPassword").innerText = password;
        
        document.getElementById("resultBox").style.display = "block";
    }
</script>

</body>
</html># pas
