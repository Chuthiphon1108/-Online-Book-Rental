<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ระบบเช่าหนังสือออนไลน์</title>
  <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet">
  <style>
    :root {
      --navy: #0f2b4a; --navy2: #163a60; --blue: #1d6fa4; --sky: #2ea8d5;
      --teal: #0fb8a0; --bg: #f0f4f8; --white: #fff; --border: #dde3ea;
      --text: #1a2636; --text2: #4a6080; --text3: #8498ae;
      --yellow: #f5a623; --orange: #f07033; --red: #e03b3b;
      --green: #19a97b; --green2: #e6f7f2; --sidebar-w: 230px; --header-h: 60px;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: 'Sarabun', sans-serif; background: var(--bg); color: var(--text); display: flex; min-height: 100vh; }

    /* Loading overlay */
    #loading-overlay {
      position: fixed; inset: 0; background: rgba(15,43,74,.7); z-index: 9000;
      display: none; align-items: center; justify-content: center; flex-direction: column; gap: 16px;
    }
    #loading-overlay.show { display: flex; }
    .spinner {
      width: 44px; height: 44px; border: 4px solid rgba(255,255,255,.2);
      border-top-color: var(--sky); border-radius: 50%; animation: spin .7s linear infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }
    .loading-msg { color: white; font-size: 14px; font-weight: 500; }

    /* --- MANGA STYLE LOGIN --- */
    .login-wrap {
        width: 100vw; min-height: 100vh; display: flex; align-items: center; justify-content: center;
        /* พื้นหลัง Speed Lines แบบมังงะ และ Halftone Dots */
        background-color: var(--navy);
        background-image:
            radial-gradient(circle at center, transparent 30%, var(--navy) 100%),
            repeating-conic-gradient(from 0deg at 50% 50%, rgba(255,255,255,0.03) 0deg 2deg, transparent 2deg 5deg),
            radial-gradient(rgba(255,255,255,0.1) 2px, transparent 2px);
        background-size: cover, cover, 20px 20px;
        background-blend-mode: normal, normal, overlay;
    }
    .login-box {
        /* เปลี่ยนจากกระจก เป็นกล่อง Comic Panel */
        background: var(--white);
        border: 4px solid var(--text); /* เส้นขอบหนาสีดำ */
        border-radius: 8px; /* มุมโค้งเล็กน้อย */
        padding: 40px 44px; width: 400px;
        /* เงาแบบทึบ (Hard Shadow) เหมือนหมึกพิมพ์ */
        box-shadow: 12px 12px 0 var(--text), 12px 12px 0 4px var(--white) inset;
        position: relative;
    }
    /* เพิ่มเอฟเฟกต์กระดาษการ์ตูน */
    .login-box::before {
        content: ''; position: absolute; inset: 0;
        background-image: radial-gradient(rgba(0,0,0,0.05) 1px, transparent 1px);
        background-size: 10px 10px; pointer-events: none; opacity: 0.5; z-index: 0;
    }
    /* ปรับแต่ง Input ให้เข้าธีม */
    .login-box .form-input {
        border: 3px solid var(--text); border-radius: 4px;
        box-shadow: 3px 3px 0 rgba(0,0,0,0.1);
        font-weight: 600; transition: all .1s;
    }
    .login-box .form-input:focus {
        border-color: var(--blue); box-shadow: 5px 5px 0 var(--yellow);
        transform: translate(-2px, -2px);
    }
    /* ปรับแต่งปุ่มให้ดูมีพลัง */
    .login-box .btn-primary {
        border: 3px solid var(--text); border-radius: 4px;
        box-shadow: 4px 4px 0 var(--text);
        text-transform: uppercase; letter-spacing: 1px; font-weight: 800;
        transition: all .1s;
    }
    .login-box .btn-primary:hover {
        transform: translate(-2px, -2px); box-shadow: 6px 6px 0 var(--text);
        background: var(--blue);
    }
    .login-box .btn-primary:active {
        transform: translate(4px, 4px); box-shadow: 0 0 0 var(--text);
    }
    /* ปรับแต่ง Tabs */
    .login-box .login-tabs { border: 3px solid var(--text); border-radius: 6px; background: var(--text); gap: 3px; }
    .login-box .login-tab { border-radius: 2px; background: var(--white); color: var(--text); }
    .login-box .login-tab.active { background: var(--blue); color: var(--white); font-weight: 800; }
    /* --- END MANGA STYLE --- */
    .login-logo { text-align: center; margin-: 28px; }
    .login-badge { background: var(--sky); color: white; font-size: 10px; font-family: 'IBM Plex Mono', monospace; letter-spacing: 1.5px; padding: 3px 10px; border-radius: 4px; display: inline-block; margin-bottom: 8px; }
    .login-title { font-size: 20px; font-weight: 700; color: var(--text); }
    .login-sub { font-size: 12px; color: var(--text3); margin-top: 4px; }
    .login-tabs { display: flex; border: 1.5px solid var(--border); border-radius: 8px; overflow: hidden; margin-bottom: 20px; }
    .login-tab { flex: 1; padding: 9px; text-align: center; font-size: 13px; font-weight: 600; cursor: pointer; color: var(--text2); background: var(--bg); transition: all .15s; }
    .login-tab.active { background: var(--blue); color: white; }
    .login-hint { font-size: 11px; color: var(--text3); text-align: center; margin-top: 14px; }
    .reg-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .login-error { background: #fff1f1; border: 1px solid #fca5a5; color: #dc2626; font-size: 12px; padding: 8px 12px; border-radius: 8px; margin-bottom: 12px; display: none; }

    /* Forms & Buttons */
    .form-group { display: flex; flex-direction: column; gap: 6px; margin-bottom: 14px; }
    .form-label { font-size: 12px; font-weight: 600; color: var(--text2); }
    .form-input, .form-select, .form-textarea { padding: 10px 14px; border: 1.5px solid var(--border); border-radius: 8px; font-size: 13.5px; font-family: 'Sarabun', sans-serif; color: var(--text); background: var(--white); outline: none; transition: border-color .15s; width: 100%; }
    .form-input:focus, .form-select:focus, .form-textarea:focus { border-color: var(--blue); box-shadow: 0 0 0 3px rgba(29,111,164,.08); }
    .form-textarea { resize: vertical; min-height: 80px; }
    .form-select { appearance: none; background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%238498ae' d='M6 8L1 3h10z'/%3E%3C/svg%3E"); background-repeat: no-repeat; background-position: right 12px center; padding-right: 32px; }
    .btn { display: inline-flex; align-items: center; justify-content: center; gap: 6px; padding: 10px 18px; border-radius: 8px; font-size: 13px; font-weight: 600; font-family: 'Sarabun', sans-serif; cursor: pointer; border: none; transition: all .15s; }
    .btn-primary { background: var(--blue); color: white; }
    .btn-primary:hover { background: var(--navy2); }
    .btn-outline { background: transparent; border: 1.5px solid var(--border); color: var(--text2); }
    .btn-outline:hover { border-color: var(--blue); color: var(--blue); }
    .btn-ghost { background: var(--bg); color: var(--text2); }
    .btn-sm { padding: 6px 12px; font-size: 12px; }
    .btn-danger { background: var(--red); color: white; }
    .btn-success { background: var(--green); color: white; }

    /* Layout */
    .sidebar { width: var(--sidebar-w); background: var(--navy); min-height: 100vh; position: fixed; top: 0; left: 0; display: flex; flex-direction: column; z-index: 100; }
    .sidebar-logo { padding: 20px 20px 16px; border-bottom: 1px solid rgba(255,255,255,.08); }
    .logo-badge { background: var(--sky); color: white; font-size: 10px; font-family: 'IBM Plex Mono', monospace; letter-spacing: 1.5px; padding: 3px 8px; border-radius: 4px; display: inline-block; margin-bottom: 8px; }
    .logo-title { color: white; font-size: 14px; font-weight: 700; line-height: 1.3; }
    .logo-sub { color: rgba(255,255,255,.4); font-size: 11px; margin-top: 3px; font-family: 'IBM Plex Mono', monospace; }
    .sidebar-section { padding: 14px 12px 4px; }
    .sidebar-section-label { font-size: 9px; letter-spacing: 2px; text-transform: uppercase; color: rgba(255,255,255,.3); padding: 0 8px; margin-bottom: 6px; font-family: 'IBM Plex Mono', monospace; }
    .nav-item { display: flex; align-items: center; gap: 10px; padding: 9px 12px; border-radius: 8px; cursor: pointer; color: rgba(255,255,255,.55); font-size: 13px; font-weight: 500; transition: all .18s; margin-bottom: 2px; position: relative; }
    .nav-item:hover { background: rgba(255,255,255,.07); color: rgba(255,255,255,.85); }
    .nav-item.active { background: rgba(46,168,213,.2); color: var(--sky); }
    .nav-item.active::before { content: ''; position: absolute; left: 0; top: 6px; bottom: 6px; width: 3px; background: var(--sky); border-radius: 0 3px 3px 0; }
    .nav-icon { font-size: 15px; width: 20px; text-align: center; }
    .nav-badge { margin-left: auto; background: var(--red); color: white; font-size: 10px; font-family: 'IBM Plex Mono', monospace; padding: 1px 6px; border-radius: 10px; min-width: 20px; text-align: center; }
    .nav-badge.yellow { background: var(--yellow); }
    .sidebar-footer { margin-top: auto; padding: 14px; border-top: 1px solid rgba(255,255,255,.08); }
    .user-card { display: flex; align-items: center; gap: 10px; }
    .user-avatar { width: 34px; height: 34px; border-radius: 50%; background: linear-gradient(135deg, var(--sky), var(--teal)); display: flex; align-items: center; justify-content: center; color: white; font-size: 13px; font-weight: 700; flex-shrink: 0; }
    .user-name { color: rgba(255,255,255,.85); font-size: 13px; font-weight: 600; }
    .user-role { color: rgba(255,255,255,.35); font-size: 11px; }
    .logout-btn { display: flex; align-items: center; gap: 6px; color: rgba(255,255,255,.35); font-size: 12px; cursor: pointer; margin-top: 8px; padding: 4px 0; }
    .logout-btn:hover { color: var(--red); }

    .main { margin-left: var(--sidebar-w); flex: 1; display: flex; flex-direction: column; min-height: 100vh; }
    .header { background: var(--white); border-bottom: 1px solid var(--border); height: var(--header-h); display: flex; align-items: center; padding: 0 28px; gap: 16px; position: sticky; top: 0; z-index: 50; }
    .header-title { font-size: 16px; font-weight: 700; color: var(--text); flex: 1; }
    .header-breadcrumb { font-size: 12px; color: var(--text3); font-family: 'IBM Plex Mono', monospace; }
    .header-actions { display: flex; align-items: center; gap: 10px; }
    .bell-btn { width: 36px; height: 36px; border-radius: 8px; background: var(--bg); border: none; display: flex; align-items: center; justify-content: center; font-size: 17px; cursor: pointer; position: relative; }
    .bell-dot { position: absolute; top: 6px; right: 6px; width: 8px; height: 8px; background: var(--red); border-radius: 50%; border: 2px solid var(--white); }
    .content { flex: 1; padding: 28px; overflow-y: auto; }
    .page { display: none; }
    .page.active { display: block; }
    .page-title { font-size: 22px; font-weight: 700; color: var(--text); margin-bottom: 4px; }
    .page-sub { font-size: 13px; color: var(--text3); margin-bottom: 24px; }

    /* Components */
    .card { background: var(--white); border: 1px solid var(--border); border-radius: 12px; padding: 20px; }
    .card-title { font-size: 14px; font-weight: 700; color: var(--text); margin-bottom: 16px; display: flex; align-items: center; justify-content: space-between; }
    .kpi-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; margin-bottom: 24px; }
    .kpi-card { background: var(--white); border: 1px solid var(--border); border-radius: 12px; padding: 20px; position: relative; overflow: hidden; }
    .kpi-card::after { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 3px; background: var(--kpi-color, var(--blue)); }
    .kpi-label { font-size: 12px; color: var(--text3); margin-bottom: 8px; }
    .kpi-value { font-size: 32px; font-weight: 700; color: var(--kpi-color, var(--blue)); font-family: 'IBM Plex Mono', monospace; margin-bottom: 6px; line-height: 1; }
    .kpi-trend { font-size: 11px; color: var(--green); }
    .kpi-trend.down { color: var(--red); }
    .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin-bottom: 20px; }
    .status-pill { display: inline-flex; align-items: center; gap: 5px; padding: 3px 10px; border-radius: 100px; font-size: 11px; font-weight: 600; white-space: nowrap; }
    .status-active { background: #dcfce7; color: #166534; }
    .status-paid { background: var(--green2); color: #065f46; }
    .status-unpaid { background: #fff1f1; color: #7f1d1d; }
    .data-table { width: 100%; border-collapse: collapse; }
    .data-table th { text-align: left; font-size: 11px; font-weight: 600; color: var(--text3); letter-spacing: 1px; text-transform: uppercase; padding: 0 12px 10px; border-bottom: 1px solid var(--border); font-family: 'IBM Plex Mono', monospace; }
    .data-table td { padding: 11px 12px; font-size: 13px; color: var(--text); border-bottom: 1px solid var(--bg); }
    .data-table tr:last-child td { border-bottom: none; }
    .data-table tr:hover td { background: var(--bg); }
    .form-section { background: var(--white); border: 1px solid var(--border); border-radius: 12px; padding: 24px; margin-bottom: 20px; }
    .form-section-title { font-size: 14px; font-weight: 700; color: var(--text); margin-bottom: 4px; display: flex; align-items: center; gap: 8px; }
    .form-section-sub { font-size: 12px; color: var(--text3); margin-bottom: 18px; }
    .form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin-bottom: 16px; }
    .form-group.span2 { grid-column: span 2; }

    /* Payment */
    .payment-methods { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 16px; }
    .pay-method { border: 2px solid var(--border); border-radius: 10px; padding: 14px; cursor: pointer; text-align: center; transition: all .15s; }
    .pay-method:hover { border-color: var(--blue); }
    .pay-method.selected { border-color: var(--blue); background: #eff6ff; }
    .pay-method-icon { font-size: 26px; margin-bottom: 6px; }
    .pay-method-name { font-size: 13px; font-weight: 600; color: var(--text); }
    .pay-method-sub { font-size: 11px; color: var(--text3); }
    .pay-summary { background: var(--bg); border-radius: 10px; padding: 16px; margin-bottom: 16px; }
    .pay-row { display: flex; justify-content: space-between; align-items: center; font-size: 13px; padding: 4px 0; }
    .pay-row.total { font-weight: 700; font-size: 15px; border-top: 1px solid var(--border); margin-top: 8px; padding-top: 10px; }
    .pay-val { font-family: 'IBM Plex Mono', monospace; font-size: 13px; }
    .qr-box { background: var(--bg); border: 2px dashed var(--border); border-radius: 12px; padding: 24px; text-align: center; margin-bottom: 14px; }
    .qr-placeholder { width: 120px; height: 120px; background: var(--white); border: 1px solid var(--border); border-radius: 8px; margin: 0 auto 10px; display: flex; align-items: center; justify-content: center; font-size: 48px; }

    /* Timeline */
    .payment-timeline { display: flex; flex-direction: column; }
    .ptl-item { display: flex; gap: 12px; padding-bottom: 16px; position: relative; }
    .ptl-item::before { content: ''; position: absolute; left: 13px; top: 26px; bottom: 0; width: 2px; background: var(--border); }
    .ptl-item:last-child::before { display: none; }
    .ptl-dot { width: 28px; height: 28px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 11px; color: white; font-weight: 700; flex-shrink: 0; z-index: 1; }
    .ptl-dot.green { background: var(--green); }
    .ptl-dot.blue { background: var(--blue); }
    .ptl-dot.gray { background: var(--text3); }
    .ptl-dot.orange { background: var(--orange); }
    .ptl-body { flex: 1; padding-top: 4px; }
    .ptl-title { font-size: 13px; font-weight: 600; color: var(--text); }
    .ptl-meta { font-size: 11px; color: var(--text3); margin-top: 2px; font-family: 'IBM Plex Mono', monospace; }
    .ptl-note { font-size: 12px; color: var(--text2); margin-top: 5px; background: var(--bg); border-radius: 6px; padding: 7px 10px; }

    /* Member & Admin */
    .member-card { border: 1px solid var(--border); border-radius: 10px; padding: 14px; display: flex; align-items: center; gap: 12px; }
    .member-avatar { width: 42px; height: 42px; border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-size: 16px; font-weight: 700; flex-shrink: 0; }
    .member-name { font-size: 14px; font-weight: 700; color: var(--text); }
    .member-info { font-size: 11px; color: var(--text3); margin-top: 2px; }
    .tier-badge { display: inline-block; padding: 2px 8px; border-radius: 100px; font-size: 10px; font-weight: 700; }
    .tier-silver { background: #e2e8f0; color: #475569; }
    .tier-gold { background: #fef9c3; color: #713f12; }
    .tier-platinum { background: #ede9fe; color: #5b21b6; }
    .progress-bar-wrap { background: var(--bg); border-radius: 100px; height: 6px; margin-top: 4px; }
    .progress-bar { height: 6px; border-radius: 100px; background: var(--blue); }

    /* Books */
    .book-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
    .book-card { border: 1px solid var(--border); border-radius: 10px; padding: 14px; cursor: pointer; transition: all .15s; }
    .book-card:hover { border-color: var(--blue); box-shadow: 0 4px 12px rgba(0,0,0,.06); }
    .book-cover { font-size: 36px; text-align: center; padding: 10px; background: var(--bg); border-radius: 8px; margin-bottom: 10px; }
    .book-title { font-size: 13px; font-weight: 700; color: var(--text); margin-bottom: 2px; }
    .book-author { font-size: 11px; color: var(--text3); }
    .book-price { font-size: 13px; font-weight: 700; color: var(--blue); font-family: 'IBM Plex Mono', monospace; margin-top: 6px; }
    .book-available { font-size: 10px; color: var(--green); font-weight: 600; }
    .book-unavailable { font-size: 10px; color: var(--red); font-weight: 600; }

    /* Modal & Toast */
    .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,.45); z-index: 200; display: none; align-items: center; justify-content: center; }
    .modal-overlay.open { display: flex; }
    .modal { background: var(--white); border-radius: 16px; padding: 28px; width: 440px; max-width: 95vw; box-shadow: 0 24px 60px rgba(0,0,0,.2); }
    .modal-title { font-size: 17px; font-weight: 700; margin-bottom: 4px; }
    .modal-sub { font-size: 12px; color: var(--text3); margin-bottom: 20px; }
    .modal-actions { display: flex; gap: 10px; justify-content: flex-end; margin-top: 20px; }
    #toast { display: none; position: fixed; bottom: 24px; right: 24px; background: var(--navy); color: white; padding: 12px 20px; border-radius: 10px; font-size: 13px; font-family: 'Sarabun', sans-serif; z-index: 9999; box-shadow: 0 8px 24px rgba(0,0,0,.2); border-left: 4px solid var(--teal); max-width: 320px; }

    /* API Status Banner */
    #api-status-bar {
      background: #fff3cd; border-bottom: 1px solid #ffc107; padding: 8px 20px;
      font-size: 12px; color: #664d03; display: none; align-items: center; gap: 8px;
    }
    #api-status-bar.show { display: flex; }
    #api-status-bar.error { background: #fff1f1; border-color: #fca5a5; color: #7f1d1d; }
    #api-status-bar.ok { background: var(--green2); border-color: #a7f3d0; color: #065f46; }
  </style>
</head>
<body>

<!-- Loading Overlay -->
<div id="loading-overlay">
  <div class="spinner"></div>
  <div class="loading-msg" id="loading-msg">กำลังเชื่อมต่อ...</div>
</div>

<!-- API Status Banner -->
<div id="api-status-bar">
  <span id="api-status-icon">⚠️</span>
  <span id="api-status-text">กำลังตรวจสอบการเชื่อมต่อ Google Sheets...</span>
</div>

<!-- LOGIN -->
<div id="login-wrap" class="login-wrap">
  <div class="login-box">
    <div class="login-logo" style="position:relative; z-index:1;">
      <div style="width: 80px; height: 80px; background: var(--yellow); border: 4px solid var(--text); border-radius: 50%; display: flex; align-items: center; justify-content: center; margin: 0 auto 16px; box-shadow: 5px 5px 0 var(--text); transform: rotate(-3deg);">
        <div style="font-size: 42px; line-height: 1; transform: rotate(3deg); filter: drop-shadow(2px 2px 0 var(--text));">📚</div>
      </div>
      
      <div class="login-badge" style="border: 2px solid var(--text); background: var(--blue); color: white; font-weight: 800; box-shadow: 3px 3px 0 var(--text);">BOOK RENTAL</div>
      <div class="login-title" style="font-family: 'IBM Plex Mono', monospace; text-transform: uppercase; letter-spacing: -1px; font-size: 24px; text-shadow: 2px 2px 0 rgba(0,0,0,0.1);">YomiSpace</div>
      <div class="login-sub" style="font-weight: 600;">— กรุณาเข้าสู่ระบบ หรือ สมัครสมาชิก —</div>
    </div>
    <div class="login-tabs">
      <div class="login-tab active" id="tab-btn-login">เข้าสู่ระบบ</div>
      <div class="login-tab" id="tab-btn-register">สมัครสมาชิก</div>
    </div>
    <div id="login-error" class="login-error"></div>
    <div id="login-error-reg" class="login-error"></div>

    <div id="tab-login">
      <div class="form-group">
        <label class="form-label">อีเมล</label>
        <input class="form-input" id="login-email" type="email" placeholder="example@email.com" value="member@book.com">
      </div>
      <div class="form-group">
        <label class="form-label">รหัสผ่าน</label>
        <input class="form-input" id="login-pass" type="password" placeholder="••••••••" value="1234">
      </div>
      <button class="btn btn-primary" id="btn-login" style="width:100%;padding:12px">เข้าสู่ระบบ</button>
      <button class="btn btn-outline" type="button" style="width:100%; padding:12px; margin-bottom:16px;">
        <svg style="width:18px; height:18px; margin-right:6px; margin-bottom:-4px;" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"/>
          <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"/>
          <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z" fill="#FBBC05"/>
          <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z" fill="#EA4335"/>
        </svg>
        เข้าสู่ระบบด้วย Google
      </button>
      <div class="login-hint">
        💡 บัญชีทดสอบ: member@book.com / 1234 &nbsp;|&nbsp; admin@book.com / admin<br>
        <span style="color:var(--blue);margin-top:4px;display:block">🔗 เชื่อมต่อ Google Sheets จริง</span>
      </div>
    </div>

    <div id="tab-register" style="display:none">
      <div class="reg-row">
        <div class="form-group">
          <label class="form-label">ชื่อ</label>
          <input class="form-input" id="reg-fname" placeholder="ชื่อ">
        </div>
        <div class="form-group">
          <label class="form-label">นามสกุล</label>
          <input class="form-input" id="reg-lname" placeholder="นามสกุล">
        </div>
      </div>
      <div class="form-group">
        <label class="form-label">อีเมล</label>
        <input class="form-input" id="reg-email" type="text" placeholder="example@email.com">
        <span id="email-error" style="display:none;font-size:11px;color:var(--red);margin-top:2px">⚠️ อีเมลต้องมี @ และลงท้ายด้วย .com หรือ .ac.th เท่านั้น</span>
      </div>
      <div class="form-group">
        <label class="form-label">เบอร์โทรศัพท์</label>
        <input class="form-input" id="reg-phone" placeholder="0xxxxxxxxx" maxlength="10" inputmode="numeric">
        <span id="phone-error" style="display:none;font-size:11px;color:var(--red);margin-top:2px">⚠️ เบอร์โทรต้องมี 10 หลักเท่านั้น</span>
      </div>
      <div class="form-group">
        <label class="form-label">รหัสผ่าน</label>
        <input class="form-input" id="reg-pass" type="password" placeholder="อย่างน้อย 6 ตัวอักษร">
      </div>
      <button class="btn btn-primary" id="reg-submit-btn" style="width:100%;padding:12px;opacity:.45;cursor:not-allowed" disabled>สมัครสมาชิก</button>
    </div>
  </div>
</div>

<!-- APP -->
<div id="app" style="display:none;width:100%;flex-direction:column">
  <div style="display:flex;flex:1">
    <aside class="sidebar">
      <div class="sidebar-logo">
        <div class="logo-badge">BOOK RENTAL</div>
        <div class="logo-title">ระบบเช่าหนังสือออนไลน์</div>
        <div class="logo-sub" id="sidebar-role">Member Portal</div>
      </div>
      <div class="sidebar-section">
        <div class="sidebar-section-label">เมนูหลัก</div>
        <div class="nav-item active" id="nav-dashboard"><span class="nav-icon">📊</span> ภาพรวม</div>
        <div class="nav-item" id="nav-books"><span class="nav-icon">📚</span> เช่าหนังสือ</div>
        <div class="nav-item" id="nav-myrental"><span class="nav-icon">📋</span> รายการเช่าของฉัน<span class="nav-badge yellow" id="badge-rental" style="display:none">0</span></div>
      </div>
      <div class="sidebar-section">
        <div class="sidebar-section-label">การเงิน</div>
        <div class="nav-item" id="nav-payment"><span class="nav-icon">💳</span> ชำระเงิน<span class="nav-badge" id="badge-payment" style="display:none">0</span></div>
        <div class="nav-item" id="nav-payhistory"><span class="nav-icon">📅</span> ติดตามการคืน<span class="nav-badge yellow" id="badge-return" style="display:none">0</span></div>
      </div>
      <div class="sidebar-section">
        <div class="sidebar-section-label">บัญชี</div>
        <div class="nav-item" id="nav-member"><span class="nav-icon">👤</span> ข้อมูลสมาชิก</div>
        <div class="nav-item" id="nav-admin" style="display:none"><span class="nav-icon">⚙️</span> จัดการสมาชิก</div>
      </div>
      <div class="sidebar-footer">
        <div class="user-card">
          <div class="user-avatar" id="sb-avatar">ม</div>
          <div>
            <div class="user-name" id="sb-name">สมาชิก</div>
            <div class="user-role" id="sb-email">member@book.com</div>
          </div>
        </div>
        <div class="logout-btn" id="btn-logout">🚪 ออกจากระบบ</div>
      </div>
    </aside>

    <div class="main">
      <div class="header">
        <div>
          <div class="header-title" id="header-title">ภาพรวม</div>
          <div class="header-breadcrumb" id="header-bread">หน้าหลัก › ภาพรวม</div>
        </div>
        <div class="header-actions">
          <button class="bell-btn">🔔<span class="bell-dot"></span></button>
          <button class="btn btn-primary btn-sm" id="btn-header-rent">+ เช่าหนังสือ</button>
        </div>
      </div>

      <div class="content">

        <!-- DASHBOARD -->
        <div class="page active" id="page-dashboard">
          <div class="page-title">ภาพรวมบัญชีสมาชิก</div>
          <div class="page-sub" id="dash-greeting">ยินดีต้อนรับ!</div>
          <div class="kpi-row">
            <div class="kpi-card" style="--kpi-color:var(--blue)"><div class="kpi-label">หนังสือที่เช่าอยู่</div><div class="kpi-value" id="kpi-active">—</div><div class="kpi-trend">รายการที่ยังไม่คืน</div></div>
            <div class="kpi-card" style="--kpi-color:var(--red)"><div class="kpi-label">ยอดค้างชำระ</div><div class="kpi-value" id="kpi-unpaid">—</div><div class="kpi-trend down" id="kpi-unpaid-sub">รอชำระเงิน</div></div>
            <div class="kpi-card" style="--kpi-color:var(--green)"><div class="kpi-label">ชำระแล้วทั้งหมด</div><div class="kpi-value" id="kpi-paid">—</div><div class="kpi-trend" id="kpi-paid-sub">รายการ</div></div>
            <div class="kpi-card" style="--kpi-color:var(--teal)"><div class="kpi-label">คะแนนสมาชิก</div><div class="kpi-value" id="kpi-points">—</div><div class="kpi-trend" id="kpi-tier">ระดับ Silver</div></div>
          </div>
          <div class="grid-2">
            <div class="card">
              <div class="card-title">📚 รายการเช่าปัจจุบัน</div>
              <table class="data-table">
                <thead><tr><th>หนังสือ</th><th>วันคืน</th><th>สถานะ</th></tr></thead>
                <tbody id="dash-rental-tbody"><tr><td colspan="3" style="text-align:center;color:var(--text3);padding:20px">กำลังโหลด...</td></tr></tbody>
              </table>
            </div>
            <div class="card">
              <div class="card-title">💳 สถานะการชำระเงินล่าสุด</div>
              <div id="dash-pay-list" style="display:flex;flex-direction:column;gap:8px">
                <div style="text-align:center;color:var(--text3);padding:20px;font-size:13px">กำลังโหลด...</div>
              </div>
            </div>
          </div>
          <div class="card">
            <div class="card-title">🏆 ระดับสมาชิกของคุณ</div>
            <div style="display:flex;align-items:center;gap:20px;flex-wrap:wrap">
              <div style="font-size:48px" id="dash-tier-icon">🥈</div>
              <div style="flex:1">
                <div style="font-size:16px;font-weight:700" id="dash-tier-label">Silver Member <span class="tier-badge tier-silver">SILVER</span></div>
                <div style="font-size:12px;color:var(--text3);margin-top:4px" id="dash-tier-sub">คะแนน — / 500 เพื่อขึ้น Gold</div>
                <div class="progress-bar-wrap" style="margin-top:8px;height:8px"><div class="progress-bar" id="dash-tier-bar" style="width:0%;background:linear-gradient(90deg,var(--sky),var(--teal))"></div></div>
              </div>
            </div>
          </div>
        </div>

        <!-- BOOKS -->
        <div class="page" id="page-books">
          <div class="page-title">เลือกเช่าหนังสือ</div>
          <div class="page-sub">เลือกหนังสือที่ต้องการเช่า — ราคาต่อ 7 วัน</div>
          <div class="book-grid" id="book-grid">
            <div style="grid-column:span 3;text-align:center;padding:40px;color:var(--text3)">กำลังโหลดรายการหนังสือ...</div>
          </div>
        </div>

        <!-- MY RENTALS -->
        <div class="page" id="page-myrental">
          <div class="page-title">รายการเช่าของฉัน</div>
          <div class="page-sub">ประวัติการเช่าและสถานะหนังสือทั้งหมด</div>
          <div class="card">
            <table class="data-table">
              <thead><tr><th>รหัส</th><th>หนังสือ</th><th>วันเช่า</th><th>วันคืน</th><th>ค่าเช่า</th><th>สถานะชำระ</th><th>สถานะคืน</th><th></th></tr></thead>
              <tbody id="myrental-tbody"><tr><td colspan="8" style="text-align:center;color:var(--text3);padding:24px">กำลังโหลด...</td></tr></tbody>
            </table>
          </div>
        </div>

        <!-- PAYMENT -->
        <div class="page" id="page-payment">
          <div class="page-title">ชำระเงิน</div>
          <div class="page-sub">เลือกช่องทางและยืนยันการชำระเงิน</div>
          <div style="display:grid;grid-template-columns:1fr 300px;gap:18px">
            <div>
              <div class="form-section">
                <div class="form-section-title">📋 รายการที่ต้องชำระ</div>
                <div style="background:#fff1f1;border:1px solid #fca5a5;border-radius:10px;padding:14px;margin-bottom:12px">
                  <div style="display:flex;align-items:center;justify-content:space-between">
                    <div><div style="font-weight:700" id="pay-item-title">—</div><div style="font-size:12px;color:var(--text3);margin-top:3px" id="pay-item-dates">—</div></div>
                    <div style="font-family:'IBM Plex Mono';font-size:18px;font-weight:700;color:var(--red)" id="pay-item-price">—</div>
                  </div>
                </div>
                <div class="pay-summary">
                  <div class="pay-row"><span>ค่าเช่าหนังสือ</span><span class="pay-val" id="pay-summary-base">—</span></div>
                  <div class="pay-row"><span id="pay-disc-label">ส่วนลดสมาชิก Silver (5%)</span><span class="pay-val" style="color:var(--green)" id="pay-summary-disc">—</span></div>
                  <div class="pay-row"><span>VAT 7%</span><span class="pay-val" id="pay-summary-vat">—</span></div>
                  <div class="pay-row total"><span>รวมทั้งสิ้น</span><span class="pay-val" style="font-size:20px;color:var(--blue)" id="pay-summary-total">—</span></div>
                </div>
              </div>
              <div class="form-section">
                <div class="form-section-title">💳 เลือกวิธีชำระเงิน</div>
                <div class="payment-methods">
                  <div class="pay-method selected pay-method-btn" data-type="qr"><div class="pay-method-icon">📱</div><div class="pay-method-name">QR Code</div><div class="pay-method-sub">PromptPay</div></div>
                  <div class="pay-method pay-method-btn" data-type="card"><div class="pay-method-icon">💳</div><div class="pay-method-name">บัตรเครดิต/เดบิต</div><div class="pay-method-sub">Visa / Mastercard</div></div>
                  <div class="pay-method pay-method-btn" data-type="transfer"><div class="pay-method-icon">🏦</div><div class="pay-method-name">โอนเงินธนาคาร</div><div class="pay-method-sub">Internet Banking</div></div>
                </div>
                <div id="pay-panel-qr">
                  <div class="qr-box">
                    <div class="qr-placeholder">QR</div>
                    <div style="font-size:13px;font-weight:600">สแกน QR Code เพื่อชำระเงิน</div>
                    <div style="font-size:11px;color:var(--text3);margin-top:4px">พร้อมเพย์ · 0-6512-34567-8</div>
                    <div style="font-size:18px;font-weight:700;font-family:'IBM Plex Mono';color:var(--blue);margin-top:8px" id="pay-qr-amount">—</div>
                  </div>
                  <div style="text-align:center;font-size:12px;color:var(--text3);margin-bottom:14px">QR Code หมดอายุใน <strong style="color:var(--orange);font-family:'IBM Plex Mono'" id="qr-timer">15:00</strong></div>
                  <button class="btn btn-success confirm-pay-btn" style="width:100%" data-method="qr">✓ แจ้งชำระเงินแล้ว</button>
                </div>
                <div id="pay-panel-card" style="display:none">
                  <div class="form-grid">
                    <div class="form-group span2"><label class="form-label">หมายเลขบัตร</label><input class="form-input" placeholder="0000 0000 0000 0000" maxlength="19"></div>
                    <div class="form-group"><label class="form-label">วันหมดอายุ</label><input class="form-input" placeholder="MM/YY" maxlength="5"></div>
                    <div class="form-group"><label class="form-label">CVV</label><input class="form-input" placeholder="•••" maxlength="3" type="password"></div>
                    <div class="form-group span2"><label class="form-label">ชื่อบนบัตร</label><input class="form-input" placeholder="SOMCHAI JAONAATHI"></div>
                  </div>
                  <button class="btn btn-primary confirm-pay-btn" style="width:100%" id="pay-card-amount" data-method="card">💳 ชำระเงิน —</button>
                </div>
                <div id="pay-panel-transfer" style="display:none">
                  <div style="background:var(--bg);border-radius:10px;padding:16px;margin-bottom:14px">
                    <div style="font-size:13px;font-weight:700;margin-bottom:10px">ข้อมูลบัญชีธนาคาร</div>
                    <div style="display:flex;flex-direction:column;gap:6px;font-size:13px">
                      <div style="display:flex;justify-content:space-between"><span style="color:var(--text3)">ธนาคาร</span><strong>กสิกรไทย (KBANK)</strong></div>
                      <div style="display:flex;justify-content:space-between"><span style="color:var(--text3)">เลขบัญชี</span><strong style="font-family:'IBM Plex Mono'">123-4-56789-0</strong></div>
                      <div style="display:flex;justify-content:space-between"><span style="color:var(--text3)">จำนวนเงิน</span><strong style="color:var(--blue);font-family:'IBM Plex Mono'" id="pay-transfer-amount">—</strong></div>
                    </div>
                  </div>
                  <div class="form-group"><label class="form-label">แนบหลักฐานการโอน</label><input class="form-input" type="file" accept="image/*"></div>
                  <button class="btn btn-primary confirm-pay-btn" style="width:100%;margin-top:6px" data-method="transfer">📤 ส่งหลักฐานการชำระ</button>
                </div>
              </div>
            </div>
            <div>
              <div class="card" style="margin-bottom:14px">
                <div class="card-title">🔐 ความปลอดภัย</div>
                <div style="display:flex;flex-direction:column;gap:8px;font-size:12px;color:var(--text2)">
                  <div>🔒 SSL 256-bit Encryption</div>
                  <div>✅ PCI-DSS Compliant</div>
                  <div>🛡 ข้อมูลบัตรไม่ถูกจัดเก็บ</div>
                  <div>☁️ บันทึกใน Google Sheets</div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- RETURN TRACKING -->
        <div class="page" id="page-payhistory">
          <div class="page-title">ติดตามการคืนหนังสือ</div>
          <div class="page-sub">ประวัติและสถานะการยืม-คืนหนังสือทั้งหมด</div>
          <div class="kpi-row" style="grid-template-columns:repeat(3,1fr)">
            <div class="kpi-card" style="--kpi-color:var(--blue)"><div class="kpi-label">กำลังยืม</div><div class="kpi-value" id="ret-kpi-active">—</div><div class="kpi-trend">เล่ม</div></div>
            <div class="kpi-card" style="--kpi-color:var(--red)"><div class="kpi-label">ค้างชำระ</div><div class="kpi-value" id="ret-kpi-unpaid">—</div><div class="kpi-trend down">รายการ</div></div>
            <div class="kpi-card" style="--kpi-color:var(--green)"><div class="kpi-label">คืนสำเร็จแล้ว</div><div class="kpi-value" id="ret-kpi-returned">—</div><div class="kpi-trend">เล่ม</div></div>
          </div>
          <div class="card">
            <div class="card-title">📋 ประวัติการยืม-คืน</div>
            <table class="data-table">
              <thead><tr><th>รหัส</th><th>หนังสือ</th><th>วันยืม</th><th>กำหนดคืน</th><th>วันคืน</th><th>ชำระ</th><th>สถานะ</th><th></th></tr></thead>
              <tbody id="return-tbody"><tr><td colspan="8" style="text-align:center;color:var(--text3);padding:24px">กำลังโหลด...</td></tr></tbody>
            </table>
          </div>
        </div>

        <!-- MEMBER -->
        <div class="page" id="page-member">
          <div class="page-title">ข้อมูลสมาชิก</div>
          <div class="page-sub">จัดการโปรไฟล์และข้อมูลส่วนตัว</div>
          <div class="grid-2">
            <div>
              <div class="form-section" style="margin-bottom:14px">
                <div class="form-section-title">👤 ข้อมูลส่วนตัว</div>
                <div style="text-align:center;margin-bottom:20px">
                  <div style="width:72px;height:72px;border-radius:50%;background:linear-gradient(135deg,var(--sky),var(--teal));display:flex;align-items:center;justify-content:center;color:white;font-size:28px;font-weight:700;margin:0 auto 10px" id="profile-avatar-big">ม</div>
                  <div style="font-size:15px;font-weight:700" id="profile-name-display">สมาชิก</div>
                  <span class="tier-badge tier-silver" id="profile-tier-badge">SILVER MEMBER</span>
                </div>
                <div class="form-grid">
                  <div class="form-group"><label class="form-label">ชื่อ</label><input class="form-input" id="profile-fname" value=""></div>
                  <div class="form-group"><label class="form-label">นามสกุล</label><input class="form-input" id="profile-lname" value=""></div>
                  <div class="form-group span2"><label class="form-label">อีเมล</label><input class="form-input" id="profile-email" readonly style="background:var(--bg);color:var(--text3)"></div>
                  <div class="form-group span2"><label class="form-label">เบอร์โทร</label><input class="form-input" id="profile-phone" maxlength="10"></div>
                </div>
                <button class="btn btn-primary" id="save-profile-btn">💾 บันทึกข้อมูล</button>
              </div>
            </div>
            <div>
              <div class="card">
                <div class="card-title">🏆 ระดับสมาชิก</div>
                <div style="font-size:16px;font-weight:700;margin-bottom:8px" id="member-tier-label">🥈 Silver</div>
                <div style="font-size:12px;color:var(--text2)" id="member-tier-sub">เหลืออีก — คะแนน เพื่อขึ้น Gold</div>
                <div class="progress-bar-wrap" style="margin-top:8px"><div class="progress-bar" id="member-tier-bar" style="width:0%"></div></div>
                <div style="margin-top:14px;font-size:12px;color:var(--text3)">รหัสสมาชิก: <span style="font-family:'IBM Plex Mono'" id="member-id-display">—</span></div>
              </div>
            </div>
          </div>
        </div>

        <!-- ADMIN -->
        <div class="page" id="page-admin">
          <div class="page-title">จัดการสมาชิก (Admin)</div>
          <div class="page-sub">ดูและจัดการสมาชิกทั้งหมดในระบบ</div>
          <div class="kpi-row">
            <div class="kpi-card" style="--kpi-color:var(--blue)"><div class="kpi-label">สมาชิกทั้งหมด</div><div class="kpi-value" id="admin-kpi-members">—</div></div>
            <div class="kpi-card" style="--kpi-color:var(--green)"><div class="kpi-label">Active วันนี้</div><div class="kpi-value" id="admin-kpi-active">—</div></div>
            <div class="kpi-card" style="--kpi-color:var(--red)"><div class="kpi-label">ค้างชำระรวม</div><div class="kpi-value" id="admin-kpi-unpaid">—</div></div>
            <div class="kpi-card" style="--kpi-color:var(--teal)"><div class="kpi-label">รายได้เดือนนี้</div><div class="kpi-value" id="admin-kpi-income">—</div></div>
          </div>
          <div class="card">
            <div class="card-title">รายชื่อสมาชิก</div>
            <div id="admin-member-list" style="display:flex;flex-direction:column;gap:10px">
              <div style="text-align:center;padding:20px;color:var(--text3)">กำลังโหลด...</div>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</div>

<!-- RENT MODAL -->
<div class="modal-overlay" id="modal-rent">
  <div class="modal">
    <div class="modal-title" id="rent-modal-title">เช่าหนังสือ</div>
    <div class="modal-sub">เลือกจำนวนวันที่ต้องการเช่า</div>
    <div style="display:flex;gap:12px;margin-bottom:16px">
      <div style="font-size:40px" id="rent-modal-icon">📗</div>
      <div><div style="font-size:15px;font-weight:700" id="rent-modal-name">-</div><div style="font-size:12px;color:var(--text3)" id="rent-modal-author">-</div><div style="font-size:14px;font-weight:700;color:var(--blue);font-family:'IBM Plex Mono';margin-top:4px" id="rent-modal-price">฿0 / 7 วัน</div></div>
    </div>
    <div class="form-group">
      <label class="form-label">จำนวนวันที่ต้องการเช่า</label>
      <select class="form-select" id="rent-days">
        <option value="7">7 วัน</option>
        <option value="14">14 วัน (+80%)</option>
        <option value="30">30 วัน (+150%)</option>
      </select>
    </div>
    <div class="pay-summary">
      <div class="pay-row"><span>ค่าเช่า</span><span class="pay-val" id="rent-cost">฿0</span></div>
      <div class="pay-row"><span id="rent-disc-label">ส่วนลด Silver (5%)</span><span class="pay-val" style="color:var(--green)" id="rent-discount">-฿0</span></div>
      <div class="pay-row total"><span>รวม</span><span class="pay-val" id="rent-total">฿0</span></div>
    </div>
    <div class="modal-actions">
      <button class="btn btn-outline" id="btn-close-modal">ยกเลิก</button>
      <button class="btn btn-primary" id="btn-confirm-rent">✓ ยืนยันเช่า</button>
    </div>
  </div>
</div>

<!-- RETURN MODAL -->
<div class="modal-overlay" id="modal-return">
  <div class="modal" style="width:380px;text-align:center">
    <div style="font-size:48px;margin-bottom:12px">📚</div>
    <div class="modal-title" style="font-size:20px;">คืนหนังสือแล้วหรือยัง?</div>
    <div class="modal-sub" style="margin-bottom:24px;font-size:14px;">ยืนยันการคืนหนังสือ <strong id="return-modal-book-name" style="color:var(--text)">-</strong></div>
    <div class="modal-actions" style="justify-content:center;gap:12px;">
      <button class="btn btn-outline" id="btn-cancel-return">ยังไม่คืน</button>
      <button class="btn btn-primary" id="btn-confirm-return">✓ ยืนยันว่าคืนแล้ว</button>
    </div>
  </div>
</div>

<div id="toast"></div>

<script>
// ============================================================
//  ⚙️ CONFIG — อัปเดตลิงก์ API ล่าสุดแล้ว
// ============================================================
const API_URL = 'https://script.google.com/macros/s/AKfycbwcY2R2wlGfCKlEqhxdQeSdLD-ZSpoyl9fwTlYM7PyTpR8h1D1tblHJwSczCn2ZZmgCTg/exec';
// ============================================================

const EMAIL_RE = /^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.(com|ac\.th)$/;
const DISCOUNT_RATE = 0.05;
const VAT_RATE = 0.07;

let currentUser    = null;
let currentRentals = [];
let rentBook       = {};
let currentRentalForPayment = null;
let currentReturnRentalID   = null;
let qrInterval;
let toastTimer;

// ============================================================
//  API Helper
// ============================================================

async function apiCall(params, loadingMsg) {
  if (loadingMsg) showLoading(loadingMsg);
  try {
    const res = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'text/plain' },
      body: JSON.stringify(params),
      redirect: 'follow',
    });
    const data = await res.json();
    return data;
  } catch (err) {
    console.error('API Error:', err);
    return { success: false, error: err.message };
  } finally {
    if (loadingMsg) hideLoading();
  }
}

// ============================================================
//  Loading / Toast / Status
// ============================================================

function showLoading(msg) {
  document.getElementById('loading-msg').textContent = msg || 'กำลังดำเนินการ...';
  document.getElementById('loading-overlay').classList.add('show');
}
function hideLoading() {
  document.getElementById('loading-overlay').classList.remove('show');
}

function showToast(msg, type) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.style.display = 'block';
  t.style.opacity = '1';
  t.style.borderLeftColor = type === 'error' ? 'var(--red)' : 'var(--teal)';
  clearTimeout(toastTimer);
  toastTimer = setTimeout(function() {
    t.style.opacity = '0';
    setTimeout(function() { t.style.display = 'none'; }, 300);
  }, 3000);
}

function setApiStatus(status, msg) {
  const bar  = document.getElementById('api-status-bar');
  const icon = document.getElementById('api-status-icon');
  const text = document.getElementById('api-status-text');
  bar.className = 'show ' + (status === 'ok' ? 'ok' : status === 'error' ? 'error' : '');
  icon.textContent = status === 'ok' ? '✅' : status === 'error' ? '❌' : '⚠️';
  text.textContent = msg;
  if (status === 'ok') setTimeout(function() { bar.classList.remove('show'); }, 3000);
}

// ============================================================
//  Utilities
// ============================================================

function formatThaiDate(dateStr) {
  if (!dateStr) return '—';
  try {
    const d = new Date(dateStr);
    if (isNaN(d.getTime())) return dateStr;
    const dd = String(d.getDate()).padStart(2, '0');
    const mm = String(d.getMonth() + 1).padStart(2, '0');
    const yy = String(d.getFullYear() + 543).slice(-2);
    return dd + '/' + mm + '/' + yy;
  } catch(e) { return dateStr; }
}

function todayISOStr() {
  const now = new Date();
  return now.getFullYear() + '-' + String(now.getMonth()+1).padStart(2,'0') + '-' + String(now.getDate()).padStart(2,'0');
}

function addDays(dateStr, days) {
  const d = new Date(dateStr);
  d.setDate(d.getDate() + days);
  return d.toISOString().split('T')[0];
}

function tierInfo(points) {
  const p = Number(points) || 0;
  if (p >= 1000) return { tier:'Platinum', icon:'💜', next:'MAX', badge:'tier-platinum', nextPts:0, progress:100 };
  if (p >= 500)  return { tier:'Gold',     icon:'🥇', next:1000, badge:'tier-gold',     nextPts:1000-p, progress:Math.round((p-500)/5) };
  return             { tier:'Silver',   icon:'🥈', next:500,  badge:'tier-silver',   nextPts:500-p,  progress:Math.round(p/5) };
}

function calcPrice(basePrice, days) {
  const mult  = days === 7 ? 1 : days === 14 ? 1.8 : 2.5;
  const base  = Math.round(basePrice * mult);
  const disc  = Math.round(base * DISCOUNT_RATE);
  const after = base - disc;
  const vat   = parseFloat((after * VAT_RATE).toFixed(2));
  const total = parseFloat((after + vat).toFixed(2));
  return { base, disc, after, vat, total };
}

// ============================================================
//  Tabs
// ============================================================

document.getElementById('tab-btn-login').addEventListener('click', function() { switchTab('login'); });
document.getElementById('tab-btn-register').addEventListener('click', function() { switchTab('register'); });

function switchTab(t) {
  document.getElementById('tab-btn-login').classList.toggle('active', t === 'login');
  document.getElementById('tab-btn-register').classList.toggle('active', t === 'register');
  document.getElementById('tab-login').style.display    = t === 'login'    ? '' : 'none';
  document.getElementById('tab-register').style.display = t === 'register' ? '' : 'none';
  document.getElementById('login-error').style.display     = 'none';
  document.getElementById('login-error-reg').style.display = 'none';
}

// ============================================================
//  Login
// ============================================================

document.getElementById('btn-login').addEventListener('click', async function() {
  const email = document.getElementById('login-email').value.trim();
  const pass  = document.getElementById('login-pass').value;
  const errEl = document.getElementById('login-error');
  errEl.style.display = 'none';

  if (!email || !pass) {
    errEl.textContent = 'กรุณากรอกอีเมลและรหัสผ่าน';
    errEl.style.display = 'block'; return;
  }

  const res = await apiCall({ action: 'login', email, password: pass }, 'กำลังตรวจสอบบัญชี...');

  if (!res.success) {
    errEl.textContent = res.error || 'อีเมลหรือรหัสผ่านไม่ถูกต้อง';
    errEl.style.display = 'block'; return;
  }

  currentUser = res.user;
  launchApp();
});

// ============================================================
//  Register
// ============================================================

const regEmailInput = document.getElementById('reg-email');
const regPhoneInput = document.getElementById('reg-phone');
const regSubmitBtn  = document.getElementById('reg-submit-btn');

regEmailInput.addEventListener('input', validateEmail);
regPhoneInput.addEventListener('input', function() {
  this.value = this.value.replace(/\D/g, '');
  if (this.value.length > 0 && this.value[0] !== '0') this.value = '';
  validatePhone(this);
});
document.getElementById('reg-fname').addEventListener('input', function() { filterNameInput(this); });
document.getElementById('reg-lname').addEventListener('input', function() { filterNameInput(this); });

function validateEmail() {
  const val   = regEmailInput.value;
  const valid = EMAIL_RE.test(val);
  const hasAt = val.includes('@');
  const err   = document.getElementById('email-error');
  if (val.length > 0) {
    err.style.display = valid ? 'none' : 'block';
    err.textContent   = !hasAt ? '⚠️ อีเมลต้องมี @' : '⚠️ อีเมลต้องลงท้ายด้วย .com หรือ .ac.th เท่านั้น';
    regEmailInput.style.borderColor = valid ? 'var(--green)' : 'var(--red)';
  } else {
    err.style.display = 'none';
    regEmailInput.style.borderColor = '';
  }
  updateRegBtn();
}

function validatePhone(el) {
  const err = document.getElementById('phone-error');
  if (el.value.length > 0 && el.value.length < 10) {
    err.style.display = 'block'; el.style.borderColor = 'var(--red)';
  } else if (el.value.length === 10) {
    err.style.display = 'none';  el.style.borderColor = 'var(--green)';
  } else {
    err.style.display = 'none';  el.style.borderColor = '';
  }
  updateRegBtn();
}

function updateRegBtn() {
  const ok = EMAIL_RE.test(regEmailInput.value.trim());
  regSubmitBtn.disabled = !ok;
  regSubmitBtn.style.opacity = ok ? '1' : '.45';
  regSubmitBtn.style.cursor  = ok ? 'pointer' : 'not-allowed';
}

function filterNameInput(el) {
  const cleaned = el.value.replace(/[^ก-๙a-zA-Z\s]/g, '');
  if (cleaned !== el.value) el.value = cleaned;
}

regSubmitBtn.addEventListener('click', async function() {
  const fn    = document.getElementById('reg-fname').value.trim();
  const ln    = document.getElementById('reg-lname').value.trim();
  const em    = regEmailInput.value.trim();
  const ph    = regPhoneInput.value.trim();
  const ps    = document.getElementById('reg-pass').value;
  const errEl = document.getElementById('login-error-reg');

  errEl.style.display = 'none';
  if (!fn || !ln)             { errEl.style.display = 'block'; errEl.textContent = 'กรุณากรอกชื่อและนามสกุล'; return; }
  if (!EMAIL_RE.test(em))     { errEl.style.display = 'block'; errEl.textContent = 'อีเมลไม่ถูกต้อง'; return; }
  if (!ph || ph.length !== 10){ errEl.style.display = 'block'; errEl.textContent = 'เบอร์โทรต้องมี 10 หลัก'; return; }
  if (!ps || ps.length < 6)   { errEl.style.display = 'block'; errEl.textContent = 'รหัสผ่านต้องมีอย่างน้อย 6 ตัวอักษร'; return; }

  const res = await apiCall({ action:'register', firstName:fn, lastName:ln, email:em, phone:ph, password:ps }, 'กำลังสมัครสมาชิก...');

  if (!res.success) {
    errEl.style.display = 'block';
    errEl.textContent   = res.error || 'สมัครสมาชิกไม่สำเร็จ'; return;
  }

  currentUser = res.user;
  launchApp();
  showToast('สมัครสมาชิกสำเร็จ! ยินดีต้อนรับ 🎉');
});

// ============================================================
//  Launch App
// ============================================================

async function launchApp() {
  document.getElementById('login-wrap').style.display = 'none';
  document.getElementById('app').style.display = 'flex';

  const n = currentUser.firstName || currentUser.name || 'สมาชิก';
  document.getElementById('sb-name').textContent    = n;
  document.getElementById('sb-email').textContent   = currentUser.email;
  document.getElementById('sb-avatar').textContent  = n[0].toUpperCase();
  document.getElementById('sidebar-role').textContent = currentUser.role === 'admin' ? 'Admin Panel' : 'Member Portal';

  document.getElementById('profile-avatar-big').textContent    = n[0].toUpperCase();
  document.getElementById('profile-name-display').textContent  = n + ' ' + (currentUser.lastName || '');
  document.getElementById('profile-fname').value  = currentUser.firstName || n;
  document.getElementById('profile-lname').value  = currentUser.lastName  || '';
  document.getElementById('profile-email').value  = currentUser.email;
  document.getElementById('profile-phone').value  = currentUser.phone || '';
  document.getElementById('member-id-display').textContent = currentUser.memberID || '—';

  if (currentUser.role === 'admin') document.getElementById('nav-admin').style.display = 'flex';

  setApiStatus('ok', '✅ เชื่อมต่อ Google Sheets สำเร็จ!');
  startQrTimer();
  await loadBooks();
  await loadMemberData();
  navigate('dashboard');
}

// ============================================================
//  Load Books
// ============================================================

async function loadBooks() {
  const res  = await apiCall({ action: 'getBooks' }, 'กำลังโหลดหนังสือ...');
  const grid = document.getElementById('book-grid');

  if (!res.success || !res.books || res.books.length === 0) {
    grid.innerHTML = '<div style="grid-column:span 3;text-align:center;padding:40px;color:var(--text3)">ไม่พบรายการหนังสือ หรือโปรดรันคำสั่ง initSheets() ใน Google Apps Script</div>';
    return;
  }

  grid.innerHTML = '';
  res.books.forEach(function(book) {
    const avail   = Number(book.AvailableCopies) || 0;
    const unavail = avail <= 0;
    const card    = document.createElement('div');
    card.className = 'book-card book-item';
    card.style.opacity = unavail ? '.5' : '1';
    card.style.cursor  = unavail ? 'not-allowed' : 'pointer';
    card.innerHTML = `
      <div class="book-cover">${book.Icon || '📖'}</div>
      <div class="book-title">${book.Title}</div>
      <div class="book-author">${book.Author}</div>
      <div class="book-price">฿${book.PricePerWeek} / 7 วัน</div>
      <div class="${unavail ? 'book-unavailable' : 'book-available'}">
        ${unavail ? '✗ ไม่มีให้เช่าขณะนี้' : '✓ มีให้เช่า ' + avail + ' เล่ม'}
      </div>`;
    if (!unavail) card.addEventListener('click', function() { openRentModal(book); });
    grid.appendChild(card);
  });
}

// ============================================================
//  Load Member Data
// ============================================================

async function loadMemberData() {
  const res = await apiCall({ action: 'getMemberStats', memberID: currentUser.memberID }, 'กำลังโหลดข้อมูล...');
  if (!res.success) return;

  currentRentals = res.rentals || [];

  document.getElementById('kpi-active').textContent = res.activeRentals || 0;
  document.getElementById('kpi-unpaid').textContent = '฿' + (parseFloat(res.unpaidTotal) || 0);
  document.getElementById('kpi-paid').textContent   = '฿' + (parseFloat(res.paidTotal)   || 0);
  document.getElementById('kpi-points').textContent = res.points || 0;

  const ti = tierInfo(res.points);
  document.getElementById('kpi-tier').textContent      = 'ระดับ ' + ti.tier;
  document.getElementById('dash-tier-icon').textContent = ti.icon;
  document.getElementById('dash-tier-bar').style.width  = Math.min(ti.progress, 100) + '%';
  document.getElementById('dash-tier-label').innerHTML  = ti.icon + ' ' + ti.tier + ' Member <span class="tier-badge ' + ti.badge + '">' + ti.tier.toUpperCase() + '</span>';
  document.getElementById('dash-tier-sub').textContent  = ti.tier === 'Platinum' ? 'ระดับสูงสุด! 🎉' : 'คะแนน ' + (res.points || 0) + ' / ' + ti.next + ' เพื่อขึ้น ' + (ti.tier === 'Silver' ? 'Gold' : 'Platinum');

  document.getElementById('member-tier-label').innerHTML = ti.icon + ' ' + ti.tier + ' <span class="tier-badge ' + ti.badge + '">' + ti.tier.toUpperCase() + '</span>';
  document.getElementById('member-tier-sub').textContent = ti.tier === 'Platinum' ? 'ระดับสูงสุดแล้ว!' : 'เหลืออีก ' + ti.nextPts + ' คะแนน เพื่อขึ้น ' + (ti.tier === 'Silver' ? 'Gold' : 'Platinum');
  document.getElementById('member-tier-bar').style.width  = Math.min(ti.progress, 100) + '%';
  document.getElementById('profile-tier-badge').textContent = ti.tier.toUpperCase() + ' MEMBER';
  document.getElementById('profile-tier-badge').className  = 'tier-badge ' + ti.badge;

  const now = new Date();
  const thaiMonths = ['มกราคม','กุมภาพันธ์','มีนาคม','เมษายน','พฤษภาคม','มิถุนายน','กรกฎาคม','สิงหาคม','กันยายน','ตุลาคม','พฤศจิกายน','ธันวาคม'];
  document.getElementById('dash-greeting').textContent = 'ยินดีต้อนรับ, ' + (currentUser.firstName || '') + '! — ' + now.getDate() + ' ' + thaiMonths[now.getMonth()] + ' ' + (now.getFullYear() + 543);

  renderRentals();
  updateBadges();
  if (currentUser.role === 'admin') loadAdminData();
}

// ============================================================
//  Render Rentals
// ============================================================

function renderRentals() {
  const rentals = currentRentals;

  // Dashboard table
  const dashTbody = document.getElementById('dash-rental-tbody');
  const active = rentals.filter(r => r.ReturnStatus === 'reading');
  dashTbody.innerHTML = active.length === 0
    ? '<tr><td colspan="3" style="text-align:center;color:var(--text3);padding:16px">ไม่มีรายการเช่าที่ใช้งานอยู่</td></tr>'
    : active.slice(0, 5).map(function(r) {
        const paid = r.PaymentStatus === 'paid';
        return `<tr>
          <td><strong>${r.BookTitle}</strong></td>
          <td style="font-family:'IBM Plex Mono';font-size:11px">${formatThaiDate(r.DueDate)}</td>
          <td><span class="status-pill ${paid ? 'status-paid' : 'status-unpaid'}">${paid ? '✓ ชำระแล้ว' : '⚠ ค้างชำระ'}</span></td>
        </tr>`;
      }).join('');

  // Dashboard pay list
  const payList = document.getElementById('dash-pay-list');
  payList.innerHTML = rentals.length === 0
    ? '<div style="text-align:center;color:var(--text3);padding:16px;font-size:13px">ยังไม่มีรายการเช่า</div>'
    : rentals.slice(0, 4).map(function(r) {
        const paid = r.PaymentStatus === 'paid';
        return `<div style="padding:10px;background:${paid ? 'var(--green2)' : '#fff1f1'};border:1px solid ${paid ? '#a7f3d0' : '#fca5a5'};border-radius:8px;display:flex;justify-content:space-between;align-items:center">
          <div>
            <div style="font-size:13px;font-weight:600;color:${paid ? 'var(--green)' : 'var(--red)'}">${r.BookTitle}</div>
            <div style="font-size:11px;color:var(--text3)">${paid ? 'ชำระแล้ว' : 'ค้างชำระ'} · ${formatThaiDate(r.DueDate)}</div>
          </div>
          <div style="text-align:right">
            <div style="font-family:'IBM Plex Mono';font-weight:700;color:${paid ? 'var(--green)' : 'var(--red)'}">฿${r.TotalAmount}</div>
            ${!paid ? `<button class="btn btn-danger btn-sm dash-pay-btn" style="margin-top:4px" data-rentalid="${r.RentalID}">ชำระ</button>` : ''}
          </div>
        </div>`;
      }).join('');

  payList.querySelectorAll('.dash-pay-btn').forEach(function(btn) {
    btn.addEventListener('click', function() { openPaymentPage(btn.dataset.rentalid); });
  });

  // My Rentals table
  const myTbody = document.getElementById('myrental-tbody');
  myTbody.innerHTML = rentals.length === 0
    ? '<tr><td colspan="8" style="text-align:center;color:var(--text3);padding:24px">ยังไม่มีประวัติการเช่า</td></tr>'
    : rentals.map(function(r) {
        const paid     = r.PaymentStatus === 'paid';
        const returned = r.ReturnStatus  === 'returned';
        return `<tr>
          <td style="font-family:'IBM Plex Mono';font-size:11px;color:var(--blue)">${r.RentalID}</td>
          <td><strong>${r.BookTitle}</strong></td>
          <td style="font-family:'IBM Plex Mono';font-size:11px">${formatThaiDate(r.RentDate)}</td>
          <td style="font-family:'IBM Plex Mono';font-size:11px;color:${returned ? 'var(--text3)' : 'var(--orange)'}">${formatThaiDate(r.DueDate)}</td>
          <td style="font-family:'IBM Plex Mono';font-weight:700;color:${paid ? 'var(--green)' : 'var(--red)'}">฿${r.TotalAmount}</td>
          <td><span class="status-pill ${paid ? 'status-paid' : 'status-unpaid'}">${paid ? '✓ ชำระแล้ว' : '⚠ ค้างชำระ'}</span></td>
          <td><span class="status-pill ${returned ? '' : 'status-active'}" ${returned ? 'style="background:#e0f2fe;color:#0369a1"' : ''}>${returned ? '✓ คืนแล้ว' : '📖 กำลังอ่าน'}</span></td>
          <td>${!paid ? `<button class="btn btn-danger btn-sm" onclick="openPaymentPage('${r.RentalID}')">ชำระ</button>` : '—'}</td>
        </tr>`;
      }).join('');

  // Return tracking table
  const retTbody   = document.getElementById('return-tbody');
  const retActive   = rentals.filter(r => r.ReturnStatus === 'reading').length;
  const retUnpaid   = rentals.filter(r => r.PaymentStatus === 'unpaid').length;
  const retReturned = rentals.filter(r => r.ReturnStatus === 'returned').length;

  document.getElementById('ret-kpi-active').textContent   = retActive;
  document.getElementById('ret-kpi-unpaid').textContent   = retUnpaid;
  document.getElementById('ret-kpi-returned').textContent = retReturned;

  retTbody.innerHTML = rentals.length === 0
    ? '<tr><td colspan="8" style="text-align:center;color:var(--text3);padding:24px">ยังไม่มีประวัติ</td></tr>'
    : rentals.map(function(r) {
        const paid      = r.PaymentStatus === 'paid';
        const returned  = r.ReturnStatus  === 'returned';
        const canReturn = paid && !returned;
        return `<tr>
          <td style="font-family:'IBM Plex Mono';font-size:11px;color:${paid ? 'var(--blue)' : 'var(--red)'}">${r.RentalID}</td>
          <td>${r.BookTitle}</td>
          <td style="font-family:'IBM Plex Mono';font-size:11px">${formatThaiDate(r.RentDate)}</td>
          <td style="font-family:'IBM Plex Mono';font-size:11px;color:${returned ? 'var(--text3)' : 'var(--orange)'}">${formatThaiDate(r.DueDate)}</td>
          <td style="font-family:'IBM Plex Mono';font-size:11px">${r.ReturnDate ? formatThaiDate(r.ReturnDate) : '—'}</td>
          <td><span class="status-pill ${paid ? 'status-paid' : 'status-unpaid'}">${paid ? '✓ ชำระแล้ว' : '⚠ ค้าง'}</span></td>
          <td><span class="status-pill ${returned ? '' : (paid ? 'status-active' : 'status-unpaid')}" ${returned ? 'style="background:#e0f2fe;color:#0369a1"' : ''}>${returned ? '✓ คืนแล้ว' : (paid ? '📖 กำลังอ่าน' : '⏳ รอชำระ')}</span></td>
          <td>${canReturn ? `<button class="btn btn-outline btn-sm return-book-btn" data-rentalid="${r.RentalID}" data-title="${r.BookTitle}">คืนหนังสือ</button>` : '—'}</td>
        </tr>`;
      }).join('');

  retTbody.querySelectorAll('.return-book-btn').forEach(function(btn) {
    btn.addEventListener('click', function() {
      currentReturnRentalID = btn.dataset.rentalid;
      document.getElementById('return-modal-book-name').textContent = btn.dataset.title;
      document.getElementById('modal-return').classList.add('open');
    });
  });
}

function updateBadges() {
  const unpaid    = currentRentals.filter(r => r.PaymentStatus === 'unpaid').length;
  const reading   = currentRentals.filter(r => r.ReturnStatus === 'reading' && r.PaymentStatus === 'paid').length;
  const allActive = currentRentals.filter(r => r.ReturnStatus === 'reading').length;
  setBadge('badge-rental',  allActive);
  setBadge('badge-payment', unpaid);
  setBadge('badge-return',  reading);
}

function setBadge(id, count) {
  const el = document.getElementById(id);
  if (count > 0) { el.textContent = count; el.style.display = 'inline-block'; }
  else { el.style.display = 'none'; }
}

// ============================================================
//  Payment Page
// ============================================================

function openPaymentPage(rentalID) {
  const rental = currentRentals.find(r => r.RentalID === rentalID);
  if (!rental) { showToast('ไม่พบรายการเช่า', 'error'); return; }

  currentRentalForPayment = rental;

  document.getElementById('pay-item-title').textContent    = rental.BookTitle + ' — ' + rental.RentalID;
  document.getElementById('pay-item-dates').textContent    = 'เช่าวันที่ ' + formatThaiDate(rental.RentDate) + ' · ครบกำหนด ' + formatThaiDate(rental.DueDate);
  document.getElementById('pay-item-price').textContent    = '฿' + rental.TotalAmount;
  document.getElementById('pay-summary-base').textContent  = '฿' + parseFloat(rental.BaseFee).toFixed(2);
  document.getElementById('pay-summary-disc').textContent  = '-฿' + parseFloat(rental.Discount).toFixed(2);
  document.getElementById('pay-summary-vat').textContent   = '฿' + parseFloat(rental.VAT).toFixed(2);
  document.getElementById('pay-summary-total').textContent = '฿' + parseFloat(rental.TotalAmount).toFixed(2);
  document.getElementById('pay-qr-amount').textContent     = '฿' + parseFloat(rental.TotalAmount).toFixed(2);
  document.getElementById('pay-card-amount').textContent   = '💳 ชำระเงิน ฿' + parseFloat(rental.TotalAmount).toFixed(2);
  document.getElementById('pay-transfer-amount').textContent = '฿' + parseFloat(rental.TotalAmount).toFixed(2);

  navigate('payment');
}

// ============================================================
//  Confirm Payment
// ============================================================

document.querySelectorAll('.confirm-pay-btn').forEach(function(btn) {
  btn.addEventListener('click', async function() {
    if (!currentRentalForPayment) { showToast('ไม่พบรายการที่จะชำระ', 'error'); return; }

    const method = btn.dataset.method || 'qr';
    const res = await apiCall({
      action:   'confirmPayment',
      rentalID: currentRentalForPayment.RentalID,
      memberID: currentUser.memberID,
      amount:   currentRentalForPayment.TotalAmount,
      method:   method,
    }, 'กำลังยืนยันการชำระเงิน...');

    if (!res.success) { showToast('❌ ' + (res.error || 'ชำระเงินไม่สำเร็จ'), 'error'); return; }

    const idx = currentRentals.findIndex(r => r.RentalID === currentRentalForPayment.RentalID);
    if (idx >= 0) currentRentals[idx].PaymentStatus = 'paid';

    if (res.totalPoints !== undefined) {
      currentUser.points = res.totalPoints;
      currentUser.tier   = res.tier;
      document.getElementById('kpi-points').textContent = res.totalPoints;
    }

    currentRentalForPayment = null;
    renderRentals();
    updateBadges();
    showToast('ชำระเงินสำเร็จ! ได้รับ +' + (res.earnedPoints || 0) + ' คะแนน 💚');
    navigate('myrental');
  });
});

// ============================================================
//  Return Book
// ============================================================

document.getElementById('btn-cancel-return').addEventListener('click', function() {
  document.getElementById('modal-return').classList.remove('open');
  currentReturnRentalID = null;
});

document.getElementById('btn-confirm-return').addEventListener('click', async function() {
  if (!currentReturnRentalID) return;

  const res = await apiCall({
    action:   'returnBook',
    rentalID: currentReturnRentalID,
    memberID: currentUser.memberID,
  }, 'กำลังบันทึกการคืนหนังสือ...');

  document.getElementById('modal-return').classList.remove('open');

  if (!res.success) {
    showToast('❌ ' + (res.error || 'คืนหนังสือไม่สำเร็จ'), 'error');
    currentReturnRentalID = null; return;
  }

  const idx = currentRentals.findIndex(r => r.RentalID === currentReturnRentalID);
  if (idx >= 0) { currentRentals[idx].ReturnStatus = 'returned'; currentRentals[idx].ReturnDate = todayISOStr(); }

  currentReturnRentalID = null;
  renderRentals();
  updateBadges();
  showToast('คืนหนังสือเรียบร้อยแล้ว ขอบคุณที่ใช้บริการ! 📚');
});

// ============================================================
//  Book Rental Modal
// ============================================================

function openRentModal(book) {
  rentBook = book;
  document.getElementById('rent-modal-title').textContent  = 'เช่า: ' + book.Title;
  document.getElementById('rent-modal-name').textContent   = book.Title;
  document.getElementById('rent-modal-author').textContent = book.Author;
  document.getElementById('rent-modal-icon').textContent   = book.Icon || '📖';
  document.getElementById('rent-modal-price').textContent  = '฿' + book.PricePerWeek + ' / 7 วัน';
  document.getElementById('rent-days').value = '7';
  updateRentTotal();
  document.getElementById('modal-rent').classList.add('open');
}

document.getElementById('rent-days').addEventListener('change', updateRentTotal);

function updateRentTotal() {
  if (!rentBook.PricePerWeek) return;
  const days = parseInt(document.getElementById('rent-days').value);
  const p    = calcPrice(rentBook.PricePerWeek, days);
  document.getElementById('rent-cost').textContent     = '฿' + p.base;
  document.getElementById('rent-discount').textContent = '-฿' + p.disc;
  document.getElementById('rent-total').textContent    = '฿' + p.after;
}

document.getElementById('btn-close-modal').addEventListener('click', function() {
  document.getElementById('modal-rent').classList.remove('open');
});

document.getElementById('btn-confirm-rent').addEventListener('click', async function() {
  document.getElementById('modal-rent').classList.remove('open');

  const days  = parseInt(document.getElementById('rent-days').value);
  const p     = calcPrice(rentBook.PricePerWeek, days);
  const today = todayISOStr();
  const due   = addDays(today, days);

  const res = await apiCall({
    action:      'addRental',
    memberID:    currentUser.memberID,
    bookID:      rentBook.BookID,
    bookTitle:   rentBook.Title,
    bookIcon:    rentBook.Icon || '📖',
    rentDate:    today,
    dueDate:     due,
    rentalDays:  days,
    baseFee:     p.base,
    discount:    p.disc,
    vat:         p.vat,
    totalAmount: p.total,
  }, 'กำลังบันทึกรายการเช่า...');

  if (!res.success) { showToast('❌ ' + (res.error || 'เพิ่มรายการเช่าไม่สำเร็จ'), 'error'); return; }

  const newRental = {
    RentalID: res.rentalID, MemberID: currentUser.memberID,
    BookID: rentBook.BookID, BookTitle: rentBook.Title,
    RentDate: today, DueDate: due, ReturnDate: '',
    RentalDays: days, BaseFee: p.base, Discount: p.disc,
    VAT: p.vat, TotalAmount: p.total,
    PaymentStatus: 'unpaid', ReturnStatus: 'reading',
  };
  currentRentals.unshift(newRental);
  currentRentalForPayment = newRental;

  document.getElementById('pay-item-title').textContent    = rentBook.Title + ' — ' + res.rentalID;
  document.getElementById('pay-item-dates').textContent    = 'เช่าวันที่ ' + formatThaiDate(today) + ' · ครบกำหนด ' + formatThaiDate(due);
  document.getElementById('pay-item-price').textContent    = '฿' + p.after;
  document.getElementById('pay-summary-base').textContent  = '฿' + p.base.toFixed(2);
  document.getElementById('pay-summary-disc').textContent  = '-฿' + p.disc.toFixed(2);
  document.getElementById('pay-summary-vat').textContent   = '฿' + p.vat.toFixed(2);
  document.getElementById('pay-summary-total').textContent = '฿' + p.total.toFixed(2);
  document.getElementById('pay-qr-amount').textContent     = '฿' + p.total.toFixed(2);
  document.getElementById('pay-card-amount').textContent   = '💳 ชำระเงิน ฿' + p.total.toFixed(2);
  document.getElementById('pay-transfer-amount').textContent = '฿' + p.total.toFixed(2);

  renderRentals();
  updateBadges();
  await loadBooks();
  navigate('payment');
  showToast('เพิ่มรายการเช่าแล้ว! กรุณาชำระเงิน 💳');
});

// ============================================================
//  Admin
// ============================================================

async function loadAdminData() {
  const res = await apiCall({ action: 'getDashboard', requesterRole: 'admin' });
  if (!res.success) return;

  const s = res.stats || {};
  document.getElementById('admin-kpi-members').textContent = s.totalMembers  || 0;
  document.getElementById('admin-kpi-active').textContent  = s.activeRentals || 0;
  document.getElementById('admin-kpi-unpaid').textContent  = '฿' + (s.unpaidTotal   || '0');
  document.getElementById('admin-kpi-income').textContent  = '฿' + (s.monthlyIncome || '0');

  const list   = document.getElementById('admin-member-list');
  const colors = ['linear-gradient(135deg,var(--sky),var(--teal))','linear-gradient(135deg,var(--yellow),var(--orange))','linear-gradient(135deg,var(--blue),var(--navy2))'];

  list.innerHTML = res.members && res.members.length > 0
    ? res.members.map(function(m, i) {
        const ti = tierInfo(m.Points);
        return `<div class="member-card">
          <div class="member-avatar" style="background:${colors[i % colors.length]}">${(m.FirstName || '?')[0]}</div>
          <div>
            <div class="member-name">${m.FirstName} ${m.LastName} <span class="tier-badge ${ti.badge}">${ti.tier.toUpperCase()}</span></div>
            <div class="member-info">${m.Email} · ${m.Points || 0} คะแนน</div>
          </div>
          <div style="margin-left:auto"><button class="btn btn-outline btn-sm">${m.MemberID}</button></div>
        </div>`;
      }).join('')
    : '<div style="text-align:center;padding:20px;color:var(--text3)">ไม่พบข้อมูลสมาชิก</div>';
}

// ============================================================
//  Save Profile
// ============================================================

document.getElementById('save-profile-btn').addEventListener('click', async function() {
  const fn = document.getElementById('profile-fname').value.trim();
  const ln = document.getElementById('profile-lname').value.trim();
  const ph = document.getElementById('profile-phone').value.trim();

  const res = await apiCall({
    action: 'updateProfile', memberID: currentUser.memberID,
    firstName: fn, lastName: ln, phone: ph,
  }, 'กำลังบันทึก...');

  if (res.success) {
    currentUser.firstName = fn; currentUser.lastName = ln; currentUser.phone = ph;
    document.getElementById('sb-name').textContent = fn;
    document.getElementById('profile-name-display').textContent = fn + ' ' + ln;
    showToast('บันทึกข้อมูลเรียบร้อยแล้ว ✓');
  } else {
    showToast('❌ ' + (res.error || 'บันทึกไม่สำเร็จ'), 'error');
  }
});

// ============================================================
//  Navigation
// ============================================================

const pageMap = {
  dashboard:  { title:'ภาพรวม',             bread:'หน้าหลัก › ภาพรวม' },
  books:      { title:'เลือกเช่าหนังสือ',    bread:'หน้าหลัก › เช่าหนังสือ' },
  myrental:   { title:'รายการเช่าของฉัน',    bread:'หน้าหลัก › รายการเช่า' },
  payment:    { title:'ชำระเงิน',            bread:'หน้าหลัก › ชำระเงิน' },
  payhistory: { title:'ติดตามการคืนหนังสือ', bread:'หน้าหลัก › ติดตามการคืน' },
  member:     { title:'ข้อมูลสมาชิก',        bread:'หน้าหลัก › โปรไฟล์' },
  admin:      { title:'จัดการสมาชิก',        bread:'หน้าหลัก › Admin' },
};

function navigate(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById('page-' + id).classList.add('active');
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  const navEl = document.getElementById('nav-' + id);
  if (navEl) navEl.classList.add('active');
  const pg = pageMap[id];
  if (pg) {
    document.getElementById('header-title').textContent = pg.title;
    document.getElementById('header-bread').textContent = pg.bread;
  }
  document.querySelector('.content').scrollTop = 0;
  if (id === 'admin' && currentUser && currentUser.role === 'admin') loadAdminData();
}

['dashboard','books','myrental','payment','payhistory','member','admin'].forEach(function(id) {
  const el = document.getElementById('nav-' + id);
  if (el) el.addEventListener('click', function() { navigate(id); });
});

document.getElementById('btn-header-rent').addEventListener('click', function() { navigate('books'); });

// ============================================================
//  Payment Method Toggle
// ============================================================

document.querySelectorAll('.pay-method-btn').forEach(function(btn) {
  btn.addEventListener('click', function() {
    document.querySelectorAll('.pay-method-btn').forEach(b => b.classList.remove('selected'));
    btn.classList.add('selected');
    const type = btn.dataset.type;
    document.getElementById('pay-panel-qr').style.display       = type === 'qr'       ? '' : 'none';
    document.getElementById('pay-panel-card').style.display     = type === 'card'     ? '' : 'none';
    document.getElementById('pay-panel-transfer').style.display = type === 'transfer' ? '' : 'none';
  });
});

// ============================================================
//  Logout
// ============================================================

document.getElementById('btn-logout').addEventListener('click', function() {
  currentUser = null; currentRentals = []; currentRentalForPayment = null;
  clearInterval(qrInterval);
  document.getElementById('app').style.display        = 'none';
  document.getElementById('login-wrap').style.display = 'flex';
  document.getElementById('api-status-bar').classList.remove('show');
});

// ============================================================
//  QR Timer
// ============================================================

/*************  ✨ Windsurf Command ⭐  *************/
/**
 * Start a 15-minute timer for QR code payment.
 * The timer will display "mm:ss" format on the element with id "qr-timer".
 * When the timer reaches 0, it will stop and display "00:00" on the element.
 */
/*******  f24f9a2a-8d2d-46f3-9daf-40cfdfb81d03  *******/function startQrTimer() {
  let sec = 15 * 60;
  clearInterval(qrInterval);
  qrInterval = setInterval(function() {
    const el = document.getElementById('qr-timer');
    if (!el) return;
    sec--;
    if (sec <= 0) { clearInterval(qrInterval); el.textContent = '00:00'; return; }
    el.textContent = String(Math.floor(sec / 60)).padStart(2, '0') + ':' + String(sec % 60).padStart(2, '0');
  }, 1000);
}
</script>
</body>
</html>
