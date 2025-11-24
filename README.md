<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>分享我的電子名片</title>
    <script charset="utf-8" src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #f5f6fa;
            color: #333;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }

        .card {
            background: white;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            padding: 40px 30px;
            text-align: center;
            max-width: 400px;
            width: 100%;
        }

        .avatar {
            width: 100px;
            height: 100px;
            background-color: #e1e1e1;
            border-radius: 50%;
            margin: 0 auto 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 40px;
        }

        h1 {
            margin: 0 0 10px;
            font-size: 24px;
            color: #1a1a1a;
        }

        p {
            margin: 0 0 30px;
            color: #666;
            line-height: 1.6;
        }

        .btn {
            background-color: #06C755;
            color: white;
            border: none;
            padding: 16px 32px;
            border-radius: 50px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            width: 100%;
            transition: transform 0.2s, background-color 0.2s;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }

        .btn:active {
            transform: scale(0.98);
            background-color: #05b34c;
        }

        .btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }

        .status {
            margin-top: 15px;
            font-size: 14px;
            color: #999;
        }

        /* 隱藏未登入時的介面，避免閃爍 */
        #content {
            display: none;
        }
    </style>
</head>
<body>

    <div id="loading">載入中...</div>

    <div class="card" id="content">
        <div class="avatar">👤</div>
        <h1>分享電子名片</h1>
        <p>點擊下方按鈕，選擇好友或群組，<br>立即發送精美的多頁式名片！</p>

        <button id="shareBtn" class="btn">
            <span>分享給好友</span>
        </button>

        <div class="status" id="statusText">準備就緒</div>
    </div>

    <script>
        // ==========================================
        // 設定區域 (請修改這裡)
        // ==========================================

        // 1. 請填入你在 LINE Developers 申請到的 LIFF ID
        const MY_LIFF_ID = "2008560642-Vk3EQw2Y";

        // 2. 這是你要發送的 Flex Message (這裡預設是一個範例名片)
        // 你可以使用 Flex Message Simulator 製作好後，把 JSON 貼過來替換這裡
        const FLEX_MESSAGE = {
            "type": "flex",
            "altText": "這是一張電子名片，請在手機上查看",
            "contents": {
                "type": "carousel",
                "contents": [
                {
                    "type": "bubble",
                    "size": "mega",
                    "hero": {
                    "type": "image",
                    "url": "https://scdn.line-apps.com/n/channel_devcenter/img/fx/01_1_cafe.png",
                    "size": "full",
                    "aspectRatio": "20:13",
                    "aspectMode": "cover"
                    },
                    "body": {
                    "type": "box",
                    "layout": "vertical",
                    "contents": [
                        {
                        "type": "text",
                        "text": "Brown Cafe",
                        "weight": "bold",
                        "size": "xl"
                        },
                        {
                        "type": "text",
                        "text": "台北市信義區咖啡路一段 123 號",
                        "size": "sm",
                        "color": "#999999",
                        "margin": "md",
                        "flex": 0
                        }
                    ]
                    },
                    "footer": {
                    "type": "box",
                    "layout": "vertical",
                    "spacing": "sm",
                    "contents": [
                        {
                        "type": "button",
                        "style": "link",
                        "height": "sm",
                        "action": {
                            "type": "uri",
                            "label": "撥打電話",
                            "uri": "https://line.me/"
                        }
                        },
                        {
                        "type": "button",
                        "style": "link",
                        "height": "sm",
                        "action": {
                            "type": "uri",
                            "label": "查看網站",
                            "uri": "https://line.me/"
                        }
                        }
                    ],
                    "flex": 0
                    }
                }
                ]
            }
        };

        // ==========================================
        // 程式邏輯 (通常不需要修改)
        // ==========================================

        const statusText = document.getElementById('statusText');
        const shareBtn = document.getElementById('shareBtn');
        const loadingDiv = document.getElementById('loading');
        const contentDiv = document.getElementById('content');

        // 初始化 LIFF
        async function initializeLiff() {
            try {
                await liff.init({ liffId: MY_LIFF_ID });

                loadingDiv.style.display = 'none';
                contentDiv.style.display = 'block';

                if (!liff.isLoggedIn()) {
                    // 如果沒登入，就去登入
                    liff.login();
                } else {
                    // 已登入，檢查環境
                    if (!liff.isInClient()) {
                        statusText.innerText = "建議在 LINE App 內開啟此連結";
                    }
                }
            } catch (error) {
                console.error('LIFF 初始化失敗', error);
                loadingDiv.innerText = "初始化失敗，請檢查 LIFF ID";
            }
        }

        // 綁定按鈕事件
        shareBtn.addEventListener('click', async () => {
            if (!liff.isLoggedIn()) {
                liff.login();
                return;
            }

            if (liff.isApiAvailable('shareTargetPicker')) {
                try {
                    const pickerResult = await liff.shareTargetPicker([FLEX_MESSAGE]);

                    if (pickerResult) {
                        // 發送成功
                        alert('名片已成功發送！');
                        liff.closeWindow(); // 關閉視窗
                    } else {
                        // 使用者取消
                        statusText.innerText = "已取消發送";
                    }
                } catch (error) {
                    console.error('發送錯誤', error);
                    statusText.innerText = "發送失敗，請確認 LIFF ID 設定";
                    alert('發送失敗，請確認你的 LINE Developers 有開啟 Share Target Picker');
                }
            } else {
                alert('您的 LINE 版本不支援此功能，請更新 LINE App');
            }
        });

        // 執行初始化
        initializeLiff();
    </script>
</body>
</html>
