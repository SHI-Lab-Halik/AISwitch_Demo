# AISwitch_Demo
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>SecureAI Chat — Runtime Analysis</title>
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Exo+2:wght@300;400;600;700&display=swap" rel="stylesheet" />
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg:        #080c10;
    --bg2:       #0e1520;
    --bg3:       #111c2a;
    --border:    #1e3248;
    --glow-a:    #00c8ff;
    --glow-b:    #ff6a00;
    --text:      #c8dbe8;
    --text-dim:  #4a6278;
    --text-hi:   #e8f4ff;
    --panel:     rgba(14,21,32,0.95);
    --prisma:    #00c8ff;
    --custom:    #ff6a00;
    --font-mono: 'Share Tech Mono', monospace;
    --font-ui:   'Exo 2', sans-serif;
    --active-color: var(--prisma);
  }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--font-ui);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  /* ── GRID BACKGROUND ── */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background-image:
      linear-gradient(rgba(0,200,255,.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,200,255,.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  /* ── SCANLINE ── */
  body::after {
    content: '';
    position: fixed; inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0,0,0,.08) 2px,
      rgba(0,0,0,.08) 4px
    );
    pointer-events: none;
    z-index: 1;
  }

  #app {
    position: relative; z-index: 2;
    display: flex; flex-direction: column;
    height: 100vh; max-width: 960px;
    margin: 0 auto; padding: 0 16px;
  }

  /* ── HEADER ── */
  header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 18px 0 14px;
    border-bottom: 1px solid var(--border);
    flex-shrink: 0;
  }

  .logo {
    display: flex; align-items: center; gap: 10px;
  }
  .logo-icon {
    width: 36px; height: 36px;
    border: 1.5px solid var(--active-color);
    border-radius: 6px;
    display: flex; align-items: center; justify-content: center;
    position: relative;
    box-shadow: 0 0 12px color-mix(in srgb, var(--active-color) 40%, transparent);
    transition: border-color .4s, box-shadow .4s;
  }
  .logo-icon svg { width: 20px; height: 20px; }
  .logo-text { font-family: var(--font-mono); font-size: 1.05rem; color: var(--text-hi); letter-spacing: .08em; }
  .logo-sub { font-size: .65rem; color: var(--text-dim); letter-spacing: .12em; text-transform: uppercase; margin-top: 1px; }

  /* STATUS BADGE */
  .status-badge {
    display: flex; align-items: center; gap: 6px;
    font-family: var(--font-mono); font-size: .7rem;
    color: var(--text-dim); letter-spacing: .1em;
  }
  .status-dot {
    width: 7px; height: 7px; border-radius: 50%;
    background: var(--active-color);
    box-shadow: 0 0 6px var(--active-color);
    animation: pulse 2s infinite;
    transition: background .4s, box-shadow .4s;
  }
  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.4} }

  /* ── TOGGLE SECTION ── */
  .toggle-section {
    padding: 14px 0;
    border-bottom: 1px solid var(--border);
    flex-shrink: 0;
  }
  .toggle-label {
    font-size: .68rem; color: var(--text-dim); letter-spacing: .14em;
    text-transform: uppercase; font-family: var(--font-mono);
    margin-bottom: 10px;
  }

  .engine-toggle {
    display: flex; gap: 0;
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 8px; overflow: hidden;
    width: fit-content;
  }
  .engine-btn {
    padding: 8px 22px;
    font-family: var(--font-ui); font-size: .82rem; font-weight: 600;
    letter-spacing: .06em; color: var(--text-dim);
    background: transparent; border: none; cursor: pointer;
    transition: all .25s; position: relative; user-select: none;
  }
  .engine-btn::after {
    content: ''; position: absolute; bottom: 0; left: 10%; width: 80%; height: 2px;
    background: currentColor; transform: scaleX(0);
    transition: transform .25s;
  }
  .engine-btn.active { color: var(--active-color); }
  .engine-btn.active::after { transform: scaleX(1); }
  .engine-btn:hover:not(.active) { color: var(--text); }

  .engine-sep { width: 1px; background: var(--border); }

  /* CONFIG STRIP */
  .config-strip {
    margin-top: 10px;
    display: flex; gap: 8px; align-items: center; flex-wrap: wrap;
  }
  .cfg-field {
    display: flex; flex-direction: column; gap: 3px; flex: 1; min-width: 160px; max-width: 320px;
  }
  .cfg-field label {
    font-size: .62rem; color: var(--text-dim); letter-spacing: .1em; font-family: var(--font-mono); text-transform: uppercase;
  }
  .cfg-field input {
    background: var(--bg3); border: 1px solid var(--border);
    border-radius: 5px; padding: 6px 10px;
    font-family: var(--font-mono); font-size: .78rem; color: var(--text-hi);
    outline: none; transition: border-color .2s, box-shadow .2s;
    width: 100%;
  }
  .cfg-field input:focus {
    border-color: var(--active-color);
    box-shadow: 0 0 0 2px color-mix(in srgb, var(--active-color) 15%, transparent);
  }
  .cfg-field input::placeholder { color: var(--text-dim); }

  .save-btn {
    padding: 7px 18px; margin-top: 14px;
    background: color-mix(in srgb, var(--active-color) 12%, transparent);
    border: 1px solid var(--active-color);
    border-radius: 5px; color: var(--active-color);
    font-family: var(--font-mono); font-size: .75rem; letter-spacing: .1em;
    cursor: pointer; transition: all .2s; flex-shrink: 0;
  }
  .save-btn:hover {
    background: color-mix(in srgb, var(--active-color) 22%, transparent);
    box-shadow: 0 0 10px color-mix(in srgb, var(--active-color) 30%, transparent);
  }

  .engine-tag {
    display: inline-flex; align-items: center; gap: 5px;
    font-family: var(--font-mono); font-size: .68rem;
    padding: 3px 10px; border-radius: 20px;
    border: 1px solid var(--active-color);
    color: var(--active-color);
    background: color-mix(in srgb, var(--active-color) 8%, transparent);
    letter-spacing: .08em;
    transition: all .4s;
    flex-shrink: 0; margin-top: 14px;
  }

  /* ── CHAT AREA ── */
  .chat-wrap {
    flex: 1; overflow-y: auto; padding: 16px 0;
    display: flex; flex-direction: column; gap: 12px;
    scroll-behavior: smooth;
  }
  .chat-wrap::-webkit-scrollbar { width: 4px; }
  .chat-wrap::-webkit-scrollbar-track { background: transparent; }
  .chat-wrap::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  .msg {
    display: flex; gap: 10px; align-items: flex-start;
    animation: fadeUp .25s ease;
  }
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(8px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  .msg.user { flex-direction: row-reverse; }

  .avatar {
    width: 30px; height: 30px; border-radius: 6px;
    display: flex; align-items: center; justify-content: center;
    font-size: .75rem; flex-shrink: 0;
    border: 1px solid var(--border);
  }
  .msg.ai .avatar {
    background: color-mix(in srgb, var(--active-color) 10%, var(--bg2));
    border-color: var(--active-color);
    color: var(--active-color);
    transition: background .4s, border-color .4s, color .4s;
  }
  .msg.user .avatar {
    background: var(--bg3); color: var(--text-dim);
  }

  .bubble {
    max-width: 78%;
    padding: 10px 14px;
    border-radius: 8px;
    font-size: .88rem; line-height: 1.6;
    border: 1px solid var(--border);
    position: relative;
  }
  .msg.ai .bubble {
    background: var(--bg2);
    border-color: color-mix(in srgb, var(--active-color) 25%, var(--border));
    color: var(--text);
  }
  .msg.user .bubble {
    background: var(--bg3);
    color: var(--text-hi);
    text-align: right;
  }

  .bubble .engine-watermark {
    font-family: var(--font-mono); font-size: .6rem;
    color: var(--active-color); opacity: .5;
    letter-spacing: .1em; margin-bottom: 4px;
    display: block;
  }

  /* TYPING INDICATOR */
  .typing-dots { display: flex; gap: 4px; padding: 4px 0; }
  .typing-dots span {
    width: 6px; height: 6px; border-radius: 50%;
    background: var(--active-color); opacity: .4;
    animation: blink 1.2s infinite;
    transition: background .4s;
  }
  .typing-dots span:nth-child(2) { animation-delay: .2s; }
  .typing-dots span:nth-child(3) { animation-delay: .4s; }
  @keyframes blink { 0%,80%,100%{opacity:.2} 40%{opacity:1} }

  /* ── INPUT BAR ── */
  .input-bar {
    display: flex; gap: 8px; align-items: flex-end;
    padding: 12px 0 16px;
    border-top: 1px solid var(--border);
    flex-shrink: 0;
  }
  textarea#chatInput {
    flex: 1;
    background: var(--bg2); border: 1px solid var(--border);
    border-radius: 8px; padding: 10px 14px;
    font-family: var(--font-ui); font-size: .88rem; color: var(--text-hi);
    resize: none; outline: none; line-height: 1.5;
    min-height: 44px; max-height: 120px;
    transition: border-color .2s, box-shadow .2s;
  }
  textarea#chatInput:focus {
    border-color: var(--active-color);
    box-shadow: 0 0 0 2px color-mix(in srgb, var(--active-color) 12%, transparent);
  }
  textarea#chatInput::placeholder { color: var(--text-dim); }

  #sendBtn {
    width: 44px; height: 44px; border-radius: 8px;
    background: color-mix(in srgb, var(--active-color) 15%, var(--bg2));
    border: 1px solid var(--active-color);
    color: var(--active-color);
    display: flex; align-items: center; justify-content: center;
    cursor: pointer; transition: all .2s; flex-shrink: 0;
  }
  #sendBtn:hover {
    background: color-mix(in srgb, var(--active-color) 28%, var(--bg2));
    box-shadow: 0 0 14px color-mix(in srgb, var(--active-color) 35%, transparent);
  }
  #sendBtn:disabled { opacity: .4; cursor: not-allowed; }

  /* ── THEME TRANSITIONS ── */
  .prisma-mode { --active-color: var(--prisma); }
  .custom-mode  { --active-color: var(--custom); }

  /* WELCOME BLOCK */
  .welcome {
    text-align: center; padding: 32px 20px;
    color: var(--text-dim); font-size: .85rem; line-height: 1.7;
  }
  .welcome h2 {
    font-family: var(--font-mono); font-size: 1rem;
    color: var(--active-color); margin-bottom: 8px; letter-spacing: .1em;
    transition: color .4s;
  }

  /* NOTIFICATION TOAST */
  #toast {
    position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%) translateY(60px);
    background: var(--bg3); border: 1px solid var(--active-color);
    border-radius: 6px; padding: 8px 18px;
    font-family: var(--font-mono); font-size: .75rem; color: var(--active-color);
    transition: transform .3s, opacity .3s, border-color .4s, color .4s;
    opacity: 0; pointer-events: none; z-index: 100;
  }
  #toast.show { transform: translateX(-50%) translateY(0); opacity: 1; }

</style>
</head>
<body class="prisma-mode">
<div id="app">

  <!-- HEADER -->
  <header>
    <div class="logo">
      <div class="logo-icon" id="logoIcon">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" style="color:var(--active-color);transition:color .4s">
          <path d="M12 2L2 7l10 5 10-5-10-5z"/>
          <path d="M2 17l10 5 10-5"/>
          <path d="M2 12l10 5 10-5"/>
        </svg>
      </div>
      <div>
        <div class="logo-text">SecureAI<span style="color:var(--active-color);transition:color .4s">Chat</span></div>
        <div class="logo-sub">Runtime Threat Analysis</div>
      </div>
    </div>
    <div class="status-badge">
      <div class="status-dot" id="statusDot"></div>
      <span id="statusLabel">PRISMA AIRS CONNECTED</span>
    </div>
  </header>

  <!-- TOGGLE -->
  <div class="toggle-section">
    <div class="toggle-label">&#9654; Select Analysis Engine</div>
    <div style="display:flex;align-items:center;gap:14px;flex-wrap:wrap;">
      <div class="engine-toggle">
        <button class="engine-btn active" id="btnPrisma" onclick="switchEngine('prisma')">Palo Alto Prisma AIRS</button>
        <div class="engine-sep"></div>
        <button class="engine-btn" id="btnCustom" onclick="switchEngine('custom')">Custom API</button>
      </div>
      <div class="engine-tag" id="engineTag">&#x25CF; PRISMA AIRS</div>
    </div>

    <!-- CONFIG FIELDS -->
    <div class="config-strip" id="configStrip">
      <!-- injected by JS -->
    </div>
  </div>

  <!-- CHAT -->
  <div class="chat-wrap" id="chatWrap">
    <div class="welcome" id="welcomeMsg">
      <h2 id="welcomeTitle">// PRISMA AIRS ENGINE READY</h2>
      <p>Connected to <strong style="color:var(--text-hi)">Palo Alto Prisma AI Runtime Security</strong>.<br/>
      Enter your API key above, then ask about threat analysis, anomaly detection, or security posture.</p>
    </div>
  </div>

  <!-- INPUT -->
  <div class="input-bar">
    <textarea id="chatInput" rows="1" placeholder="Ask about threats, anomalies, policy violations…" onkeydown="handleKey(event)"></textarea>
    <button id="sendBtn" onclick="sendMessage()" title="Send">
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
        <line x1="22" y1="2" x2="11" y2="13"/>
        <polygon points="22 2 15 22 11 13 2 9 22 2"/>
      </svg>
    </button>
  </div>

</div>

<div id="toast"></div>

<script>
// ── STATE ──
let currentEngine = 'prisma';
let config = {
  prisma: { apiKey: '', tenantId: '', region: 'us' },
  custom: { endpoint: '', apiKey: '', headerName: 'Authorization' }
};

const ENGINES = {
  prisma: {
    name: 'Palo Alto Prisma AIRS',
    tag: '◉ PRISMA AIRS',
    status: 'PRISMA AIRS CONNECTED',
    mode: 'prisma-mode',
    welcome: '// PRISMA AIRS ENGINE READY',
    welcomeBody: 'Connected to <strong style="color:var(--text-hi)">Palo Alto Prisma AI Runtime Security</strong>.<br/>Enter your API key above, then ask about threat analysis, anomaly detection, or security posture.',
    fields: [
      { key:'apiKey',    label:'API Key',    type:'password', placeholder:'••••••••••••••••••••' },
      { key:'tenantId',  label:'Tenant ID',  type:'text',     placeholder:'your-tenant-id' },
      { key:'region',    label:'Region',     type:'text',     placeholder:'us / eu / ap' }
    ]
  },
  custom: {
    name: 'Custom API Endpoint',
    tag: '◉ CUSTOM API',
    status: 'CUSTOM ENDPOINT CONNECTED',
    mode: 'custom-mode',
    welcome: '// CUSTOM API ENGINE READY',
    welcomeBody: 'Generic AI/security API endpoint active.<br/>Configure your endpoint URL and auth header below, then start querying.',
    fields: [
      { key:'endpoint',   label:'Endpoint URL',   type:'text',     placeholder:'https://api.yourtool.com/v1/analyze' },
      { key:'apiKey',     label:'API Key / Token', type:'password', placeholder:'••••••••••••••••••••' },
      { key:'headerName', label:'Auth Header',     type:'text',     placeholder:'Authorization' }
    ]
  }
};

// ── INIT ──
function init() {
  renderConfig();
  autoGrow();
}

function switchEngine(engine) {
  if (currentEngine === engine) return;
  currentEngine = engine;
  const e = ENGINES[engine];

  // theme
  document.body.className = e.mode;

  // buttons
  document.getElementById('btnPrisma').classList.toggle('active', engine === 'prisma');
  document.getElementById('btnCustom').classList.toggle('active', engine === 'custom');

  // status
  document.getElementById('engineTag').textContent = e.tag;
  document.getElementById('statusLabel').textContent = e.status;

  // welcome
  document.getElementById('welcomeTitle').textContent = e.welcome;
  document.getElementById('welcomeMsg').querySelector('p').innerHTML = e.welcomeBody;

  // placeholder input
  document.getElementById('chatInput').placeholder =
    engine === 'prisma'
      ? 'Ask about threats, anomalies, policy violations…'
      : 'Send a query to your custom AI endpoint…';

  renderConfig();
  showToast('Engine switched → ' + e.name);
}

function renderConfig() {
  const strip = document.getElementById('configStrip');
  const e = ENGINES[currentEngine];
  const cfg = config[currentEngine];

  strip.innerHTML = '';
  e.fields.forEach(f => {
    const wrap = document.createElement('div');
    wrap.className = 'cfg-field';
    wrap.innerHTML = `
      <label>${f.label}</label>
      <input type="${f.type}" placeholder="${f.placeholder}" value="${cfg[f.key] || ''}"
        data-key="${f.key}" oninput="updateConfig(this)" autocomplete="off" spellcheck="false"/>
    `;
    strip.appendChild(wrap);
  });

  const btn = document.createElement('button');
  btn.className = 'save-btn';
  btn.textContent = 'SAVE CONFIG';
  btn.onclick = saveConfig;
  strip.appendChild(btn);
}

function updateConfig(input) {
  config[currentEngine][input.dataset.key] = input.value;
}

function saveConfig() {
  showToast('Configuration saved ✓');
}

// ── CHAT ──
let chatHistory = []; // [{role, content}]
let isTyping = false;

function handleKey(e) {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
  autoGrow();
}

function autoGrow() {
  const ta = document.getElementById('chatInput');
  ta.style.height = 'auto';
  ta.style.height = Math.min(ta.scrollHeight, 120) + 'px';
}

async function sendMessage() {
  const ta = document.getElementById('chatInput');
  const text = ta.value.trim();
  if (!text || isTyping) return;

  // clear welcome
  const welcome = document.getElementById('welcomeMsg');
  if (welcome) welcome.remove();

  appendMsg('user', text);
  ta.value = '';
  ta.style.height = 'auto';
  chatHistory.push({ role: 'user', content: text });

  const btn = document.getElementById('sendBtn');
  btn.disabled = true;
  isTyping = true;

  const typingEl = appendTyping();

  try {
    const reply = await callEngine(text);
    typingEl.remove();
    appendMsg('ai', reply);
    chatHistory.push({ role: 'assistant', content: reply });
  } catch (err) {
    typingEl.remove();
    appendMsg('ai', '⚠ ' + err.message);
  }

  btn.disabled = false;
  isTyping = false;
}

async function callEngine(userText) {
  const cfg = config[currentEngine];
  const e = ENGINES[currentEngine];

  if (currentEngine === 'prisma') {
    if (!cfg.apiKey || !cfg.tenantId) {
      return simulatePrisma(userText);
    }
    return await callPrismaAIRS(userText, cfg);
  } else {
    if (!cfg.endpoint || !cfg.apiKey) {
      return simulateCustom(userText);
    }
    return await callCustomAPI(userText, cfg);
  }
}

// ── PRISMA AIRS REAL CALL ──
async function callPrismaAIRS(text, cfg) {
  // Palo Alto Prisma AIRS — adjust endpoint path to match your deployment
  const url = `https://api.prismacloud.io/ai-runtime/v1/chat`;
  const res = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-redlock-auth': cfg.apiKey,
      'x-tenant-id': cfg.tenantId
    },
    body: JSON.stringify({
      query: text,
      region: cfg.region || 'us',
      history: chatHistory.slice(-10)
    })
  });
  if (!res.ok) throw new Error(`Prisma AIRS API error: ${res.status} ${res.statusText}`);
  const data = await res.json();
  return data.response || data.answer || JSON.stringify(data);
}

// ── CUSTOM API REAL CALL ──
async function callCustomAPI(text, cfg) {
  const headerVal = cfg.headerName.toLowerCase().includes('bearer') || cfg.headerName === 'Authorization'
    ? `Bearer ${cfg.apiKey}` : cfg.apiKey;

  const res = await fetch(cfg.endpoint, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      [cfg.headerName]: headerVal
    },
    body: JSON.stringify({ query: text, messages: chatHistory.slice(-10) })
  });
  if (!res.ok) throw new Error(`API error: ${res.status} ${res.statusText}`);
  const data = await res.json();
  // try common response keys
  return data.response || data.answer || data.result || data.output
    || data.choices?.[0]?.message?.content
    || JSON.stringify(data, null, 2);
}

// ── SIMULATED RESPONSES (no API key) ──
const prismaDemo = [
  "**[PRISMA AIRS — DEMO MODE]**\n\nNo API key configured. In production, Prisma AIRS would scan this query against your cloud workloads and return real-time threat context.\n\n_Configure your API Key and Tenant ID above to connect live._",
  "**[PRISMA AIRS — DEMO MODE]**\n\nPrisma AI Runtime Security monitors AI model inputs/outputs for prompt injection, data exfiltration attempts, and policy violations. Enter your credentials above to enable live analysis.",
  "**[PRISMA AIRS — DEMO MODE]**\n\nIn a live deployment, this response would include anomaly scores, policy match results, and remediation recommendations from your Prisma Cloud tenant.",
];
const customDemo = [
  "**[CUSTOM API — DEMO MODE]**\n\nNo endpoint configured. Enter your API endpoint URL and authentication token above.\n\nThis connector supports any REST API that accepts `{ query, messages }` and returns a `response` or `answer` field.",
  "**[CUSTOM API — DEMO MODE]**\n\nOnce configured, every message will be forwarded to your endpoint with full conversation history. Common integrations: AWS GuardDuty AI, Microsoft Defender XDR, Splunk AI, Wiz, or any custom ML inference endpoint.",
];

let demoIdx = { prisma: 0, custom: 0 };
function simulatePrisma(text) {
  const arr = prismaDemo;
  const r = arr[demoIdx.prisma % arr.length]; demoIdx.prisma++;
  return new Promise(res => setTimeout(() => res(r), 900 + Math.random() * 600));
}
function simulateCustom(text) {
  const arr = customDemo;
  const r = arr[demoIdx.custom % arr.length]; demoIdx.custom++;
  return new Promise(res => setTimeout(() => res(r), 800 + Math.random() * 500));
}

// ── DOM HELPERS ──
function appendMsg(role, content) {
  const wrap = document.getElementById('chatWrap');
  const el = document.createElement('div');
  el.className = `msg ${role === 'user' ? 'user' : 'ai'}`;

  const avatar = document.createElement('div');
  avatar.className = 'avatar';
  avatar.textContent = role === 'user' ? 'YOU' : 'AI';

  const bubble = document.createElement('div');
  bubble.className = 'bubble';

  if (role === 'ai') {
    const wm = document.createElement('span');
    wm.className = 'engine-watermark';
    wm.textContent = currentEngine === 'prisma' ? '▸ PRISMA AIRS' : '▸ CUSTOM API';
    bubble.appendChild(wm);
  }

  const body = document.createElement('div');
  body.innerHTML = formatMsg(content);
  bubble.appendChild(body);

  el.appendChild(avatar);
  el.appendChild(bubble);
  wrap.appendChild(el);
  wrap.scrollTop = wrap.scrollHeight;
  return el;
}

function appendTyping() {
  const wrap = document.getElementById('chatWrap');
  const el = document.createElement('div');
  el.className = 'msg ai';
  el.innerHTML = `
    <div class="avatar">AI</div>
    <div class="bubble">
      <span class="engine-watermark">${currentEngine === 'prisma' ? '▸ PRISMA AIRS' : '▸ CUSTOM API'}</span>
      <div class="typing-dots"><span></span><span></span><span></span></div>
    </div>`;
  wrap.appendChild(el);
  wrap.scrollTop = wrap.scrollHeight;
  return el;
}

function formatMsg(text) {
  return text
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    .replace(/_(.*?)_/g, '<em>$1</em>')
    .replace(/`([^`]+)`/g, '<code style="background:var(--bg3);padding:1px 5px;border-radius:3px;font-family:var(--font-mono);font-size:.82em">$1</code>')
    .replace(/\n/g, '<br/>');
}

function showToast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2400);
}

// textarea auto-grow
document.getElementById('chatInput').addEventListener('input', autoGrow);

init();
</script>
</body>
</html>
