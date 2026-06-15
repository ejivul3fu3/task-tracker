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
  --green-l: #7ad898;
  --green-bg: rgba(90,184,120,0.14);
  --jade: #4a9e7a;
  --jade-l: #6abf98;
  --jade-bg: rgba(74,158,122,0.14);
  --red: #e05c5c;
  --red-bg: rgba(224,92,92,0.14);
  --amber: #e0a93c;
  --amber-bg: rgba(224,169,60,0.14);
  --gold: #f0c060;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html { background: #0f0f13; }
body {
  font-family: 'Noto Sans TC', sans-serif;
  background: #0f0f13;
  color: #f5f4ee;
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
.content { display: none; padding: 1rem; }
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
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 8px;
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
.score-val { font-size: 12px; font-weight: 700; color: var(--text); }
.score-val.good { color: var(--green); }
.score-val.warn { color: var(--amber); }
.score-val.bad { color: var(--red); }
.score-val.week { color: #7ab8ff; }

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


/* Item/Equipment Section */
.items-section {
  background: var(--bg2);
  border: 1px solid rgba(201,168,76,0.3);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.items-header {
  display: flex; align-items: center; gap: 10px; margin-bottom: 12px;
}
.items-icon {
  font-size: 18px; width: 36px; height: 36px;
  background: var(--gold-bg); border-radius: 10px;
  display: flex; align-items: center; justify-content: center; flex-shrink: 0;
}
.items-title { font-size: 15px; font-weight: 700; color: var(--text); }
.items-sub { font-size: 11px; color: var(--text3); margin-top: 2px; }
.items-grid { display: flex; flex-direction: column; gap: 10px; }
.item-row {
  background: var(--bg3); border-radius: 10px; padding: 10px 12px;
  display: flex; align-items: flex-start; gap: 10px;
}
.item-emoji { font-size: 22px; flex-shrink: 0; margin-top: 2px; }
.item-name { font-size: 13px; font-weight: 700; color: var(--amber); margin-bottom: 4px; }
.item-owners { font-size: 11px; color: var(--text2); line-height: 1.6; }
.item-owners span { 
  display: inline-block; background: var(--gold-bg);
  border: 1px solid rgba(201,168,76,0.25);
  color: var(--text); border-radius: 4px;
  padding: 1px 7px; margin: 1px 2px; font-size: 10px;
}

/* Footer */
.footer {
  text-align: center;
  padding: 1rem;
  font-size: 11px;
  color: var(--text3);
}

/* Alert colors per group */
.av-purple { background: rgba(124,109,245,0.2); color: var(--purple-l); }
.av-green  { background: rgba(90,184,120,0.2);  color: var(--jade-l); }
.av-red    { background: rgba(224,92,92,0.2);   color: var(--red); }
.av-amber  { background: rgba(224,169,60,0.2);  color: var(--amber); }
.av-blue   { background: rgba(80,128,200,0.2);  color: #80b8ff; }
.av-teal   { background: rgba(50,180,190,0.2);  color: #60d8ee; }
.av-pink   { background: rgba(210,100,160,0.2); color: #f080b8; }


/* Gap Chart v2 */
.gap-chart-section { background: #17171e; border: 1px solid rgba(255,255,255,0.08); border-radius: 16px; padding: 1rem; margin-bottom: 1.25rem; }
.gap-chart-title { font-size: 14px; font-weight: 700; color: #f5f4ee; margin-bottom: 2px; }
.gap-chart-sub { font-size: 11px; color: #8a8880; margin-bottom: 12px; }
.gc-list { display: flex; flex-direction: column; gap: 6px; }
.gc-row { background: #1e1e28; border-radius: 10px; overflow: hidden; }
.gc-bar-wrap { display: flex; align-items: center; gap: 8px; padding: 9px 10px; cursor: pointer; user-select: none; }
.gc-bar-wrap:active { background: rgba(255,255,255,0.04); }
.gc-name { width: 52px; font-size: 11px; font-weight: 700; text-align: right; flex-shrink: 0; }
.gc-track { flex: 1; height: 18px; background: rgba(255,255,255,0.06); border-radius: 5px; overflow: hidden; }
.gc-fill { height: 100%; border-radius: 5px; display: flex; align-items: center; padding-left: 5px; min-width: 18px; transition: width 0.6s ease; }
.gc-fill-txt { font-size: 9px; font-weight: 700; color: rgba(255,255,255,0.9); white-space: nowrap; }
.gc-pct { width: 32px; font-size: 10px; font-weight: 700; text-align: right; flex-shrink: 0; }
.gc-arrow { width: 14px; font-size: 9px; color: rgba(255,255,255,0.3); transition: transform 0.2s; flex-shrink: 0; }
.gc-arrow.open { transform: rotate(180deg); }
.gc-detail { display: none; padding: 0 10px 10px; border-top: 1px solid rgba(255,255,255,0.05); }
.gc-detail.open { display: block; }
.gc-section-label { font-size: 10px; color: #8a8880; font-weight: 700; margin: 8px 0 5px; }
.gc-chips { display: flex; flex-wrap: wrap; gap: 5px; }
.gc-chip { font-size: 10px; font-weight: 700; padding: 3px 8px; border-radius: 6px; }
.gc-done { background: rgba(90,184,120,0.15); color: #5ab878; border: 1px solid rgba(90,184,120,0.2); }
.gc-miss { background: rgba(224,92,92,0.1); color: #f87171; border: 1px solid rgba(224,92,92,0.15); }

</style>
</head>
<body>

<div class="header">
  <div class="header-badge">筱君大隊</div>
  <h1>🏆 任務追蹤儀表板</h1>
  <div class="sub">第4週 6/15（一）～ 6/21（日）</div>
  <div class="date">截至 6/15 09:32 更新（已含全隊24人）</div>
  <div class="week-tag">🆕 新的一週開始！本週特殊任務與定課歸零重新計算</div>
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
        <div class="hug-name">🌟 新的一週開始！</div>
        <div class="hug-score">本週特殊任務與定課已重新計算</div>
        <div class="hug-block">
          <div class="hug-label">💪 給全組的話</div>
          <div class="hug-text">第4週（6/15-6/21）正式開始！上週的努力都已經累積到總分，本週讓我們從零開始衝刺新的特殊任務與每日定課，一起加油！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g9chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_王薏涵')">
            <div class="gc-name" style="color:#a78bfa">王薏涵</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#a78bfa"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g9chart_王薏涵">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_王薏涵">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_王岑芯')">
            <div class="gc-name" style="color:#60a5fa">王岑芯</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#60a5fa"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_王岑芯">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_王岑芯">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_王宏榮')">
            <div class="gc-name" style="color:#34d399">王宏榮</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#34d399"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g9chart_王宏榮">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_王宏榮">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_黃芯璿')">
            <div class="gc-name" style="color:#fbbf24">黃芯璿</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#fbbf24"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_黃芯璿">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_黃芯璿">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_鄒念穎')">
            <div class="gc-name" style="color:#f87171">鄒念穎</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#f87171"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_鄒念穎">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_鄒念穎">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_黃雅琪')">
            <div class="gc-name" style="color:#e879f9">黃雅琪</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#e879f9"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_黃雅琪">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_黃雅琪">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_廖志裕')">
            <div class="gc-name" style="color:#fb923c">廖志裕</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#fb923c"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g9chart_廖志裕">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_廖志裕">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
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
      <div class="angel-week-label">本週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 王宏榮（布丁）&amp; 王岑芯</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 黃芯璿（Bella）&amp; 黃雅琪（琪琪）</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 廖志裕 &amp; 鄒念穎（小鄒）</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <div class="items-section">
    <div class="items-header">
      <span class="items-icon">⚔️</span>
      <div>
        <div class="items-title">道具持有紀錄</div>
        <div class="items-sub">第9組｜親證班第三週</div>
      </div>
    </div>
    <div class="items-grid">
      <div class="item-row">
        <div class="item-emoji">🔱</div>
        <div>
          <div class="item-name">龍宮玉印</div>
          <div class="item-owners"><span>王宏榮</span><span>廖志裕</span><span>鄒念穎</span><span>黃芯璿</span><span>黃雅琪</span><span>王岑芯</span><span>王薏涵</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🛡️</div>
        <div>
          <div class="item-name">天罡戰鎧</div>
          <div class="item-owners"><span>鄒念穎</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🔫</div>
        <div>
          <div class="item-name">破曉槍</div>
          <div class="item-owners"><span>王薏涵</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🪄</div>
        <div>
          <div class="item-name">如意金箍棒</div>
          <div class="item-owners"><span>王薏涵</span></div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg9">✨【第九組｜佛系但暴富組】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x2)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

佛系也要暴富，新一週繼續衝🙏✨</div>
    <button class="copy-btn" onclick="copyMsg('msg9', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">薏</div><div><div class="card-name">王薏涵</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,720</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">1,120</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,120</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">岑</div><div><div class="card-name">王岑芯</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">19,400</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">宏</div><div><div class="card-name">王宏榮</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,980</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">3,360</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,360</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">流動情緒(觀呼吸)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-green">芯</div><div><div class="card-name">黃芯璿</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,700</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">念</div><div><div class="card-name">鄒念穎</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,400</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">志</div><div><div class="card-name">廖志裕</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">16,400</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">3,800</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,800</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">流動情緒(觀呼吸)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">雅</div><div><div class="card-name">黃雅琪</div><div class="card-role">龍女（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,300</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

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
        <div class="hug-name">🌟 新的一週開始！</div>
        <div class="hug-score">本週特殊任務與定課已重新計算</div>
        <div class="hug-block">
          <div class="hug-label">💪 給全組的話</div>
          <div class="hug-text">第4週（6/15-6/21）正式開始！上週的努力都已經累積到總分，本週讓我們從零開始衝刺新的特殊任務與每日定課，一起加油！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g10chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_蔡鎔庄')">
            <div class="gc-name" style="color:#34d399">蔡鎔庄</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#34d399"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g10chart_蔡鎔庄">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_蔡鎔庄">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 參加心成活動(x2)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_游佳霖')">
            <div class="gc-name" style="color:#a78bfa">游佳霖</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#a78bfa"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g10chart_游佳霖">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_游佳霖">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_游文君')">
            <div class="gc-name" style="color:#60a5fa">游文君</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#60a5fa"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g10chart_游文君">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_游文君">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 參加心成活動(x2)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_王依涵')">
            <div class="gc-name" style="color:#f87171">王依涵</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#f87171"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g10chart_王依涵">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_王依涵">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_羅萱')">
            <div class="gc-name" style="color:#fbbf24">羅萱</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#fbbf24"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g10chart_羅萱">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_羅萱">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_李雯萱')">
            <div class="gc-name" style="color:#fb923c">李雯萱</div>
            <div class="gc-track"><div class="gc-fill" style="width:38%;background:#fb923c"><span class="gc-fill-txt">3/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">38%</div>
            <div class="gc-arrow" id="arr_g10chart_李雯萱">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_李雯萱">
            <div class="gc-section-label">✅ 已完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 參加心成活動(x2)</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
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
      <div class="angel-week-label">本週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 游佳霖 &amp; 羅萱</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 游文君 &amp; 李雯萱</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 蔡鎔庄 &amp; 王依涵</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <div class="items-section">
    <div class="items-header">
      <span class="items-icon">⚔️</span>
      <div>
        <div class="items-title">道具持有紀錄</div>
        <div class="items-sub">第10組｜6/8–6/14 親證班第三週</div>
      </div>
    </div>
    <div class="items-grid">
      <div class="item-row">
        <div class="item-emoji">🔫</div>
        <div>
          <div class="item-name">破曉火尖槍</div>
          <div class="item-owners"><span>游文君</span><span>蔡鎔庄</span><span>王依涵</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🛡️</div>
        <div>
          <div class="item-name">天罡戰鎧</div>
          <div class="item-owners"><span>王依涵</span><span>羅萱</span><span>游佳霖</span><span>蔡鎔庄</span><span>李雯萱</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🪄</div>
        <div>
          <div class="item-name">如意金箍棒</div>
          <div class="item-owners"><span>游文君</span></div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg10">💥【第十組｜十破天驚】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x2)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

十破天驚新週新氣象，繼續衝🔥</div>
    <button class="copy-btn" onclick="copyMsg('msg10', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">庄</div><div><div class="card-name">蔡鎔庄</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">30,620</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">3,060</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,060</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">霖</div><div><div class="card-name">游佳霖</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,180</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">960</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+960</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">萱</div><div><div class="card-name">羅萱</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,320</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">2,920</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+4,320</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">流動情緒(觀呼吸)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">文</div><div><div class="card-name">游文君</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">22,400</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">720</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+2,000</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">依</div><div><div class="card-name">王依涵</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">17,700</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,200</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">萱</div><div><div class="card-name">李雯萱</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">19,960</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">1,260</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+5,060</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

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
        <div class="hug-name">🌟 新的一週開始！</div>
        <div class="hug-score">本週特殊任務與定課已重新計算</div>
        <div class="hug-block">
          <div class="hug-label">💪 給全組的話</div>
          <div class="hug-text">第4週（6/15-6/21）正式開始！上週的努力都已經累積到總分，本週讓我們從零開始衝刺新的特殊任務與每日定課，一起加油！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g11chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_郭筱婷')">
            <div class="gc-name" style="color:#a78bfa">郭筱婷</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#a78bfa"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_郭筱婷">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_郭筱婷">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_黃湘庭')">
            <div class="gc-name" style="color:#fbbf24">黃湘庭</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#fbbf24"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_黃湘庭">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_黃湘庭">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 參加心成活動(x2)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_許哲豪')">
            <div class="gc-name" style="color:#fb923c">許哲豪</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#fb923c"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g11chart_許哲豪">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_許哲豪">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_王芷盈')">
            <div class="gc-name" style="color:#60a5fa">王芷盈</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#60a5fa"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g11chart_王芷盈">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_王芷盈">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_陳惠玲')">
            <div class="gc-name" style="color:#34d399">陳惠玲</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#34d399"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_陳惠玲">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_陳惠玲">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_賴冠臻')">
            <div class="gc-name" style="color:#f87171">賴冠臻</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#f87171"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g11chart_賴冠臻">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_賴冠臻">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
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
        <div class="angel-sub">為期2週，每週一次通話｜6/15 – 6/28</div>
      </div>
    </div>
    <div class="angel-week">
      <div class="angel-week-label">第1週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 郭筱婷 &amp; 許哲豪</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 陳惠玲 &amp; 賴冠臻</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 王芷盈 &amp; 黃湘庭</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 任務負責分工 -->
  <div class="duty-section">
    <div class="duty-header">
      <span class="duty-icon">📋</span>
      <div>
        <div class="duty-title">任務提醒分工</div>
        <div class="duty-sub">隊長確認未完成名單 → 通知對應負責人追蹤｜本週新任務尚未分工</div>
      </div>
    </div>
    <div class="duty-grid">
      <div class="duty-card">
        <div class="duty-person">👑 郭筱婷（Yoyo）負責</div>
        <div class="duty-tasks"><span class="duty-tag">圓夢計劃親證(x2)</span><span class="duty-tag">參加心成活動(x2)</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
      <div class="duty-card">
        <div class="duty-person">🔔 許哲豪 負責</div>
        <div class="duty-tasks"><span class="duty-tag">欣賞夥伴</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
      <div class="duty-card">
        <div class="duty-person">🔔 黃湘庭 負責</div>
        <div class="duty-tasks"><span class="duty-tag">蓋雅的召喚</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
      <div class="duty-card">
        <div class="duty-person">🔔 陳惠玲 負責</div>
        <div class="duty-tasks"><span class="duty-tag">主題親證2</span><span class="duty-tag">巔峰取經</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg11">🌸【第十一組｜修心之路】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x2)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

修心之路新一週一起加油✨</div>
    <button class="copy-btn" onclick="copyMsg('msg11', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">筱</div><div><div class="card-name">郭筱婷</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">25,040</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+540</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">湘</div><div><div class="card-name">黃湘庭</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">25,200</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+900</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">哲</div><div><div class="card-name">許哲豪</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">24,380</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">惠</div><div><div class="card-name">陳惠玲</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,460</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">760</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,560</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-red">芷</div><div><div class="card-name">王芷盈</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">18,300</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">冠</div><div><div class="card-name">賴冠臻</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">13,300</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">0</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

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
        <div class="hug-name">🌟 新的一週開始！</div>
        <div class="hug-score">本週特殊任務與定課已重新計算</div>
        <div class="hug-block">
          <div class="hug-label">💪 給全組的話</div>
          <div class="hug-text">第4週（6/15-6/21）正式開始！上週的努力都已經累積到總分，本週讓我們從零開始衝刺新的特殊任務與每日定課，一起加油！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g12chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_盧家淑')">
            <div class="gc-name" style="color:#34d399">盧家淑</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#34d399"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g12chart_盧家淑">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_盧家淑">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 欣賞夥伴</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_黃怡駿')">
            <div class="gc-name" style="color:#a78bfa">黃怡駿</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#a78bfa"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g12chart_黃怡駿">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_黃怡駿">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_林嘉慈')">
            <div class="gc-name" style="color:#fb923c">林嘉慈</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#fb923c"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g12chart_林嘉慈">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_林嘉慈">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_許玲慧')">
            <div class="gc-name" style="color:#fbbf24">許玲慧</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#fbbf24"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g12chart_許玲慧">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_許玲慧">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_郭丞浤')">
            <div class="gc-name" style="color:#f87171">郭丞浤</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#f87171"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g12chart_郭丞浤">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_郭丞浤">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_洪煜棠')">
            <div class="gc-name" style="color:#60a5fa">洪煜棠</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#60a5fa"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g12chart_洪煜棠">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_洪煜棠">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x2)</span><span class="gc-chip gc-miss">✗ 主題親證3</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
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
        <div class="angel-sub">為期2週，每週一次通話｜6/15 – 6/28</div>
      </div>
    </div>
    <div class="angel-week">
      <div class="angel-week-label">第1週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 洪煜棠 &amp; 郭丞浤</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 林嘉慈 &amp; 盧家淑</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 黃怡駿 &amp; 許玲慧</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg12">🔥【第十二組｜齊天戰神突擊隊】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x2)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

戰神突擊隊新週再戰💪</div>
    <button class="copy-btn" onclick="copyMsg('msg12', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">淑</div><div><div class="card-name">盧家淑</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">29,700</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">1,340</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+2,440</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">當下之舞</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">駿</div><div><div class="card-name">黃怡駿</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">19,100</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">500</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,700</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">慈</div><div><div class="card-name">林嘉慈</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,320</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">1,780</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,780</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-purple">玲</div><div><div class="card-name">許玲慧</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">20,400</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">300</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+300</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">丞</div><div><div class="card-name">郭丞浤</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">27,840</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">860</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+860</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">煜</div><div><div class="card-name">洪煜棠</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">23,840</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">220</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,140</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

  </div>
</div>

<div class="footer">筱君大隊任務追蹤 · 截至 6/12 11:40 更新</div>

<script>
function toggleGap(uid) {
  const det = document.getElementById('det_' + uid);
  const arr = document.getElementById('arr_' + uid);
  det.classList.toggle('open');
  arr.classList.toggle('open');
}
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
