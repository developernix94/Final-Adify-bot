<!doctype html>
<html lang="bn">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Adify — Telegram Web App</title>
  <meta name="description" content="Adify — Telegram Mini App" />
  <style>
    :root{--bg:#f6f8fb;--card:#fff;--accent:#1766d1;--muted:#6b7280;--pad:14px;--radius:12px}
    *{box-sizing:border-box}
    body{font-family:Inter,system-ui,Arial,sans-serif;margin:0;background:var(--bg);color:#111;line-height:1.3}
    .wrap{max-width:920px;margin:28px auto;padding:16px}
    .card{background:var(--card);border-radius:var(--radius);padding:var(--pad);box-shadow:0 6px 24px rgba(20,20,50,0.06);margin-bottom:16px}
    header{display:flex;align-items:center;gap:12px}
    header h1{margin:0;font-size:20px}
    .muted{color:var(--muted);font-size:13px}
    .row{display:flex;gap:12px;flex-wrap:wrap}
    .col{flex:1;min-width:220px}
    .big{font-size:28px;font-weight:700}
    button{background:var(--accent);color:#fff;border:0;padding:10px 12px;border-radius:8px;cursor:pointer;font-weight:600}
    input,select,textarea{width:100%;padding:10px;border-radius:8px;border:1px solid #e6e9ee}
    label{display:block;font-size:13px;margin-bottom:6px;color:var(--muted)}
    .small{font-size:13px;color:var(--muted)}
    .notice{background:#fffbe8;border:1px solid #ffe7b5;padding:10px;border-radius:8px;color:#6a4a00}
    .success{background:#ecfdf5;border:1px solid #bbf7d0;padding:10px;border-radius:8px;color:#065f46}
    footer{text-align:center;color:var(--muted);font-size:13px;margin-top:8px}
    .disabled{opacity:0.5;pointer-events:none}
    @media (max-width:700px){.row{flex-direction:column}}
    /* Monetag ad container style (banner) */
    .ad-banner{display:block;text-align:center;padding:8px;border-radius:8px;background:#fff;margin-top:10px}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <header>
        <div>
          <h1>Adify</h1>
          <p class="muted">Earn points by watching ads — 100 points = $0.10 — Withdraw min 100 points</p>
        </div>
        <div style="margin-left:auto;text-align:right">
          <div id="userName" class="muted">Not signed in</div>
          <div id="userId" class="small"></div>
        </div>
      </header>
    </div>

    <div class="row">
      <div class="col card">
        <div class="muted">Points</div>
        <div class="big" id="points">0</div>
        <div class="small">Equivalent: <span id="dollars">$0.00</span></div>
        <div style="margin-top:12px">
          <button id="watchInterstitialBtn">Watch Interstitial (Rewarded)</button>
          <button id="watchPopupBtn" style="background:#10b981;margin-left:8px">Watch Popup (Rewarded)</button>
        </div>

        <div style="margin-top:12px;">
          <button id="openInAppBtn" style="background:#8b5cf6">Enable In-App Interstitial</button>
        </div>

        <!-- banner ad slot (Monetag will inject) -->
        <div id="adBanner" class="ad-banner">
          <!-- Monetag banner placeholder will be placed here -->
        </div>
      </div>

      <div class="col card">
        <div class="muted">Profile</div>
        <div id="profileBox">
          <div class="small">Name: <strong id="pfName">—</strong></div>
          <div class="small">Username: <strong id="pfUsername">—</strong></div>
          <div class="small">Registered: <strong id="pfRegistered">—</strong></div>
        </div>
        <div style="margin-top:12px">
          <button id="openBotBtn">Open Bot Chat</button>
        </div>
      </div>
    </div>

    <div class="card">
      <div class="muted">Withdraw</div>
      <div class="small" style="margin-bottom:8px">Minimum withdraw: <strong>100 points (= $0.10)</strong></div>

      <div style="display:flex;gap:8px;flex-wrap:wrap">
        <div style="flex:1">
          <label>Amount (points)</label>
          <input id="withdrawPoints" placeholder="100" />
        </div>
        <div style="flex:1">
          <label>Method</label>
          <select id="withdrawMethod">
            <option value="bKash">bKash</option>
            <option value="Nagad">Nagad</option>
            <option value="BankTransfer">BankTransfer</option>
            <option value="Other">Other</option>
          </select>
        </div>
      </div>

      <div style="margin-top:10px">
        <label>Details (account number / notes)</label>
        <textarea id="withdrawDetails" rows="2" placeholder="Account number or instructions"></textarea>
      </div>

      <div style="margin-top:10px;display:flex;gap:8px">
        <button id="submitWithdrawBtn">Submit Withdraw</button>
        <button id="refreshBtn" style="background:#6b7280">Refresh</button>
      </div>

      <div style="margin-top:10px">
        <div id="withdrawMsg" class="small muted"></div>
      </div>
    </div>

    <div class="card">
      <div class="muted">Join Channel (required)</div>
      <div class="small">Please join the channel <strong>@XpertzXProfitz</strong> if required. Then press Check Join.</div>
      <div style="margin-top:8px;display:flex;gap:8px">
        <button id="joinChannelBtn">Open Channel</button>
        <button id="checkJoinBtn">Check Join</button>
      </div>
      <div style="margin-top:8px"><div id="joinStatus" class="small muted">Status: unknown</div></div>
    </div>

    <footer class="muted">Adify • Bot: @Adify_01_Bot • Your GitHub Pages UI</footer>
  </div>

  <!-- Monetag SDK: zone 10136294 -->
  <script src="//libtl.com/sdk.js" data-zone="10136294" data-sdk="show_10136294"></script>

  <script>
  /*************************************************************************
   * Final Adify frontend
   * - Monetag integrated (zone 10136294)
   * - All ad types enabled (interstitial, popup, inApp)
   * - 1 ad = +1 point. 100 points = $0.10
   * - Withdraw minimum 100 points
   *
   * Edit SERVER_URL below when your backend is deployed (Render/Railway).
   *************************************************************************/

  // ===== CONFIGURE =====
  const SERVER_URL = ''; // <-- set to your backend API base URL (e.g. https://adify-backend.onrender.com). If empty, frontend uses localStorage only.
  const BOT_USERNAME = '@Adify_01_Bot';
  const REQUIRED_CHANNEL = '@XpertzXProfitz';
  const MIN_POINTS_WITHDRAW = 100;
  const POINTS_TO_USD = 0.001; // 1 point = $0.001 (100 points = $0.10)
  const MONETAG_SHOW = window.show_10136294; // provided by Monetag SDK

  // ===== state =====
  let TG_USER = null; // {id, first_name, username}
  let points = 0;
  let synced = false; // whether we synced to server
  let inAppEnabled = false;

  // ===== helpers =====
  const el = id => document.getElementById(id);
  function setText(id, txt){ const e = el(id); if(e) e.textContent = txt; }
  function formatUSD(n){ return '$' + Number(n).toFixed(2); }

  // Load points from backend or localStorage
  async function loadPoints() {
    points = 0;
    // try backend if available
    if (SERVER_URL && TG_USER) {
      try {
        const r = await fetch(`${SERVER_URL}/api/user?user_id=${encodeURIComponent(TG_USER.id)}`);
        const j = await r.json();
        if (j && j.ok && j.user) {
          // backend must expose `points` optionally; else use balance conversion if provided.
          if (typeof j.user.points !== 'undefined') {
            points = Number(j.user.points) || 0;
            synced = true;
          } else if (typeof j.user.balance !== 'undefined') {
            // if backend stores balance in dollars, convert to points: points = balance / 0.001
            const bal = Number(j.user.balance) || 0;
            points = Math.round(bal / POINTS_TO_USD);
            synced = true;
          }
        }
      } catch (e) {
        console.warn('loadPoints backend failed', e);
      }
    }
    if (!synced) {
      const ls = localStorage.getItem('adify_points_' + (TG_USER ? TG_USER.id : 'anon'));
      points = ls ? Number(ls) : 0;
    }
    updateUI();
  }

  // Save points to backend if endpoint exists, otherwise to localStorage
  async function savePoints() {
    if (SERVER_URL && TG_USER) {
      try {
        // attempt to POST to /api/sync_points (optional endpoint). Server may ignore; handle gracefully.
        const resp = await fetch(`${SERVER_URL}/api/sync_points`, {
          method: 'POST',
          headers: {'Content-Type':'application/json'},
          body: JSON.stringify({ user_id: TG_USER.id, points })
        });
        if (resp.ok) { synced = true; return; }
      } catch(e){ console.warn('sync_points failed', e); }
    }
    // fallback localStorage
    localStorage.setItem('adify_points_' + (TG_USER ? TG_USER.id : 'anon'), String(points));
  }

  function updateUI() {
    setText('points', String(points));
    setText('dollars', formatUSD(points * POINTS_TO_USD));
    setText('pfName', TG_USER ? (TG_USER.first_name || '—') : '—');
    setText('pfUsername', TG_USER ? (TG_USER.username || '—') : '—');
    setText('pfRegistered', '—');
    if (points >= MIN_POINTS_WITHDRAW) {
      el('submitWithdrawBtn').classList.remove('disabled');
    } else {
      el('submitWithdrawBtn').classList.add('disabled');
    }
  }

  // ===== Monetag ad wrappers =====
  async function onAdWatched() {
    // Called after Monetag indicates ad was watched/completed
    points += 1;
    await savePoints();
    updateUI();
    // Optionally inform backend of new points via /api/add_points (best-effort)
    if (SERVER_URL && TG_USER) {
      try {
        await fetch(`${SERVER_URL}/api/add_points`, {
          method: 'POST',
          headers: {'Content-Type':'application/json'},
          body: JSON.stringify({ user_id: TG_USER.id, points: 1 })
        });
      } catch(e){ /* ignore */ }
    }
    alert('✅ You earned 1 point!');
  }

  async function showRewardedInterstitial() {
    if (!MONETAG_SHOW) return alert('Monetag SDK not loaded yet.');
    try {
      // default rewarded interstitial
      MONETAG_SHOW().then(async () => {
        await onAdWatched();
      }).catch(err=>{
        console.warn('interstitial ad error',err);
      });
    } catch(e){ console.warn(e); }
  }

  async function showRewardedPopup() {
    if (!MONETAG_SHOW) return alert('Monetag SDK not loaded yet.');
    try {
      MONETAG_SHOW('pop').then(async () => {
        await onAdWatched();
      }).catch(e=>{
        console.warn('popup ad error',e);
      });
    } catch(e){ console.warn(e); }
  }

  async function enableInAppInterstitial() {
    if (!MONETAG_SHOW) return alert('Monetag SDK not loaded yet.');
    try {
      // inApp configuration as you gave: frequency:2, capping:0.1h, interval:30s, timeout:5s, everyPage:false
      MONETAG_SHOW({
        type: 'inApp',
        inAppSettings: {
          frequency: 2,
          capping: 0.1,
          interval: 30,
          timeout: 5,
          everyPage: false
        }
      });
      inAppEnabled = true;
      el('openInAppBtn').textContent = 'In-App Interstitial Enabled';
      // The in-app ads run automatically; we still need a handler to reward for ads when SDK exposes callbacks.
      // Monetag's SDK calls returned promise only for explicit calls; auto inApp may not return per-ad promises.
      // If SDK supports event/callback to JS, integrate onAdWatched there. Otherwise, keep manual watch buttons.
    } catch(e){ console.warn(e); }
  }

  // ===== Withdraw flow =====
  async function submitWithdraw() {
    const userId = TG_USER ? TG_USER.id : prompt('Enter your Telegram user id:');
    if (!userId) return alert('User id required');
    const ptsRaw = el('withdrawPoints').value.trim();
    const pts = Number(ptsRaw);
    if (!pts || pts < MIN_POINTS_WITHDRAW) return alert(`Minimum withdraw is ${MIN_POINTS_WITHDRAW} points.`);
    if (pts > points) return alert('You do not have enough points.');
    const method = el('withdrawMethod').value;
    const details = el('withdrawDetails').value.trim();
    if (/binance/i.test(method) || /binance/i.test(details)) return alert('Binance is not allowed.');

    // convert points to dollars: 1 pt = $0.001
    const dollars = +(pts * POINTS_TO_USD).toFixed(6);

    // If server exists, send withdraw request to backend
    if (SERVER_URL) {
      try {
        el('submitWithdrawBtn').disabled = true;
        setText('withdrawMsg','Submitting withdraw request...');
        const resp = await fetch(`${SERVER_URL}/api/withdraw`, {
          method:'POST',
          headers:{'Content-Type':'application/json'},
          body: JSON.stringify({ user_id: userId, amount: dollars, method, details })
        });
        const j = await resp.json();
        if (!resp.ok) throw new Error(j.error || 'Server error');
        // on success, deduct points locally and update
        points -= pts;
        await savePoints();
        updateUI();
        setText('withdrawMsg','Withdraw request submitted. Admin will review.');
      } catch (err) {
        console.error('withdraw error', err);
        setText('withdrawMsg', 'Failed to submit withdraw: ' + (err.message || err));
      } finally { el('submitWithdrawBtn').disabled = false; }
      return;
    }

    // If no server, just record locally (admin must manually process)
    // create a local withdraw record in localStorage
    const rec = { id: 'local-' + Date.now(), user_id: userId, points: pts, amount: dollars, method, details, status:'pending', created_at: new Date().toISOString() };
    const list = JSON.parse(localStorage.getItem('adify_local_withdraws') || '[]');
    list.unshift(rec);
    localStorage.setItem('adify_local_withdraws', JSON.stringify(list));
    points -= pts;
    await savePoints();
    updateUI();
    setText('withdrawMsg', 'Withdraw recorded locally. Admin must process manually.');
  }

  // ===== Join check =====
  async function checkJoin() {
    const uid = TG_USER ? TG_USER.id : prompt('Enter your Telegram id:');
    if (!uid) return;
    if (SERVER_URL) {
      try {
        setText('joinStatus','Checking...');
        const r = await fetch(`${SERVER_URL}/api/check_join?user_id=${encodeURIComponent(uid)}`);
        const j = await r.json();
        if (j && j.ok) {
          setText('joinStatus', j.joined ? 'Status: Joined ✅' : 'Status: Not joined ❌');
          return;
        }
      } catch(e){ console.warn('check_join failed', e); }
    }
    setText('joinStatus','Unable to verify (server not configured).');
  }

  // ===== Open bot chat =====
  function openBotChat() {
    const bot = BOT_USERNAME.replace('@','');
    const tgUrl = `tg://resolve?domain=${bot}`;
    const webUrl = `https://t.me/${bot}`;
    window.location.href = tgUrl;
    setTimeout(()=> window.open(webUrl, '_blank'), 700);
  }

  // ===== Init Telegram/WebApp data =====
  function initTelegram() {
    try {
      const tg = window.Telegram && window.Telegram.WebApp;
      if (tg && tg.initDataUnsafe && tg.initDataUnsafe.user) {
        const u = tg.initDataUnsafe.user;
        TG_USER = { id: u.id, first_name: u.first_name || '', username: u.username || '' };
      }
    } catch(e){ console.warn('Telegram init failed', e); }
    if (TG_USER) {
      setText('userName', TG_USER.first_name + (TG_USER.username ? ' (@' + TG_USER.username + ')' : ''));
      setText('userId', 'ID: ' + TG_USER.id);
      // try to fetch profile details from server
      fetchProfile();
    } else {
      setText('userName', 'Open inside Telegram to auto-fill user');
      setText('userId','');
    }
  }

  async function fetchProfile(){
    if (!TG_USER) return;
    if (!SERVER_URL) return loadPoints();
    try {
      const r = await fetch(`${SERVER_URL}/api/user?user_id=${encodeURIComponent(TG_USER.id)}`);
      const j = await r.json();
      if (j && j.ok && j.user) {
        setText('pfName', j.user.first_name || '');
        setText('pfUsername', j.user.username || '');
        setText('pfRegistered', j.user.created_at || '—');
        // try to extract points from server user (if server returns points)
        if (typeof j.user.points !== 'undefined') {
          points = Number(j.user.points) || 0;
          await savePoints();
        }
      }
    } catch(e){ console.warn('fetchProfile failed', e); }
    await loadPoints();
  }

  // ===== Bind events =====
  document.addEventListener('DOMContentLoaded', () => {
    // Monetag banner injection: insert script element if needed (the SDK already loaded above)
    // If you want a banner display, Monetag SDK sometimes supports rendering to a container.
    // We'll insert a generic script tag (Monetag handles rendering based on zone).
    const adBanner = el('adBanner');
    if (adBanner) {
      // If Monetag supports auto-rendering inside a container, this will work.
      // For safety, just show a small note until monetag injects content.
      adBanner.innerHTML = '<div style="font-size:13px;color:#6b7280">Ad loading...</div>';
    }

    initTelegram();
    el('watchInterstitialBtn').addEventListener('click', showRewardedInterstitial);
    el('watchPopupBtn').addEventListener('click', showRewardedPopup);
    el('openInAppBtn').addEventListener('click', enableInAppInterstitial);
    el('submitWithdrawBtn').addEventListener('click', submitWithdraw);
    el('refreshBtn').addEventListener('click', fetchProfile);
    el('openBotBtn').addEventListener('click', openBotChat);
    el('joinChannelBtn').addEventListener('click', () => {
      const url = `tg://resolve?domain=${REQUIRED_CHANNEL.replace('@','')}`;
      window.location.href = url;
      setTimeout(()=> window.open(`https://t.me/${REQUIRED_CHANNEL.replace('@','')}`,'_blank'),700);
    });
    el('checkJoinBtn').addEventListener('click', checkJoin);

    loadPoints();
    // Optionally, place a Monetag banner script inside adBanner:
    // Monetag global SDK may allow rendering like: show_10136294('banner', {container: '#adBanner'});
    // But exact API depends on Monetag docs. The SDK we included will auto-render typical slots.
  });

  // Expose functions for debugging
  window.Adify = { showRewardedInterstitial, showRewardedPopup, enableInAppInterstitial, points, loadPoints, savePoints };

  </script>
</body>
</html>
