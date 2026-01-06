<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>新店顏家 - 模擬聊天系統</title>
    <style>
        :root { --bg: #f0f2f5; --line-green: #00c300; }
        body { font-family: sans-serif; background: var(--bg); display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
        #setup-screen { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); text-align: center; }
        #chat-screen { display: none; width: 400px; height: 600px; background: #84a1c1; flex-direction: column; border-radius: 10px; overflow: hidden; }
        
        /* 聊天室 UI */
        #messages { flex: 1; overflow-y: auto; padding: 15px; display: flex; flex-direction: column; }
        .msg { margin-bottom: 10px; max-width: 80%; padding: 8px 12px; border-radius: 15px; font-size: 14px; position: relative; }
        .msg.bot { background: white; align-self: flex-start; }
        .msg.user { background: #95ec69; align-self: flex-end; }
        .name-label { font-size: 10px; color: #555; margin-bottom: 2px; }
        
        .input-area { background: white; padding: 10px; display: flex; border-top: 1px solid #ddd; }
        input { flex: 1; border: 1px solid #ddd; border-radius: 5px; padding: 8px; outline: none; }
        button { margin-left: 5px; cursor: pointer; border-radius: 5px; border: none; background: var(--line-green); color: white; padding: 0 15px; }
        
        /* 情緒顯示 */
        .status-bar { background: rgba(0,0,0,0.05); padding: 5px 10px; font-size: 11px; color: #333; }
    </style>
</head>
<body>

<div id="setup-screen">
    <h2>選擇模擬模式</h2>
    <div style="margin-bottom: 20px;">
        <p>選擇你要聊天的對象：</p>
        <select id="mode-select" onchange="toggleSingleSelect()">
            <option value="group">全體群組 (隨機回覆)</option>
            <option value="single">單一朋友</option>
        </select>
        <select id="friend-select" style="display:none; margin-top: 10px;">
            <option value="熊剛剛Jimmy">熊剛剛Jimmy</option>
            <option value="王水源">王水源</option>
            <option value="黃子庭 Troy">黃子庭 Troy</option>
            <option value="Gee Homie">Gee Homie</option>
        </select>
    </div>
    <button onclick="startChat()" style="padding: 10px 30px;">開始對話</button>
</div>

<div id="chat-screen">
    <div class="status-bar" id="system-status">模式：載入中...</div>
    <div id="messages"></div>
    <div class="input-area">
        <input type="text" id="user-input" placeholder="輸入訊息..." onkeypress="if(event.keyCode==13) sendMessage()">
        <button onclick="sendMessage()">傳送</button>
    </div>
</div>

<script>
// --- 核心數據庫 ---
const friendData = {
    "熊剛剛Jimmy": {
        mood: 0, // 0: 平靜, 1: 興奮/笑死, 2: 懶得理你
        keywords: [
            { word: "分手", res: ["風險對沖加槓桿的概念啦", "勸分不勸和，真的"], moodBoost: 1 },
            { word: "加班", res: ["耍大牌直接走人啊", "總經理可以沒特助，但我不能沒瓜吃"], moodBoost: 0 }
        ],
        timeRes: { night: "大半夜不睡覺", day: "嘿嘿，早安啊" },
        fallbacks: ["笑死", "我不信", "尊都假嘟", "好扯", "牛逼"]
    },
    "王水源": {
        mood: 0, // 0: 正常, 1: 痛苦, 2: 暴躁
        keywords: [
            { word: "嘉恩", res: ["我先修理她", "媽的想找個正常人交往好難", "還是要去東京"], moodBoost: 1 },
            { word: "錢", res: ["為了我的錢包謝謝", "爛貨，不然虧更多＝＝"], moodBoost: 2 }
        ],
        timeRes: { night: "影片", day: "圖片" }, // 水源愛發圖影
        fallbacks: ["都可以啊", "假賽啦", "幹你娘會被你們害死", "無言"]
    },
    "黃子庭 Troy": {
        mood: 0, // 0: 正常, 1: 煩燥
        keywords: [
            { word: "機票", res: ["改來改去操他媽，浪費我錢", "為什麼只有我是局外人"], moodBoost: 1 },
            { word: "日本", res: ["就說不要這麼早訂", "她們贏了，18年"], moodBoost: 1 }
        ],
        timeRes: { night: "白癡", day: "小硬" },
        fallbacks: ["操你媽", "Fuckable", "煩都煩死", "圖片"]
    },
    "Gee Homie": {
        mood: 0,
        keywords: [
            { word: "約", res: ["幾點到吳興街？", "明天晚上集合"], moodBoost: 0 },
            { word: "吃啥", res: ["看要吃啥都行除了火鍋", "慢來，吃東西不揪？"], moodBoost: 0 }
        ],
        timeRes: { night: "笑死", day: "行" },
        fallbacks: ["愛了", "不愛了", "三小啦", "好的"]
    }
};

let currentMode = 'group';
let selectedFriend = '';

function toggleSingleSelect() {
    const mode = document.getElementById('mode-select').value;
    document.getElementById('friend-select').style.display = (mode === 'single') ? 'block' : 'none';
}

function startChat() {
    currentMode = document.getElementById('mode-select').value;
    selectedFriend = document.getElementById('friend-select').value;
    
    document.getElementById('setup-screen').style.display = 'none';
    document.getElementById('chat-screen').style.display = 'flex';
    document.getElementById('system-status').innerText = `目前模式：${currentMode === 'group' ? '群組模擬' : '單挑 ' + selectedFriend}`;
    
    addMessage("系統", "連線成功，現在可以開始聊天了。", 'bot');
}

function sendMessage() {
    const input = document.getElementById('user-input');
    const text = input.value.trim();
    if (!text) return;

    // 1. 顯示使用者發送的訊息
    addMessage("你", text, 'user');
    input.value = '';

    // 2. 計算隨機延遲 (500ms 基礎 + 隨機 0~2000ms)
    // 這樣延遲會在 0.5 秒到 2.5 秒之間，比較像真人
    const randomDelay = 500 + Math.floor(Math.random() * 2000);

    // 3. 顯示「正在打字」狀態
    const status = document.getElementById('system-status');
    const originalStatus = status.innerText;
    status.innerText = "💬 某人正在輸入中...";

    setTimeout(() => {
        let responderName = "";
        let reply = "";

        if (currentMode === 'single') {
            responderName = selectedFriend;
            reply = generateLogic(selectedFriend, text);
        } else {
            // 群組模式：隨機挑選一位朋友
            const friends = Object.keys(friendData);
            responderName = friends[Math.floor(Math.random() * friends.length)];
            reply = generateLogic(responderName, text);
        }

        // 4. 發送回覆並還原狀態
        addMessage(responderName, reply, 'bot');
        status.innerText = originalStatus;

        // 5. 額外的小細節：如果是群組模式，有 20% 機率會有第二個人跟著回話（插嘴）
        if (currentMode === 'group' && Math.random() > 0.8) {
            triggerDoubleReply(responderName, text);
        }

    }, randomDelay);
}

// 模擬群組中「第二個人插嘴」的邏輯
function triggerDoubleReply(firstResponder, originalText) {
    const extraDelay = 800 + Math.floor(Math.random() * 1000);
    setTimeout(() => {
        const friends = Object.keys(friendData).filter(n => n !== firstResponder);
        const secondPerson = friends[Math.floor(Math.random() * friends.length)];
        const reply = generateLogic(secondPerson, originalText);
        addMessage(secondPerson, reply, 'bot');
    }, extraDelay);
}
function generateLogic(name, input) {
    const data = friendData[name];
    const now = new Date();
    const hour = now.getHours();

    // 1. 時效性判斷 (凌晨 1點-6點)
    if ((hour >= 1 && hour <= 6) && Math.random() > 0.7) {
        return data.timeRes.night;
    }

    // 2. 情緒與關鍵字邏輯
    for (let item of data.keywords) {
        if (input.includes(item.word)) {
            data.mood = item.moodBoost; // 設定情緒狀態
            const res = item.res[Math.floor(Math.random() * item.res.length)];
            
            // 情緒系統：如果暴躁度高，加個髒話結尾
            if (data.mood >= 1 && name === "黃子庭 Troy") return res + " 操";
            if (data.mood >= 1 && name === "王水源") return res + " 媽的痛苦";
            
            return res;
        }
    }

    // 3. 隨機兜底 (受情緒影響)
    if (data.mood > 0 && Math.random() > 0.5) {
        return name === "熊剛剛Jimmy" ? "笑死，真的" : data.fallbacks[0];
    }

    return data.fallbacks[Math.floor(Math.random() * data.fallbacks.length)];
}

function addMessage(sender, text, type) {
    const container = document.getElementById('messages');
    const div = document.createElement('div');
    div.className = `msg ${type}`;
    div.innerHTML = `<div class="name-label">${sender}</div>${text}`;
    container.appendChild(div);
    container.scrollTop = container.scrollHeight;
}
</script>

</body>
</html>
