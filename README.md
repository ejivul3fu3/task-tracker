[index.html](https://github.com/user-attachments/files/28728888/index.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Expires" content="0">
<title>筱君大隊 任務追蹤</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
<style>
:root {
  --bg: #0f0f13;
  --bg2: #17171e;
  --bg3: #1e1e28;
  --border: rgba(255,255,255,0.10);
  --border2: rgba(255,255,255,0.18);
  --text: #f5f4ee;
  --text2: #c8c6bc;
  --text3: #8a8880;
  --purple: #7c6df5;
  --purple-l: #bbb0ff;
  --purple-bg: rgba(124,109,245,0.14);
  --green: #5ab878;
  --green-bg: rgba(90,184,120,0.14);
  --red: #e05c5c;
  --red-bg: rgba(224,92,92,0.14);
  --amber: #e0a93c;
  --amber-bg: rgba(224,169,60,0.14);
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
  gap: 8px;
  padding: 12px 1rem;
  overflow-x: auto;
  scrollbar-width: none;
  -webkit-overflow-scrolling: touch;
  border-bottom: 1px solid var(--border);
}
.tabs::-webkit-scrollbar { display: none; }
.tab {
  flex-shrink: 0;
  padding: 10px 18px;
  font-size: 14px;
  font-weight: 500;
  color: var(--text2);
  border-radius: 20px;
  cursor: pointer;
  border: 1px solid var(--border);
  transition: all 0.2s;
  background: transparent;
  white-space: nowrap;
  min-height: 44px;
  display: flex;
  align-items: center;
}
.tab:hover { color: var(--text); background: var(--bg2); }
.tab.active {
  color: var(--purple-l);
  background: var(--purple-bg);
  border-color: rgba(124,109,245,0.4);
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


/* Hug Section */
.hug-section {
  background: linear-gradient(135deg, rgba(255,182,193,0.08) 0%, rgba(255,218,185,0.08) 100%);
  border: 1px solid rgba(255,182,193,0.25);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.hug-header { text-align: center; margin-bottom: 1rem; }
.hug-title { font-size: 16px; font-weight: 700; color: #f5a0b8; margin: 8px 0 4px; }
.hug-sub { font-size: 12px; color: var(--text2); }

/* Pikmin SVG animation */
.pikmin-svg {
  width: 100%;
  max-width: 320px;
  height: 70px;
  overflow: visible;
}
.pk { animation: pk-bounce 1.2s ease-in-out infinite; }
.pk1 { animation-delay: 0s; }
.pk2 { animation-delay: 0.2s; }
.pk3 { animation-delay: 0.4s; }
.pk4 { animation-delay: 0.6s; }
.pk5 { animation-delay: 0.8s; }
.pk6 { animation-delay: 1.0s; }
@keyframes pk-bounce {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-8px); }
}

.hug-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 10px; }
.hug-card {
  background: var(--bg2);
  border: 1px solid rgba(255,182,193,0.2);
  border-radius: 12px;
  padding: 12px;
}
.hug-name { font-size: 14px; font-weight: 700; color: #f5a0b8; margin-bottom: 4px; }
.hug-score { font-size: 11px; color: var(--text3); margin-bottom: 10px; }
.hug-block { margin-bottom: 8px; }
.hug-label { font-size: 10px; font-weight: 700; color: var(--amber); margin-bottom: 3px; }
.hug-text { font-size: 11px; color: var(--text2); line-height: 1.6; }

/* Duty Section */
.duty-section {
  background: var(--bg2);
  border: 1px solid rgba(124,109,245,0.25);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.duty-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 1rem;
}
.duty-icon {
  font-size: 18px;
  width: 36px; height: 36px;
  background: var(--purple-bg);
  border-radius: 10px;
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
}
.duty-title { font-size: 15px; font-weight: 700; color: var(--text); }
.duty-sub { font-size: 11px; color: var(--text3); margin-top: 2px; }
.duty-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 8px;
  margin-bottom: 12px;
}
.duty-card {
  background: var(--bg3);
  border-radius: 10px;
  padding: 10px 12px;
}
.duty-person {
  font-size: 12px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 6px;
}
.duty-tasks { display: flex; flex-wrap: wrap; gap: 4px; }
.duty-tag {
  font-size: 10px;
  background: var(--purple-bg);
  color: var(--purple-l);
  border: 1px solid rgba(124,109,245,0.25);
  border-radius: 4px;
  padding: 2px 7px;
}
.duty-note {
  font-size: 11px;
  color: var(--text3);
  background: var(--bg3);
  border-radius: 8px;
  padding: 8px 12px;
  line-height: 1.6;
}

.duty-status-block { margin-top: 8px; display: flex; flex-direction: column; gap: 3px; }
.duty-status-row { font-size: 10px; line-height: 1.5; }
.duty-status-label { color: var(--text2); font-weight: 600; margin-right: 4px; }
.duty-done-list { color: var(--green); }
.duty-miss-label { color: var(--red); font-weight: 700; margin-right: 4px; }
.duty-miss-list { color: var(--text); }
.angel-pair-alert {
  font-size: 10px;
  color: var(--amber);
  background: var(--amber-bg);
  border-radius: 5px;
  padding: 4px 8px;
  margin-top: 5px;
  line-height: 1.5;
}
.angel-pair-note { font-size: 10px; color: var(--text2); line-height: 1.5; margin-top: 4px; }

/* Angel Call Section */
.angel-section {
  background: var(--bg2);
  border: 1px solid rgba(90,184,120,0.25);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.angel-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 1rem;
}
.angel-icon {
  font-size: 22px;
  width: 36px; height: 36px;
  background: var(--green-bg);
  border-radius: 10px;
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
}
.angel-title { font-size: 15px; font-weight: 700; color: var(--text); }
.angel-sub { font-size: 11px; color: var(--text3); margin-top: 2px; }
.angel-week { margin-bottom: 1rem; }
.angel-week-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  color: var(--text3);
  text-transform: uppercase;
  margin-bottom: 8px;
  padding-bottom: 4px;
  border-bottom: 1px solid var(--border);
}
.angel-pairs { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 8px; }
.angel-pair {
  background: var(--bg3);
  border-radius: 10px;
  padding: 10px 12px;
  border-left: 3px solid transparent;
}
.angel-pair.done { border-left-color: var(--green); }
.angel-pair.pending { border-left-color: var(--amber); }
.angel-pair.upcoming { border-left-color: var(--border2); opacity: 0.6; }
.angel-pair-names { font-size: 12px; font-weight: 700; color: var(--text); margin-bottom: 3px; }
.angel-pair-time { font-size: 10px; color: var(--text2); margin-bottom: 4px; }
.angel-pair-status { font-size: 10px; font-weight: 700; margin-bottom: 4px; }
.status-done { color: var(--green); }
.status-pending { color: var(--amber); }
.status-upcoming { color: var(--text3); }
.angel-pair-note { font-size: 10px; color: var(--text3); line-height: 1.5; }

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
  <div class="date">截至 6/9 06:17 更新</div>
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
      <span>隊長：王薏涵</span>
    </div>
  </div>

  
  <!-- 需要愛的抱抱 -->
  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <!-- Blue Pikmin -->
  <g class="pk pk1">
    <line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"/>
    <ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"/>
    <ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"/>
    <ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"/>
    <circle cx="17" cy="25" r="2" fill="white"/>
    <circle cx="23" cy="25" r="2" fill="white"/>
    <circle cx="17.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="23.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"/>
    <line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Yellow Pikmin -->
  <g class="pk pk2">
    <line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"/>
    <ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"/>
    <ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"/>
    <ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"/>
    <circle cx="67" cy="25" r="2" fill="white"/>
    <circle cx="73" cy="25" r="2" fill="white"/>
    <circle cx="67.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="73.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"/>
    <ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"/>
    <ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"/>
    <line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Red Pikmin -->
  <g class="pk pk3">
    <line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"/>
    <ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"/>
    <ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"/>
    <ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"/>
    <circle cx="117" cy="25" r="2" fill="white"/>
    <circle cx="123" cy="25" r="2" fill="white"/>
    <circle cx="117.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="123.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"/>
    <line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Purple Pikmin -->
  <g class="pk pk4">
    <line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"/>
    <ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"/>
    <ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"/>
    <ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"/>
    <circle cx="166" cy="26" r="2.5" fill="white"/>
    <circle cx="174" cy="26" r="2.5" fill="white"/>
    <circle cx="166.5" cy="26.5" r="1.2" fill="#222"/>
    <circle cx="174.5" cy="26.5" r="1.2" fill="#222"/>
    <ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"/>
    <line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
  </g>
  <!-- White Pikmin -->
  <g class="pk pk5">
    <line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"/>
    <ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"/>
    <ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"/>
    <ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"/>
    <circle cx="217" cy="25" r="2.5" fill="#ff4466"/>
    <circle cx="223" cy="25" r="2.5" fill="#ff4466"/>
    <circle cx="217.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="223.5" cy="25.5" r="1" fill="#222"/>
    <line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Rock Pikmin -->
  <g class="pk pk6">
    <line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"/>
    <ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"/>
    <ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"/>
    <ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"/>
    <ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"/>
    <circle cx="266" cy="24" r="2" fill="white"/>
    <circle cx="274" cy="23" r="2" fill="white"/>
    <circle cx="266.5" cy="24.5" r="1" fill="#222"/>
    <circle cx="274.5" cy="23.5" r="1" fill="#222"/>
    <line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/>
  </g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！這兩位夥伴需要你的支持 ✨</div>
    </div>

    <div class="hug-cards">

      <div class="hug-card">
        <div class="hug-name">😔 廖志裕（丁丁）</div>
        <div class="hug-score">總分 10,600 ｜ 本週 700</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：深夜 10:30 後｜適合聯繫：下午 9:00 ～ 10:00</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">本週特殊任務全未做，來得及衝！蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2 都還可以完成，加油！</div>
        </div>
      </div>

      <div class="hug-card">
        <div class="hug-name">😔 黃雅琪（抱抱）</div>
        <div class="hug-score">總分 10,300 ｜ 本週 1,600</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：早上 7:00 或晚間 10:00 前後｜適合聯繫：上午 7:00 ～ 8:00 或 下午 9:30 ～ 10:30</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2 還未做，特殊任務有在做，繼續加油！</div>
        </div>
      </div>

    </div>
  </div>


  <div class="members-grid">

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-purple">薏</div><div><div class="card-name">王薏涵</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">16,560</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>

      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>

    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-purple">岑</div><div><div class="card-name">王岑芯</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>

      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>

    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">宏</div><div><div class="card-name">王宏榮</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>

      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>

    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-green">芯</div><div><div class="card-name">黃芯璿</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,700</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>

      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>

    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">念</div><div><div class="card-name">鄒念穎</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>

      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>

    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">志</div><div><div class="card-name">廖志裕</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">10,600</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
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
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>

    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">雅</div><div><div class="card-name">黃雅琪</div><div class="card-role">龍女（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">10,300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">+0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>

      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
<div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
<div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>

    </div>
  </div>

    <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg9">✨【第九組｜佛系但暴富組】6/9 提醒！

━━━━━━━━━━━━━━
📋 今日定課

✅ 三項都完成：（無）
還差兩項：（無）
還差一項：（無）
❌ 都還沒打卡：王薏涵、王岑芯、王宏榮、黃芯璿、鄒念穎、廖志裕、黃雅琪

━━━━━━━━━━━━━━
🎯 本週特殊任務

蓋雅的召喚
✅ 完成：王宏榮、黃雅琪、鄒念穎
❌ 未完成：王薏涵、王岑芯、黃芯璿、廖志裕

天使通話
✅ 完成：王岑芯、黃芯璿
❌ 未完成：王薏涵、王宏榮、鄒念穎、廖志裕、黃雅琪

欣賞夥伴
✅ 完成：王薏涵、王岑芯、王宏榮、黃芯璿、鄒念穎
❌ 未完成：廖志裕、黃雅琪

親證分享
✅ 完成：王薏涵、王岑芯、王宏榮、黃雅琪
❌ 未完成：黃芯璿、鄒念穎、廖志裕

圓夢計畫(x2)
✅ 完成：王薏涵、王宏榮
❌ 未完成：王岑芯、黃芯璿、鄒念穎、廖志裕、黃雅琪

參加活動(x2)
✅ 完成：王薏涵、王岑芯、黃芯璿
❌ 未完成：王宏榮、鄒念穎、廖志裕、黃雅琪

主題親證2
✅ 完成：（無）
❌ 未完成：全員

⚠️ 主題親證2（2週內完成，+2800！）
⚠️ 小組聚會（本月內完成1次，+2000！）
巔峰取經
⚠️ 已挑戰待打卡：王薏涵、鄒念穎、黃雅琪
❌ 未完成：王岑芯、王宏榮、黃芯璿、廖志裕
佛系也要暴富，大家衝🙏✨</div>
    <button class="copy-btn" onclick="copyMsg('msg9', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>

<!-- 第十組 -->
<div class="content" id="tab1">
  <div class="group-info">
    <div class="group-name">十破天驚</div>
    <div class="group-stats"><span>6人</span><span>隊長：游佳霖</span></div>
  </div>

  
  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <!-- Blue Pikmin -->
  <g class="pk pk1">
    <line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"/>
    <ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"/>
    <ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"/>
    <ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"/>
    <circle cx="17" cy="25" r="2" fill="white"/>
    <circle cx="23" cy="25" r="2" fill="white"/>
    <circle cx="17.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="23.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"/>
    <line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Yellow Pikmin -->
  <g class="pk pk2">
    <line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"/>
    <ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"/>
    <ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"/>
    <ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"/>
    <circle cx="67" cy="25" r="2" fill="white"/>
    <circle cx="73" cy="25" r="2" fill="white"/>
    <circle cx="67.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="73.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"/>
    <ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"/>
    <ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"/>
    <line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Red Pikmin -->
  <g class="pk pk3">
    <line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"/>
    <ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"/>
    <ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"/>
    <ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"/>
    <circle cx="117" cy="25" r="2" fill="white"/>
    <circle cx="123" cy="25" r="2" fill="white"/>
    <circle cx="117.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="123.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"/>
    <line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Purple Pikmin -->
  <g class="pk pk4">
    <line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"/>
    <ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"/>
    <ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"/>
    <ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"/>
    <circle cx="166" cy="26" r="2.5" fill="white"/>
    <circle cx="174" cy="26" r="2.5" fill="white"/>
    <circle cx="166.5" cy="26.5" r="1.2" fill="#222"/>
    <circle cx="174.5" cy="26.5" r="1.2" fill="#222"/>
    <ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"/>
    <line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
  </g>
  <!-- White Pikmin -->
  <g class="pk pk5">
    <line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"/>
    <ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"/>
    <ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"/>
    <ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"/>
    <circle cx="217" cy="25" r="2.5" fill="#ff4466"/>
    <circle cx="223" cy="25" r="2.5" fill="#ff4466"/>
    <circle cx="217.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="223.5" cy="25.5" r="1" fill="#222"/>
    <line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Rock Pikmin -->
  <g class="pk pk6">
    <line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"/>
    <ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"/>
    <ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"/>
    <ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"/>
    <ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"/>
    <circle cx="266" cy="24" r="2" fill="white"/>
    <circle cx="274" cy="23" r="2" fill="white"/>
    <circle cx="266.5" cy="24.5" r="1" fill="#222"/>
    <circle cx="274.5" cy="23.5" r="1" fill="#222"/>
    <line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/>
  </g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！這兩位夥伴需要你的支持 ✨</div>
    </div>
    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">😔 羅萱（抱抱）</div>
        <div class="hug-score">總分 15,400 ｜ 本週 1,000</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：晚間 10:30 前後｜適合聯繫：下午 9:30 ～ 10:30</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">天使通話、欣賞夥伴、主題親證2 還未做，特殊任務有在動，繼續衝！</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">😔 游佳霖（隊長）</div>
        <div class="hug-score">總分 13,900 ｜ 本週 1,200</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：上午 8:57 前後｜適合聯繫：上午 8:00 ～ 9:00</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">蓋雅、天使通話、親證分享、圓夢計畫(x2)、主題親證2 還未做，隊長衝起來帶大家飛！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 天使通話分組 -->
  <div class="angel-section">
    <div class="angel-header">
      <span class="angel-icon">📞</span>
      <div>
        <div class="angel-title">天使通話分組</div>
        <div class="angel-sub">本週通話配對｜完成後記得打卡！</div>
      </div>
    </div>

    <div class="angel-week">
      <div class="angel-week-label">本週（6/8 – 6/14）</div>
      <div class="angel-pairs">

        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 游佳霖 &amp; 羅萱（蜜卡）</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
          <div class="angel-pair-note">請提醒：游佳霖、羅萱 確認通話時間，完成後記得打卡！</div>
        </div>

        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 游文君 &amp; 李雯萱</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
          <div class="angel-pair-note">請提醒：游文君、李雯萱 確認通話時間，完成後記得打卡！</div>
        </div>

        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 蔡鎔庄 &amp; 王依涵</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
          <div class="angel-pair-note">請提醒：蔡鎔庄、王依涵 確認通話時間，完成後記得打卡！</div>
        </div>

      </div>
    </div>
  </div>


  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">庄</div><div><div class="card-name">蔡鎔庄</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">22,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">萱</div><div><div class="card-name">羅萱</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">文</div><div><div class="card-name">游文君</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,020</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">霖</div><div><div class="card-name">游佳霖</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">依</div><div><div class="card-name">王依涵</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計畫(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">萱</div><div><div class="card-name">李雯萱</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">12,100</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>
    </div>

  </div>

    <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg10">💥【第十組｜十破天驚】6/9 提醒！

━━━━━━━━━━━━━━
📋 今日定課

✅ 三項都完成：（無）
還差兩項：（無）
還差一項：（無）
❌ 都還沒打卡：蔡鎔庄、羅萱、游文君、游佳霖、王依涵、李雯萱

━━━━━━━━━━━━━━
🎯 本週特殊任務

蓋雅的召喚
✅ 完成：蔡鎔庄、羅萱、王依涵
❌ 未完成：游文君、游佳霖、李雯萱

天使通話
✅ 完成：蔡鎔庄、王依涵
❌ 未完成：羅萱、游文君、游佳霖、李雯萱

欣賞夥伴
✅ 完成：蔡鎔庄、游文君、王依涵
❌ 未完成：羅萱、游佳霖、李雯萱

親證分享
✅ 完成：羅萱、游文君、王依涵
❌ 未完成：蔡鎔庄、游佳霖、李雯萱

圓夢計畫(x2)
✅ 完成：蔡鎔庄、羅萱、游文君、王依涵
❌ 未完成：游佳霖、李雯萱

參加活動(x2)
✅ 完成：羅萱、游文君、李雯萱
❌ 未完成：蔡鎔庄、游佳霖、王依涵

主題親證2
✅ 完成：蔡鎔庄
❌ 未完成：羅萱、游文君、游佳霖、王依涵、李雯萱

⚠️ 主題親證2（2週內完成，+2800！）
⚠️ 小組聚會（本月內完成1次，+2000！）
巔峰取經
⚠️ 已挑戰待打卡：羅萱、游佳霖、游文君
❌ 未完成：蔡鎔庄、王依涵、李雯萱
十破天驚繼續衝🔥</div>
    <button class="copy-btn" onclick="copyMsg('msg10', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>

<!-- 第十一組 -->
<div class="content" id="tab2">
  <div class="group-info">
    <div class="group-name">修心之路</div>
    <div class="group-stats"><span>6人</span><span>隊長：郭筱婷</span></div>
  </div>

  
  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <!-- Blue Pikmin -->
  <g class="pk pk1">
    <line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"/>
    <ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"/>
    <ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"/>
    <ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"/>
    <circle cx="17" cy="25" r="2" fill="white"/>
    <circle cx="23" cy="25" r="2" fill="white"/>
    <circle cx="17.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="23.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"/>
    <line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Yellow Pikmin -->
  <g class="pk pk2">
    <line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"/>
    <ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"/>
    <ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"/>
    <ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"/>
    <circle cx="67" cy="25" r="2" fill="white"/>
    <circle cx="73" cy="25" r="2" fill="white"/>
    <circle cx="67.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="73.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"/>
    <ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"/>
    <ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"/>
    <line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Red Pikmin -->
  <g class="pk pk3">
    <line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"/>
    <ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"/>
    <ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"/>
    <ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"/>
    <circle cx="117" cy="25" r="2" fill="white"/>
    <circle cx="123" cy="25" r="2" fill="white"/>
    <circle cx="117.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="123.5" cy="25.5" r="1" fill="#222"/>
    <ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"/>
    <line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Purple Pikmin -->
  <g class="pk pk4">
    <line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"/>
    <ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"/>
    <ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"/>
    <ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"/>
    <circle cx="166" cy="26" r="2.5" fill="white"/>
    <circle cx="174" cy="26" r="2.5" fill="white"/>
    <circle cx="166.5" cy="26.5" r="1.2" fill="#222"/>
    <circle cx="174.5" cy="26.5" r="1.2" fill="#222"/>
    <ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"/>
    <line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
    <line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/>
  </g>
  <!-- White Pikmin -->
  <g class="pk pk5">
    <line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"/>
    <ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"/>
    <ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"/>
    <ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"/>
    <circle cx="217" cy="25" r="2.5" fill="#ff4466"/>
    <circle cx="223" cy="25" r="2.5" fill="#ff4466"/>
    <circle cx="217.5" cy="25.5" r="1" fill="#222"/>
    <circle cx="223.5" cy="25.5" r="1" fill="#222"/>
    <line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
    <line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/>
  </g>
  <!-- Rock Pikmin -->
  <g class="pk pk6">
    <line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"/>
    <ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"/>
    <ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"/>
    <ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"/>
    <ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"/>
    <circle cx="266" cy="24" r="2" fill="white"/>
    <circle cx="274" cy="23" r="2" fill="white"/>
    <circle cx="266.5" cy="24.5" r="1" fill="#222"/>
    <circle cx="274.5" cy="23.5" r="1" fill="#222"/>
    <line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/>
    <line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/>
  </g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！這兩位夥伴需要你的支持 ✨</div>
    </div>
    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">😔 賴冠臻（副隊長）</div>
        <div class="hug-score">總分 9,200 ｜ 本週 1,600</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：晚間 10:59 前後｜適合聯繫：下午 10:00 ～ 11:00</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2 還未做，你做得到的，衝！</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">😔 王芷盈（丁丁）</div>
        <div class="hug-score">總分 13,100 ｜ 本週 800</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：下午 9:25 前後｜適合聯繫：下午 8:30 ～ 9:30</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">蓋雅、天使通話、欣賞夥伴、親證分享、圓夢計畫(x2)、參加活動(x2)、主題親證2 全未做，週才剛開始，來得及全部完成！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 天使通話分組 -->
  <div class="angel-section">
    <div class="angel-header">
      <span class="angel-icon">📞</span>
      <div>
        <div class="angel-title">天使通話分組</div>
        <div class="angel-sub">為期2週，每週一次通話｜6/8 – 6/21</div>
      </div>
    </div>

    <div class="angel-week">
      <div class="angel-week-label">第1週（6/8 – 6/14）</div>
      <div class="angel-pairs">

        <div class="angel-pair done">
          <div class="angel-pair-names">👥 郭筱婷 &amp; 許哲豪</div>
          <div class="angel-pair-time">6/9 22:00</div>
          <div class="angel-pair-status status-done">✅ 已完成通話</div>
          <div class="angel-pair-alert">⚠️ 提醒打卡：許哲豪 尚未在遊戲內打卡「天使通話」</div>
        </div>

        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 陳惠玲 &amp; 賴冠臻</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
          <div class="angel-pair-note">請提醒：陳惠玲、賴冠臻 確認通話時間，完成後記得打卡！</div>
        </div>

        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 王芷盈 &amp; 黃湘庭</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
          <div class="angel-pair-note">請提醒：王芷盈、黃湘庭 確認通話時間，完成後記得打卡！</div>
        </div>

      </div>
    </div>

    <div class="angel-week">
      <div class="angel-week-label">第2週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair upcoming"><div class="angel-pair-names">👥 郭筱婷 &amp; 許哲豪</div><div class="angel-pair-status status-upcoming">🔜 即將到來</div></div>
        <div class="angel-pair upcoming"><div class="angel-pair-names">👥 陳惠玲 &amp; 賴冠臻</div><div class="angel-pair-status status-upcoming">🔜 即將到來</div></div>
        <div class="angel-pair upcoming"><div class="angel-pair-names">👥 王芷盈 &amp; 黃湘庭</div><div class="angel-pair-status status-upcoming">🔜 即將到來</div></div>
      </div>
    </div>
  </div>

  <!-- 任務負責分工 -->
  <div class="duty-section">
    <div class="duty-header">
      <span class="duty-icon">📋</span>
      <div>
        <div class="duty-title">任務提醒分工</div>
        <div class="duty-sub">隊長確認未完成名單 → 通知對應負責人追蹤</div>
      </div>
    </div>

    <div class="duty-grid">

      <div class="duty-card">
        <div class="duty-person">👑 郭筱婷（Yoyo）負責</div>
        <div class="duty-tasks"><span class="duty-tag">圓夢計畫(x2)</span><span class="duty-tag">圓夢親證</span><span class="duty-tag">心成活動</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row">
            <span class="duty-status-label">心成活動</span>
            <span class="duty-done-list">✅ 黃湘庭、陳惠玲</span>
          </div>
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 未完成</span>
            <span class="duty-miss-list">郭筱婷、許哲豪、王芷盈、賴冠臻</span>
          </div>
          <div class="duty-status-row">
            <span class="duty-status-label">圓夢計畫</span>
          </div>
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 未完成</span>
            <span class="duty-miss-list">全員</span>
          </div>
        </div>
      </div>

      <div class="duty-card">
        <div class="duty-person">🔔 許哲豪 負責</div>
        <div class="duty-tasks"><span class="duty-tag">欣賞夥伴</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row">
            <span class="duty-done-list">✅ 郭筱婷</span>
          </div>
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 未完成</span>
            <span class="duty-miss-list">黃湘庭、陳惠玲、許哲豪、王芷盈、賴冠臻</span>
          </div>
        </div>
      </div>

      <div class="duty-card">
        <div class="duty-person">🔔 黃湘庭 負責</div>
        <div class="duty-tasks"><span class="duty-tag">蓋雅的召喚</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row">
            <span class="duty-done-list">✅ 黃湘庭、賴冠臻</span>
          </div>
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 未完成</span>
            <span class="duty-miss-list">郭筱婷、陳惠玲、許哲豪、王芷盈</span>
          </div>
        </div>
      </div>

      <div class="duty-card">
        <div class="duty-person">🔔 賴冠臻 負責</div>
        <div class="duty-tasks"><span class="duty-tag">天使通話</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row">
            <span class="duty-done-list">✅ 郭筱婷（已通話，待打卡）</span>
          </div>
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 未打卡</span>
            <span class="duty-miss-list">許哲豪（已通話未打卡）、陳惠玲、王芷盈、黃湘庭、賴冠臻</span>
          </div>
        </div>
      </div>

      <div class="duty-card">
        <div class="duty-person">🔔 王芷盈 負責</div>
        <div class="duty-tasks"><span class="duty-tag">每日定課</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 今日未打卡</span>
            <span class="duty-miss-list">全員（截至 6/9 更新時）</span>
          </div>
        </div>
      </div>

      <div class="duty-card">
        <div class="duty-person">🔔 陳惠玲 負責</div>
        <div class="duty-tasks"><span class="duty-tag">主題親證2</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row">
            <span class="duty-miss-label">❌ 未完成</span>
            <span class="duty-miss-list">全員</span>
          </div>
        </div>
      </div>

    </div>
  </div>


  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">湘</div><div><div class="card-name">黃湘庭</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">18,000</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">小組聚會(本月1次)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">惠</div><div><div class="card-name">陳惠玲</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,700</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">參加活動(x1)</span><span class="badge warn">還差1次</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">哲</div><div><div class="card-name">許哲豪</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
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
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">筱</div><div><div class="card-name">郭筱婷</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,500</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
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
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">芷</div><div><div class="card-name">王芷盈</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
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
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">冠</div><div><div class="card-name">賴冠臻</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">9,200</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

  </div>

    <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg11">🌸【第十一組｜修心之路】6/9 提醒！

━━━━━━━━━━━━━━
📋 今日定課

✅ 三項都完成：（無）
還差兩項：（無）
還差一項：（無）
❌ 都還沒打卡：郭筱婷、陳惠玲、許哲豪、王芷盈、黃湘庭、賴冠臻

━━━━━━━━━━━━━━
🎯 本週特殊任務

蓋雅的召喚
✅ 完成：黃湘庭、賴冠臻
❌ 未完成：郭筱婷、陳惠玲、許哲豪、王芷盈

天使通話（含已完成通話待打卡）
✅ 完成：（無打卡紀錄）
⚠️ 已通話待打卡：郭筱婷、許哲豪
❌ 未完成：陳惠玲、王芷盈、黃湘庭、賴冠臻

欣賞夥伴
✅ 完成：郭筱婷
❌ 未完成：陳惠玲、許哲豪、王芷盈、黃湘庭、賴冠臻

親證分享
✅ 完成：郭筱婷、賴冠臻
❌ 未完成：陳惠玲、許哲豪、王芷盈、黃湘庭

圓夢計畫(x2)
✅ 完成：（無）
❌ 未完成：全員

參加活動(x2)
✅ 完成：黃湘庭
❌ 未完成：郭筱婷、陳惠玲、許哲豪、王芷盈、賴冠臻

主題親證2
✅ 完成：（無）
❌ 未完成：全員

⚠️ 主題親證2（2週內完成，+2800！）
⚠️ 小組聚會（本月內完成1次，+2000！）
巔峰取經
⚠️ 已挑戰待打卡：郭筱婷、許哲豪、黃湘庭、賴冠臻
❌ 未完成：陳惠玲、王芷盈
修心之路一起加油✨</div>
    <button class="copy-btn" onclick="copyMsg('msg11', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>
<!-- 第十二組 -->
<div class="content" id="tab3">
  <div class="group-info">
    <div class="group-name">齊天戰神突擊隊</div>
    <div class="group-stats"><span>6人</span><span>隊長：黃怡駿</span></div>
  </div>

  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <g class="pk pk1"><line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"/><ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"/><ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"/><ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"/><circle cx="17" cy="25" r="2" fill="white"/><circle cx="23" cy="25" r="2" fill="white"/><circle cx="17.5" cy="25.5" r="1" fill="#222"/><circle cx="23.5" cy="25.5" r="1" fill="#222"/><ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"/><line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/><line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/><line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/><line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"/></g>
  <g class="pk pk2"><line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"/><ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"/><ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"/><ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"/><circle cx="67" cy="25" r="2" fill="white"/><circle cx="73" cy="25" r="2" fill="white"/><circle cx="67.5" cy="25.5" r="1" fill="#222"/><circle cx="73.5" cy="25.5" r="1" fill="#222"/><ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"/><ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"/><ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"/><line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/><line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"/></g>
  <g class="pk pk3"><line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"/><ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"/><ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"/><ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"/><circle cx="117" cy="25" r="2" fill="white"/><circle cx="123" cy="25" r="2" fill="white"/><circle cx="117.5" cy="25.5" r="1" fill="#222"/><circle cx="123.5" cy="25.5" r="1" fill="#222"/><ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"/><line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/><line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/><line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/><line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"/></g>
  <g class="pk pk4"><line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"/><ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"/><ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"/><ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"/><circle cx="166" cy="26" r="2.5" fill="white"/><circle cx="174" cy="26" r="2.5" fill="white"/><circle cx="166.5" cy="26.5" r="1.2" fill="#222"/><circle cx="174.5" cy="26.5" r="1.2" fill="#222"/><ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"/><line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/><line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/><line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/><line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"/></g>
  <g class="pk pk5"><line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"/><ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"/><ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"/><ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"/><circle cx="217" cy="25" r="2.5" fill="#ff4466"/><circle cx="223" cy="25" r="2.5" fill="#ff4466"/><circle cx="217.5" cy="25.5" r="1" fill="#222"/><circle cx="223.5" cy="25.5" r="1" fill="#222"/><line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/><line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/><line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/><line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"/></g>
  <g class="pk pk6"><line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"/><ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"/><ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"/><ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"/><ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"/><circle cx="266" cy="24" r="2" fill="white"/><circle cx="274" cy="23" r="2" fill="white"/><circle cx="266.5" cy="24.5" r="1" fill="#222"/><circle cx="274.5" cy="23.5" r="1" fill="#222"/><line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/><line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"/><line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/><line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"/></g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！這兩位夥伴需要你的支持 ✨</div>
    </div>
    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">😔 洪煜棠（副隊長）</div>
        <div class="hug-score">總分 13,400 ｜ 本週 600</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：上午 12:27 前後｜適合聯繫：上午 12:00 ～ 下午 1:00</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">蓋雅、天使通話、欣賞夥伴、圓夢計畫(x2)、參加活動(x2)、主題親證2 還未做，週才開始來得及衝！</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">😔 黃怡駿（隊長）</div>
        <div class="hug-score">總分 13,800 ｜ 本週 2,000</div>
        <div class="hug-block">
          <div class="hug-label">🕐 打卡習性</div>
          <div class="hug-text">推測作業時間：下午 10:28～10:50｜適合聯繫：下午 10:00 ～ 11:00</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💪 需要支持</div>
          <div class="hug-text">天使通話、圓夢計畫(x2)、參加活動(x2)、主題親證2 還未做，隊長帶頭衝帶全組飛！</div>
        </div>
      </div>
    </div>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">淑</div><div><div class="card-name">盧家淑</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">18,600</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+500</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x1)</span><span class="badge warn">還差1次</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-purple">玲</div><div><div class="card-name">許玲慧</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">18,300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">丞</div><div><div class="card-name">郭丞浤</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">18,200</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">破曉打拳（+200）</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x1)</span><span class="badge warn">還差1次</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">慈</div><div><div class="card-name">林嘉慈</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">15,200</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-green">駿</div><div><div class="card-name">黃怡駿</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,800</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計畫(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">煜</div><div><div class="card-name">洪煜棠</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+100</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
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
      <div class="task-row"><span class="task-name">小組聚會(本月1次)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經(活動結束前)</span> <span class="badge warn">已挑戰待打卡</span></div>
    </div>

  </div>
  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg12">🔥【第十二組｜齊天戰神突擊隊】6/9 提醒！

━━━━━━━━━━━━━━
📋 今日定課

✅ 三項都完成：盧家淑
還差兩項：洪煜棠
還差一項：（無）
❌ 都還沒打卡：許玲慧、郭丞浤、林嘉慈、黃怡駿

━━━━━━━━━━━━━━
🎯 本週特殊任務

蓋雅的召喚
✅ 完成：盧家淑、許玲慧、郭丞浤、黃怡駿
❌ 未完成：林嘉慈、洪煜棠

天使通話
✅ 完成：盧家淑、許玲慧、林嘉慈
❌ 未完成：郭丞浤、黃怡駿、洪煜棠

欣賞夥伴
✅ 完成：盧家淑、黃怡駿
❌ 未完成：許玲慧、郭丞浤、林嘉慈、洪煜棠

親證分享
✅ 完成：盧家淑、郭丞浤、林嘉慈、黃怡駿、洪煜棠
❌ 未完成：許玲慧

圓夢計畫(x2)
✅ 完成：（無）
❌ 未完成：全員

參加活動(x2)
✅ 完成：許玲慧、林嘉慈
⚠️ 還差1次：盧家淑、郭丞浤
❌ 未完成：黃怡駿、洪煜棠

主題親證2
✅ 完成：（無）
❌ 未完成：全員

⚠️ 主題親證2（2週內完成，+2800！）
⚠️ 小組聚會（本月內完成1次，+2000！）
巔峰取經
⚠️ 已挑戰待打卡：全員（盧家淑、許玲慧、郭丞浤、林嘉慈、黃怡駿、洪煜棠）
戰神突擊隊加油衝💪</div>
    <button class="copy-btn" onclick="copyMsg('msg12', this)">一鍵複製貼到 Line ↗</button>
  </div>
</div>


<div class="footer">筱君大隊任務追蹤 · 截至 6/9 06:17 更新</div>

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
