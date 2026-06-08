# task-tracker[index.html](https://github.com/user-attachments/files/28710868/_1.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>筱君大隊 任務追蹤</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
<style>
:root {
  --bg: #0f0f13;
  --bg2: #17171e;
  --bg3: #1e1e28;
  --border: rgba(255,255,255,0.08);
  --border2: rgba(255,255,255,0.14);
  --text: #f0efe8;
  --text2: #9a9891;
  --text3: #5c5b56;
  --purple: #7c6df5;
  --purple-l: #a99ef8;
  --purple-bg: rgba(124,109,245,0.12);
  --green: #5ab878;
  --green-bg: rgba(90,184,120,0.12);
  --red: #e05c5c;
  --red-bg: rgba(224,92,92,0.12);
  --amber: #e0a93c;
  --amber-bg: rgba(224,169,60,0.12);
  --gold: #f0c060;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: 'Noto Sans TC', sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  padding-bottom: 3rem;
}

/* Header */
.header {
  background: linear-gradient(180deg, #1a1830 0%, var(--bg) 100%);
  border-bottom: 1px solid var(--border);
  padding: 2rem 1.5rem 1.5rem;
  text-align: center;
  position: relative;
  overflow: hidden;
}
.header::before {
  content: '';
  position: absolute;
  top: -60px; left: 50%; transform: translateX(-50%);
  width: 400px; height: 200px;
  background: radial-gradient(ellipse, rgba(124,109,245,0.2) 0%, transparent 70%);
  pointer-events: none;
}
.header-badge {
  display: inline-block;
  font-size: 11px;
  letter-spacing: 0.12em;
  color: var(--purple-l);
  background: var(--purple-bg);
  border: 1px solid rgba(124,109,245,0.3);
  padding: 4px 12px;
  border-radius: 20px;
  margin-bottom: 12px;
}
.header h1 {
  font-size: 26px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 6px;
  letter-spacing: -0.01em;
}
.header .sub {
  font-size: 13px;
  color: var(--text2);
}
.header .date {
  margin-top: 8px;
  font-size: 12px;
  color: var(--text3);
}

/* Week badge */
.week-tag {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: var(--amber-bg);
  border: 1px solid rgba(224,169,60,0.25);
  color: var(--amber);
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 20px;
  margin-top: 10px;
}

/* Tabs */
.tabs {
  display: flex;
  gap: 4px;
  padding: 1rem 1.5rem 0;
  overflow-x: auto;
  scrollbar-width: none;
  border-bottom: 1px solid var(--border);
}
.tabs::-webkit-scrollbar { display: none; }
.tab {
  flex-shrink: 0;
  padding: 8px 16px;
  font-size: 13px;
  font-weight: 500;
  color: var(--text2);
  border-radius: 8px 8px 0 0;
  cursor: pointer;
  border: 1px solid transparent;
  border-bottom: none;
  transition: all 0.2s;
  background: transparent;
}
.tab:hover { color: var(--text); background: var(--bg2); }
.tab.active {
  color: var(--purple-l);
  background: var(--bg2);
  border-color: var(--border);
  border-bottom-color: var(--bg2);
  margin-bottom: -1px;
}

/* Content */
.content { display: none; padding: 1.5rem; }
.content.active { display: block; }

/* Group info */
.group-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.25rem;
  flex-wrap: wrap;
  gap: 8px;
}
.group-name {
  font-size: 18px;
  font-weight: 700;
  color: var(--text);
}
.group-stats {
  font-size: 12px;
  color: var(--text2);
  display: flex;
  gap: 12px;
}
.group-stats span { display: flex; align-items: center; gap: 4px; }

/* Grid */
.members-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 10px;
  margin-bottom: 1.5rem;
}

/* Card */
.member-card {
  background: var(--bg2);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 12px;
  transition: border-color 0.2s;
}
.member-card:hover { border-color: var(--border2); }
.member-card.alert { border-color: rgba(224,92,92,0.35); }

/* Card header */
.card-top {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}
.avatar {
  width: 32px; height: 32px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 12px; font-weight: 700;
  flex-shrink: 0;
}
.card-name { font-size: 13px; font-weight: 700; color: var(--text); line-height: 1.2; }
.card-role { font-size: 10px; color: var(--text3); margin-top: 1px; }

/* Scores */
.scores {
  display: flex;
  gap: 6px;
  margin-bottom: 10px;
}
.score-box {
  flex: 1;
  background: var(--bg3);
  border-radius: 8px;
  padding: 5px 7px;
  text-align: center;
}
.score-label { font-size: 9px; color: var(--text3); margin-bottom: 1px; }
.score-val { font-size: 13px; font-weight: 700; color: var(--text); }
.score-val.good { color: var(--green); }
.score-val.warn { color: var(--amber); }
.score-val.bad { color: var(--red); }

/* Section */
.section-title {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: var(--text3);
  text-transform: uppercase;
  margin: 8px 0 4px;
  padding-top: 8px;
  border-top: 1px solid var(--border);
}

/* Task rows */
.task-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 3px;
  font-size: 10px;
  gap: 4px;
}
.task-num {
  font-size: 9px;
  color: var(--text3);
  min-width: 12px;
  flex-shrink: 0;
}
.task-name { color: var(--text); flex: 1; }
.task-name.done { color: var(--text3); text-decoration: line-through; text-decoration-color: var(--text3); }
.badge {
  font-size: 9px;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: 4px;
  flex-shrink: 0;
  white-space: nowrap;
}
.badge.done { background: var(--green-bg); color: var(--green); }
.badge.miss { background: var(--red-bg); color: var(--red); }
.badge.warn { background: var(--amber-bg); color: var(--amber); }

/* Notify section */
.notify-section {
  background: var(--bg2);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 1.25rem;
}
.notify-title {
  font-size: 14px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  gap: 8px;
}
.notify-icon {
  width: 28px; height: 28px;
  background: var(--purple-bg);
  border-radius: 8px;
  display: flex; align-items: center; justify-content: center;
  font-size: 14px;
}
.notify-preview {
  background: var(--bg3);
  border-radius: 10px;
  padding: 12px 14px;
  font-size: 12px;
  line-height: 1.8;
  color: var(--text2);
  white-space: pre-wrap;
  max-height: 260px;
  overflow-y: auto;
  margin-bottom: 10px;
  scrollbar-width: thin;
  scrollbar-color: var(--bg3) transparent;
}
.copy-btn {
  width: 100%;
  padding: 10px;
  background: var(--purple);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 700;
  font-family: 'Noto Sans TC', sans-serif;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  letter-spacing: 0.02em;
}
.copy-btn:hover { background: #6b5ee0; }
.copy-btn:active { transform: scale(0.98); }
.copy-btn.copied { background: var(--green); }

/* Footer */
.footer {
  text-align: center;
  padding: 1rem;
  font-size: 11px;
  color: var(--text3);
}

/* Alert colors per group */
.av-purple { background: rgba(124,109,245,0.2); color: var(--purple-l); }
.av-green  { background: rgba(90,184,120,0.2);  color: var(--green); }
.av-amber  { background: rgba(224,169,60,0.2);   color: var(--amber); }
.av-red    { background: rgba(224,92,92,0.2);    color: var(--red); }
.av-teal   { background: rgba(60,180,180,0.2);   color: #60d4d4; }
.av-pink   { background: rgba(220,100,160,0.2);  color: #f08ec8; }
</style>
</head>
<body>

<div class="header">
  <div class="header-badge">筱君大隊</div>
  <h1>🏆 任務追蹤儀表板</h1>
  <div class="sub">第3週 6/8（一）～ 6/14（日）</div>
  <div class="date">截至 6/8 更新</div>
  <div class="week-tag">⚠️ 主題親證2 開跑！2週內完成 +2800</div>
</div>

<div class="tabs">
  <div class="tab active" onclick="switchTab(0)">✨ 第九組</div>
  <div class="tab" onclick="switchTab(1)">💥 第十組</div>
  <div class="tab" onclick="switchTab(2)">🔥 第十一組</div>
  <div class="tab" onclick="switchTab(3)">🌸 第十二組</div>
</div>

<!-- 第九組 -->
<div class="content active" id="tab0">
  <div class="group-info">
    <div class="group-name">佛系但暴富組</div>
    <div class="group-stats">
      <span>7人</span>
      <span>隊長：王慧涵</span>
    </div>
  </div>
  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">慧</div><div><div class="card-name">王慧涵</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,700</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">岑</div><div><div class="card-name">王岑芯</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">子時入睡</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">宏</div><div><div class="card-name">王宏榮</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">芯</div><div><div class="card-name">黃芯璿</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,000</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">念</div><div><div class="card-name">鄒念穎</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">11,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val warn">+300</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">志</div><div><div class="card-name">廖志裕</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">10,200</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">雅</div><div><div class="card-name">黃雅琪</div><div class="card-role">龍女（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">9,700</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,000</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg9">✨【第九組｜佛系但暴富組】6/8 提醒！

━━━━━━━━━━━━━━
📋 今日定課：

👤 王宏榮（衝衝）→ 定課1完成，還差2個
👤 鄒念穎（金金）→ 定課2完成，還差1個
👤 廖志裕（丁丁）→ 今日尚未打卡
👤 黃雅琪（抱抱）→ 今日尚未打卡

━━━━━━━━━━━━━━
🎯 本週特殊任務：

👤 王慧涵（孫悟空）
✓ 親證分享
❌ 蓋雅、天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 王岑芯（觀音）
✓ 親證分享
❌ 蓋雅、天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 王宏榮（衝衝）
✓ 親證分享、欣賞夥伴、圓夢計畫(x2)
❌ 蓋雅、天使通話、參加活動(x2)、主題親證2

👤 黃芯璿（樂樂）
✓ 天使通話
❌ 蓋雅、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 鄒念穎（金金）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 廖志裕（丁丁）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 黃雅琪（抱抱）
✓ 親證分享
❌ 蓋雅、天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2

⚠️ 主題親證2（6/8開跑，2週內完成，+2800！）
佛系也要暴富，新的一週衝🙏✨</div>
    <button class="copy-btn" onclick="copyMsg('msg9', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>

<!-- 第十組 -->
<div class="content" id="tab1">
  <div class="group-info">
    <div class="group-name">十破天驚</div>
    <div class="group-stats"><span>6人</span><span>隊長：游佳霖</span></div>
  </div>
  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">庄</div><div><div class="card-name">蔡鎔庄</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">21,200</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,700</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">子時入睡</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">萱</div><div><div class="card-name">羅萱</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">文</div><div><div class="card-name">游文君</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+500</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">子時入睡</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">依</div><div><div class="card-name">王依涵</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">霖</div><div><div class="card-name">游佳霖</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,700</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">萱</div><div><div class="card-name">李雯萱</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">11,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+2,400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

  </div>
  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg10">💥【第十組｜十破天驚】6/8 提醒！

━━━━━━━━━━━━━━
📋 今日定課：

👤 羅萱（抱抱）→ 今日尚未打卡
👤 游佳霖（隊長）→ 今日尚未打卡

━━━━━━━━━━━━━━
🎯 本週特殊任務：

👤 蔡鎔庄（金金）
✓ 主題親證2、天使通話、欣賞夥伴、圓夢計畫(x2)
❌ 蓋雅、親證分享、參加活動(x2)

👤 羅萱（抱抱）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 游文君（觀音）
✓ 欣賞夥伴、圓夢計畫(x2)
❌ 蓋雅、天使通話、親證分享、參加活動(x2)、主題親證2

👤 王依涵（丁丁）
✓ 天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)
❌ 蓋雅、參加活動(x2)、主題親證2

👤 游佳霖（隊長）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 李雯萱（樂樂）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

⚠️ 主題親證2（6/8開跑，2週內完成，+2800！）
十破天驚繼續衝🔥</div>
    <button class="copy-btn" onclick="copyMsg('msg10', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>

<!-- 第十一組 -->
<div class="content" id="tab2">
  <div class="group-info">
    <div class="group-name">齊天戰神突擊隊</div>
    <div class="group-stats"><span>6人</span><span>隊長：黃怡駿</span></div>
  </div>
  <div class="members-grid">

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">玲</div><div><div class="card-name">許玲慧</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">17,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">丞</div><div><div class="card-name">郭丞淕</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,800</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+800</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">淑</div><div><div class="card-name">盧家淑</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+500</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">流動情緒</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">子時入睡</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">慈</div><div><div class="card-name">林嘉慈</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,500</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">煜</div><div><div class="card-name">洪煜棠</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,800</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">駿</div><div><div class="card-name">黃怡駿</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+500</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

  </div>
  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg11">🔥【第十一組｜齊天戰神突擊隊】6/8 提醒！

━━━━━━━━━━━━━━
📋 今日定課：

👤 許玲慧（抱抱）→ 今日尚未打卡
👤 林嘉慈（樂樂）→ 今日尚未打卡

━━━━━━━━━━━━━━
🎯 本週特殊任務：

👤 許玲慧（抱抱）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 郭丞淕（衝衝）
✓ 天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)
❌ 蓋雅、參加活動(x2)、主題親證2

👤 盧家淑（丁丁）
✓ 天使通話、欣賞夥伴、親證分享
❌ 蓋雅、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 林嘉慈（樂樂）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 洪煜棠（副隊長）
✓ 親證分享
❌ 蓋雅、天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 黃怡駿（隊長）
✓ 親證分享、欣賞夥伴
❌ 蓋雅、天使通話、圓夢計畫(x2)、參加活動(x2)、主題親證2

⚠️ 主題親證2（6/8開跑，2週內完成，+2800！）
戰神突擊隊加油衝💪</div>
    <button class="copy-btn" onclick="copyMsg('msg11', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>

<!-- 第十二組 -->
<div class="content" id="tab3">
  <div class="group-info">
    <div class="group-name">修心之路</div>
    <div class="group-stats"><span>6人</span><span>隊長：郭筱婷</span></div>
  </div>
  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">惠</div><div><div class="card-name">陳惠玲</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">湘</div><div><div class="card-name">黃湘庭</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val warn">+200</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">哲</div><div><div class="card-name">許哲豪</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">筱</div><div><div class="card-name">郭筱婷</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,200</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">芷</div><div><div class="card-name">王芷盈</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,700</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">冠</div><div><div class="card-name">賴冠臻</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">8,600</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,000</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
    </div>

  </div>
  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg12">🌸【第十二組｜修心之路】6/8 提醒！

━━━━━━━━━━━━━━
📋 今日定課：

👤 許哲豪（衝衝）→ 今日尚未打卡
👤 王芷盈（丁丁）→ 今日尚未打卡
👤 黃湘庭（樂樂）→ 定課1完成，還差2個
👤 郭筱婷（隊長）→ 定課2完成，還差1個
👤 賴冠臻（副隊長）→ 定課1完成，還差2個

━━━━━━━━━━━━━━
🎯 本週特殊任務：

👤 陳惠玲（抱抱）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 黃湘庭（樂樂）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 許哲豪（衝衝）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 郭筱婷（隊長）
✓ 欣賞夥伴、親證分享
❌ 蓋雅、天使通話、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 王芷盈（丁丁）
❌ 蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2

👤 賴冠臻（副隊長）
✓ 欣賞夥伴、親證分享
❌ 蓋雅、天使通話、圓夢計畫(x2)、參加活動(x2)、主題親證2

⚠️ 主題親證2（6/8開跑，2週內完成，+2800！）
修心之路一起加油✨</div>
    <button class="copy-btn" onclick="copyMsg('msg12', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>

<div class="footer">筱君大隊任務追蹤 · 截至 6/8 更新</div>

<script>
function switchTab(n) {
  document.querySelectorAll('.tab').forEach((t,i) => t.classList.toggle('active', i===n));
  document.querySelectorAll('.content').forEach((c,i) => c.classList.toggle('active', i===n));
}
function copyMsg(id, btn) {
  const text = document.getElementById(id).innerText;
  navigator.clipboard.writeText(text).then(() => {
    btn.textContent = '✓ 已複製！';
    btn.classList.add('copied');
    setTimeout(() => {
      btn.textContent = '一鍵複製貼到 Line ↗';
      btn.classList.remove('copied');
    }, 2000);
  }).catch(() => {
    const el = document.getElementById(id);
    const range = document.createRange();
    range.selectNode(el);
    window.getSelection().removeAllRanges();
    window.getSelection().addRange(range);
    document.execCommand('copy');
    window.getSelection().removeAllRanges();
    btn.textContent = '✓ 已複製！';
    btn.classList.add('copied');
    setTimeout(() => {
      btn.textContent = '一鍵複製貼到 Line ↗';
      btn.classList.remove('copied');
    }, 2000);
  });
}
</script>
</body>
</html>
