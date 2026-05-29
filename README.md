<!-- index.html -->
<!doctype html>
<html lang="zh-HK">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>立體浮動對話框示例</title>
  <style>
    :root{
      --bg:#0f1724;
      --card:#0b1220;
      --accent:#7dd3fc;
      --text:#e6eef8;
      --shadow-color: rgba(2,6,23,0.6);
      --card-radius:16px;
      --card-padding:18px;
    }

    html,body{
      height:100%;
      margin:0;
      font-family:system-ui,-apple-system,"Segoe UI",Roboto,"Noto Sans","Helvetica Neue",Arial;
      /* 已更新為你選擇的五色橫向漸變背景 */
      background: linear-gradient(to right, #030304 100%, #04080C 25%, #060E15 50%, #050A0E 75%, #030304 100%);
      color:var(--text);
      display:flex;
      align-items:center;
      justify-content:center;
    }

    /* 包裹容器提供透視 */
    .stage{
      width:360px;
      max-width:90vw;
      perspective:1200px; /* 控制立體感強弱 */
      display:flex;
      flex-direction:column;
      gap:18px;
      align-items:center;
    }

    /* 浮動卡片 */
    .floating-card{
      width:100%;
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:var(--card-radius);
      padding:var(--card-padding);
      box-shadow:
        0 30px 60px -20px var(--shadow-color),
        0 10px 30px -10px rgba(0,0,0,0.6);
      transform-style:preserve-3d;
      transition: transform 420ms cubic-bezier(.2,.9,.3,1), box-shadow 420ms;
      transform: translateZ(0) rotateX(0deg) rotateY(0deg);
      position:relative;
      border: 1px solid rgba(255,255,255,0.03);
      backdrop-filter: blur(6px) saturate(120%);
    }

    /* 卡片被激活時更浮起 */
    .floating-card.is-floating{
      transform: translateY(-18px) translateZ(40px) rotateX(6deg);
      box-shadow:
        0 60px 90px -30px rgba(2,6,23,0.75),
        0 18px 40px -12px rgba(0,0,0,0.6);
    }

    /* 對話內容 */
    .chat-line{
      display:flex;
      gap:12px;
      align-items:flex-start;
    }

    .avatar{
      width:44px;
      height:44px;
      border-radius:10px;
      background:linear-gradient(135deg,var(--accent),#60a5fa);
      flex:0 0 44px;
      transform: translateZ(30px);
      box-shadow: 0 6px 18px rgba(0,0,0,0.45);
    }

    .bubble{
      background: linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.01));
      padding:12px 14px;
      border-radius:12px;
      max-width:78%;
      box-shadow: 0 6px 18px rgba(0,0,0,0.35);
      transform: translateZ(20px);
      line-height:1.45;
      font-size:14px;
    }

    /* 小尾巴 */
    .bubble::after{
      content:"";
      position:absolute;
      width:12px;
      height:12px;
      transform: rotate(45deg) translateZ(20px);
      background: linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.01));
      left:72px;
      top:56px;
      border-radius:2px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.25);
    }

    /* 控制面板 */
    .controls{
      display:flex;
      gap:8px;
      align-items:center;
    }

    .btn{
      background:transparent;
      color:var(--text);
      border:1px solid rgba(255,255,255,0.06);
      padding:8px 12px;
      border-radius:10px;
      cursor:pointer;
      font-size:13px;
      transition: background .18s, transform .12s;
    }
    .btn:active{ transform: translateY(1px); }
    .btn.primary{
      background: linear-gradient(90deg,#0369a1,#7dd3fc);
      color:#021022;
      border:none;
      box-shadow: 0 6px 18px rgba(7,89,133,0.18);
    }

    /* 小動畫：微幅擺動，讓卡片像在空中漂浮 */
    @keyframes floaty {
      0% { transform: translateY(0) translateZ(0) rotateX(0deg); }
      50% { transform: translateY(-6px) translateZ(8px) rotateX(2deg); }
      100% { transform: translateY(0) translateZ(0) rotateX(0deg); }
    }
    .floating-card.subtle{
      animation: floaty 4.8s ease-in-out infinite;
    }

    /* 響應式微調 */
    @media (max-width:420px){
      .avatar{ width:40px; height:40px; }
      .bubble{ font-size:13px; }
    }
  </style>
</head>
<body>
  <div class="stage" aria-live="polite">
    <div id="card" class="floating-card subtle" role="region" aria-label="對話框">
      <div class="chat-line" style="position:relative;">
        <div class="avatar" aria-hidden="true"></div>
        <div class="bubble">嗨，我係示範對話框。你可以按下面按鈕切換更強烈的立體浮動效果。</div>
      </div>
    </div>

    <div class="controls" aria-hidden="false">
      <button id="toggleFloat" class="btn">切換浮動</button>
      <button id="toggleSubtle" class="btn primary">開關微動</button>
    </div>
  </div>

  <script>
    const card = document.getElementById('card');
    const toggle = document.getElementById('toggleFloat');
    const subtleBtn = document.getElementById('toggleSubtle');

    toggle.addEventListener('click', () => {
      card.classList.toggle('is-floating');
    });

    subtleBtn.addEventListener('click', () => {
      card.classList.toggle('subtle');
      subtleBtn.textContent = card.classList.contains('subtle') ? '關閉微動' : '開啟微動';
    });

    // 可選：滑鼠移入時根據游標位置微調旋轉，增加互動立體感
    card.addEventListener('mousemove', (e) => {
      const rect = card.getBoundingClientRect();
      const cx = rect.left + rect.width/2;
      const cy = rect.top + rect.height/2;
      const dx = (e.clientX - cx) / rect.width;
      const dy = (e.clientY - cy) / rect.height;
      card.style.transform = `translateY(-6px) translateZ(30px) rotateX(${dy * 6}deg) rotateY(${dx * 8}deg)`;
    });
    card.addEventListener('mouseleave', () => {
      if (card.classList.contains('is-floating')) {
        card.style.transform = '';
      } else {
        card.style.transform = '';
      }
    });
  </script>
</body>
</html>
  
