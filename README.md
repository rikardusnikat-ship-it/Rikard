<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Portal Ujian Pendidikan Fisika · UNM — Multi-Device</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
  <link href="https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,400;14..32,500;14..32,600;14..32,700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/@emailjs/browser@4/dist/email.min.js"></script>

  <script type="module">
    import { initializeApp, getApps } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import {
      getFirestore, collection, doc, setDoc, getDoc, getDocs,
      onSnapshot, query, where, deleteDoc, updateDoc, serverTimestamp,
      enableIndexedDbPersistence
    } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

    // ⚙️ KONFIGURASI FIREBASE — GANTI DENGAN MILIK ANDA
    const firebaseConfig = {
      apiKey: "AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      authDomain: "portal-ujian-unm.firebaseapp.com",
      projectId: "portal-ujian-unm",
      storageBucket: "portal-ujian-unm.appspot.com",
      messagingSenderId: "123456789",
      appId: "1:123456789:web:abcdef123456"
    };

    let app = null, db = null;
    let FIREBASE_ENABLED = false;

    try {
      if (!firebaseConfig.apiKey.includes('XXXX')) {
        app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];
        db = getFirestore(app);
        FIREBASE_ENABLED = true;
        console.log('%c🔥 Firebase AKTIF', 'background:#15803d;color:white;padding:8px 16px;border-radius:8px;font-weight:bold;');
        
        // Enable offline persistence
        try {
          enableIndexedDbPersistence(db).then(() => {
            console.log('%c💾 Offline persistence AKTIF', 'background:#0891b2;color:white;padding:6px 12px;border-radius:8px;');
          }).catch((err) => {
            if (err.code === 'failed-precondition') console.warn('Persistence gagal: multiple tabs');
            else if (err.code === 'unimplemented') console.warn('Browser tidak support persistence');
          });
        } catch(e) {}
      } else {
        console.warn('%c⚠️ Firebase belum dikonfigurasi — mode lokal', 'background:#fbbf24;color:#78350f;padding:8px 16px;border-radius:8px;font-weight:bold;');
      }
    } catch (e) { console.warn('Firebase init error:', e.message); }

    window.FIREBASE_DB = db;
    window.FIREBASE_ENABLED = FIREBASE_ENABLED;
    window.FIREBASE_FUNCS = { collection, doc, setDoc, getDoc, getDocs, onSnapshot, query, where, deleteDoc, updateDoc, serverTimestamp };
    
    // Generate Device ID
    let deviceId = localStorage.getItem('unm_device_id');
    if (!deviceId) {
      deviceId = 'dev_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
      localStorage.setItem('unm_device_id', deviceId);
    }
    window.DEVICE_ID = deviceId;
    window.DEVICE_INFO = {
      userAgent: navigator.userAgent,
      platform: navigator.platform,
      language: navigator.language,
      screen: `${screen.width}x${screen.height}`,
      isMobile: /Mobi|Android|iPhone|iPad/i.test(navigator.userAgent)
    };
    
    window.dispatchEvent(new Event('firebase-ready'));
  </script>

  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Inter', sans-serif;
      background: linear-gradient(145deg, #0b1a2e 0%, #1b2f47 100%);
      min-height: 100vh; display: flex; align-items: center; justify-content: center; padding: 20px;
    }
    .portal-container {
      width: 100%; max-width: 1400px;
      background: rgba(255, 255, 255, 0.06);
      backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
      border-radius: 48px; padding: 24px;
      box-shadow: 0 30px 50px rgba(0, 0, 0, 0.6), inset 0 1px 2px rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(255, 255, 255, 0.08);
    }
    .main-card { background: #ffffff; border-radius: 36px; overflow: hidden; box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4); }

    .realtime-badge {
      position: fixed; top: 20px; right: 80px;
      background: white; padding: 10px 18px;
      border-radius: 30px; font-weight: 700; font-size: 0.8rem;
      display: flex; align-items: center; gap: 8px;
      box-shadow: 0 4px 16px rgba(0,0,0,0.15);
      z-index: 10001;
    }
    .realtime-badge.online { background: #dcfce7; color: #166534; border: 2px solid #22c55e; }
    .realtime-badge.offline { background: #fef3c7; color: #92400e; border: 2px solid #fbbf24; }
    .realtime-badge .dot { width: 10px; height: 10px; border-radius: 50%; background: currentColor; animation: pulse-realtime 1.5s infinite; }
    @keyframes pulse-realtime { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }

    /* Notification Bell */
    .notif-bell {
      position: fixed; top: 20px; right: 20px;
      background: white; border: 2px solid #e2e8f0;
      width: 48px; height: 48px; border-radius: 50%;
      cursor: pointer; transition: 0.2s;
      display: flex; align-items: center; justify-content: center;
      color: #475569; font-size: 1.1rem;
      z-index: 10001;
      box-shadow: 0 4px 16px rgba(0,0,0,0.15);
    }
    .notif-bell:hover { background: #f0f6fe; border-color: #1e4b7c; color: #1e4b7c; }
    .notif-bell .notif-count {
      position: absolute; top: -4px; right: -4px;
      background: #dc2626; color: white;
      min-width: 20px; height: 20px;
      border-radius: 50%; font-size: 0.7rem;
      font-weight: 800; display: flex;
      align-items: center; justify-content: center;
      border: 2px solid white; padding: 0 4px;
    }
    .notif-bell.has-notif { animation: bellShake 0.6s; }
    @keyframes bellShake {
      0%, 100% { transform: rotate(0); }
      25% { transform: rotate(-15deg); }
      75% { transform: rotate(15deg); }
    }

    .notif-panel {
      position: fixed; top: 80px; right: 20px;
      width: 380px; max-width: calc(100vw - 40px);
      background: white; border-radius: 20px;
      box-shadow: 0 20px 50px rgba(0,0,0,0.2);
      z-index: 10002;
      transform: translateX(500px);
      transition: transform 0.3s ease-out;
      max-height: 500px; overflow-y: auto;
    }
    .notif-panel.show { transform: translateX(0); }
    .notif-panel-header {
      padding: 18px 20px; border-bottom: 2px solid #e2e8f0;
      display: flex; justify-content: space-between; align-items: center;
      position: sticky; top: 0; background: white;
      border-radius: 20px 20px 0 0;
    }
    .notif-panel-header h4 {
      font-size: 1rem; font-weight: 800; color: #0b1a2e;
      display: flex; align-items: center; gap: 8px;
    }
    .notif-panel-body { padding: 8px; }
    .notif-item {
      padding: 12px 16px; border-radius: 12px;
      display: flex; gap: 12px; align-items: flex-start;
      cursor: pointer; transition: 0.15s;
      border-left: 3px solid transparent;
    }
    .notif-item:hover { background: #f8fafc; border-left-color: #3b82f6; }
    .notif-item.unread { background: #eff6ff; border-left-color: #2563eb; }
    .notif-item .notif-icon {
      width: 36px; height: 36px; border-radius: 50%;
      display: flex; align-items: center; justify-content: center;
      color: white; font-size: 0.9rem; flex-shrink: 0;
    }
    .notif-item .notif-icon.info { background: #2563eb; }
    .notif-item .notif-icon.success { background: #16a34a; }
    .notif-item .notif-icon.warning { background: #d97706; }
    .notif-item .notif-icon.danger { background: #dc2626; }
    .notif-item .notif-content { flex: 1; min-width: 0; }
    .notif-item .notif-title { font-weight: 700; color: #0b1a2e; font-size: 0.88rem; }
    .notif-item .notif-desc { font-size: 0.78rem; color: #64748b; margin-top: 3px; line-height: 1.4; }
    .notif-item .notif-time { font-size: 0.7rem; color: #94a3b8; font-weight: 600; margin-top: 4px; }
    .notif-empty { padding: 40px 20px; text-align: center; color: #94a3b8; font-size: 0.9rem; }
    .notif-empty i { font-size: 2.5rem; margin-bottom: 10px; display: block; color: #cbd5e1; }

    /* Offline Queue Badge */
    .offline-queue-badge {
      position: fixed; bottom: 20px; left: 20px;
      background: #fef3c7; color: #92400e;
      padding: 12px 20px; border-radius: 30px;
      font-weight: 700; font-size: 0.85rem;
      display: flex; align-items: center; gap: 10px;
      box-shadow: 0 8px 20px rgba(251, 191, 36, 0.3);
      border: 2px solid #fbbf24;
      z-index: 10001;
      transform: translateY(100px);
      transition: transform 0.3s;
    }
    .offline-queue-badge.show { transform: translateY(0); }
    .offline-queue-badge .queue-count {
      background: #d97706; color: white;
      min-width: 24px; height: 24px;
      border-radius: 50%; display: flex;
      align-items: center; justify-content: center;
      font-size: 0.75rem; font-weight: 800;
    }

    /* Device Conflict Alert */
    .device-conflict-alert {
      position: fixed; top: 50%; left: 50%;
      transform: translate(-50%, -50%) scale(0.9);
      background: white; border-radius: 24px;
      padding: 32px; max-width: 480px; width: calc(100% - 40px);
      box-shadow: 0 30px 60px rgba(0,0,0,0.4);
      z-index: 10003; opacity: 0; pointer-events: none;
      transition: 0.3s; text-align: center;
    }
    .device-conflict-alert.show { opacity: 1; pointer-events: auto; transform: translate(-50%, -50%) scale(1); }
    .device-conflict-alert .alert-icon {
      width: 80px; height: 80px; margin: 0 auto 20px;
      background: linear-gradient(135deg, #fef3c7, #fde68a);
      border-radius: 50%; display: flex;
      align-items: center; justify-content: center;
      color: #d97706; font-size: 2.5rem;
    }
    .device-conflict-alert h3 { font-size: 1.3rem; font-weight: 800; color: #0b1a2e; margin-bottom: 12px; }
    .device-conflict-alert p { color: #64748b; font-size: 0.92rem; line-height: 1.6; margin-bottom: 20px; }
    .device-conflict-alert .conflict-actions { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; }

    /* Device Sync Panel */
    .device-sync-panel {
      background: linear-gradient(135deg, #f0fdf4 0%, #dcfce7 100%);
      border-radius: 24px; padding: 24px 28px;
      border: 2px solid #22c55e;
      margin-bottom: 28px;
      box-shadow: 0 8px 24px rgba(34, 197, 94, 0.15);
    }
    .device-sync-header {
      display: flex; justify-content: space-between; align-items: center;
      margin-bottom: 18px; flex-wrap: wrap; gap: 12px;
    }
    .device-sync-header h3 {
      font-size: 1.15rem; font-weight: 800; color: #065f46;
      display: flex; align-items: center; gap: 10px;
    }
    .sync-status-bar {
      display: flex; align-items: center; gap: 8px;
      padding: 8px 14px; border-radius: 30px;
      font-size: 0.78rem; font-weight: 700;
      transition: 0.3s;
    }
    .sync-status-bar.synced { background: #dcfce7; color: #166534; }
    .sync-status-bar.syncing { background: #fef3c7; color: #92400e; }
    .sync-status-bar.error { background: #fee2e2; color: #b91c1c; }
    .sync-status-bar.offline { background: #f1f5f9; color: #64748b; }
    .sync-status-bar.syncing i { animation: spin 1s linear infinite; }
    @keyframes spin { to { transform: rotate(360deg); } }

    .device-stats-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
      gap: 12px; margin-bottom: 20px;
    }
    .device-stat {
      background: white; border-radius: 16px;
      padding: 14px 18px; border: 1px solid #bbf7d0;
      display: flex; align-items: center; gap: 12px;
    }
    .device-stat .icon-box {
      width: 40px; height: 40px; border-radius: 12px;
      display: flex; align-items: center; justify-content: center;
      color: white; font-size: 1.1rem; flex-shrink: 0;
    }
    .device-stat .icon-box.green { background: #16a34a; }
    .device-stat .icon-box.blue { background: #2563eb; }
    .device-stat .icon-box.orange { background: #ea580c; }
    .device-stat .icon-box.purple { background: #7c3aed; }
    .device-stat .stat-info { flex: 1; min-width: 0; }
    .device-stat .stat-info .val { font-size: 1.4rem; font-weight: 800; color: #065f46; line-height: 1; }
    .device-stat .stat-info .lbl { font-size: 0.72rem; color: #64748b; font-weight: 600; margin-top: 3px; text-transform: uppercase; }

    .device-list {
      display: flex; flex-direction: column; gap: 10px;
      max-height: 400px; overflow-y: auto; padding-right: 4px;
    }
    .device-card {
      background: white; border-radius: 16px;
      padding: 14px 18px; border: 1px solid #bbf7d0;
      display: flex; align-items: center; gap: 14px;
      transition: 0.2s; cursor: pointer;
    }
    .device-card:hover {
      border-color: #16a34a;
      box-shadow: 0 4px 16px rgba(34, 197, 94, 0.15);
      transform: translateX(4px);
    }
    .device-card .device-icon {
      width: 44px; height: 44px; border-radius: 12px;
      display: flex; align-items: center; justify-content: center;
      color: white; font-size: 1.2rem; flex-shrink: 0;
      position: relative;
    }
    .device-card .device-icon.mobile { background: linear-gradient(135deg, #7c3aed, #6d28d9); }
    .device-card .device-icon.desktop { background: linear-gradient(135deg, #2563eb, #1d4ed8); }
    .device-card .device-icon.tablet { background: linear-gradient(135deg, #ea580c, #c2410c); }
    .device-card .device-icon .online-indicator {
      position: absolute; bottom: -2px; right: -2px;
      width: 14px; height: 14px; border-radius: 50%;
      background: #22c55e; border: 2px solid white;
      animation: devicePulse 2s infinite;
    }
    @keyframes devicePulse {
      0% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7); }
      70% { box-shadow: 0 0 0 12px rgba(34, 197, 94, 0); }
      100% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0); }
    }
    .device-card .device-info { flex: 1; min-width: 0; }
    .device-card .device-info .device-name {
      font-weight: 700; color: #065f46; font-size: 0.95rem;
      display: flex; align-items: center; gap: 8px;
    }
    .device-card .device-info .device-meta {
      font-size: 0.75rem; color: #64748b; margin-top: 3px;
      display: flex; flex-wrap: wrap; gap: 10px;
    }
    .device-card .device-info .device-meta span { display: flex; align-items: center; gap: 4px; }
    .device-card .device-badge {
      padding: 5px 12px; border-radius: 30px;
      font-size: 0.7rem; font-weight: 800;
      text-transform: uppercase; white-space: nowrap;
    }
    .device-card .device-badge.active { background: #dcfce7; color: #166534; }
    .device-card .device-badge.idle { background: #fef3c7; color: #92400e; }

    /* QR Pairing */
    .qr-container {
      display: flex; flex-direction: column; align-items: center;
      padding: 24px; background: #f8fafc;
      border-radius: 20px; margin: 16px 0;
    }
    .qr-code-box {
      background: white; padding: 16px;
      border-radius: 16px; box-shadow: 0 8px 24px rgba(0,0,0,0.1);
      margin-bottom: 16px;
    }
    .qr-code-box canvas, .qr-code-box img { display: block; width: 240px; height: 240px; }
    .qr-instruction {
      font-size: 0.85rem; color: #64748b;
      text-align: center; line-height: 1.5;
      max-width: 320px;
    }
    .pairing-code {
      font-family: 'Courier New', monospace;
      font-size: 1.6rem; font-weight: 800;
      color: #7c3aed; letter-spacing: 4px;
      background: white; padding: 12px 24px;
      border-radius: 12px; border: 2px dashed #a78bfa;
      margin: 12px 0;
    }

    .login-section { padding: 40px 40px 30px; background: #ffffff; display: block; }
    .login-header { display: flex; align-items: center; gap: 18px; margin-bottom: 28px; }
    .login-header .logo-unm { width: 72px; height: 72px; object-fit: contain; flex-shrink: 0; filter: drop-shadow(0 4px 12px rgba(30, 75, 124, 0.15)); }
    .login-header .header-text { display: flex; flex-direction: column; }
    .login-header h1 { font-size: 1.9rem; font-weight: 700; color: #0b1a2e; letter-spacing: -0.5px; }
    .login-header span { font-weight: 400; font-size: 1rem; color: #4a5f7a; display: block; margin-top: 4px; }

    .role-tabs { display: flex; gap: 12px; background: #f0f4fa; padding: 8px; border-radius: 60px; margin-bottom: 28px; width: fit-content; flex-wrap: wrap; }
    .role-btn {
      padding: 12px 28px; border: none; background: transparent;
      font-weight: 600; font-size: 0.95rem; border-radius: 40px;
      cursor: pointer; color: #3a4e66; transition: 0.2s;
      display: flex; align-items: center; gap: 8px;
    }
    .role-btn.active { background: #1e4b7c; color: white; box-shadow: 0 8px 18px rgba(30, 75, 124, 0.3); }
    .role-btn.admin.active { background: #7c3aed; box-shadow: 0 8px 18px rgba(124, 58, 237, 0.3); }

    .auth-tabs { display: flex; gap: 8px; margin-bottom: 24px; border-bottom: 2px solid #e2e8f0; }
    .auth-tab {
      padding: 10px 22px; border: none; background: transparent;
      font-weight: 600; font-size: 0.95rem; color: #64748b;
      cursor: pointer; border-bottom: 3px solid transparent;
      transition: 0.2s; margin-bottom: -2px;
    }
    .auth-tab.active { color: #1e4b7c; border-bottom-color: #1e4b7c; }

    .login-form { display: flex; flex-direction: column; gap: 20px; max-width: 540px; }
    .input-group { display: flex; flex-direction: column; gap: 6px; }
    .input-group label { font-weight: 600; font-size: 0.9rem; color: #1f3449; display: flex; align-items: center; gap: 6px; }
    .input-group input, .input-group select, .input-group textarea {
      padding: 14px 20px; border: 2px solid #dbe4ee; border-radius: 20px;
      font-size: 1rem; font-family: 'Inter', sans-serif; transition: 0.2s;
      background: #fafcff;
    }
    .input-group input:focus, .input-group select:focus, .input-group textarea:focus {
      outline: none; border-color: #1e4b7c; box-shadow: 0 0 0 4px rgba(30, 75, 124, 0.12); background: white;
    }
    .input-hint { font-size: 0.75rem; color: #64748b; margin-top: 2px; }
    .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }

    .register-role-banner {
      display: flex; align-items: center; gap: 14px;
      padding: 16px 20px; border-radius: 20px; margin-bottom: 4px;
    }
    .register-role-banner.mahasiswa { background: linear-gradient(135deg, #eef4ff 0%, #dbeafe 100%); border: 1px solid #93c5fd; }
    .register-role-banner.dosen { background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%); border: 1px solid #fbbf24; }
    .register-role-banner .icon-circle {
      width: 48px; height: 48px; border-radius: 50%;
      display: flex; align-items: center; justify-content: center;
      color: white; font-size: 1.3rem; flex-shrink: 0;
    }
    .register-role-banner.mahasiswa .icon-circle { background: #1e4b7c; }
    .register-role-banner.dosen .icon-circle { background: #d97706; }
    .register-role-banner .banner-text h4 { font-size: 1rem; font-weight: 700; margin-bottom: 2px; }
    .register-role-banner.mahasiswa .banner-text h4 { color: #1e3a8a; }
    .register-role-banner.dosen .banner-text h4 { color: #78350f; }
    .register-role-banner .banner-text p { font-size: 0.8rem; color: #475569; line-height: 1.4; }

    .btn-login {
      background: #1e4b7c; color: white; border: none;
      padding: 16px 28px; border-radius: 40px; font-weight: 700;
      font-size: 1.05rem; display: flex; align-items: center;
      justify-content: center; gap: 12px; cursor: pointer;
      transition: 0.2s; box-shadow: 0 12px 24px rgba(30, 75, 124, 0.3);
      margin-top: 4px;
    }
    .btn-login:hover { background: #0f3a60; transform: translateY(-2px); }
    .btn-login:disabled { background: #94a3b8; cursor: not-allowed; transform: none; }
    .btn-login.admin-btn { background: linear-gradient(135deg, #7c3aed, #6d28d9); }
    .btn-login.admin-btn:hover { background: linear-gradient(135deg, #6d28d9, #5b21b6); }

    .btn-register {
      background: #1e4b7c; color: white; border: none;
      padding: 16px 28px; border-radius: 40px; font-weight: 700;
      font-size: 1rem; display: flex; align-items: center;
      justify-content: center; gap: 10px; cursor: pointer;
      transition: 0.2s; box-shadow: 0 8px 20px rgba(30, 75, 124, 0.3);
      margin-top: 8px;
    }
    .btn-register:hover { background: #0f3a60; transform: translateY(-2px); }

    .forgot-password-wrapper { display: flex; justify-content: center; margin-top: -6px; margin-bottom: 4px; }
    .btn-forgot-password {
      background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
      color: #92400e; border: 2px solid #fbbf24;
      padding: 12px 24px; border-radius: 40px;
      font-weight: 700; font-size: 0.9rem;
      cursor: pointer; display: inline-flex;
      align-items: center; justify-content: center; gap: 8px;
      transition: 0.2s; box-shadow: 0 4px 12px rgba(251, 191, 36, 0.25);
      font-family: 'Inter', sans-serif;
    }
    .btn-forgot-password:hover { background: linear-gradient(135deg, #fde68a 0%, #fcd34d 100%); border-color: #d97706; transform: translateY(-2px); }

    .login-note {
      margin-top: 16px; font-size: 0.85rem; color: #64748b;
      border-top: 1px solid #e2e8f0; padding-top: 16px; line-height: 1.6;
    }
    .login-note i { color: #1e4b7c; margin-right: 6px; }

    .portal-footer {
      margin-top: 32px; padding-top: 24px; border-top: 1px solid #e2e8f0;
      display: flex; align-items: center; gap: 16px; flex-wrap: wrap;
    }
    .portal-footer .footer-logo { width: 44px; height: 44px; object-fit: contain; opacity: 0.85; }
    .portal-footer .footer-text { font-size: 0.8rem; color: #94a3b8; line-height: 1.5; }
    .portal-footer .footer-text strong { color: #1e4b7c; display: block; font-size: 0.85rem; margin-bottom: 2px; }
    .hidden { display: none !important; }

    /* Admin Dashboard */
    .admin-dashboard { display: none; padding: 32px 36px 40px; background: #ffffff; }
    .admin-header {
      display: flex; justify-content: space-between; align-items: center;
      margin-bottom: 24px; flex-wrap: wrap; gap: 16px;
      background: linear-gradient(135deg, #ede9fe 0%, #ddd6fe 100%);
      padding: 20px 24px; border-radius: 24px; border: 2px solid #a78bfa;
    }
    .admin-header .admin-title-group { display: flex; align-items: center; gap: 14px; }
    .admin-header .admin-icon {
      width: 60px; height: 60px; border-radius: 50%;
      background: linear-gradient(135deg, #7c3aed, #6d28d9);
      display: flex; align-items: center; justify-content: center;
      color: white; font-size: 1.6rem;
      box-shadow: 0 8px 20px rgba(124, 58, 237, 0.35);
    }
    .admin-header h2 { font-size: 1.6rem; font-weight: 700; color: #4c1d95; display: flex; align-items: center; gap: 10px; margin: 0; }
    .admin-header h2 small { display: block; font-size: 0.75rem; font-weight: 500; color: #6d28d9; margin-top: 2px; }
    .admin-badge {
      background: linear-gradient(135deg, #7c3aed, #6d28d9);
      color: white; padding: 6px 16px; border-radius: 30px;
      font-size: 0.75rem; font-weight: 800;
      display: inline-flex; align-items: center; gap: 6px;
      box-shadow: 0 4px 12px rgba(124, 58, 237, 0.3);
    }

    /* Dosen Dashboard */
    .dosen-dashboard { display: none; padding: 32px 36px 40px; background: #ffffff; }
    .dosen-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; flex-wrap: wrap; gap: 16px; }
    .dosen-header .dosen-title-group { display: flex; align-items: center; gap: 14px; }
    .dosen-header .dosen-logo { width: 52px; height: 52px; object-fit: contain; flex-shrink: 0; }
    .dosen-header h2 { font-size: 1.6rem; font-weight: 700; color: #0b1a2e; display: flex; align-items: center; gap: 10px; margin: 0; }
    .dosen-header h2 small { display: block; font-size: 0.75rem; font-weight: 500; color: #64748b; margin-top: 2px; }
    .badge-role-dosen { background: #fef3c7; color: #92400e; padding: 3px 12px; border-radius: 30px; font-size: 0.75rem; font-weight: 700; }

    .dashboard-tabs { display: flex; gap: 8px; margin-bottom: 24px; border-bottom: 2px solid #e2e8f0; flex-wrap: wrap; }
    .dashboard-tab {
      padding: 12px 24px; border: none; background: transparent;
      font-weight: 600; font-size: 0.95rem; color: #64748b;
      cursor: pointer; border-bottom: 3px solid transparent;
      transition: 0.2s; margin-bottom: -2px;
      display: flex; align-items: center; gap: 8px;
    }
    .dashboard-tab.active { color: #1e4b7c; border-bottom-color: #1e4b7c; }
    .dashboard-tab-content { display: none; }
    .dashboard-tab-content.active { display: block; }

    .admin-stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 16px; margin-bottom: 28px; }
    .admin-stat-card {
      background: white; border-radius: 20px;
      padding: 20px 24px; border: 2px solid #e2e8f0;
      transition: 0.2s; display: flex; flex-direction: column; gap: 8px;
    }
    .admin-stat-card.purple { border-color: #c4b5fd; background: linear-gradient(135deg, #f5f3ff 0%, #ede9fe 100%); }
    .admin-stat-card.green { border-color: #bbf7d0; background: linear-gradient(135deg, #f0fdf4 0%, #dcfce7 100%); }
    .admin-stat-card.blue { border-color: #bfdbfe; background: linear-gradient(135deg, #eff6ff 0%, #dbeafe 100%); }
    .admin-stat-card.orange { border-color: #fed7aa; background: linear-gradient(135deg, #fff7ed 0%, #ffedd5 100%); }
    .admin-stat-card .stat-icon-lg {
      width: 48px; height: 48px; border-radius: 14px;
      background: #7c3aed; color: white;
      display: flex; align-items: center; justify-content: center;
      font-size: 1.4rem;
    }
    .admin-stat-card .stat-icon-lg.green { background: #16a34a; }
    .admin-stat-card .stat-icon-lg.blue { background: #2563eb; }
    .admin-stat-card .stat-icon-lg.orange { background: #ea580c; }
    .admin-stat-card .stat-number { font-size: 2rem; font-weight: 800; color: #0b1a2e; line-height: 1; }
    .admin-stat-card .stat-label { font-size: 0.85rem; color: #64748b; font-weight: 600; }

    .user-mgmt-panel {
      background: linear-gradient(135deg, #f0f6fe 0%, #e6f0fa 100%);
      border-radius: 24px; padding: 24px 28px; margin-bottom: 28px;
      border: 1px solid #b8d7f0;
    }
    .user-mgmt-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 18px; flex-wrap: wrap; gap: 12px; }
    .user-mgmt-header h3 { font-size: 1.2rem; font-weight: 700; color: #103456; display: flex; align-items: center; gap: 8px; }
    .user-filter-row { display: flex; gap: 12px; flex-wrap: wrap; margin-bottom: 16px; align-items: center; }
    .user-filter-row select, .user-filter-row input {
      padding: 10px 16px; border: 2px solid #dbe4ee;
      border-radius: 30px; font-size: 0.9rem;
      font-family: 'Inter', sans-serif; background: white;
    }
    .user-list { display: flex; flex-direction: column; gap: 10px; max-height: 600px; overflow-y: auto; padding-right: 4px; }
    .user-row {
      display: flex; align-items: center; gap: 14px;
      background: white; padding: 14px 18px; border-radius: 18px;
      border: 1px solid #e2e8f0; transition: 0.2s;
    }
    .user-row:hover { border-color: #1e4b7c; box-shadow: 0 4px 12px rgba(30, 75, 124, 0.1); }
    .user-row .avatar {
      width: 44px; height: 44px; border-radius: 50%;
      display: flex; align-items: center; justify-content: center;
      font-weight: 700; color: white; font-size: 0.95rem; flex-shrink: 0;
    }
    .user-row .avatar.admin { background: linear-gradient(135deg, #7c3aed, #6d28d9); }
    .user-row .avatar.dosen { background: linear-gradient(135deg, #d97706, #f59e0b); }
    .user-row .avatar.mahasiswa { background: linear-gradient(135deg, #1e4b7c, #3b82f6); }
    .user-row .user-info { flex: 1; min-width: 0; }
    .user-row .user-info .name { font-weight: 700; color: #0b2a44; font-size: 0.95rem; display: flex; align-items: center; gap: 8px; }
    .user-row .user-info .nim { font-size: 0.8rem; color: #64748b; margin-top: 2px; }
    .user-row .user-info .email { font-size: 0.72rem; color: #94a3b8; margin-top: 2px; }
    .user-role-badge { padding: 5px 14px; border-radius: 30px; font-size: 0.72rem; font-weight: 800; text-transform: uppercase; white-space: nowrap; }
    .user-role-badge.admin { background: #ede9fe; color: #6d28d9; }
    .user-role-badge.dosen { background: #fef3c7; color: #92400e; }
    .user-role-badge.mahasiswa { background: #dbeafe; color: #1e40af; }
    .user-actions { display: flex; gap: 6px; }
    .btn-user-action {
      background: #f1f5f9; border: none;
      width: 36px; height: 36px; border-radius: 10px;
      cursor: pointer; transition: 0.2s;
      display: flex; align-items: center; justify-content: center;
      color: #475569; font-size: 0.85rem;
    }
    .btn-user-action:hover { background: #e2e8f0; }
    .btn-user-action.danger { color: #b91c1c; }
    .btn-user-action.danger:hover { background: #fee2e2; }
    .btn-user-action.primary { color: #1e4b7c; }
    .btn-user-action.primary:hover { background: #dbeafe; }

    /* Live Monitor */
    .live-monitor-panel {
      background: linear-gradient(135deg, #f0f9ff 0%, #dbeafe 100%);
      border-radius: 24px; padding: 24px 28px;
      border: 2px solid #60a5fa;
      margin-bottom: 28px;
      box-shadow: 0 8px 24px rgba(59, 130, 246, 0.15);
    }
    .live-monitor-header {
      display: flex; justify-content: space-between; align-items: center;
      margin-bottom: 20px; flex-wrap: wrap; gap: 12px;
    }
    .live-monitor-header h3 {
      font-size: 1.15rem; font-weight: 800; color: #1e40af;
      display: flex; align-items: center; gap: 10px;
    }
    .live-pulse {
      width: 12px; height: 12px; border-radius: 50%;
      background: #dc2626;
      box-shadow: 0 0 0 0 rgba(220, 38, 38, 0.7);
      animation: livePulse 1.8s infinite;
    }
    @keyframes livePulse {
      0% { box-shadow: 0 0 0 0 rgba(220, 38, 38, 0.7); }
      70% { box-shadow: 0 0 0 12px rgba(220, 38, 38, 0); }
      100% { box-shadow: 0 0 0 0 rgba(220, 38, 38, 0); }
    }
    .live-monitor-stats {
      display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
      gap: 12px; margin-bottom: 20px;
    }
    .live-stat-mini {
      background: white; border-radius: 14px;
      padding: 12px 14px; border: 1px solid #bfdbfe;
      text-align: center;
    }
    .live-stat-mini .val { font-size: 1.4rem; font-weight: 800; color: #1e40af; line-height: 1; }
    .live-stat-mini .lbl { font-size: 0.72rem; color: #64748b; font-weight: 600; margin-top: 4px; text-transform: uppercase; }

    .live-activity-list { display: flex; flex-direction: column; gap: 8px; max-height: 340px; overflow-y: auto; }
    .live-activity-item {
      display: flex; align-items: center; gap: 12px;
      background: white; padding: 12px 16px;
      border-radius: 14px; border-left: 4px solid #3b82f6;
      transition: 0.2s; animation: slideInActivity 0.4s ease-out;
      cursor: pointer;
    }
    @keyframes slideInActivity {
      from { opacity: 0; transform: translateX(-20px); }
      to { opacity: 1; transform: translateX(0); }
    }
    .live-activity-item.writing { border-left-color: #d97706; background: #fffbeb; }
    .live-activity-item.submitted { border-left-color: #16a34a; background: #f0fdf4; }
    .live-activity-item .act-icon {
      width: 36px; height: 36px; border-radius: 50%;
      display: flex; align-items: center; justify-content: center;
      color: white; font-size: 0.9rem; flex-shrink: 0;
    }
    .live-activity-item.writing .act-icon { background: #d97706; }
    .live-activity-item.submitted .act-icon { background: #16a34a; }
    .live-activity-item .act-info { flex: 1; min-width: 0; }
    .live-activity-item .act-name { font-weight: 700; color: #0b2a44; font-size: 0.88rem; }
    .live-activity-item .act-detail { font-size: 0.75rem; color: #64748b; margin-top: 2px; }
    .live-activity-item .act-time { font-size: 0.72rem; color: #94a3b8; font-weight: 600; white-space: nowrap; }
    .live-empty { padding: 32px; text-align: center; color: #94a3b8; font-size: 0.9rem; }
    .live-empty i { font-size: 2.5rem; margin-bottom: 10px; display: block; color: #cbd5e1; }

    .writing-indicator {
      display: inline-flex; align-items: center; gap: 6px;
      background: #fef3c7; color: #92400e;
      padding: 3px 10px; border-radius: 30px;
      font-size: 0.7rem; font-weight: 700; margin-left: 8px;
    }
    .writing-indicator .pulse-dot {
      width: 6px; height: 6px; background: #d97706;
      border-radius: 50%; animation: pulse 1.2s infinite;
    }
    @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.4} }

    /* Modal */
    .modal-overlay {
      position: fixed; inset: 0; background: rgba(11, 26, 46, 0.85);
      backdrop-filter: blur(8px); display: none;
      align-items: center; justify-content: center; z-index: 9999;
      padding: 20px; overflow-y: auto;
    }
    .modal-overlay.show { display: flex; }
    .modal-box {
      background: white; border-radius: 32px; padding: 36px;
      max-width: 620px; width: 100%;
      box-shadow: 0 30px 60px rgba(0, 0, 0, 0.5);
      animation: slideUp 0.3s ease-out; margin: auto;
    }
    @keyframes slideUp { from { transform: translateY(30px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    .modal-header {
      display: flex; justify-content: space-between; align-items: center;
      margin-bottom: 24px; padding-bottom: 16px;
      border-bottom: 2px solid #e2e8f0;
    }
    .modal-header h3 { font-size: 1.4rem; font-weight: 700; color: #0b1a2e; display: flex; align-items: center; gap: 10px; }
    .modal-close {
      background: #f1f5f9; border: none; width: 40px; height: 40px;
      border-radius: 50%; cursor: pointer; font-size: 1.1rem;
      color: #64748b; transition: 0.2s;
      display: flex; align-items: center; justify-content: center;
    }
    .modal-close:hover { background: #fee2e2; color: #dc2626; }
    .modal-actions {
      display: flex; gap: 12px; justify-content: flex-end;
      margin-top: 24px; padding-top: 20px;
      border-top: 2px solid #e2e8f0;
    }
    .btn-modal {
      padding: 12px 28px; border-radius: 40px; font-weight: 700;
      font-size: 0.95rem; cursor: pointer; transition: 0.2s;
      border: none; display: flex; align-items: center; gap: 8px;
      font-family: 'Inter', sans-serif;
    }
    .btn-modal-primary { background: #1e4b7c; color: white; box-shadow: 0 8px 20px rgba(30, 75, 124, 0.3); }
    .btn-modal-primary:hover { background: #0f3a60; transform: translateY(-2px); }
    .btn-modal-purple { background: linear-gradient(135deg, #7c3aed, #6d28d9); color: white; box-shadow: 0 8px 20px rgba(124, 58, 237, 0.3); }
    .btn-modal-purple:hover { transform: translateY(-2px); }
    .btn-modal-secondary { background: #f1f5f9; color: #475569; }
    .btn-modal-secondary:hover { background: #e2e8f0; }
    .btn-modal-danger { background: #dc2626; color: white; box-shadow: 0 8px 20px rgba(220, 38, 38, 0.3); }
    .btn-modal-danger:hover { background: #b91c1c; transform: translateY(-2px); }

    .sesi-portal { display: none; padding: 40px 36px 40px; background: #ffffff; }
    .sesi-portal-header { display: flex; align-items: center; gap: 18px; margin-bottom: 32px; flex-wrap: wrap; justify-content: space-between; }
    .sesi-portal-header .title-group { display: flex; align-items: center; gap: 16px; }
    .sesi-portal-header .sesi-logo { width: 60px; height: 60px; object-fit: contain; flex-shrink: 0; }
    .sesi-portal-header h2 { font-size: 1.8rem; font-weight: 700; color: #0b1a2e; display: flex; align-items: center; gap: 10px; margin: 0; }
    .sesi-portal-header h2 small { display: block; font-size: 0.8rem; font-weight: 500; color: #64748b; margin-top: 4px; }
    .user-badge-sesi { display: flex; align-items: center; gap: 12px; background: #f0f6fe; padding: 8px 18px 8px 14px; border-radius: 60px; font-weight: 500; color: #1e4b7c; }
    .verified-tag { background: #dcfce7; color: #166534; padding: 3px 10px; border-radius: 30px; font-size: 0.7rem; font-weight: 700; display: inline-flex; align-items: center; gap: 4px; }
    .logout-btn-sesi { background: none; border: 1.5px solid #cbd5e1; padding: 8px 18px; border-radius: 30px; font-weight: 500; color: #475569; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 6px; }
    .logout-btn-sesi:hover { background: #fee2e2; border-color: #f87171; color: #b91c1c; }
    .sesi-intro { background: linear-gradient(135deg, #f0f6fe 0%, #e6f0fa 100%); border-radius: 24px; padding: 24px 28px; border: 1px solid #b8d7f0; margin-bottom: 28px; }
    .sesi-intro h3 { font-size: 1.15rem; font-weight: 700; color: #103456; margin-bottom: 8px; display: flex; align-items: center; gap: 8px; }
    .sesi-intro p { color: #475569; font-size: 0.92rem; line-height: 1.6; }
    .sesi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 24px; margin-bottom: 32px; }
    .sesi-card { background: white; border-radius: 24px; border: 2px solid #dbe4ee; overflow: hidden; transition: 0.3s; cursor: pointer; display: flex; flex-direction: column; position: relative; }
    .sesi-card:hover { transform: translateY(-6px); box-shadow: 0 16px 40px rgba(30, 75, 124, 0.18); border-color: #1e4b7c; }
    .sesi-card.pretest .sesi-card-header { background: linear-gradient(135deg, #d97706 0%, #f59e0b 100%); }
    .sesi-card.posttest .sesi-card-header { background: linear-gradient(135deg, #15803d 0%, #22c55e 100%); }
    .sesi-card-header { padding: 28px 24px; color: white; text-align: center; }
    .sesi-card-header .sesi-icon { width: 72px; height: 72px; background: rgba(255,255,255,0.22); border-radius: 50%; display: flex; align-items: center; justify-content: center; margin: 0 auto 16px; font-size: 2rem; border: 3px solid rgba(255,255,255,0.4); }
    .sesi-card-header h4 { font-size: 1.5rem; font-weight: 800; letter-spacing: 0.5px; margin-bottom: 6px; }
    .sesi-card-header p { font-size: 0.85rem; opacity: 0.92; font-weight: 500; }
    .sesi-card-body { padding: 22px 24px; flex: 1; display: flex; flex-direction: column; gap: 14px; }
    .sesi-card-body .info-line { display: flex; align-items: flex-start; gap: 10px; font-size: 0.88rem; color: #1f3a57; line-height: 1.5; }
    .sesi-card-body .info-line i { color: #1e4b7c; margin-top: 3px; flex-shrink: 0; font-size: 0.95rem; }
    .sesi-card-footer { padding: 18px 24px 24px; border-top: 1px dashed #e2e8f0; }
    .btn-pilih-sesi { width: 100%; padding: 14px 20px; border-radius: 40px; font-weight: 700; font-size: 0.98rem; cursor: pointer; border: none; display: flex; align-items: center; justify-content: center; gap: 10px; transition: 0.2s; font-family: 'Inter', sans-serif; }
    .btn-pilih-sesi.pretest { background: linear-gradient(135deg, #d97706 0%, #f59e0b 100%); color: white; box-shadow: 0 8px 20px rgba(217, 119, 6, 0.3); }
    .btn-pilih-sesi.pretest:hover { transform: translateY(-2px); }
    .btn-pilih-sesi.posttest { background: linear-gradient(135deg, #15803d 0%, #22c55e 100%); color: white; box-shadow: 0 8px 20px rgba(21, 128, 61, 0.3); }
    .btn-pilih-sesi.posttest:hover { transform: translateY(-2px); }
    .sesi-info-panel { background: #fffbeb; border: 1px solid #fde68a; border-radius: 20px; padding: 20px 24px; color: #78350f; font-size: 0.88rem; line-height: 1.65; display: flex; align-items: flex-start; gap: 12px; }
    .sesi-info-panel i { font-size: 1.3rem; color: #d97706; margin-top: 2px; flex-shrink: 0; }

    .exam-section { display: none; padding: 32px 36px 40px; background: #ffffff; }
    .exam-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; flex-wrap: wrap; gap: 16px; }
    .exam-header .exam-title-group { display: flex; align-items: center; gap: 14px; }
    .exam-header .exam-logo { width: 52px; height: 52px; object-fit: contain; flex-shrink: 0; }
    .exam-header h2 { font-size: 1.6rem; font-weight: 700; color: #0b1a2e; display: flex; align-items: center; gap: 10px; margin: 0; }
    .exam-header h2 small { display: block; font-size: 0.75rem; font-weight: 500; color: #64748b; margin-top: 2px; }
    .user-badge { display: flex; align-items: center; gap: 12px; background: #f0f6fe; padding: 8px 18px 8px 14px; border-radius: 60px; font-weight: 500; color: #1e4b7c; }
    .logout-btn { background: none; border: 1.5px solid #cbd5e1; padding: 8px 18px; border-radius: 30px; font-weight: 500; color: #475569; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 6px; }
    .logout-btn:hover { background: #fee2e2; border-color: #f87171; color: #b91c1c; }
    .soal-nav { display: flex; gap: 10px; margin-bottom: 24px; background: #f0f4fa; padding: 8px; border-radius: 60px; width: fit-content; flex-wrap: wrap; }
    .soal-nav-btn { padding: 10px 28px; border: none; background: transparent; font-weight: 600; font-size: 0.95rem; border-radius: 40px; cursor: pointer; color: #3a4e66; transition: 0.2s; }
    .soal-nav-btn.active { background: #1e4b7c; color: white; }
    .soal-content { display: none; }
    .soal-content.active { display: block; }
    .case-box { background: #f8fbff; border-left: 6px solid #1e4b7c; padding: 22px 26px; border-radius: 24px; margin-bottom: 28px; border: 1px solid #dbe7f5; }
    .case-box h3 { font-size: 1.2rem; font-weight: 700; color: #103456; margin-bottom: 12px; display: flex; align-items: center; gap: 8px; }
    .case-box p { color: #1f3a57; line-height: 1.6; margin-bottom: 10px; font-size: 0.98rem; white-space: pre-wrap; }
    .pertanyaan-label { font-weight: 700; color: #0b2a44; margin-top: 18px; margin-bottom: 6px; font-size: 1rem; }
    .soal-item { margin-top: 10px; padding-left: 8px; }
    .soal-item p { margin-bottom: 6px; font-weight: 500; }
    .sub-soal-group { background: #fafdff; border-radius: 20px; padding: 20px 24px; margin-bottom: 20px; border: 2px solid #dbe4ee; transition: 0.2s; }
    .sub-soal-group:focus-within { border-color: #1e4b7c; box-shadow: 0 0 0 4px rgba(30, 75, 124, 0.08); background: white; }
    .sub-soal-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 12px; flex-wrap: wrap; gap: 10px; }
    .sub-soal-label { display: flex; align-items: center; gap: 10px; font-weight: 700; color: #0b1a2e; font-size: 1rem; }
    .sub-soal-label .badge-abcd { background: #1e4b7c; color: white; width: 30px; height: 30px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 800; font-size: 0.9rem; flex-shrink: 0; }
    .lock-badge { display: flex; align-items: center; gap: 6px; background: #fef3c7; color: #92400e; padding: 5px 12px; border-radius: 30px; font-size: 0.72rem; font-weight: 700; }
    .sub-soal-textarea { width: 100%; padding: 16px 20px; border: 2px solid #dbe4ee; border-radius: 16px; font-family: 'Inter', sans-serif; font-size: 0.95rem; line-height: 1.6; resize: vertical; min-height: 130px; background: white; transition: 0.2s; }
    .sub-soal-textarea:focus { outline: none; border-color: #1e4b7c; box-shadow: 0 0 0 4px rgba(30, 75, 124, 0.1); }
    .sub-soal-textarea.paste-blocked { animation: shake 0.4s; border-color: #dc2626; background: #fef2f2; }
    @keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-6px); } 75% { transform: translateX(6px); } }
    .char-counter { text-align: right; font-size: 0.75rem; color: #94a3b8; margin-top: 6px; font-weight: 500; }
    .char-counter.warn { color: #d97706; }
    .char-counter.ok { color: #15803d; }
    .paste-warning { background: #fef2f2; border-left: 4px solid #dc2626; padding: 10px 16px; border-radius: 12px; margin-top: 10px; color: #991b1b; font-size: 0.82rem; font-weight: 600; display: none; align-items: center; gap: 8px; }
    .paste-warning.show { display: flex; }
    .ai-detection-panel { display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 16px; margin: 16px 0 10px; background: #f1f5f9; padding: 14px 22px; border-radius: 20px; }
    .ai-status { display: flex; align-items: center; gap: 12px; font-weight: 500; color: #1e293b; }
    .ai-status .badge { background: #e2e8f0; padding: 6px 14px; border-radius: 40px; font-size: 0.85rem; font-weight: 600; color: #334155; }
    .btn-check-ai { background: white; border: 1.5px solid #cbd5e1; padding: 8px 20px; border-radius: 40px; font-weight: 600; color: #1e4b7c; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 8px; font-size: 0.9rem; }
    .btn-check-ai:hover { background: #eef4ff; border-color: #1e4b7c; }
    .warning-message { background: #fef2f2; border-left: 6px solid #dc2626; padding: 14px 20px; border-radius: 16px; margin: 12px 0; color: #991b1b; font-weight: 500; display: none; align-items: center; gap: 12px; font-size: 0.9rem; }
    .btn-submit { background: #1e4b7c; color: white; border: none; padding: 16px 32px; border-radius: 60px; font-weight: 700; font-size: 1rem; display: flex; align-items: center; justify-content: center; gap: 10px; cursor: pointer; transition: 0.2s; box-shadow: 0 10px 20px rgba(30, 75, 124, 0.25); width: 100%; margin-top: 16px; }
    .btn-submit:hover { background: #0f3a60; transform: translateY(-2px); }
    .btn-submit:disabled { background: #94a3b8; box-shadow: none; cursor: not-allowed; }
    .result-panel { margin-top: 24px; background: #f0f9ff; border-radius: 28px; padding: 24px; border: 1px solid #b8d7f0; display: none; }
    .result-panel h3 { font-size: 1.3rem; color: #0b2a44; margin-bottom: 16px; display: flex; align-items: center; gap: 10px; }
    .score-display { font-size: 2.6rem; font-weight: 800; color: #1e4b7c; line-height: 1; margin-bottom: 6px; }
    .feedback-list { margin-top: 20px; list-style: none; }
    .feedback-list li { display: flex; align-items: flex-start; gap: 12px; padding: 10px 0; border-bottom: 1px solid #d9e6f2; color: #1e3a5f; font-weight: 500; font-size: 0.9rem; }
    .feedback-dosen-panel { margin-top: 24px; background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%); border-radius: 24px; padding: 24px 28px; border: 2px solid #fbbf24; display: none; }
    .feedback-dosen-panel.show { display: block; }
    .feedback-dosen-panel h3 { font-size: 1.2rem; font-weight: 700; color: #78350f; margin-bottom: 14px; display: flex; align-items: center; gap: 10px; }
    .feedback-dosen-panel .fb-dosen-meta { display: flex; gap: 20px; flex-wrap: wrap; margin-bottom: 14px; font-size: 0.85rem; color: #92400e; }
    .feedback-dosen-panel .fb-dosen-text { background: white; padding: 18px 22px; border-radius: 16px; border: 1px solid #fcd34d; line-height: 1.7; color: #1e293b; font-size: 0.95rem; white-space: pre-wrap; }
    .fb-nilai-badge { display: inline-flex; align-items: center; gap: 6px; background: #1e4b7c; color: white; padding: 6px 16px; border-radius: 30px; font-weight: 800; font-size: 0.95rem; }
    .email-notif-panel { margin-top: 24px; background: linear-gradient(135deg, #ecfdf5 0%, #d1fae5 100%); border-radius: 24px; padding: 24px 28px; border: 2px solid #10b981; display: none; }
    .email-notif-panel.show { display: block; }
    .email-notif-panel h3 { font-size: 1.2rem; font-weight: 700; color: #065f46; margin-bottom: 12px; display: flex; align-items: center; gap: 10px; }
    .email-notif-panel p { color: #047857; font-size: 0.92rem; margin-bottom: 16px; line-height: 1.6; }
    .email-info-box { background: white; padding: 16px 20px; border-radius: 16px; border: 1px solid #6ee7b7; margin-bottom: 16px; font-size: 0.9rem; color: #065f46; }
    .email-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px dashed #d1fae5; }
    .email-row:last-child { border-bottom: none; }
    .email-label { color: #047857; font-weight: 500; }
    .email-value { color: #065f46; font-weight: 700; }
    .btn-send-email { background: linear-gradient(135deg, #10b981, #059669); color: white; border: none; padding: 14px 32px; border-radius: 40px; font-weight: 700; font-size: 0.95rem; display: flex; align-items: center; justify-content: center; gap: 10px; cursor: pointer; transition: 0.2s; box-shadow: 0 8px 20px rgba(16, 185, 129, 0.3); width: 100%; }
    .btn-send-email:hover { transform: translateY(-2px); }
    .btn-send-email:disabled { background: #94a3b8; cursor: not-allowed; transform: none; }

    .monitoring-panel { background: linear-gradient(135deg, #f0f6fe 0%, #e6f0fa 100%); border-radius: 24px; padding: 24px 28px; margin-bottom: 28px; border: 1px solid #b8d7f0; }
    .monitoring-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 18px; flex-wrap: wrap; gap: 12px; }
    .monitoring-header h3 { font-size: 1.2rem; font-weight: 700; color: #103456; display: flex; align-items: center; gap: 8px; }
    .live-badge { display: flex; align-items: center; gap: 6px; background: #dcfce7; color: #166534; padding: 6px 14px; border-radius: 30px; font-size: 0.8rem; font-weight: 700; }
    .live-dot { width: 8px; height: 8px; background: #16a34a; border-radius: 50%; animation: pulse 1.5s infinite; }
    .monitoring-stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 12px; margin-bottom: 20px; }
    .stat-card { background: white; border-radius: 16px; padding: 14px 18px; border: 1px solid #dbe7f5; text-align: center; }
    .stat-card .stat-value { font-size: 1.6rem; font-weight: 800; color: #1e4b7c; line-height: 1; }
    .stat-card .stat-label { font-size: 0.78rem; color: #64748b; font-weight: 600; margin-top: 4px; text-transform: uppercase; }
    .student-list { display: flex; flex-direction: column; gap: 10px; max-height: 480px; overflow-y: auto; padding-right: 4px; }
    .student-row { display: flex; align-items: center; gap: 14px; background: white; padding: 14px 18px; border-radius: 18px; border: 1px solid #e2e8f0; transition: 0.2s; cursor: pointer; position: relative; }
    .student-row:hover { border-color: #1e4b7c; box-shadow: 0 4px 12px rgba(30, 75, 124, 0.1); transform: translateX(4px); }
    .student-row.selected { border-color: #1e4b7c; background: #f0f6fe; }
    .student-row.writing-now { border-left: 5px solid #d97706; background: #fffbeb; }
    .student-avatar { width: 44px; height: 44px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 700; color: white; font-size: 0.95rem; flex-shrink: 0; position: relative; }
    .student-avatar .online-dot { position: absolute; bottom: 0; right: 0; width: 12px; height: 12px; border-radius: 50%; background: #16a34a; border: 2px solid white; }
    .student-avatar .writing-dot { position: absolute; bottom: 0; right: 0; width: 12px; height: 12px; border-radius: 50%; background: #d97706; border: 2px solid white; animation: pulse 1.2s infinite; }
    .student-info { flex: 1; min-width: 0; }
    .student-info .name { font-weight: 700; color: #0b2a44; font-size: 0.95rem; display: flex; align-items: center; gap: 8px; }
    .student-info .nim { font-size: 0.8rem; color: #64748b; margin-top: 2px; }
    .student-info .email-mini { font-size: 0.72rem; color: #94a3b8; margin-top: 2px; display: flex; align-items: center; gap: 4px; }
    .student-info .sesi-mini { font-size: 0.72rem; color: #92400e; margin-top: 2px; display: flex; align-items: center; gap: 4px; font-weight: 600; }
    .student-status { display: flex; flex-direction: column; align-items: flex-end; gap: 4px; min-width: 110px; }
    .status-pill { padding: 5px 14px; border-radius: 30px; font-size: 0.72rem; font-weight: 700; text-transform: uppercase; }
    .status-pill.mengerjakan { background: #fef3c7; color: #92400e; }
    .status-pill.selesai { background: #dcfce7; color: #166534; }
    .status-pill.pretest { background: #fed7aa; color: #9a3412; }
    .status-pill.posttest { background: #bbf7d0; color: #166534; }
    .student-score-badge { font-size: 1.1rem; font-weight: 800; color: #1e4b7c; }
    .student-score-badge.good { color: #15803d; }
    .student-score-badge.warn { color: #d97706; }
    .student-score-badge.bad { color: #b91c1c; }
    .btn-action-small { background: #eef4ff; color: #1e4b7c; border: 1.5px solid #b8d7f0; padding: 8px 12px; border-radius: 12px; font-size: 0.8rem; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 4px; font-weight: 700; }
    .btn-action-small:hover { background: #1e4b7c; color: white; border-color: #1e4b7c; transform: scale(1.05); }
    .btn-delete-student { background: #fef2f2; color: #b91c1c; border: 1.5px solid #fecaca; padding: 8px 12px; border-radius: 12px; font-size: 0.8rem; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 4px; font-weight: 700; }
    .btn-delete-student:hover { background: #dc2626; color: white; border-color: #dc2626; transform: scale(1.05); }
    .soal-penilaian-selector { display: flex; gap: 10px; margin: 0 0 20px 0; background: #f0f4fa; padding: 8px; border-radius: 60px; width: fit-content; flex-wrap: wrap; }
    .soal-penilaian-btn { padding: 12px 28px; border: none; background: transparent; font-weight: 700; font-size: 0.95rem; border-radius: 40px; cursor: pointer; color: #3a4e66; transition: 0.2s; display: flex; align-items: center; gap: 8px; }
    .soal-penilaian-btn.active { background: #1e4b7c; color: white; }
    .soal-penilaian-btn .status-dot { width: 8px; height: 8px; border-radius: 50%; background: #cbd5e1; }
    .soal-penilaian-btn.sudah-dinilai .status-dot { background: #15803d; }
    .student-submission-card { background: #f8fbff; border-radius: 24px; padding: 24px 28px; margin-bottom: 24px; border: 1px solid #dbe7f5; }
    .student-submission-card h3 { font-size: 1.2rem; font-weight: 700; color: #103456; margin-bottom: 12px; display: flex; align-items: center; gap: 8px; }
    .student-meta { display: flex; gap: 20px; flex-wrap: wrap; margin-bottom: 16px; color: #475569; font-size: 0.9rem; }
    .student-answer-preview { background: white; padding: 16px 20px; border-radius: 16px; border: 1px solid #e2e8f0; line-height: 1.6; color: #1e293b; font-size: 0.95rem; margin-bottom: 16px; max-height: 320px; overflow-y: auto; }
    .sub-answer-block { margin-bottom: 14px; padding-bottom: 12px; border-bottom: 1px dashed #e2e8f0; }
    .sub-answer-block:last-child { border-bottom: none; margin-bottom: 0; padding-bottom: 0; }
    .sub-answer-label { font-weight: 700; color: #1e4b7c; font-size: 0.85rem; margin-bottom: 6px; display: block; }
    .rubrik-aspect-container { display: flex; flex-direction: column; gap: 20px; margin: 20px 0; }
    .rubrik-aspect-block { background: white; border-radius: 20px; border: 2px solid #dbe4ee; overflow: hidden; transition: 0.2s; }
    .rubrik-aspect-block:hover { border-color: #1e4b7c; box-shadow: 0 8px 24px rgba(30, 75, 124, 0.1); }
    .rubrik-aspect-header { background: linear-gradient(135deg, #1e4b7c 0%, #3b82f6 100%); color: white; padding: 18px 24px; display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 12px; }
    .rubrik-aspect-header .aspect-title { display: flex; align-items: center; gap: 12px; font-weight: 700; font-size: 1.05rem; }
    .rubrik-aspect-header .aspect-title .aspect-number { background: rgba(255,255,255,0.25); width: 36px; height: 36px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 800; font-size: 1rem; flex-shrink: 0; }
    .rubrik-aspect-header .aspect-title small { display: block; font-weight: 400; font-size: 0.78rem; opacity: 0.9; margin-top: 2px; }
    .rubrik-aspect-header .aspect-score-badge { background: rgba(255,255,255,0.2); padding: 8px 18px; border-radius: 30px; font-weight: 800; font-size: 1.1rem; min-width: 80px; text-align: center; border: 2px solid rgba(255,255,255,0.3); }
    .rubrik-aspect-header .aspect-score-badge.has-value { background: #15803d; border-color: #15803d; }
    .rubrik-descriptor-list { padding: 8px 0; }
    .rubrik-descriptor-item { display: flex; align-items: flex-start; gap: 14px; padding: 14px 24px; border-bottom: 1px solid #f1f5f9; cursor: pointer; transition: 0.15s; }
    .rubrik-descriptor-item:last-child { border-bottom: none; }
    .rubrik-descriptor-item:hover { background: #f8fbff; }
    .rubrik-descriptor-item.selected { background: linear-gradient(90deg, #dcfce7 0%, #f0fdf4 100%); border-left: 6px solid #15803d; padding-left: 18px; }
    .rubrik-descriptor-item .radio-wrapper { display: flex; align-items: center; gap: 10px; min-width: 70px; flex-shrink: 0; padding-top: 2px; }
    .rubrik-descriptor-item input[type="radio"] { width: 22px; height: 22px; accent-color: #1e4b7c; cursor: pointer; flex-shrink: 0; }
    .rubrik-descriptor-item .skor-label { font-weight: 800; color: #1e4b7c; font-size: 1.1rem; min-width: 24px; text-align: center; }
    .rubrik-descriptor-item.selected .skor-label { color: #15803d; }
    .rubrik-descriptor-item .descriptor-text { font-size: 0.88rem; line-height: 1.55; color: #1e293b; flex: 1; }
    .rubrik-descriptor-item.selected .descriptor-text { color: #065f46; font-weight: 500; }
    .rubrik-total-panel { background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%); border-radius: 20px; padding: 24px; margin: 20px 0; border: 2px solid #b8d7f0; display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 16px; }
    .rubrik-total-panel .total-label { font-size: 1rem; font-weight: 600; color: #0b2a44; }
    .rubrik-total-panel .total-label small { display: block; font-size: 0.78rem; color: #64748b; font-weight: 400; margin-top: 2px; }
    .rubrik-total-panel .total-value { background: #1e4b7c; color: white; padding: 14px 32px; border-radius: 40px; font-size: 2rem; font-weight: 800; min-width: 130px; text-align: center; box-shadow: 0 8px 20px rgba(30, 75, 124, 0.3); }
    .rubrik-progress-info { font-size: 0.82rem; color: #64748b; margin-top: 6px; text-align: right; }
    .rubrik-progress-info.complete { color: #15803d; font-weight: 600; }
    .feedback-input-section { background: #fffbeb; border-radius: 20px; padding: 20px 24px; margin: 20px 0; border: 1px solid #fde68a; }
    .feedback-input-section label { font-weight: 700; color: #78350f; display: flex; align-items: center; gap: 8px; margin-bottom: 10px; font-size: 1rem; }
    .feedback-input-section textarea { width: 100%; padding: 14px 18px; border: 2px solid #fcd34d; border-radius: 16px; font-family: 'Inter', sans-serif; font-size: 0.95rem; line-height: 1.6; resize: vertical; min-height: 100px; background: white; }
    .btn-save-grade { background: #15803d; color: white; border: none; padding: 16px 32px; border-radius: 40px; font-weight: 700; font-size: 1rem; display: flex; align-items: center; justify-content: center; gap: 10px; cursor: pointer; transition: 0.2s; box-shadow: 0 8px 20px rgba(21, 128, 61, 0.3); width: 100%; }
    .btn-save-grade:hover { background: #166534; transform: translateY(-2px); }
    .btn-save-grade:disabled { background: #94a3b8; box-shadow: none; cursor: not-allowed; }
    .grade-saved-notif { background: #dcfce7; border-left: 6px solid #15803d; padding: 16px 22px; border-radius: 16px; margin-top: 16px; color: #166534; font-weight: 600; display: none; align-items: center; gap: 12px; }
    .rubrik-legend { background: #fffbeb; border: 1px solid #fde68a; padding: 14px 20px; border-radius: 16px; margin-bottom: 20px; color: #92400e; font-size: 0.85rem; line-height: 1.6; display: flex; align-items: flex-start; gap: 10px; }
    .db-panel { background: linear-gradient(135deg, #f0f6fe 0%, #e6f0fa 100%); border-radius: 24px; padding: 24px 28px; margin-bottom: 24px; border: 1px solid #b8d7f0; }
    .db-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; flex-wrap: wrap; gap: 12px; }
    .db-header h3 { font-size: 1.2rem; font-weight: 700; color: #103456; display: flex; align-items: center; gap: 8px; }
    .db-filters { display: flex; gap: 12px; flex-wrap: wrap; margin-bottom: 20px; align-items: center; }
    .db-filters select, .db-filters input { padding: 10px 16px; border: 2px solid #dbe4ee; border-radius: 30px; font-size: 0.9rem; font-family: 'Inter', sans-serif; background: white; }
    .btn-download-csv { background: linear-gradient(135deg, #15803d 0%, #166534 100%); color: white; border: none; padding: 12px 24px; border-radius: 40px; font-weight: 700; font-size: 0.95rem; display: flex; align-items: center; gap: 10px; cursor: pointer; transition: 0.2s; box-shadow: 0 8px 20px rgba(21, 128, 61, 0.3); margin-left: auto; }
    .btn-download-csv:hover { transform: translateY(-2px); }
    .db-summary { display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 12px; margin-bottom: 20px; }
    .db-summary-card { background: white; border-radius: 16px; padding: 14px 18px; border: 1px solid #dbe7f5; text-align: center; }
    .db-summary-card .value { font-size: 1.5rem; font-weight: 800; color: #1e4b7c; line-height: 1; }
    .db-summary-card .label { font-size: 0.75rem; color: #64748b; font-weight: 600; margin-top: 4px; text-transform: uppercase; }
    .db-table-wrapper { overflow-x: auto; border-radius: 16px; box-shadow: 0 4px 20px rgba(0,0,0,0.06); background: white; }
    .db-table { width: 100%; border-collapse: collapse; min-width: 1200px; }
    .db-table th { background: #1e4b7c; color: white; padding: 12px 14px; text-align: left; font-weight: 600; font-size: 0.8rem; white-space: nowrap; }
    .db-table td { padding: 12px 14px; border-bottom: 1px solid #e8eef5; color: #1e293b; font-size: 0.85rem; vertical-align: middle; }
    .db-table .aspek-score { text-align: center; font-weight: 600; color: #475569; }
    .status-db { padding: 4px 12px; border-radius: 30px; font-size: 0.7rem; font-weight: 700; text-transform: uppercase; white-space: nowrap; }
    .status-db.selesai { background: #dcfce7; color: #166534; }
    .status-db.mengerjakan { background: #fef3c7; color: #92400e; }
    .status-db.belum { background: #f1f5f9; color: #64748b; }
    .ai-tag { padding: 4px 10px; border-radius: 30px; font-size: 0.7rem; font-weight: 700; }
    .ai-tag.bebas { background: #dcfce7; color: #166534; }
    .ai-tag.indikasi { background: #fee2e2; color: #b91c1c; }
    .sesi-tag { padding: 4px 10px; border-radius: 30px; font-size: 0.7rem; font-weight: 700; }
    .sesi-tag.pretest { background: #fed7aa; color: #9a3412; }
    .sesi-tag.posttest { background: #bbf7d0; color: #166534; }
    .btn-delete-db { background: #fef2f2; color: #b91c1c; border: 1.5px solid #fecaca; padding: 6px 12px; border-radius: 10px; font-size: 0.75rem; font-weight: 700; cursor: pointer; transition: 0.2s; display: inline-flex; align-items: center; gap: 4px; }
    .btn-delete-db:hover { background: #dc2626; color: white; border-color: #dc2626; }
    .db-empty { padding: 40px; text-align: center; color: #94a3b8; font-size: 0.95rem; }
    .db-empty i { font-size: 3rem; margin-bottom: 12px; display: block; }
    .toast { position: fixed; bottom: 30px; right: 30px; background: white; padding: 18px 24px; border-radius: 20px; box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2); display: flex; align-items: center; gap: 14px; z-index: 10000; transform: translateX(500px); transition: transform 0.4s ease-out; max-width: 380px; border-left: 6px solid #1e4b7c; }
    .toast.show { transform: translateX(0); }
    .toast.success { border-left-color: #15803d; }
    .toast.error { border-left-color: #dc2626; }
    .toast i { font-size: 1.6rem; }
    .toast.success i { color: #15803d; }
    .toast.error i { color: #dc2626; }
    .toast.info i { color: #1e4b7c; }
    .toast .toast-title { font-weight: 700; color: #0b1a2e; font-size: 0.95rem; margin-bottom: 2px; }
    .toast .toast-message { font-size: 0.85rem; color: #64748b; line-height: 1.4; }
    .soal-edit-form { display: flex; flex-direction: column; gap: 16px; }
    .soal-edit-form textarea { min-height: 100px; }
    .soal-edit-list { display: flex; flex-direction: column; gap: 12px; margin-bottom: 24px; }
    .soal-edit-item { background: #f8fbff; border-radius: 16px; padding: 16px 20px; border: 1px solid #dbe7f5; cursor: pointer; transition: 0.2s; display: flex; justify-content: space-between; align-items: center; gap: 12px; }
    .soal-edit-item:hover { border-color: #1e4b7c; background: #f0f6fe; transform: translateX(4px); }
    .soal-edit-item .soal-title { font-weight: 700; color: #0b2a44; font-size: 0.95rem; flex: 1; }
    .soal-edit-item .soal-title small { display: block; font-weight: 400; color: #64748b; font-size: 0.8rem; margin-top: 2px; }
    .soal-edit-item .soal-badge { background: #1e4b7c; color: white; padding: 6px 14px; border-radius: 30px; font-weight: 700; font-size: 0.8rem; }
    .sync-indicator { display: inline-flex; align-items: center; gap: 6px; background: #dcfce7; color: #166534; padding: 4px 12px; border-radius: 30px; font-size: 0.72rem; font-weight: 700; }
    .sync-indicator.syncing { background: #fef3c7; color: #92400e; }
    .sync-indicator i { font-size: 0.8rem; }
    .soal-edit-actions { display: flex; gap: 8px; align-items: center; }
    .btn-edit-soal { background: #eef4ff; color: #1e4b7c; border: 1.5px solid #b8d7f0; padding: 8px 14px; border-radius: 10px; font-size: 0.8rem; font-weight: 700; cursor: pointer; transition: 0.2s; display: inline-flex; align-items: center; gap: 4px; }
    .btn-edit-soal:hover { background: #1e4b7c; color: white; border-color: #1e4b7c; }
    .btn-delete-soal { background: #fef2f2; color: #b91c1c; border: 1.5px solid #fecaca; padding: 8px 14px; border-radius: 10px; font-size: 0.8rem; font-weight: 700; cursor: pointer; transition: 0.2s; display: inline-flex; align-items: center; gap: 4px; }
    .btn-delete-soal:hover { background: #dc2626; color: white; border-color: #dc2626; }
    .btn-add-soal { background: linear-gradient(135deg, #15803d 0%, #166534 100%); color: white; border: none; padding: 14px 24px; border-radius: 40px; font-weight: 700; font-size: 0.95rem; cursor: pointer; transition: 0.2s; display: inline-flex; align-items: center; gap: 8px; box-shadow: 0 8px 20px rgba(21, 128, 61, 0.3); margin-bottom: 20px; font-family: 'Inter', sans-serif; }
    .btn-add-soal:hover { transform: translateY(-2px); }
    .loading-overlay { position: fixed; inset: 0; background: rgba(11, 26, 46, 0.9); backdrop-filter: blur(8px); display: none; align-items: center; justify-content: center; z-index: 99999; flex-direction: column; gap: 20px; }
    .loading-overlay.show { display: flex; }
    .loading-overlay .spinner { width: 60px; height: 60px; border: 6px solid rgba(255,255,255,0.2); border-top-color: #3b82f6; border-radius: 50%; animation: spin 1s linear infinite; }
    .loading-overlay .loading-text { color: white; font-size: 1rem; font-weight: 600; }

    /* Reset Modal Styles */
    .reset-icon { width: 80px; height: 80px; margin: 0 auto 20px; background: linear-gradient(135deg, #1e4b7c, #3b82f6); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-size: 2rem; }
    .reset-icon.email { background: linear-gradient(135deg, #7c3aed, #6d28d9); }
    .reset-icon.success { background: linear-gradient(135deg, #16a34a, #22c55e); }
    .reset-error { background: #fef2f2; color: #b91c1c; padding: 12px 16px; border-radius: 12px; margin-bottom: 16px; font-size: 0.85rem; font-weight: 600; display: none; align-items: center; gap: 8px; }
    .reset-error.show { display: flex; }
    .reset-success { background: #dcfce7; color: #166534; padding: 12px 16px; border-radius: 12px; margin-bottom: 16px; font-size: 0.85rem; font-weight: 600; display: none; align-items: center; gap: 8px; }
    .reset-success.show { display: flex; }
    .otp-input-group { display: flex; gap: 10px; justify-content: center; margin-bottom: 20px; }
    .otp-input-group input { width: 52px; height: 60px; text-align: center; font-size: 1.6rem; font-weight: 700; border: 2px solid #dbe4ee; border-radius: 14px; color: #1e4b7c; background: #fafcff; transition: 0.2s; font-family: 'Inter', sans-serif; }
    .otp-input-group input:focus { outline: none; border-color: #1e4b7c; box-shadow: 0 0 0 4px rgba(30, 75, 124, 0.12); background: white; }
    .otp-input-group input.filled { background: #eff6ff; border-color: #3b82f6; }
    .reset-actions { display: flex; gap: 12px; justify-content: center; margin-top: 16px; flex-wrap: wrap; }
    .btn-reset { padding: 12px 28px; border-radius: 40px; font-weight: 700; font-size: 0.95rem; cursor: pointer; transition: 0.2s; border: none; display: flex; align-items: center; gap: 8px; font-family: 'Inter', sans-serif; }
    .btn-reset-primary { background: #1e4b7c; color: white; box-shadow: 0 8px 20px rgba(30, 75, 124, 0.3); }
    .btn-reset-primary:hover { background: #0f3a60; transform: translateY(-2px); }
    .btn-reset-secondary { background: #f1f5f9; color: #475569; }
    .btn-reset-secondary:hover { background: #e2e8f0; }
    .btn-reset-success { background: #16a34a; color: white; box-shadow: 0 8px 20px rgba(22, 163, 74, 0.3); }
    .btn-reset-success:hover { background: #15803d; transform: translateY(-2px); }
    .reset-timer { text-align: center; margin-top: 16px; font-size: 0.85rem; color: #64748b; font-weight: 600; }
    .reset-timer strong { color: #1e4b7c; }
    .password-strength { height: 6px; background: #e2e8f0; border-radius: 10px; margin-top: 8px; overflow: hidden; }
    .password-strength-fill { height: 100%; width: 0%; border-radius: 10px; transition: 0.3s; }
    .password-strength-text { font-size: 0.75rem; font-weight: 600; margin-top: 4px; }

    @media (max-width: 768px) {
      .device-stats-grid { grid-template-columns: 1fr 1fr; }
      .notif-panel { width: calc(100vw - 20px); right: 10px; top: 70px; }
      .device-sync-panel { padding: 18px 16px; }
      .device-sync-header h3 { font-size: 1rem; }
      .portal-container { padding: 12px; border-radius: 24px; }
      .main-card { border-radius: 20px; }
      .login-section, .admin-dashboard, .dosen-dashboard, .sesi-portal, .exam-section { padding: 20px 16px; }
      .realtime-badge { right: 70px; font-size: 0.7rem; padding: 8px 12px; }
      .notif-bell { width: 42px; height: 42px; }
    }
  </style>
</head>
<body>

<!-- Realtime Badge -->
<div class="realtime-badge offline" id="realtimeBadge">
  <span class="dot"></span>
  <span id="realtimeText">Menghubungkan...</span>
</div>

<!-- Notification Bell -->
<button class="notif-bell" id="notifBell" title="Notifikasi">
  <i class="fas fa-bell"></i>
  <span class="notif-count" id="notifCount" style="display:none;">0</span>
</button>

<!-- Notification Panel -->
<div class="notif-panel" id="notifPanel">
  <div class="notif-panel-header">
    <h4><i class="fas fa-bell" style="color:#1e4b7c;"></i> Notifikasi</h4>
    <button class="btn-action-small" id="markAllReadBtn" style="padding:6px 12px;font-size:0.75rem;">
      <i class="fas fa-check-double"></i> Tandai Dibaca
    </button>
  </div>
  <div class="notif-panel-body" id="notifPanelBody">
    <div class="notif-empty">
      <i class="fas fa-bell-slash"></i>
      <div>Belum ada notifikasi</div>
    </div>
  </div>
</div>

<!-- Offline Queue Badge -->
<div class="offline-queue-badge" id="offlineQueueBadge">
  <i class="fas fa-cloud-upload-alt"></i>
  <span>Menunggu koneksi...</span>
  <span class="queue-count" id="queueCount">0</span>
</div>

<!-- Device Conflict Alert -->
<div class="device-conflict-alert" id="deviceConflictAlert">
  <div class="alert-icon"><i class="fas fa-exclamation-triangle"></i></div>
  <h3>Perangkat Lain Terdeteksi</h3>
  <p id="conflictMessage">Akun Anda sedang aktif di perangkat lain. Apakah Anda ingin melanjutkan di perangkat ini?</p>
  <div class="conflict-actions">
    <button class="btn-modal btn-modal-secondary" id="conflictCancelBtn">
      <i class="fas fa-times"></i> Batalkan
    </button>
    <button class="btn-modal btn-modal-primary" id="conflictContinueBtn">
      <i class="fas fa-check"></i> Lanjutkan di Sini
    </button>
  </div>
</div>

<!-- Loading -->
<div class="loading-overlay" id="loadingOverlay">
  <div class="spinner"></div>
  <div class="loading-text">Memuat data dari cloud...</div>
</div>

<div class="portal-container">
  <div class="main-card">
    <!-- LOGIN -->
    <div id="loginSection" class="login-section">
      <div class="login-header">
        <img src="https://ftvsinetron.wordpress.com/wp-content/uploads/2025/05/00d61-logo-kampus-unm-bintang-dua-tanpa-latar-belakang-terbaru.png"
             alt="Logo UNM" class="logo-unm" onerror="this.style.display='none'">
        <div class="header-text">
          <h1>Portal Ujian Pendidikan Fisika</h1>
          <span>Universitas Negeri Makassar · Multi-Device</span>
        </div>
      </div>
      <div class="role-tabs">
        <button id="roleMahasiswaBtn" class="role-btn active"><i class="fas fa-user-graduate"></i> Mahasiswa</button>
        <button id="roleDosenBtn" class="role-btn"><i class="fas fa-chalkboard-teacher"></i> Dosen</button>
        <button id="roleAdminBtn" class="role-btn admin"><i class="fas fa-user-shield"></i> Admin</button>
      </div>
      <div class="auth-tabs">
        <button id="tabLogin" class="auth-tab active"><i class="fas fa-sign-in-alt"></i> Masuk</button>
        <button id="tabRegister" class="auth-tab"><i class="fas fa-user-plus"></i> Daftar</button>
      </div>
      <div id="loginFormContainer" class="login-form">
        <div class="input-group">
          <label><i class="fas fa-user"></i> <span id="loginLabelText">Nama Lengkap / NIM</span></label>
          <input type="text" id="loginName" placeholder="Contoh: 2101010101 / Budi Santoso">
        </div>
        <div class="input-group">
          <label><i class="fas fa-lock"></i> Password</label>
          <input type="password" id="loginPassword" placeholder="••••••••">
        </div>
        <div class="forgot-password-wrapper">
          <button type="button" id="forgotPasswordBtn" class="btn-forgot-password">
            <i class="fas fa-key"></i> Lupa Password? Reset di sini
          </button>
        </div>
        <button id="loginBtn" class="btn-login"><i class="fas fa-arrow-right-to-bracket"></i> Masuk Portal</button>
        <div class="login-note">
          <i class="fas fa-cloud"></i> <strong>Multi-Device Sync:</strong> Login dari HP, laptop, atau tablet — data otomatis tersinkron.<br>
          <i class="fas fa-shield-alt"></i> Field jawaban dikunci: paste dari luar tidak diizinkan.
        </div>
      </div>
      <div id="registerFormContainer" class="login-form hidden">
        <div id="registerBannerMahasiswa" class="register-role-banner mahasiswa">
          <div class="icon-circle"><i class="fas fa-user-graduate"></i></div>
          <div class="banner-text"><h4>Pendaftaran Mahasiswa</h4><p>Isi data diri, NIM, dan email aktif. Akun langsung tersinkron ke cloud.</p></div>
        </div>
        <div id="registerBannerDosen" class="register-role-banner dosen hidden">
          <div class="icon-circle"><i class="fas fa-chalkboard-teacher"></i></div>
          <div class="banner-text"><h4>Pendaftaran Dosen</h4><p>Isi data dosen, NIDN, dan email institusi. Akun langsung tersinkron ke cloud.</p></div>
        </div>
        <div id="registerFormMahasiswa">
          <div class="input-group"><label><i class="fas fa-user"></i> Nama Lengkap</label><input type="text" id="mhsName" placeholder="Budi Santoso"></div>
          <div class="form-row">
            <div class="input-group"><label><i class="fas fa-id-card"></i> NIM</label><input type="text" id="mhsNim" placeholder="2101010101"></div>
            <div class="input-group"><label><i class="fas fa-graduation-cap"></i> Program Studi</label>
              <select id="mhsProdi"><option>Pendidikan Fisika</option><option>Fisika</option><option>Teknik Fisika</option><option>Pendidikan IPA</option></select>
            </div>
          </div>
          <div class="form-row">
            <div class="input-group"><label><i class="fas fa-calendar"></i> Angkatan</label><input type="text" id="mhsAngkatan" placeholder="2021" maxlength="4"></div>
            <div class="input-group"><label><i class="fas fa-university"></i> Kelas</label><input type="text" id="mhsKelas" placeholder="A / B / C"></div>
          </div>
          <div class="input-group"><label><i class="fas fa-envelope"></i> Email Aktif</label><input type="email" id="mhsEmail" placeholder="nama@student.unm.ac.id"></div>
          <div class="form-row">
            <div class="input-group"><label><i class="fas fa-lock"></i> Password</label><input type="password" id="mhsPassword" placeholder="Min 6 karakter"></div>
            <div class="input-group"><label><i class="fas fa-lock"></i> Konfirmasi</label><input type="password" id="mhsPasswordConfirm" placeholder="Ulangi"></div>
          </div>
          <button id="registerMahasiswaBtn" class="btn-register"><i class="fas fa-user-plus"></i> Daftar Sebagai Mahasiswa</button>
        </div>
        <div id="registerFormDosen" class="hidden">
          <div class="input-group"><label><i class="fas fa-user-tie"></i> Nama Lengkap (dengan gelar)</label><input type="text" id="dsnName" placeholder="Dr. Budi Santoso, M.Pd"></div>
          <div class="form-row">
            <div class="input-group"><label><i class="fas fa-id-card"></i> NIDN / NIP</label><input type="text" id="dsnNidn" placeholder="0012345678"></div>
            <div class="input-group"><label><i class="fas fa-briefcase"></i> Jabatan Akademik</label>
              <select id="dsnJabatan"><option>Asisten Ahli</option><option>Lektor</option><option>Lektor Kepala</option><option>Guru Besar</option></select>
            </div>
          </div>
          <div class="form-row">
            <div class="input-group"><label><i class="fas fa-building"></i> Program Studi</label>
              <select id="dsnProdi"><option>Pendidikan Fisika</option><option>Fisika</option><option>Teknik Fisika</option><option>Pendidikan IPA</option></select>
            </div>
            <div class="input-group"><label><i class="fas fa-book"></i> Mata Kuliah Diampu</label><input type="text" id="dsnMatkul" placeholder="Media Pembelajaran Fisika"></div>
          </div>
          <div class="input-group"><label><i class="fas fa-envelope"></i> Email Institusi</label><input type="email" id="dsnEmail" placeholder="nama@unm.ac.id"></div>
          <div class="form-row">
            <div class="input-group"><label><i class="fas fa-lock"></i> Password</label><input type="password" id="dsnPassword" placeholder="Min 8 karakter"></div>
            <div class="input-group"><label><i class="fas fa-lock"></i> Konfirmasi</label><input type="password" id="dsnPasswordConfirm" placeholder="Ulangi"></div>
          </div>
          <button id="registerDosenBtn" class="btn-register"><i class="fas fa-user-plus"></i> Daftar Sebagai Dosen</button>
        </div>
      </div>
      <div class="portal-footer">
        <img src="https://ftvsinetron.wordpress.com/wp-content/uploads/2025/05/00d61-logo-kampus-unm-bintang-dua-tanpa-latar-belakang-terbaru.png"
             alt="Logo UNM" class="footer-logo" onerror="this.style.display='none'">
        <div class="footer-text">
          <strong>Universitas Negeri Makassar</strong>
          Portal Ujian · Realtime Multi-Device Monitoring
        </div>
      </div>
    </div>

    <!-- ADMIN DASHBOARD -->
    <div id="adminSection" class="admin-dashboard">
      <div class="admin-header">
        <div class="admin-title-group">
          <div class="admin-icon"><i class="fas fa-user-shield"></i></div>
          <h2>
            <div>
              Panel Admin
              <small>Kelola Pengguna · Rikardus Feribertus Nikat</small>
            </div>
          </h2>
        </div>
        <div style="display: flex; align-items: center; gap: 16px;">
          <span class="admin-badge"><i class="fas fa-crown"></i> SUPER ADMIN</span>
          <button id="logoutAdminBtn" class="logout-btn"><i class="fas fa-sign-out-alt"></i> Keluar</button>
        </div>
      </div>

      <div class="admin-stats-grid">
        <div class="admin-stat-card purple">
          <div class="stat-icon-lg"><i class="fas fa-users"></i></div>
          <div class="stat-number" id="adminTotalUsers">0</div>
          <div class="stat-label">Total Pengguna</div>
        </div>
        <div class="admin-stat-card orange">
          <div class="stat-icon-lg orange"><i class="fas fa-chalkboard-teacher"></i></div>
          <div class="stat-number" id="adminTotalDosen">0</div>
          <div class="stat-label">Total Dosen</div>
        </div>
        <div class="admin-stat-card green">
          <div class="stat-icon-lg green"><i class="fas fa-user-graduate"></i></div>
          <div class="stat-number" id="adminTotalMhs">0</div>
          <div class="stat-label">Total Mahasiswa</div>
        </div>
        <div class="admin-stat-card blue">
          <div class="stat-icon-lg blue"><i class="fas fa-book"></i></div>
          <div class="stat-number" id="adminTotalSoal">0</div>
          <div class="stat-label">Total Soal</div>
        </div>
      </div>

      <div class="user-mgmt-panel">
        <div class="user-mgmt-header">
          <h3><i class="fas fa-users-cog"></i> Kelola Pengguna (Realtime)</h3>
          <button id="btnRefreshUsers" class="btn-action-small"><i class="fas fa-sync-alt"></i> Refresh</button>
        </div>
        <div class="user-filter-row">
          <select id="adminFilterRole">
            <option value="all">Semua Role</option>
            <option value="admin">Admin</option>
            <option value="dosen">Dosen</option>
            <option value="mahasiswa">Mahasiswa</option>
          </select>
          <input type="text" id="adminSearchUser" placeholder="Cari nama / NIDN / NIM / email...">
        </div>
        <div class="user-list" id="adminUserList"></div>
      </div>
    </div>

    <!-- DOSEN DASHBOARD -->
    <div id="dosenSection" class="dosen-dashboard">
      <div class="dosen-header">
        <div class="dosen-title-group">
          <img src="https://ftvsinetron.wordpress.com/wp-content/uploads/2025/05/00d61-logo-kampus-unm-bintang-dua-tanpa-latar-belakang-terbaru.png"
               alt="Logo UNM" class="dosen-logo" onerror="this.style.display='none'">
          <h2><i class="fas fa-chalkboard-teacher" style="color:#1e4b7c;"></i>
            <div>Dashboard Dosen<small>Realtime Multi-Device · <span class="sync-indicator" id="syncIndicator"><i class="fas fa-sync-alt fa-spin"></i> <span>Menghubungkan...</span></span></small></div>
          </h2>
        </div>
        <div style="display: flex; align-items: center; gap: 16px;">
          <div class="user-badge">
            <i class="fas fa-user-tie"></i>
            <span id="dosenNameDisplay">Dosen</span>
            <span class="badge-role-dosen" id="dosenRoleBadge">Dosen</span>
          </div>
          <button id="logoutDosenBtn" class="logout-btn"><i class="fas fa-sign-out-alt"></i> Keluar</button>
        </div>
      </div>
      <div class="dashboard-tabs">
        <button class="dashboard-tab active" data-dashboard-tab="monitoring"><i class="fas fa-users"></i> Monitoring</button>
        <button class="dashboard-tab" data-dashboard-tab="penilaian"><i class="fas fa-table"></i> Penilaian</button>
        <button class="dashboard-tab" data-dashboard-tab="soal"><i class="fas fa-edit"></i> Edit Soal</button>
        <button class="dashboard-tab" data-dashboard-tab="database"><i class="fas fa-database"></i> Database</button>
      </div>

      <div class="dashboard-tab-content active" id="tabMonitoring">
        <!-- MULTI-DEVICE SYNC PANEL -->
        <div class="device-sync-panel">
          <div class="device-sync-header">
            <h3>
              <i class="fas fa-network-wired"></i> 
              Multi-Device Sync Center
            </h3>
            <div style="display:flex; gap:8px; align-items:center; flex-wrap:wrap;">
              <span class="sync-status-bar synced" id="syncStatusBar">
                <i class="fas fa-check-circle"></i>
                <span id="syncStatusText">Tersinkronisasi</span>
              </span>
              <button class="btn-action-small" id="showQrPairingBtn">
                <i class="fas fa-qrcode"></i> QR Pairing
              </button>
            </div>
          </div>
          
          <div class="device-stats-grid">
            <div class="device-stat">
              <div class="icon-box green"><i class="fas fa-laptop"></i></div>
              <div class="stat-info">
                <div class="val" id="deviceCountOnline">0</div>
                <div class="lbl">Device Online</div>
              </div>
            </div>
            <div class="device-stat">
              <div class="icon-box blue"><i class="fas fa-users"></i></div>
              <div class="stat-info">
                <div class="val" id="deviceCountStudents">0</div>
                <div class="lbl">Mahasiswa Aktif</div>
              </div>
            </div>
            <div class="device-stat">
              <div class="icon-box orange"><i class="fas fa-mobile-alt"></i></div>
              <div class="stat-info">
                <div class="val" id="deviceCountMobile">0</div>
                <div class="lbl">Via HP</div>
              </div>
            </div>
            <div class="device-stat">
              <div class="icon-box purple"><i class="fas fa-desktop"></i></div>
              <div class="stat-info">
                <div class="val" id="deviceCountDesktop">0</div>
                <div class="lbl">Via Laptop</div>
              </div>
            </div>
          </div>
          
          <h4 style="font-size:0.95rem; font-weight:700; color:#065f46; margin-bottom:12px; display:flex; align-items:center; gap:8px;">
            <i class="fas fa-list"></i> Device Terhubung
          </h4>
          <div class="device-list" id="deviceList">
            <div class="notif-empty">
              <i class="fas fa-satellite-dish"></i>
              <div>Menunggu device terhubung...</div>
            </div>
          </div>
        </div>

        <!-- LIVE MONITOR PANEL -->
        <div class="live-monitor-panel">
          <div class="live-monitor-header">
            <h3><span class="live-pulse"></span> LIVE MONITORING — Aktivitas Mahasiswa Realtime</h3>
            <div style="display: flex; gap: 8px; align-items: center;">
              <span class="live-badge"><span class="live-dot"></span><span id="liveCountBadge">0 Online</span></span>
            </div>
          </div>
          <div class="live-monitor-stats">
            <div class="live-stat-mini">
              <div class="val" id="liveOnline">0</div>
              <div class="lbl">Online</div>
            </div>
            <div class="live-stat-mini">
              <div class="val" id="liveWriting" style="color:#d97706;">0</div>
              <div class="lbl">Sedang Menulis</div>
            </div>
            <div class="live-stat-mini">
              <div class="val" id="liveSubmitted" style="color:#16a34a;">0</div>
              <div class="lbl">Selesai Kirim</div>
            </div>
            <div class="live-stat-mini">
              <div class="val" id="liveCharacters" style="color:#1e40af;">0</div>
              <div class="lbl">Total Karakter</div>
            </div>
          </div>
          <div class="live-activity-list" id="liveActivityList">
            <div class="live-empty">
              <i class="fas fa-satellite-dish"></i>
              <div>Menunggu aktivitas mahasiswa...</div>
              <div style="font-size:0.78rem; margin-top:6px; color:#cbd5e1;">Aktivitas akan muncul realtime saat mahasiswa menulis jawaban</div>
            </div>
          </div>
        </div>

        <div class="monitoring-panel">
          <div class="monitoring-header">
            <h3><i class="fas fa-users"></i> Daftar Peserta Ujian</h3>
            <div class="live-badge"><span class="live-dot"></span><span>LIVE</span></div>
          </div>
          <div class="monitoring-stats">
            <div class="stat-card"><div class="stat-value" id="statTotal">0</div><div class="stat-label">Total Peserta</div></div>
            <div class="stat-card"><div class="stat-value" style="color:#d97706;" id="statMengerjakan">0</div><div class="stat-label">Mengerjakan</div></div>
            <div class="stat-card"><div class="stat-value" style="color:#15803d;" id="statSelesai">0</div><div class="stat-label">Selesai</div></div>
            <div class="stat-card"><div class="stat-value" style="color:#b91c1c;" id="statIndikasi">0</div><div class="stat-label">Indikasi AI</div></div>
          </div>
          <div class="student-list" id="studentList"></div>
        </div>
      </div>

      <div class="dashboard-tab-content" id="tabPenilaian">
        <h3 style="font-size:1.1rem; font-weight:700; color:#0b1a2e; margin-bottom:12px;"><i class="fas fa-list-ol" style="color:#1e4b7c;"></i> Pilih Mahasiswa:</h3>
        <div class="soal-penilaian-selector" id="mahasiswaSelector"></div>
        <h3 style="font-size:1.1rem; font-weight:700; color:#0b1a2e; margin-bottom:12px;"><i class="fas fa-file-alt" style="color:#1e4b7c;"></i> Pilih Soal:</h3>
        <div class="soal-penilaian-selector" id="soalPenilaianSelector"></div>
        <div class="student-submission-card">
          <h3><i class="fas fa-file-signature"></i> Jawaban <span id="dosenSoalTitle">-</span></h3>
          <div class="student-meta">
            <span><i class="fas fa-user"></i> <strong id="selectedStudentName">-</strong> (<span id="selectedStudentNim">-</span>)</span>
            <span><i class="fas fa-envelope"></i> <span id="selectedStudentEmail">-</span></span>
            <span><i class="fas fa-layer-group"></i> Sesi: <strong id="selectedStudentSesi">-</strong></span>
            <span><i class="fas fa-robot"></i> AI: <strong id="selectedStudentAI">-</strong></span>
          </div>
          <div class="student-answer-preview" id="dosenAnswerPreview"></div>
        </div>
        <div class="rubrik-legend">
          <i class="fas fa-info-circle"></i>
          <div><strong>Panduan Rubrik (0–5):</strong> Nilai setiap aspek terpisah. Nilai soal = (jumlah skor 3 aspek ÷ 15) × 100.</div>
        </div>
        <div class="rubrik-aspect-container" id="rubrikAspectContainer"></div>
        <div class="rubrik-total-panel">
          <div class="total-label">Nilai Soal Ini:<small>(Jumlah skor 3 aspek ÷ 15 × 100)</small><div class="rubrik-progress-info" id="rubrikProgressInfo">0 dari 3 aspek dinilai</div></div>
          <div class="total-value" id="totalNilaiRubrik">0</div>
        </div>
        <div class="feedback-input-section">
          <label><i class="fas fa-comment-dots"></i> Feedback untuk Mahasiswa:</label>
          <textarea id="feedbackDosenInput" placeholder="Tulis feedback..."></textarea>
        </div>
        <button id="saveGradeBtn" class="btn-save-grade" disabled><i class="fas fa-save"></i> Simpan Nilai & Feedback</button>
        <div id="gradeSavedNotif" class="grade-saved-notif"><i class="fas fa-check-circle"></i><span>Nilai & feedback berhasil disimpan!</span></div>
      </div>

      <div class="dashboard-tab-content" id="tabSoal">
        <h3 style="font-size:1.2rem; font-weight:700; color:#0b1a2e; margin-bottom:16px;"><i class="fas fa-edit" style="color:#1e4b7c;"></i> Edit Soal Ujian</h3>
        <p style="color:#64748b; font-size:0.9rem; margin-bottom:20px; line-height:1.6;">Perubahan soal langsung tersinkron ke semua perangkat.</p>
        <button class="btn-add-soal" id="addSoalBtn"><i class="fas fa-plus-circle"></i> Tambah Soal Baru</button>
        <div class="soal-edit-list" id="soalEditList"></div>
        <div id="soalEditFormContainer" class="hidden">
          <div class="soal-edit-form">
            <div class="input-group"><label><i class="fas fa-heading"></i> Judul Soal</label><input type="text" id="editSoalJudul"></div>
            <div class="input-group"><label><i class="fas fa-align-left"></i> Deskripsi / Studi Kasus</label><textarea id="editSoalDeskripsi"></textarea></div>
            <div class="input-group"><label><i class="fas fa-question-circle"></i> Pertanyaan a</label><textarea id="editSoalPertanyaanA"></textarea></div>
            <div class="input-group"><label><i class="fas fa-question-circle"></i> Pertanyaan b</label><textarea id="editSoalPertanyaanB"></textarea></div>
            <div class="input-group"><label><i class="fas fa-question-circle"></i> Pertanyaan c</label><textarea id="editSoalPertanyaanC"></textarea></div>
            <div style="display:flex; gap:12px; justify-content:flex-end; margin-top:12px;">
              <button class="btn-modal btn-modal-secondary" id="cancelEditSoalBtn"><i class="fas fa-times"></i> Batal</button>
              <button class="btn-modal btn-modal-primary" id="saveEditSoalBtn"><i class="fas fa-save"></i> Simpan Perubahan</button>
            </div>
          </div>
        </div>
      </div>
      
      <div class="dashboard-tab-content" id="tabDatabase">
        <div class="db-panel">
          <div class="db-header">
            <h3><i class="fas fa-database"></i> Database Nilai (Cloud)</h3>
            <button id="downloadCsvBtn" class="btn-download-csv"><i class="fas fa-file-csv"></i> Download CSV</button>
          </div>
          <div class="db-filters">
            <select id="filterSesi"><option value="all">Semua Sesi</option><option value="pretest">Pre-Test</option><option value="posttest">Post-Test</option></select>
            <select id="filterStatus"><option value="all">Semua Status</option><option value="selesai">Selesai</option><option value="mengerjakan">Mengerjakan</option><option value="belum">Belum</option></select>
            <select id="filterAI"><option value="all">Semua AI</option><option value="bebas">Bebas AI</option><option value="terindikasi">Indikasi AI</option></select>
            <input type="text" id="filterSearch" placeholder="Cari nama / NIM / email...">
          </div>
          <div class="db-summary">
            <div class="db-summary-card"><div class="value" id="dbTotal">0</div><div class="label">Total</div></div>
            <div class="db-summary-card"><div class="value" style="color:#15803d;" id="dbRataRata">0</div><div class="label">Rata-rata</div></div>
            <div class="db-summary-card"><div class="value" style="color:#1e4b7c;" id="dbTertinggi">0</div><div class="label">Tertinggi</div></div>
            <div class="db-summary-card"><div class="value" style="color:#b91c1c;" id="dbTerendah">0</div><div class="label">Terendah</div></div>
          </div>
          <div class="db-table-wrapper">
            <table class="db-table">
              <thead><tr><th>No</th><th>Sesi</th><th>Nama</th><th>NIM</th><th>Email</th><th>Soal 1</th><th>Soal 2</th><th>Soal 3</th><th>Rata-rata</th><th>AI</th><th>Status</th><th>Aksi</th></tr></thead>
              <tbody id="dbTableBody"></tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Portal Sesi -->
    <div id="sesiPortalSection" class="sesi-portal">
      <div class="sesi-portal-header">
        <div class="title-group">
          <img src="https://ftvsinetron.wordpress.com/wp-content/uploads/2025/05/00d61-logo-kampus-unm-bintang-dua-tanpa-latar-belakang-terbaru.png"
               alt="Logo UNM" class="sesi-logo" onerror="this.style.display='none'">
          <h2><i class="fas fa-layer-group" style="color:#1e4b7c;"></i>
            <div>Pilih Sesi Ujian<small>Portal Ujian · Multi-Device Ready</small></div>
          </h2>
        </div>
        <div style="display: flex; align-items: center; gap: 16px; flex-wrap: wrap;">
          <div class="user-badge-sesi">
            <i class="fas fa-user-circle"></i>
            <span id="sesiUserName">Mahasiswa</span>
            <span class="verified-tag"><i class="fas fa-check-circle"></i> Terverifikasi</span>
          </div>
          <button id="logoutSesiBtn" class="logout-btn-sesi"><i class="fas fa-sign-out-alt"></i> Keluar</button>
        </div>
      </div>
      <div class="sesi-intro">
        <h3><i class="fas fa-info-circle"></i> Pilih Sesi Ujian Anda</h3>
        <p>Pilih <strong>Pre-Test</strong> sebelum pembelajaran atau <strong>Post-Test</strong> setelah pembelajaran. Aktivitas pengisian Anda akan dipantau realtime oleh dosen dari perangkat manapun.</p>
      </div>
      <div class="sesi-grid">
        <div class="sesi-card pretest" data-sesi="pretest">
          <div class="sesi-card-header">
            <div class="sesi-icon"><i class="fas fa-clipboard-list"></i></div>
            <h4>PRE-TEST</h4>
            <p>Tes Awal / Sebelum Pembelajaran</p>
          </div>
          <div class="sesi-card-body">
            <div class="info-line"><i class="fas fa-info-circle"></i><div><strong>Tujuan:</strong> Mengukur pengetahuan awal.</div></div>
            <div class="info-line"><i class="fas fa-question-circle"></i><div><strong>Jumlah Soal:</strong> <span id="pretestJumlahSoal">3</span> soal</div></div>
            <div class="info-line"><i class="fas fa-eye"></i><div><strong>Monitoring:</strong> Dipantau realtime</div></div>
          </div>
          <div class="sesi-card-footer">
            <button class="btn-pilih-sesi pretest" data-sesi="pretest"><i class="fas fa-play-circle"></i> Pilih Sesi Pre-Test</button>
          </div>
        </div>
        <div class="sesi-card posttest" data-sesi="posttest">
          <div class="sesi-card-header">
            <div class="sesi-icon"><i class="fas fa-graduation-cap"></i></div>
            <h4>POST-TEST</h4>
            <p>Tes Akhir / Setelah Pembelajaran</p>
          </div>
          <div class="sesi-card-body">
            <div class="info-line"><i class="fas fa-info-circle"></i><div><strong>Tujuan:</strong> Mengukur pemahaman akhir.</div></div>
            <div class="info-line"><i class="fas fa-question-circle"></i><div><strong>Jumlah Soal:</strong> <span id="posttestJumlahSoal">3</span> soal</div></div>
            <div class="info-line"><i class="fas fa-eye"></i><div><strong>Monitoring:</strong> Dipantau realtime</div></div>
          </div>
          <div class="sesi-card-footer">
            <button class="btn-pilih-sesi posttest" data-sesi="posttest"><i class="fas fa-play-circle"></i> Pilih Sesi Post-Test</button>
          </div>
        </div>
      </div>
      
      <!-- Pairing Option untuk Mahasiswa -->
      <div style="margin-top:24px; background:linear-gradient(135deg, #faf5ff 0%, #f3e8ff 100%); border-radius:20px; padding:20px 24px; border:2px solid #c4b5fd;">
        <div style="display:flex; align-items:center; gap:16px; flex-wrap:wrap;">
          <div style="width:56px; height:56px; background:linear-gradient(135deg, #7c3aed, #6d28d9); border-radius:50%; display:flex; align-items:center; justify-content:center; color:white; font-size:1.5rem; flex-shrink:0;">
            <i class="fas fa-qrcode"></i>
          </div>
          <div style="flex:1; min-width:200px;">
            <h4 style="font-size:1rem; font-weight:700; color:#5b21b6; margin-bottom:4px;">
              <i class="fas fa-link"></i> Punya Kode Pairing dari Dosen?
            </h4>
            <p style="font-size:0.82rem; color:#64748b; line-height:1.5;">
              Masukkan kode pairing untuk langsung terhubung ke sesi dosen Anda.
            </p>
          </div>
          <button id="openPairingInputBtn" class="btn-register" style="background:linear-gradient(135deg,#7c3aed,#6d28d9); padding:12px 24px; margin:0; box-shadow:0 8px 20px rgba(124,58,237,0.3);">
            <i class="fas fa-keyboard"></i> Masukkan Kode
          </button>
        </div>
      </div>
      
      <div class="sesi-info-panel" style="margin-top:20px;">
        <i class="fas fa-lightbulb"></i>
        <div><strong>Catatan:</strong> Setiap ketikan jawaban Anda otomatis tersimpan & terlihat oleh dosen secara realtime dari perangkat manapun.</div>
      </div>
    </div>

    <!-- Exam -->
    <div id="examSection" class="exam-section">
      <div class="exam-header">
        <div class="exam-title-group">
          <img src="https://ftvsinetron.wordpress.com/wp-content/uploads/2025/05/00d61-logo-kampus-unm-bintang-dua-tanpa-latar-belakang-terbaru.png"
               alt="Logo UNM" class="exam-logo" onerror="this.style.display='none'">
          <h2><i class="fas fa-file-alt" style="color:#1e4b7c;"></i>
            <div><span id="examSesiTitle">Ujian Studi Kasus</span><small>Portal Ujian UNM · 🔴 LIVE MONITORED · 📱 Multi-Device</small></div>
          </h2>
        </div>
        <div style="display: flex; align-items: center; gap: 16px;">
          <div class="user-badge">
            <i class="fas fa-user-circle"></i>
            <span id="displayUserName">Mahasiswa</span>
            <span class="verified-tag"><i class="fas fa-check-circle"></i> Terverifikasi</span>
          </div>
          <button id="logoutBtn" class="logout-btn"><i class="fas fa-sign-out-alt"></i> Keluar</button>
        </div>
      </div>
      <div class="soal-nav" id="soalNavContainer"></div>
      <div id="soalContentContainer"></div>
      <div class="email-notif-panel" id="emailNotifPanel">
        <h3><i class="fas fa-envelope-circle-check"></i> Kirim Nilai Akhir ke Email</h3>
        <p>Setelah semua soal dinilai dosen, Anda dapat mengirim rekap nilai akhir.</p>
        <div class="email-info-box">
          <div class="email-row"><span class="email-label">Sesi:</span><span class="email-value" id="emailSesi">-</span></div>
          <div class="email-row"><span class="email-label">Email:</span><span class="email-value" id="emailTarget">-</span></div>
          <div class="email-row"><span class="email-label">Nama:</span><span class="email-value" id="emailNamaMhs">-</span></div>
          <div class="email-row"><span class="email-label">Nilai Akhir:</span><span class="email-value" id="emailNilaiAkhir">-</span></div>
        </div>
        <button class="btn-send-email" id="sendEmailBtn"><i class="fas fa-paper-plane"></i> Kirim Nilai Akhir ke Email</button>
      </div>
    </div>

  </div>
</div>

<!-- Modal Konfirmasi Sesi -->
<div class="modal-overlay" id="sesiConfirmModal">
  <div class="modal-box" style="text-align:center;">
    <div class="reset-icon" id="sesiConfirmIcon" style="background:linear-gradient(135deg,#d97706,#f59e0b);"><i class="fas fa-clipboard-list"></i></div>
    <h3 id="sesiConfirmTitle" style="font-size:1.5rem; font-weight:800; color:#0b1a2e; margin-bottom:10px;">Mulai Pre-Test?</h3>
    <p id="sesiConfirmDesc" style="color:#64748b; margin-bottom:24px;">Setelah memulai, Anda akan masuk ke halaman ujian.</p>
    <div class="reset-actions">
      <button class="btn-reset btn-reset-secondary" id="sesiCancelBtn"><i class="fas fa-times"></i> Batal</button>
      <button class="btn-reset btn-reset-primary" id="sesiStartBtn"><i class="fas fa-play"></i> Mulai Sekarang</button>
    </div>
  </div>
</div>

<!-- Modal QR Pairing -->
<div class="modal-overlay" id="qrPairingModal">
  <div class="modal-box" style="max-width:520px;">
    <div class="modal-header">
      <h3><i class="fas fa-qrcode"></i> QR Code Pairing</h3>
      <button class="modal-close" id="qrPairingClose"><i class="fas fa-times"></i></button>
    </div>
    <div class="qr-container">
      <div class="qr-code-box" id="qrCodeBox">
        <div style="width:240px;height:240px;display:flex;align-items:center;justify-content:center;background:#f8fafc;border-radius:12px;">
          <i class="fas fa-spinner fa-spin" style="font-size:2rem;color:#7c3aed;"></i>
        </div>
      </div>
      <div style="font-size:0.85rem;color:#64748b;margin-bottom:6px;font-weight:600;">Kode Pairing:</div>
      <div class="pairing-code" id="pairingCodeDisplay">------</div>
      <div class="qr-instruction">
        <i class="fas fa-info-circle" style="color:#7c3aed;"></i>
        Minta mahasiswa untuk <strong>scan QR</strong> atau masukkan <strong>kode pairing</strong> di HP mereka untuk langsung terhubung ke sesi Anda.
      </div>
      <div style="display:flex;gap:10px;margin-top:20px;flex-wrap:wrap;justify-content:center;">
        <button class="btn-modal btn-modal-secondary" id="copyPairingCodeBtn"><i class="fas fa-copy"></i> Copy Kode</button>
        <button class="btn-modal btn-modal-primary" id="refreshPairingBtn"><i class="fas fa-sync-alt"></i> Generate Ulang</button>
      </div>
      <div style="margin-top:16px;padding:12px 16px;background:#fef3c7;border-radius:12px;font-size:0.78rem;color:#92400e;line-height:1.5;text-align:center;">
        <i class="fas fa-clock"></i> Kode berlaku <strong id="pairingTimer">10:00</strong>
      </div>
    </div>
    <div class="modal-actions">
      <button class="btn-modal btn-modal-secondary" id="qrPairingCancel"><i class="fas fa-times"></i> Tutup</button>
    </div>
  </div>
</div>

<!-- Modal Input Pairing (Mahasiswa) -->
<div class="modal-overlay" id="pairingInputModal">
  <div class="modal-box" style="max-width:480px; text-align:center;">
    <div style="width:80px;height:80px;margin:0 auto 20px;background:linear-gradient(135deg,#7c3aed,#6d28d9);border-radius:50%;display:flex;align-items:center;justify-content:center;color:white;font-size:2rem;">
      <i class="fas fa-keyboard"></i>
    </div>
    <h3 style="font-size:1.4rem;font-weight:800;color:#0b1a2e;margin-bottom:8px;">Masukkan Kode Pairing</h3>
    <p style="color:#64748b;font-size:0.9rem;margin-bottom:20px;">Minta kode 6 digit dari dosen Anda</p>
    <input type="text" id="pairingCodeInput" maxlength="6" placeholder="ABC123" 
           style="width:100%; padding:18px; border:3px dashed #a78bfa; border-radius:16px; text-align:center; font-size:2rem; font-weight:800; letter-spacing:8px; color:#7c3aed; font-family:'Courier New',monospace; text-transform:uppercase; background:#faf5ff;">
    <div id="pairingError" style="display:none; background:#fef2f2; color:#b91c1c; padding:12px; border-radius:12px; margin-top:14px; font-size:0.85rem; font-weight:600;">
      <i class="fas fa-exclamation-circle"></i> <span id="pairingErrorText"></span>
    </div>
    <div id="pairingSuccess" style="display:none; background:#dcfce7; color:#166534; padding:12px; border-radius:12px; margin-top:14px; font-size:0.85rem; font-weight:600;">
      <i class="fas fa-check-circle"></i> <span id="pairingSuccessText"></span>
    </div>
    <div class="modal-actions" style="justify-content:center;">
      <button class="btn-modal btn-modal-secondary" id="pairingCancelBtn"><i class="fas fa-times"></i> Batal</button>
      <button class="btn-modal btn-modal-purple" id="pairingSubmitBtn"><i class="fas fa-link"></i> Hubungkan</button>
    </div>
  </div>
</div>

<!-- Modal Edit User (Admin) -->
<div class="modal-overlay" id="editUserAdminModal">
  <div class="modal-box">
    <div class="modal-header">
      <h3><i class="fas fa-user-edit"></i> Edit Pengguna</h3>
      <button class="modal-close" id="editUserAdminClose"><i class="fas fa-times"></i></button>
    </div>
    <div class="soal-edit-form">
      <div class="form-row">
        <div class="input-group"><label><i class="fas fa-user"></i> Nama Lengkap</label><input type="text" id="adminEditNama"></div>
        <div class="input-group"><label><i class="fas fa-id-card"></i> <span id="adminEditNimLabel">NIDN / NIM</span></label><input type="text" id="adminEditNim"></div>
      </div>
      <div class="input-group"><label><i class="fas fa-envelope"></i> Email</label><input type="email" id="adminEditEmail"></div>
      <div class="input-group"><label><i class="fas fa-user-tag"></i> Role</label>
        <select id="adminEditRole"><option value="admin">Admin</option><option value="dosen">Dosen</option><option value="mahasiswa">Mahasiswa</option></select>
      </div>
      <div class="input-group"><label><i class="fas fa-lock"></i> Password Baru (opsional)</label><input type="password" id="adminEditPassword" placeholder="Kosongkan jika tidak diubah"></div>
    </div>
    <div class="modal-actions">
      <button class="btn-modal btn-modal-secondary" id="editUserAdminCancel"><i class="fas fa-times"></i> Batal</button>
      <button class="btn-modal btn-modal-purple" id="editUserAdminSave"><i class="fas fa-save"></i> Simpan</button>
    </div>
  </div>
</div>

<!-- Modal Confirm Delete -->
<div class="modal-overlay" id="confirmModal">
  <div class="modal-box" style="text-align:center;">
    <div style="width:80px;height:80px;margin:0 auto 20px;background:linear-gradient(135deg,#fee2e2,#fecaca);border-radius:50%;display:flex;align-items:center;justify-content:center;color:#dc2626;font-size:2.5rem;">
      <i class="fas fa-trash-alt"></i>
    </div>
    <h3 style="font-size:1.4rem; font-weight:700; color:#0b1a2e; margin-bottom:12px;">Hapus Data?</h3>
    <p style="color:#64748b; margin-bottom:8px;">Tindakan ini akan menghapus data secara permanen.</p>
    <span style="background:#fef2f2; color:#b91c1c; padding:12px 20px; border-radius:16px; font-weight:700; display:block; margin:16px 0 24px; border:1px solid #fecaca;" id="confirmTargetName">-</span>
    <div class="modal-actions" style="justify-content:center;">
      <button class="btn-modal btn-modal-secondary" id="confirmCancelBtn"><i class="fas fa-times"></i> Batal</button>
      <button class="btn-modal btn-modal-danger" id="confirmDeleteBtn"><i class="fas fa-trash"></i> Hapus Permanen</button>
    </div>
  </div>
</div>

<!-- Modal OTP -->
<div class="modal-overlay" id="otpModal">
  <div class="modal-box" style="text-align:center;">
    <div style="width:80px;height:80px;margin:0 auto 20px;background:linear-gradient(135deg,#1e4b7c,#3b82f6);border-radius:50%;display:flex;align-items:center;justify-content:center;color:white;font-size:2.5rem;">
      <i class="fas fa-envelope-open-text"></i>
    </div>
    <h3 style="font-size:1.5rem; font-weight:700; color:#0b1a2e; margin-bottom:10px;">Verifikasi Email</h3>
    <p style="color:#64748b; margin-bottom:24px;">Kode OTP dikirim ke:</p>
    <div style="background:#f0f6fe; color:#1e4b7c; padding:10px 20px; border-radius:30px; font-weight:700; display:inline-block; margin-bottom:20px; word-break:break-all;" id="otpEmailTarget">-</div>
    <div class="reset-error" id="otpError"><i class="fas fa-exclamation-circle"></i> <span id="otpErrorText"></span></div>
    <div class="reset-success" id="otpSuccess"><i class="fas fa-check-circle"></i> <span id="otpSuccessText"></span></div>
    <div class="otp-input-group">
      <input type="text" maxlength="1" data-otp-index="0" inputmode="numeric" disabled>
      <input type="text" maxlength="1" data-otp-index="1" inputmode="numeric" disabled>
      <input type="text" maxlength="1" data-otp-index="2" inputmode="numeric" disabled>
      <input type="text" maxlength="1" data-otp-index="3" inputmode="numeric" disabled>
      <input type="text" maxlength="1" data-otp-index="4" inputmode="numeric" disabled>
      <input type="text" maxlength="1" data-otp-index="5" inputmode="numeric" disabled>
    </div>
    <div class="reset-actions">
      <button class="btn-reset btn-reset-secondary" id="otpCancelBtn" disabled><i class="fas fa-times"></i> Batal</button>
      <button class="btn-reset btn-reset-primary" id="otpVerifyBtn" disabled><i class="fas fa-check"></i> Verifikasi</button>
    </div>
    <div class="reset-timer">Kode berlaku: <strong id="otpTimer">05:00</strong></div>
  </div>
</div>

<!-- Modal Reset Password -->
<div class="modal-overlay" id="resetModal">
  <div class="modal-box">
    <div id="resetStep1">
      <div class="reset-icon"><i class="fas fa-key"></i></div>
      <h3 style="text-align:center; font-size:1.5rem; margin-bottom:10px;">Reset Password</h3>
      <p style="text-align:center; color:#64748b; margin-bottom:24px;">Masukkan NIM/NIDN atau email terdaftar.</p>
      <div class="reset-error" id="resetError1"><i class="fas fa-exclamation-circle"></i> <span id="resetErrorText1"></span></div>
      <div class="input-group" style="margin-bottom:20px;">
        <label><i class="fas fa-id-card"></i> NIM / NIDN / Email</label>
        <input type="text" id="resetInput" placeholder="Contoh: 2101010101 / NIDN001 / nama@student.unm.ac.id">
      </div>
      <div class="reset-actions">
        <button class="btn-reset btn-reset-secondary" id="resetCancelBtn1"><i class="fas fa-times"></i> Batal</button>
        <button class="btn-reset btn-reset-primary" id="resetSendOtpBtn"><i class="fas fa-paper-plane"></i> Kirim OTP</button>
      </div>
    </div>
    <div id="resetStep2" class="hidden">
      <div class="reset-icon email"><i class="fas fa-envelope-open-text"></i></div>
      <h3 style="text-align:center; font-size:1.5rem; margin-bottom:10px;">Verifikasi OTP</h3>
      <p style="text-align:center; color:#64748b; margin-bottom:24px;">Kode OTP dikirim ke:</p>
      <div style="background:#f0f6fe; color:#1e4b7c; padding:12px 20px; border-radius:30px; font-weight:700; text-align:center; margin-bottom:20px; word-break:break-all;" id="resetEmailDisplay">-</div>
      <div class="reset-error" id="resetError2"><i class="fas fa-exclamation-circle"></i> <span id="resetErrorText2"></span></div>
      <div class="reset-success" id="resetSuccess2"><i class="fas fa-check-circle"></i> <span id="resetSuccessText2"></span></div>
      <div class="otp-input-group">
        <input type="text" maxlength="1" data-reset-otp="0" inputmode="numeric">
        <input type="text" maxlength="1" data-reset-otp="1" inputmode="numeric">
        <input type="text" maxlength="1" data-reset-otp="2" inputmode="numeric">
        <input type="text" maxlength="1" data-reset-otp="3" inputmode="numeric">
        <input type="text" maxlength="1" data-reset-otp="4" inputmode="numeric">
        <input type="text" maxlength="1" data-reset-otp="5" inputmode="numeric">
      </div>
      <div class="reset-timer">Kode berlaku: <strong id="resetOtpTimer">05:00</strong></div>
      <div class="reset-actions" style="margin-top:16px;">
        <button class="btn-reset btn-reset-secondary" id="resetBackBtn"><i class="fas fa-arrow-left"></i> Kembali</button>
        <button class="btn-reset btn-reset-primary" id="resetVerifyOtpBtn"><i class="fas fa-check"></i> Verifikasi</button>
      </div>
    </div>
    <div id="resetStep3" class="hidden">
      <div class="reset-icon success"><i class="fas fa-lock-open"></i></div>
      <h3 style="text-align:center; font-size:1.5rem; margin-bottom:10px;">Password Baru</h3>
      <p style="text-align:center; color:#64748b; margin-bottom:24px;">Buat password baru.</p>
      <div class="reset-error" id="resetError3"><i class="fas fa-exclamation-circle"></i> <span id="resetErrorText3"></span></div>
      <div class="input-group" style="margin-bottom:16px;">
        <label><i class="fas fa-lock"></i> Password Baru</label>
        <input type="password" id="resetNewPassword" placeholder="Minimal 6 karakter">
        <div class="password-strength"><div class="password-strength-fill" id="passwordStrengthFill"></div></div>
        <div class="password-strength-text" id="passwordStrengthText"></div>
      </div>
      <div class="input-group" style="margin-bottom:20px;">
        <label><i class="fas fa-lock"></i> Konfirmasi Password</label>
        <input type="password" id="resetNewPasswordConfirm" placeholder="Ulangi password">
      </div>
      <div class="reset-actions">
        <button class="btn-reset btn-reset-secondary" id="resetCancelBtn3"><i class="fas fa-times"></i> Batal</button>
        <button class="btn-reset btn-reset-success" id="resetSavePasswordBtn"><i class="fas fa-save"></i> Simpan</button>
      </div>
    </div>
  </div>
</div>

<!-- Toast -->
<div class="toast" id="toast">
  <i class="fas fa-check-circle" id="toastIcon"></i>
  <div>
    <div class="toast-title" id="toastTitle">Berhasil</div>
    <div class="toast-message" id="toastMessage">Pesan</div>
  </div>
</div>

<script>
  (function() {
    // ============================================================
    // KONFIGURASI EMAILJS
    // ============================================================
    const EMAILJS_PUBLIC_KEY = 'GANTI_DENGAN_PUBLIC_KEY_ANDA';
    const EMAILJS_SERVICE_ID = 'GANTI_DENGAN_SERVICE_ID_ANDA';
    const EMAILJS_TEMPLATE_ID = 'GANTI_DENGAN_TEMPLATE_ID_ANDA';
    const DEMO_MODE = EMAILJS_PUBLIC_KEY.includes('GANTI');
    if (!DEMO_MODE && typeof emailjs !== 'undefined') emailjs.init(EMAILJS_PUBLIC_KEY);

    // ============================================================
    // DEVICE INFO
    // ============================================================
    const DEVICE_ID = window.DEVICE_ID || ('dev_' + Date.now());
    const DEVICE_INFO = window.DEVICE_INFO || { userAgent: navigator.userAgent, isMobile: false };

    // ============================================================
    // FIREBASE
    // ============================================================
    let FIREBASE_READY = false;
    let db = null;
    let FB = null;
    let FIREBASE_INIT_DONE = false;

    function showLoading(show) {
      const el = document.getElementById('loadingOverlay');
      if (el) el.classList.toggle('show', show);
    }

    function initFirebase() {
      if (FIREBASE_INIT_DONE) return;
      FIREBASE_INIT_DONE = true;
      FIREBASE_READY = window.FIREBASE_ENABLED === true;
      db = window.FIREBASE_DB;
      FB = window.FIREBASE_FUNCS;
      updateRealtimeBadge();
      if (FIREBASE_READY && db && FB) {
        console.log('✅ Firebase siap — realtime multi-device monitoring aktif');
        showLoading(true);
        setupRealtimeListeners();
        setupLiveActivityListener();
        Promise.all([fetchAllUsers(), fetchAllStudents(), fetchAllSoal()])
          .then(() => { seedDefaultAdmin(); showLoading(false); })
          .catch(() => showLoading(false));
      } else {
        seedDefaultAdmin();
      }
    }
    window.addEventListener('firebase-ready', initFirebase);
    setTimeout(initFirebase, 2000);

    function updateRealtimeBadge() {
      const badge = document.getElementById('realtimeBadge');
      const text = document.getElementById('realtimeText');
      if (!badge || !text) return;
      if (FIREBASE_READY) { badge.className = 'realtime-badge online'; text.textContent = 'Realtime Multi-Device ON'; }
      else { badge.className = 'realtime-badge offline'; text.textContent = 'Mode Lokal'; }
    }

    // ============================================================
    // DATA CACHE
    // ============================================================
    let STUDENTS_DATA = [];
    let USERS_DATA = [];
    let SOAL_DATA = [];
    let ACTIVITIES_DATA = [];
    let DEVICES_DATA = [];
    let NOTIFICATIONS_DATA = [];
    let OFFLINE_QUEUE = [];
    let MY_DEVICE_DOC = null;
    let PAIRING_CODE = null;
    let PAIRING_TIMER = null;
    let PAIRING_SECONDS = 600;

    const LS_USERS = 'portal_ujian_users_v7';
    const LS_STUDENTS = 'portal_ujian_students_v7';
    const LS_SOAL = 'portal_ujian_soal_v7';
    const LS_ACTIVITIES = 'portal_ujian_activities_v7';
    const ADMIN_SEEDED_KEY = 'portal_ujian_admin_seeded_v7';

    function getLocal(key) { try { const raw = localStorage.getItem(key); return raw ? JSON.parse(raw) : []; } catch(e) { return []; } }
    function setLocal(key, val) { try { localStorage.setItem(key, JSON.stringify(val)); } catch(e) {} }

    // ============================================================
    // SEED ADMIN
    // ============================================================
    async function seedDefaultAdmin() {
      if (localStorage.getItem(ADMIN_SEEDED_KEY) === 'true') { USERS_DATA = getLocal(LS_USERS); return; }
      const users = getLocal(LS_USERS);
      if (!users.some(u => u.nim === 'RikardusFeribertusNikat' && u.role === 'admin')) {
        users.push({
          nama: 'Rikardus Feribertus Nikat', nim: 'RikardusFeribertusNikat',
          email: 'rikardus.nikat@unm.ac.id', password: 'admin123',
          role: 'admin', verified: true, jabatan: 'Super Admin',
          createdAt: new Date().toISOString()
        });
        setLocal(LS_USERS, users);
        USERS_DATA = users;
        if (FIREBASE_READY && db && FB) {
          try {
            await FB.setDoc(FB.doc(db, 'users', 'RikardusFeribertusNikat_admin'), {
              ...users[users.length-1], updatedAt: FB.serverTimestamp()
            });
          } catch(e) {}
        }
        localStorage.setItem(ADMIN_SEEDED_KEY, 'true');
      }
      USERS_DATA = getLocal(LS_USERS);
    }

    // ============================================================
    // RUBRIK
    // ============================================================
    const RUBRIK_DATA = [
      { id: 'skills', title: 'Knowledge of Skills and Algorithms', subtitle: 'Pengetahuan keterampilan & algoritma',
        deskriptor: {
          0: "Tidak Menjawab",
          1: "Tidak ada urutan langkah yang relevan; tidak dapat mengidentifikasi algoritma atau keterampilan yang dibutuhkan dalam perancangan media fisika digital.",
          2: "Menuliskan beberapa langkah namun tidak sistematis; urutan tidak logis atau tidak terhubung dengan tujuan perancangan media fisika.",
          3: "Menuliskan urutan langkah yang cukup sistematis; sebagian besar langkah terhubung dengan tujuan perancangan media fisika digital namun masih ada yang kurang lengkap.",
          4: "Menuliskan urutan langkah yang sistematis dan logis; hampir semua langkah terhubung dengan tujuan perancangan media fisika digital; penjelasan memadai.",
          5: "Menuliskan urutan langkah yang sangat sistematis, logis, dan komprehensif; semua langkah terhubung dengan tujuan perancangan media fisika digital; penjelasan sangat lengkap dan presisi."
        } },
      { id: 'techniques', title: 'Knowledge of Techniques and Methods', subtitle: 'Pengetahuan teknik & metode',
        deskriptor: {
          0: "Tidak Menjawab",
          1: "Tidak ada teknik atau metode yang relevan diidentifikasi; tidak menjelaskan bagaimana menerapkan teknik dalam perancangan media.",
          2: "Menuliskan teknik/metode namun tidak spesifik; tidak menjelaskan bagaimana penerapannya dalam konteks perancangan media fisika digital.",
          3: "Menuliskan teknik/metode yang cukup spesifik dan relevan; penjelasan penerapan dalam konteks perancangan media cukup memadai namun belum komprehensif.",
          4: "Menuliskan teknik/metode dengan jelas dan spesifik; menjelaskan penerapan dalam konteks perancangan media secara memadai; hanya ada sedikit kekurangan.",
          5: "Menuliskan teknik/metode dengan sangat jelas, spesifik, dan komprehensif; menjelaskan penerapan dalam perancangan media fisika digital secara sangat lengkap dan tepat."
        } },
      { id: 'criteria', title: 'Knowledge of Criteria for Determining When to Use Procedures', subtitle: 'Pengetahuan kriteria prosedur',
        deskriptor: {
          0: "Tidak Menjawab",
          1: "Tidak ada kriteria pemilihan prosedur yang relevan; tidak menjelaskan kapan dan mengapa suatu prosedur digunakan.",
          2: "Menuliskan beberapa kriteria namun tidak menjelaskan kapan dan mengapa prosedur tersebut dipilih untuk perancangan media tertentu.",
          3: "Menuliskan kriteria yang cukup relevan dan lengkap; mampu menjelaskan kapan dan mengapa prosedur digunakan, meskipun belum sempurna.",
          4: "Menuliskan kriteria yang jelas dan relevan; menjelaskan kapan dan mengapa prosedur digunakan dengan baik; hanya ada sedikit kekurangan.",
          5: "Menuliskan kriteria yang sangat jelas, relevan, dan lengkap; menjelaskan kapan dan mengapa prosedur dipilih dengan sangat tepat dan didukung alasan yang kuat."
        } }
    ];

    const DEFAULT_SOAL = [
      { id: 1, judul: "Soal 1 — Energi dan Perubahannya (SMA Kelas X)",
        deskripsi: "Sebuah video pembelajaran Energi dan Perubahannya untuk siswa SMA kelas X telah selesai dikembangkan. Hasil evaluasi menunjukkan:\n\n• ahli materi menilai konsep fisika sudah benar tetapi contoh penerapan masih kurang;\n• ahli media menemukan ukuran teks terlalu kecil;\n• siswa mengatakan video menarik tetapi terlalu cepat;\n• beberapa siswa kesulitan memahami grafik energi;\n• sebagian siswa masih mengalami miskonsepsi tentang perubahan energi.",
        pertanyaanA: "Susunlah prosedur evaluasi kelayakan video pembelajaran secara sistematis mulai dari tahap persiapan sampai pengambilan keputusan akhir.",
        pertanyaanB: "Berdasarkan hasil evaluasi pada kasus tersebut, buatlah rencana revisi video.",
        pertanyaanC: "Pilih dan jelaskan evaluasi lanjutan yang paling tepat (A: Expert Review, B: Small Group, C: Field Test)." },
      { id: 2, judul: "Soal 2 — Gelombang dan Bunyi (SMP Kelas VIII)",
        deskripsi: "Sebagai guru Fisika, Anda diminta mengembangkan video pembelajaran untuk siswa SMP kelas VIII dengan topik Gelombang dan Bunyi, mencakup frekuensi, amplitudo, panjang gelombang, serta hubungan getaran dan bunyi.",
        pertanyaanA: "Susunlah urutan langkah teknis pengembangan video pembelajaran mulai dari persiapan konten sampai video siap digunakan.",
        pertanyaanB: "Susunlah prosedur verifikasi dan validasi konten fisika sebelum video diberikan kepada siswa.",
        pertanyaanC: "Pilih salah satu alat pengembangan video dan jelaskan alasan pemilihan berdasarkan karakteristik konten." },
      { id: 3, judul: "Soal 3 — Hukum Newton (SMA Kelas X)",
        deskripsi: "Seorang guru fisika SMA kelas X mengalami kesulitan menjelaskan materi Hukum Newton tentang Gerak. Guru ingin mengembangkan video pembelajaran berdurasi 8–12 menit yang memadukan penjelasan konsep, demonstrasi sederhana, animasi, dan visualisasi vektor gaya.",
        pertanyaanA: "Susunlah langkah-langkah sistematis analisis kebutuhan sebelum video dikembangkan.",
        pertanyaanB: "Buatlah kerangka desain instruksional video pembelajaran Hukum Newton.",
        pertanyaanC: "Pilih antara Desain A (penjelasan guru di depan kamera) dan Desain B (memadukan demonstrasi eksperimen, animasi gerak, visualisasi vektor gaya, narasi, pertanyaan interaktif). Jelaskan alasan Anda." }
    ];

    // ============================================================
    // CLOUD FUNCTIONS
    // ============================================================
    async function fetchAllUsers() {
      if (!FIREBASE_READY || !db || !FB) { USERS_DATA = getLocal(LS_USERS); return USERS_DATA; }
      try {
        const snap = await FB.getDocs(FB.collection(db, 'users'));
        const users = [];
        snap.forEach(d => users.push({ ...d.data(), _docId: d.id }));
        if (users.length > 0) {
          USERS_DATA = users;
          setLocal(LS_USERS, users.map(u => { const c = {...u}; delete c._docId; return c; }));
        } else USERS_DATA = getLocal(LS_USERS);
      } catch (e) { USERS_DATA = getLocal(LS_USERS); }
      return USERS_DATA;
    }

    async function fetchAllStudents() {
      if (!FIREBASE_READY || !db || !FB) { STUDENTS_DATA = getLocal(LS_STUDENTS); return STUDENTS_DATA; }
      try {
        const snap = await FB.getDocs(FB.collection(db, 'students'));
        const students = [];
        snap.forEach(d => students.push({ ...d.data(), _docId: d.id }));
        if (students.length > 0) {
          STUDENTS_DATA = students;
          setLocal(LS_STUDENTS, students.map(s => { const c = {...s}; delete c._docId; return c; }));
        } else STUDENTS_DATA = getLocal(LS_STUDENTS);
      } catch (e) { STUDENTS_DATA = getLocal(LS_STUDENTS); }
      return STUDENTS_DATA;
    }

    async function fetchAllSoal() {
      if (!FIREBASE_READY || !db || !FB) { SOAL_DATA = getLocal(LS_SOAL); if (SOAL_DATA.length === 0) SOAL_DATA = [...DEFAULT_SOAL]; return SOAL_DATA; }
      try {
        const snap = await FB.getDocs(FB.collection(db, 'soal'));
        const soalList = [];
        snap.forEach(d => soalList.push({ ...d.data(), _docId: d.id }));
        if (soalList.length > 0) {
          SOAL_DATA = soalList.sort((a,b) => a.id - b.id);
          setLocal(LS_SOAL, SOAL_DATA.map(s => { const c = {...s}; delete c._docId; return c; }));
        } else {
          for (const s of DEFAULT_SOAL) { try { await FB.setDoc(FB.doc(db, 'soal', `soal_${s.id}`), s); } catch(e) {} }
          SOAL_DATA = [...DEFAULT_SOAL];
        }
      } catch (e) {
        SOAL_DATA = getLocal(LS_SOAL);
        if (SOAL_DATA.length === 0) SOAL_DATA = [...DEFAULT_SOAL];
      }
      return SOAL_DATA;
    }

    // ============================================================
    // REALTIME LISTENERS
    // ============================================================
    let lastStudentCount = 0;
    function setupRealtimeListeners() {
      if (!FIREBASE_READY || !db || !FB) return;

      try {
        FB.onSnapshot(FB.collection(db, 'users'), (snap) => {
          const users = [];
          snap.forEach(d => users.push({ ...d.data(), _docId: d.id }));
          if (users.length > 0) {
            USERS_DATA = users;
            setLocal(LS_USERS, users.map(u => { const c = {...u}; delete c._docId; return c; }));
            if (currentRole === 'admin') renderAdminUserList();
          }
        });
      } catch (e) {}

      try {
        FB.onSnapshot(FB.collection(db, 'students'), (snap) => {
          const students = [];
          snap.forEach(d => students.push({ ...d.data(), _docId: d.id }));
          STUDENTS_DATA = students;
          setLocal(LS_STUDENTS, students.map(s => { const c = {...s}; delete c._docId; return c; }));
          if (lastStudentCount > 0 && students.length > lastStudentCount) {
            showToast('👥 Peserta Baru', 'Ada mahasiswa baru mendaftar', 'info', 4000);
          }
          lastStudentCount = students.length;
          if (currentRole === 'dosen' && document.getElementById('dosenSection').style.display === 'block') {
            renderStudentList(); renderMahasiswaSelector(); renderDatabase();
          }
          if (currentRole === 'admin') renderAdminUserList();
        });
      } catch (e) {}

      try {
        FB.onSnapshot(FB.collection(db, 'soal'), (snap) => {
          const soalList = [];
          snap.forEach(d => soalList.push({ ...d.data(), _docId: d.id }));
          if (soalList.length > 0) {
            SOAL_DATA = soalList.sort((a,b) => a.id - b.id);
            setLocal(LS_SOAL, SOAL_DATA.map(s => { const c = {...s}; delete c._docId; return c; }));
            const jml = SOAL_DATA.length;
            if (document.getElementById('pretestJumlahSoal')) document.getElementById('pretestJumlahSoal').textContent = jml;
            if (document.getElementById('posttestJumlahSoal')) document.getElementById('posttestJumlahSoal').textContent = jml;
            if (currentRole === 'mahasiswa' && document.getElementById('examSection').style.display === 'block' && currentUser) {
              renderExamSoal(); loadPreviousAnswers(); pindahSoal(currentSoal);
            }
            if (currentRole === 'admin') updateAdminStats();
          }
        });
      } catch (e) {}

      // Devices listener
      try {
        FB.onSnapshot(FB.collection(db, 'devices'), (snap) => {
          const devices = [];
          snap.forEach(d => {
            const data = d.data();
            let ts = data.lastPing;
            if (typeof ts === 'string') ts = new Date(ts).getTime();
            else if (data.serverPing?.seconds) ts = data.serverPing.seconds * 1000;
            else ts = Date.now();
            const isOnline = (Date.now() - ts) < 120000;
            devices.push({ ...data, _docId: d.id, _isOnline: isOnline, _lastTs: ts });
          });
          DEVICES_DATA = devices;
          if (currentRole === 'dosen' && document.getElementById('dosenSection')?.style.display === 'block') {
            renderDeviceList();
          }
        });
      } catch (e) {}
    }

    // ============================================================
    // LIVE ACTIVITY LISTENER
    // ============================================================
    function setupLiveActivityListener() {
      if (!FIREBASE_READY || !db || !FB) return;
      try {
        FB.onSnapshot(FB.collection(db, 'live_activities'), (snap) => {
          const activities = [];
          snap.forEach(d => activities.push({ ...d.data(), _docId: d.id }));
          ACTIVITIES_DATA = activities;
          setLocal(LS_ACTIVITIES, activities.map(a => { const c = {...a}; delete c._docId; return c; }));
          if (currentRole === 'dosen') {
            renderLiveMonitor();
            renderStudentList();
          }
        });
      } catch (e) { console.warn('Live activity listener error:', e); }
    }

    async function saveActivity(activity) {
      const key = activity.studentId + '_' + activity.sesi;
      const idx = ACTIVITIES_DATA.findIndex(a => a._docId === key);
      if (idx >= 0) ACTIVITIES_DATA[idx] = { ...activity, _docId: key };
      else ACTIVITIES_DATA.push({ ...activity, _docId: key });
      setLocal(LS_ACTIVITIES, ACTIVITIES_DATA.map(a => { const c = {...a}; delete c._docId; return c; }));
      if (FIREBASE_READY && db && FB) {
        try {
          await FB.setDoc(FB.doc(db, 'live_activities', key), { ...activity, lastUpdate: FB.serverTimestamp() });
        } catch (e) {}
      }
    }

    // ============================================================
    // SYNC FUNCTIONS
    // ============================================================
    async function syncSaveUser(user) {
      const local = getLocal(LS_USERS);
      const docId = user.nim + '_' + user.role;
      const idx = local.findIndex(u => (u.nim + '_' + u.role) === docId);
      if (idx >= 0) local[idx] = user; else local.push(user);
      setLocal(LS_USERS, local);
      const idxMem = USERS_DATA.findIndex(u => (u.nim + '_' + u.role) === docId);
      if (idxMem >= 0) USERS_DATA[idxMem] = user; else USERS_DATA.push(user);
      if (FIREBASE_READY && db && FB) {
        try { await FB.setDoc(FB.doc(db, 'users', docId), { ...user, updatedAt: FB.serverTimestamp() }); return true; } catch (e) { return false; }
      }
      return false;
    }

    async function syncDeleteUser(user) {
      const docId = user.nim + '_' + user.role;
      setLocal(LS_USERS, getLocal(LS_USERS).filter(u => (u.nim + '_' + u.role) !== docId));
      USERS_DATA = USERS_DATA.filter(u => (u.nim + '_' + u.role) !== docId);
      if (FIREBASE_READY && db && FB) { try { await FB.deleteDoc(FB.doc(db, 'users', docId)); return true; } catch (e) { return false; } }
      return false;
    }

    async function syncSaveStudent(student) {
      const local = getLocal(LS_STUDENTS);
      const idx = local.findIndex(s => s.id === student.id);
      if (idx >= 0) local[idx] = student; else local.push(student);
      setLocal(LS_STUDENTS, local);
      const idxMem = STUDENTS_DATA.findIndex(s => s.id === student.id);
      if (idxMem >= 0) STUDENTS_DATA[idxMem] = student; else STUDENTS_DATA.push(student);
      if (FIREBASE_READY && db && FB) { try { await FB.setDoc(FB.doc(db, 'students', student.id), { ...student, updatedAt: FB.serverTimestamp() }); return true; } catch (e) { return false; } }
      return false;
    }

    async function syncDeleteStudent(id) {
      setLocal(LS_STUDENTS, getLocal(LS_STUDENTS).filter(s => s.id !== id));
      STUDENTS_DATA = STUDENTS_DATA.filter(s => s.id !== id);
      if (FIREBASE_READY && db && FB) { try { await FB.deleteDoc(FB.doc(db, 'students', id)); return true; } catch (e) { return false; } }
      return false;
    }

    async function syncSaveSoal(soalList) {
      setLocal(LS_SOAL, soalList);
      SOAL_DATA = [...soalList];
      if (FIREBASE_READY && db && FB) {
        try {
          for (const s of soalList) { await FB.setDoc(FB.doc(db, 'soal', `soal_${s.id}`), s); }
          return true;
        } catch (e) { return false; }
      }
      return false;
    }

    async function syncDeleteSoal(id) {
      const list = SOAL_DATA.filter(s => s.id !== id);
      setLocal(LS_SOAL, list);
      SOAL_DATA = list;
      if (FIREBASE_READY && db && FB) { try { await FB.deleteDoc(FB.doc(db, 'soal', `soal_${id}`)); return true; } catch (e) { return false; } }
      return false;
    }

    // ============================================================
    // STATE
    // ============================================================
    const $ = (id) => document.getElementById(id);
    let currentRole = 'mahasiswa';
    let currentUser = null;
    let currentSoal = 1;
    let selectedStudentId = null;
    let currentDosenSoal = 1;
    let currentSesi = null;
    let pendingSesi = null;
    let otpCode = '', otpTimerInterval = null, otpSecondsLeft = 300, pendingUser = null;
    let resetUser = null, resetOtpCode = '', resetOtpTimerInterval = null, resetOtpSecondsLeft = 300;
    let currentRubrikSkor = { skills: null, techniques: null, criteria: null };
    let editingSoalId = null;
    let editingUserId = null;
    let answerSaveTimeout = null;
    let conflictShown = false;

    function showToast(title, message, type = 'success', duration = 3500) {
      const toast = $('toast');
      toast.className = 'toast ' + type;
      $('toastTitle').textContent = title;
      $('toastMessage').textContent = message;
      $('toastIcon').className = type === 'error' ? 'fas fa-exclamation-circle' : type === 'info' ? 'fas fa-info-circle' : 'fas fa-check-circle';
      setTimeout(() => toast.classList.add('show'), 50);
      setTimeout(() => toast.classList.remove('show'), duration);
    }

    function getStudentSesi(student, sesi) {
      if (!student.sesi) student.sesi = { pretest: {}, posttest: {} };
      if (!student.sesi[sesi]) student.sesi[sesi] = { status: 'belum', progress: 0, nilaiSoal: {}, rubrikSkor: {}, feedback: {}, emailSent: false, jawaban: {} };
      const s = student.sesi[sesi];
      if (!s.nilaiSoal) s.nilaiSoal = {};
      if (!s.feedback) s.feedback = {};
      if (!s.rubrikSkor) s.rubrikSkor = {};
      if (!s.jawaban) s.jawaban = {};
      return s;
    }

    function escapeHtml(str) {
      if (!str) return '';
      return str.replace(/[&<>"']/g, m => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[m]));
    }

    function formatTimeAgo(ts) {
      if (!ts) return 'baru saja';
      let ms;
      if (typeof ts === 'object' && ts.seconds) ms = ts.seconds * 1000;
      else if (typeof ts === 'number') ms = ts;
      else ms = new Date(ts).getTime();
      const diff = Date.now() - ms;
      if (diff < 5000) return 'baru saja';
      if (diff < 60000) return `${Math.floor(diff/1000)}d lalu`;
      if (diff < 3600000) return `${Math.floor(diff/60000)}m lalu`;
      return `${Math.floor(diff/3600000)}j lalu`;
    }

    // ============================================================
    // DEVICE FUNCTIONS
    // ============================================================
    function getDeviceType() {
      const ua = DEVICE_INFO.userAgent;
      if (/iPad|Tablet/i.test(ua)) return 'tablet';
      if (/Mobi|Android|iPhone/i.test(ua)) return 'mobile';
      return 'desktop';
    }

    function getDeviceName() {
      const type = getDeviceType();
      const ua = DEVICE_INFO.userAgent;
      let browser = 'Browser';
      if (/Chrome/i.test(ua) && !/Edg/i.test(ua)) browser = 'Chrome';
      else if (/Safari/i.test(ua) && !/Chrome/i.test(ua)) browser = 'Safari';
      else if (/Firefox/i.test(ua)) browser = 'Firefox';
      else if (/Edg/i.test(ua)) browser = 'Edge';
      let os = 'OS';
      if (/Windows/i.test(ua)) os = 'Windows';
      else if (/Mac OS/i.test(ua)) os = 'macOS';
      else if (/Android/i.test(ua)) os = 'Android';
      else if (/iPhone|iPad/i.test(ua)) os = 'iOS';
      else if (/Linux/i.test(ua)) os = 'Linux';
      const typeLabel = type === 'mobile' ? '📱 HP' : type === 'tablet' ? '📱 Tablet' : '💻 Laptop';
      return `${typeLabel} · ${browser} · ${os}`;
    }

    async function registerDevice() {
      if (!FIREBASE_READY || !db || !FB || !currentUser) return;
      const deviceData = {
        deviceId: DEVICE_ID,
        userId: currentUser.nim,
        userName: currentUser.nama,
        userRole: currentUser.role,
        deviceType: getDeviceType(),
        deviceName: getDeviceName(),
        userAgent: DEVICE_INFO.userAgent.slice(0, 200),
        screen: DEVICE_INFO.screen || '',
        isMobile: DEVICE_INFO.isMobile || false,
        online: true,
        lastPing: new Date().toISOString(),
        session: currentSesi || null,
        currentSoal: currentSoal || null,
        loginAt: new Date().toISOString()
      };
      try {
        await FB.setDoc(FB.doc(db, 'devices', DEVICE_ID), { ...deviceData, serverPing: FB.serverTimestamp() });
        MY_DEVICE_DOC = DEVICE_ID;
        console.log('✅ Device registered:', DEVICE_ID);
      } catch (e) { console.warn('Device register failed:', e); }
    }

    function startDeviceHeartbeat() {
      setInterval(async () => {
        if (!FIREBASE_READY || !db || !FB || !currentUser || !MY_DEVICE_DOC) return;
        try {
          await FB.updateDoc(FB.doc(db, 'devices', MY_DEVICE_DOC), {
            online: true, lastPing: new Date().toISOString(),
            serverPing: FB.serverTimestamp(),
            session: currentSesi || null, currentSoal: currentSoal || null
          });
        } catch (e) {}
      }, 20000);
    }

    window.addEventListener('beforeunload', async () => {
      if (FIREBASE_READY && db && FB && MY_DEVICE_DOC) {
        try {
          await FB.updateDoc(FB.doc(db, 'devices', MY_DEVICE_DOC), { online: false, lastPing: new Date().toISOString() });
        } catch (e) {}
      }
    });

    function renderDeviceList() {
      const list = $('deviceList');
      if (!list) return;
      const mahasiswaDevices = DEVICES_DATA.filter(d => d.userRole === 'mahasiswa' && d._isOnline);
      const online = mahasiswaDevices.length;
      const mobile = mahasiswaDevices.filter(d => d.deviceType === 'mobile').length;
      const desktop = mahasiswaDevices.filter(d => d.deviceType === 'desktop').length;
      const uniqueStudents = new Set(mahasiswaDevices.map(d => d.userId)).size;
      if ($('deviceCountOnline')) $('deviceCountOnline').textContent = online;
      if ($('deviceCountStudents')) $('deviceCountStudents').textContent = uniqueStudents;
      if ($('deviceCountMobile')) $('deviceCountMobile').textContent = mobile;
      if ($('deviceCountDesktop')) $('deviceCountDesktop').textContent = desktop;
      if (mahasiswaDevices.length === 0) {
        list.innerHTML = `<div class="notif-empty"><i class="fas fa-satellite-dish"></i><div>Menunggu device terhubung...</div><div style="font-size:0.75rem;margin-top:6px;color:#cbd5e1;">Device mahasiswa akan muncul saat mereka login</div></div>`;
        return;
      }
      mahasiswaDevices.sort((a, b) => b._lastTs - a._lastTs);
      list.innerHTML = '';
      mahasiswaDevices.forEach(device => {
        const card = document.createElement('div');
        card.className = 'device-card';
        const typeIcon = device.deviceType === 'mobile' ? 'fa-mobile-alt' : device.deviceType === 'tablet' ? 'fa-tablet-alt' : 'fa-laptop';
        const typeClass = device.deviceType === 'mobile' ? 'mobile' : device.deviceType === 'tablet' ? 'tablet' : 'desktop';
        const lastPingAgo = Math.floor((Date.now() - device._lastTs) / 1000);
        const pingText = lastPingAgo < 30 ? 'Baru saja' : lastPingAgo < 60 ? `${lastPingAgo}d lalu` : `${Math.floor(lastPingAgo/60)}m lalu`;
        const isWriting = device.session && device.currentSoal;
        const badgeClass = isWriting ? 'active' : 'idle';
        const badgeText = isWriting ? `✏️ ${device.currentSoal}` : '👀 Idle';
        card.innerHTML = `
          <div class="device-icon ${typeClass}"><i class="fas ${typeIcon}"></i><div class="online-indicator"></div></div>
          <div class="device-info">
            <div class="device-name">${device.userName} <span style="font-size:0.7rem;color:#94a3b8;font-weight:500;">(${device.userId})</span></div>
            <div class="device-meta">
              <span><i class="fas fa-info-circle"></i> ${device.deviceName}</span>
              <span><i class="fas fa-clock"></i> ${pingText}</span>
              ${device.session ? `<span><i class="fas fa-layer-group"></i> ${device.session === 'pretest' ? 'Pre-Test' : 'Post-Test'}</span>` : ''}
            </div>
          </div>
          <span class="device-badge ${badgeClass}">${badgeText}</span>
        `;
        card.addEventListener('click', () => {
          const student = STUDENTS_DATA.find(s => s.nim === device.userId);
          if (student) {
            selectedStudentId = student.id;
            document.querySelectorAll('.dashboard-tab').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.dashboard-tab-content').forEach(c => c.classList.remove('active'));
            document.querySelector('[data-dashboard-tab="penilaian"]').classList.add('active');
            $('tabPenilaian').classList.add('active');
            renderStudentList(); renderMahasiswaSelector(); renderSoalPenilaianSelector();
            updateSelectedStudentInfo(student); updateRubrikUIForSoal();
          }
        });
        list.appendChild(card);
      });
    }

    // ============================================================
    // LIVE MONITOR
    // ============================================================
    function renderLiveMonitor() {
      const list = $('liveActivityList');
      if (!list) return;
      const now = Date.now();
      const fresh = ACTIVITIES_DATA.filter(a => {
        if (!a.lastUpdate) return true;
        let ts = a.lastUpdate;
        if (typeof ts === 'object' && ts.seconds) ts = ts.seconds * 1000;
        else if (typeof ts === 'string') ts = new Date(ts).getTime();
        return (now - ts) < 5 * 60 * 1000;
      });
      fresh.sort((a, b) => {
        if (a.status === 'writing' && b.status !== 'writing') return -1;
        if (a.status !== 'writing' && b.status === 'writing') return 1;
        let ta = a.lastUpdate?.seconds ? a.lastUpdate.seconds * 1000 : (a.lastUpdate ? new Date(a.lastUpdate).getTime() : 0);
        let tb = b.lastUpdate?.seconds ? b.lastUpdate.seconds * 1000 : (b.lastUpdate ? new Date(b.lastUpdate).getTime() : 0);
        return tb - ta;
      });
      const online = fresh.length;
      const writing = fresh.filter(a => a.status === 'writing').length;
      const submitted = fresh.filter(a => a.status === 'submitted').length;
      const totalChars = fresh.reduce((sum, a) => sum + (a.totalChars || 0), 0);
      if ($('liveOnline')) $('liveOnline').textContent = online;
      if ($('liveWriting')) $('liveWriting').textContent = writing;
      if ($('liveSubmitted')) $('liveSubmitted').textContent = submitted;
      if ($('liveCharacters')) $('liveCharacters').textContent = totalChars.toLocaleString('id-ID');
      if ($('liveCountBadge')) $('liveCountBadge').textContent = `${online} Online`;
      if (fresh.length === 0) {
        list.innerHTML = `<div class="live-empty"><i class="fas fa-satellite-dish"></i><div>Menunggu aktivitas mahasiswa...</div><div style="font-size:0.78rem; margin-top:6px; color:#cbd5e1;">Aktivitas akan muncul realtime saat mahasiswa menulis jawaban</div></div>`;
        return;
      }
      list.innerHTML = '';
      fresh.forEach(act => {
        const item = document.createElement('div');
        item.className = 'live-activity-item ' + (act.status === 'writing' ? 'writing' : act.status === 'submitted' ? 'submitted' : '');
        const isWriting = act.status === 'writing';
        const timeStr = formatTimeAgo(act.lastUpdate);
        item.innerHTML = `
          <div class="act-icon"><i class="fas ${isWriting ? 'fa-pen' : act.status === 'submitted' ? 'fa-check' : 'fa-user'}"></i></div>
          <div class="act-info">
            <div class="act-name">${act.nama || 'Mahasiswa'}${isWriting ? '<span class="writing-indicator"><span class="pulse-dot"></span> MENULIS</span>' : ''}</div>
            <div class="act-detail">${act.sesi === 'pretest' ? '📝 Pre-Test' : '📝 Post-Test'} · ${isWriting ? `Menulis ${act.currentSoal || ''} (${act.totalChars || 0} karakter)` : act.status === 'submitted' ? 'Mengirim jawaban' : 'Aktif'}</div>
          </div>
          <div class="act-time">${timeStr}</div>
        `;
        item.addEventListener('click', () => {
          const student = STUDENTS_DATA.find(s => s.id === act.studentId || s.nim === act.studentId);
          if (student) {
            selectedStudentId = student.id;
            document.querySelectorAll('.dashboard-tab').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.dashboard-tab-content').forEach(c => c.classList.remove('active'));
            document.querySelector('[data-dashboard-tab="penilaian"]').classList.add('active');
            $('tabPenilaian').classList.add('active');
            currentDosenSoal = act.currentSoalId || 1;
            renderStudentList(); renderMahasiswaSelector(); renderSoalPenilaianSelector();
            updateSelectedStudentInfo(student); updateRubrikUIForSoal();
          }
        });
        list.appendChild(item);
      });
    }

    // ============================================================
    // QR PAIRING
    // ============================================================
    function generatePairingCode() {
      const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
      let code = '';
      for (let i = 0; i < 6; i++) code += chars[Math.floor(Math.random() * chars.length)];
      return code;
    }

    async function createPairingSession() {
      if (!FIREBASE_READY || !db || !FB || !currentUser) return null;
      const code = generatePairingCode();
      const expiresAt = new Date(Date.now() + 10 * 60 * 1000).toISOString();
      try {
        await FB.setDoc(FB.doc(db, 'pairing_sessions', code), {
          code, dosenId: currentUser.nim, dosenName: currentUser.nama,
          createdAt: new Date().toISOString(), expiresAt, active: true,
          serverCreated: FB.serverTimestamp()
        });
        return code;
      } catch (e) { console.error('Pairing create failed:', e); return null; }
    }

    function renderQRCode(code) {
      const box = $('qrCodeBox');
      if (!box) return;
      const pairUrl = `${window.location.origin}${window.location.pathname}?pair=${code}`;
      const qrUrl = `https://api.qrserver.com/v1/create-qr-code/?size=240x240&data=${encodeURIComponent(pairUrl)}&bgcolor=ffffff&color=7c3aed&margin=10`;
      box.innerHTML = `<img src="${qrUrl}" alt="QR Code" style="width:240px;height:240px;display:block;border-radius:8px;">`;
    }

    function startPairingTimer() {
      PAIRING_SECONDS = 600;
      clearInterval(PAIRING_TIMER);
      const update = () => {
        const m = String(Math.floor(PAIRING_SECONDS / 60)).padStart(2, '0');
        const s = String(PAIRING_SECONDS % 60).padStart(2, '0');
        const el = $('pairingTimer');
        if (el) { el.textContent = `${m}:${s}`; el.style.color = PAIRING_SECONDS < 60 ? '#dc2626' : '#92400e'; }
        if (PAIRING_SECONDS <= 0) { clearInterval(PAIRING_TIMER); if ($('pairingTimer')) $('pairingTimer').textContent = 'Kadaluarsa'; }
      };
      update();
      PAIRING_TIMER = setInterval(() => { PAIRING_SECONDS--; update(); }, 1000);
    }

    $('showQrPairingBtn')?.addEventListener('click', async () => {
      $('qrPairingModal').classList.add('show');
      $('pairingCodeDisplay').textContent = '------';
      $('qrCodeBox').innerHTML = '<div style="width:240px;height:240px;display:flex;align-items:center;justify-content:center;"><i class="fas fa-spinner fa-spin" style="font-size:2rem;color:#7c3aed;"></i></div>';
      const code = await createPairingSession();
      if (code) {
        PAIRING_CODE = code;
        $('pairingCodeDisplay').textContent = code;
        renderQRCode(code);
        startPairingTimer();
      } else showToast('Gagal', 'Tidak dapat membuat pairing session.', 'error');
    });

    $('qrPairingClose')?.addEventListener('click', () => { $('qrPairingModal').classList.remove('show'); clearInterval(PAIRING_TIMER); });
    $('qrPairingCancel')?.addEventListener('click', () => { $('qrPairingModal').classList.remove('show'); clearInterval(PAIRING_TIMER); });
    $('copyPairingCodeBtn')?.addEventListener('click', () => {
      if (PAIRING_CODE) {
        navigator.clipboard.writeText(PAIRING_CODE).then(() => showToast('Terkopy!', `Kode ${PAIRING_CODE} disalin.`, 'success'))
          .catch(() => showToast('Gagal Copy', 'Salin manual: ' + PAIRING_CODE, 'error'));
      }
    });
    $('refreshPairingBtn')?.addEventListener('click', async () => {
      const code = await createPairingSession();
      if (code) {
        PAIRING_CODE = code;
        $('pairingCodeDisplay').textContent = code;
        renderQRCode(code); startPairingTimer();
        showToast('Diperbarui', 'Kode pairing baru dibuat.', 'success');
      }
    });

    $('openPairingInputBtn')?.addEventListener('click', () => {
      $('pairingInputModal').classList.add('show');
      $('pairingCodeInput').value = '';
      $('pairingError').style.display = 'none';
      $('pairingSuccess').style.display = 'none';
      setTimeout(() => $('pairingCodeInput').focus(), 100);
    });
    $('pairingCancelBtn')?.addEventListener('click', () => $('pairingInputModal').classList.remove('show'));
    $('pairingCodeInput')?.addEventListener('input', (e) => { e.target.value = e.target.value.toUpperCase().replace(/[^A-Z0-9]/g, ''); });

    $('pairingSubmitBtn')?.addEventListener('click', async () => {
      const code = $('pairingCodeInput').value.trim().toUpperCase();
      if (code.length !== 6) { $('pairingErrorText').textContent = 'Kode harus 6 karakter.'; $('pairingError').style.display = 'block'; return; }
      $('pairingSubmitBtn').disabled = true;
      $('pairingSubmitBtn').innerHTML = '<i class="fas fa-spinner fa-spin"></i> Menghubungkan...';
      try {
        if (!FIREBASE_READY || !db || !FB) { $('pairingErrorText').textContent = 'Firebase tidak aktif.'; $('pairingError').style.display = 'block'; return; }
        const docSnap = await FB.getDoc(FB.doc(db, 'pairing_sessions', code));
        if (!docSnap.exists()) { $('pairingErrorText').textContent = 'Kode tidak ditemukan.'; $('pairingError').style.display = 'block'; return; }
        const data = docSnap.data();
        if (!data.active || new Date(data.expiresAt) < new Date()) { $('pairingErrorText').textContent = 'Kode sudah kadaluarsa.'; $('pairingError').style.display = 'block'; return; }
        localStorage.setItem('unm_paired_dosen', JSON.stringify({ code, dosenId: data.dosenId, dosenName: data.dosenName, pairedAt: new Date().toISOString() }));
        if (currentUser && FIREBASE_READY && db && FB) {
          await FB.updateDoc(FB.doc(db, 'devices', DEVICE_ID), {
            pairedDosenId: data.dosenId, pairedDosenName: data.dosenName, pairedCode: code, pairedAt: new Date().toISOString()
          }).catch(() => {});
        }
        $('pairingSuccessText').textContent = `Berhasil terhubung dengan ${data.dosenName}!`;
        $('pairingSuccess').style.display = 'block';
        $('pairingError').style.display = 'none';
        showToast('Terhubung!', `Anda terhubung dengan ${data.dosenName}`, 'success', 5000);
        setTimeout(() => $('pairingInputModal').classList.remove('show'), 2000);
        sendNotification(data.dosenId, 'dosen', { title: 'Mahasiswa Terhubung', desc: `${currentUser?.nama || 'Mahasiswa'} terhubung via kode ${code}`, type: 'success', icon: 'fa-user-check' });
      } catch (err) {
        $('pairingErrorText').textContent = 'Error: ' + (err.message || 'Gagal verifikasi');
        $('pairingError').style.display = 'block';
      }
      $('pairingSubmitBtn').disabled = false;
      $('pairingSubmitBtn').innerHTML = '<i class="fas fa-link"></i> Hubungkan';
    });

    function checkUrlPairing() {
      const urlParams = new URLSearchParams(window.location.search);
      const pairCode = urlParams.get('pair');
      if (pairCode) {
        setTimeout(() => {
          $('openPairingInputBtn')?.click();
          setTimeout(() => { $('pairingCodeInput').value = pairCode.toUpperCase(); }, 200);
        }, 1000);
        window.history.replaceState({}, '', window.location.pathname);
      }
    }

    // ============================================================
    // NOTIFICATIONS
    // ============================================================
    async function sendNotification(userId, userRole, notif) {
      if (!FIREBASE_READY || !db || !FB) return;
      const notifId = 'notif_' + Date.now() + '_' + Math.random().toString(36).substr(2, 6);
      try {
        await FB.setDoc(FB.doc(db, 'notifications', notifId), {
          id: notifId, userId, userRole,
          title: notif.title, desc: notif.desc,
          type: notif.type || 'info', icon: notif.icon || 'fa-bell',
          read: false, createdAt: new Date().toISOString(),
          serverTime: FB.serverTimestamp()
        });
      } catch (e) {}
    }

    let notifUnsubscribe = null;
    function setupNotificationListener() {
      if (!FIREBASE_READY || !db || !FB || !currentUser) return;
      if (notifUnsubscribe) notifUnsubscribe();
      try {
        const q = FB.query(FB.collection(db, 'notifications'), FB.where('userId', '==', currentUser.nim));
        notifUnsubscribe = FB.onSnapshot(q, (snap) => {
          const notifs = [];
          snap.forEach(d => notifs.push({ ...d.data(), _docId: d.id }));
          notifs.sort((a, b) => {
            const ta = a.createdAt ? new Date(a.createdAt).getTime() : 0;
            const tb = b.createdAt ? new Date(b.createdAt).getTime() : 0;
            return tb - ta;
          });
          NOTIFICATIONS_DATA = notifs;
          renderNotifications();
        });
      } catch (e) {}
    }

    function renderNotifications() {
      const body = $('notifPanelBody');
      const count = $('notifCount');
      const bell = $('notifBell');
      if (!body) return;
      const unread = NOTIFICATIONS_DATA.filter(n => !n.read).length;
      if (unread > 0) {
        count.style.display = 'flex';
        count.textContent = unread > 99 ? '99+' : unread;
        bell?.classList.add('has-notif');
        setTimeout(() => bell?.classList.remove('has-notif'), 600);
      } else count.style.display = 'none';
      if (NOTIFICATIONS_DATA.length === 0) {
        body.innerHTML = '<div class="notif-empty"><i class="fas fa-bell-slash"></i><div>Belum ada notifikasi</div></div>';
        return;
      }
      body.innerHTML = '';
      NOTIFICATIONS_DATA.slice(0, 30).forEach(n => {
        const item = document.createElement('div');
        item.className = 'notif-item' + (n.read ? '' : ' unread');
        const timeAgo = formatTimeAgo(n.createdAt);
        item.innerHTML = `
          <div class="notif-icon ${n.type || 'info'}"><i class="fas ${n.icon || 'fa-bell'}"></i></div>
          <div class="notif-content">
            <div class="notif-title">${escapeHtml(n.title)}</div>
            <div class="notif-desc">${escapeHtml(n.desc)}</div>
            <div class="notif-time"><i class="fas fa-clock"></i> ${timeAgo}</div>
          </div>
        `;
        item.addEventListener('click', async () => {
          if (!n.read && FIREBASE_READY && db && FB) {
            try { await FB.updateDoc(FB.doc(db, 'notifications', n._docId), { read: true }); } catch (e) {}
          }
        });
        body.appendChild(item);
      });
    }

    $('notifBell')?.addEventListener('click', () => $('notifPanel').classList.toggle('show'));
    document.addEventListener('click', (e) => {
      const panel = $('notifPanel');
      const bell = $('notifBell');
      if (panel && panel.classList.contains('show')) {
        if (!panel.contains(e.target) && !bell.contains(e.target)) panel.classList.remove('show');
      }
    });
    $('markAllReadBtn')?.addEventListener('click', async () => {
      if (!FIREBASE_READY || !db || !FB) return;
      const unread = NOTIFICATIONS_DATA.filter(n => !n.read);
      for (const n of unread) {
        try { await FB.updateDoc(FB.doc(db, 'notifications', n._docId), { read: true }); } catch(e) {}
      }
      showToast('Ditandai', 'Semua notifikasi ditandai dibaca.', 'success');
    });

    // ============================================================
    // OFFLINE QUEUE
    // ============================================================
    function updateOfflineQueueBadge() {
      const badge = $('offlineQueueBadge');
      const count = $('queueCount');
      if (!badge || !count) return;
      if (OFFLINE_QUEUE.length > 0 && !navigator.onLine) {
        badge.classList.add('show'); count.textContent = OFFLINE_QUEUE.length;
      } else badge.classList.remove('show');
    }

    async function processOfflineQueue() {
      if (!navigator.onLine || OFFLINE_QUEUE.length === 0) return;
      if (!FIREBASE_READY || !db || !FB) return;
      console.log('🔄 Processing offline queue:', OFFLINE_QUEUE.length);
      const queue = [...OFFLINE_QUEUE];
      OFFLINE_QUEUE = [];
      for (const item of queue) {
        try {
          if (item.action.type === 'saveActivity') await FB.setDoc(FB.doc(db, 'live_activities', item.action.docId), item.action.data);
          else if (item.action.type === 'saveStudent') await FB.setDoc(FB.doc(db, 'students', item.action.id), item.action.data);
        } catch (e) {}
      }
      updateOfflineQueueBadge();
    }

    window.addEventListener('online', () => { processOfflineQueue(); showToast('Online', 'Koneksi kembali.', 'success'); });
    window.addEventListener('offline', () => { showToast('Offline', 'Mode offline aktif.', 'info', 5000); updateOfflineQueueBadge(); });

    // ============================================================
    // DEVICE CONFLICT
    // ============================================================
    async function checkDeviceConflict() {
      if (!FIREBASE_READY || !db || !FB || !currentUser) return;
      const myDevices = DEVICES_DATA.filter(d => d.userId === currentUser.nim && d._isOnline && d._docId !== DEVICE_ID);
      if (myDevices.length > 0 && !conflictShown) {
        conflictShown = true;
        const otherDevice = myDevices[0];
        $('conflictMessage').textContent = `Akun Anda aktif di ${otherDevice.deviceName} (${formatTimeAgo(otherDevice.lastPing)}). Lanjutkan di perangkat ini?`;
        $('deviceConflictAlert').classList.add('show');
      }
    }

    $('conflictCancelBtn')?.addEventListener('click', () => {
      $('deviceConflictAlert').classList.remove('show');
      setTimeout(() => logout(), 300);
    });
    $('conflictContinueBtn')?.addEventListener('click', async () => {
      $('deviceConflictAlert').classList.remove('show');
      const myDevices = DEVICES_DATA.filter(d => d.userId === currentUser?.nim && d._docId !== DEVICE_ID);
      for (const dev of myDevices) {
        try { await FB.updateDoc(FB.doc(db, 'devices', dev._docId), { online: false, forceLogout: true, lastPing: new Date().toISOString() }); } catch (e) {}
      }
      showToast('Device Lain Logout', 'Anda aktif di perangkat ini.', 'success');
    });

    // ============================================================
    // SYNC STATUS
    // ============================================================
    function updateSyncStatus(status) {
      const bar = $('syncStatusBar');
      if (!bar) return;
      bar.className = 'sync-status-bar ' + status;
      const icons = { synced: '<i class="fas fa-check-circle"></i>', syncing: '<i class="fas fa-sync-alt fa-spin"></i>', error: '<i class="fas fa-exclamation-circle"></i>', offline: '<i class="fas fa-cloud-slash"></i>' };
      const labels = { synced: 'Tersinkronisasi', syncing: 'Menyinkron...', error: 'Gagal Sync', offline: 'Mode Offline' };
      bar.innerHTML = `${icons[status]} <span>${labels[status]}</span>`;
    }

    setInterval(() => {
      if (!FIREBASE_READY || !navigator.onLine) updateSyncStatus('offline');
      else updateSyncStatus('synced');
    }, 5000);

    // ============================================================
    // INIT MULTI-DEVICE
    // ============================================================
    async function initMultiDeviceSetup() {
      if (!currentUser) return;
      await registerDevice();
      startDeviceHeartbeat();
      setupNotificationListener();
      setTimeout(checkDeviceConflict, 3000);
      setInterval(checkDeviceConflict, 30000);
      console.log('✅ Multi-device setup untuk:', currentUser.nama);
    }

    // Observer login success
    const observer = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        if (mutation.target.id === 'loginSection' && mutation.target.style.display === 'none') {
          setTimeout(() => { if (currentUser) initMultiDeviceSetup(); }, 500);
        }
      });
    });
    if ($('loginSection')) observer.observe($('loginSection'), { attributes: true, attributeFilter: ['style'] });

    // ============================================================
    // ROLE TABS
    // ============================================================
    $('roleMahasiswaBtn').addEventListener('click', () => {
      currentRole = 'mahasiswa';
      $('roleMahasiswaBtn').classList.add('active'); $('roleDosenBtn').classList.remove('active'); $('roleAdminBtn').classList.remove('active');
      $('loginLabelText').textContent = 'Nama Lengkap / NIM';
      $('loginName').placeholder = 'Contoh: 2101010101 / Budi Santoso';
      $('registerBannerMahasiswa').classList.remove('hidden'); $('registerBannerDosen').classList.add('hidden');
      $('registerFormMahasiswa').classList.remove('hidden'); $('registerFormDosen').classList.add('hidden');
      $('loginBtn').className = 'btn-login';
    });
    $('roleDosenBtn').addEventListener('click', () => {
      currentRole = 'dosen';
      $('roleDosenBtn').classList.add('active'); $('roleMahasiswaBtn').classList.remove('active'); $('roleAdminBtn').classList.remove('active');
      $('loginLabelText').textContent = 'Nama Dosen / NIDN';
      $('loginName').placeholder = 'Contoh: NIDN001 / Dr. Budi Santoso';
      $('registerBannerMahasiswa').classList.add('hidden'); $('registerBannerDosen').classList.remove('hidden');
      $('registerFormMahasiswa').classList.add('hidden'); $('registerFormDosen').classList.remove('hidden');
      $('loginBtn').className = 'btn-login';
    });
    $('roleAdminBtn').addEventListener('click', () => {
      currentRole = 'admin';
      $('roleAdminBtn').classList.add('active'); $('roleMahasiswaBtn').classList.remove('active'); $('roleDosenBtn').classList.remove('active');
      $('loginLabelText').textContent = 'Admin ID';
      $('loginName').placeholder = 'Contoh: RikardusFeribertusNikat';
      $('registerBannerMahasiswa').classList.add('hidden'); $('registerBannerDosen').classList.add('hidden');
      $('registerFormMahasiswa').classList.add('hidden'); $('registerFormDosen').classList.add('hidden');
      $('loginBtn').className = 'btn-login admin-btn';
    });
    $('tabLogin').addEventListener('click', () => { $('tabLogin').classList.add('active'); $('tabRegister').classList.remove('active'); $('loginFormContainer').classList.remove('hidden'); $('registerFormContainer').classList.add('hidden'); });
    $('tabRegister').addEventListener('click', () => { $('tabRegister').classList.add('active'); $('tabLogin').classList.remove('active'); $('registerFormContainer').classList.remove('hidden'); $('loginFormContainer').classList.add('hidden'); });

    // ============================================================
    // OTP
    // ============================================================
    function generateOTP() { return String(Math.floor(100000 + Math.random() * 900000)); }
    async function kirimEmailOTP(email, nama, kode, tipe = 'verifikasi') {
      if (DEMO_MODE) { console.log(`🔐 DEMO OTP: ${email} → ${kode}`); return { demo: true }; }
      return await emailjs.send(EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_ID, { to_email: email, to_name: nama, otp_code: kode, app_name: 'Portal Ujian UNM', tipe, reply_to: 'noreply@unm.ac.id' });
    }
    function startOtpTimer() {
      otpSecondsLeft = 300; clearInterval(otpTimerInterval);
      const update = () => { const m = String(Math.floor(otpSecondsLeft/60)).padStart(2,'0'); const s = String(otpSecondsLeft%60).padStart(2,'0'); const el = $('otpTimer'); if (el) { el.textContent = `${m}:${s}`; el.style.color = otpSecondsLeft < 60 ? '#dc2626' : '#1e4b7c'; } };
      update();
      otpTimerInterval = setInterval(() => { otpSecondsLeft--; update(); if (otpSecondsLeft <= 0) { clearInterval(otpTimerInterval); if ($('otpTimer')) $('otpTimer').textContent = 'Kadaluarsa'; } }, 1000);
    }
    async function openOtpModal(email, user) {
      pendingUser = user; otpCode = generateOTP();
      $('otpEmailTarget').textContent = email;
      $('otpError').classList.remove('show'); $('otpSuccess').classList.remove('show');
      $('otpModal').classList.add('show');
      const inputs = document.querySelectorAll('[data-otp-index]');
      inputs.forEach(inp => { inp.value = ''; inp.classList.remove('filled'); inp.disabled = true; });
      $('otpVerifyBtn').disabled = true; $('otpCancelBtn').disabled = true;
      $('otpVerifyBtn').innerHTML = '<i class="fas fa-spinner fa-spin"></i> Mengirim...';
      try {
        await kirimEmailOTP(email, user.nama, otpCode, 'verifikasi');
        inputs.forEach(inp => inp.disabled = false);
        $('otpVerifyBtn').disabled = false; $('otpCancelBtn').disabled = false;
        $('otpVerifyBtn').innerHTML = '<i class="fas fa-check"></i> Verifikasi';
        if (DEMO_MODE) { $('otpSuccessText').textContent = `DEMO: Kode OTP Anda ${otpCode}`; $('otpSuccess').classList.add('show'); }
        else showToast('Email Terkirim!', `Cek inbox ${email}`, 'success', 6000);
        startOtpTimer();
        setTimeout(() => inputs[0].focus(), 100);
      } catch (err) {
        $('otpErrorText').textContent = 'Gagal. ' + (err.text || err.message);
        $('otpError').classList.add('show');
        $('otpVerifyBtn').disabled = false; $('otpCancelBtn').disabled = false;
        $('otpVerifyBtn').innerHTML = '<i class="fas fa-check"></i> Verifikasi';
      }
    }
    document.querySelectorAll('[data-otp-index]').forEach((inp, idx, all) => {
      inp.addEventListener('input', (e) => { const val = e.target.value.replace(/\D/g, ''); e.target.value = val.slice(0, 1); if (val) { e.target.classList.add('filled'); if (idx < all.length - 1) all[idx + 1].focus(); } else e.target.classList.remove('filled'); });
      inp.addEventListener('keydown', (e) => { if (e.key === 'Backspace' && !e.target.value && idx > 0) all[idx - 1].focus(); });
      inp.addEventListener('paste', (e) => { e.preventDefault(); const p = (e.clipboardData || window.clipboardData).getData('text').replace(/\D/g, '').slice(0, 6); if (p) { all.forEach((input, i) => { input.value = p[i] || ''; if (p[i]) input.classList.add('filled'); }); if (p.length === 6) all[5].focus(); } });
    });
    $('otpCancelBtn').addEventListener('click', () => { $('otpModal').classList.remove('show'); clearInterval(otpTimerInterval); pendingUser = null; });
    $('otpVerifyBtn').addEventListener('click', async () => {
      const inputs = document.querySelectorAll('[data-otp-index]');
      const entered = Array.from(inputs).map(i => i.value).join('');
      if (entered.length < 6) { $('otpErrorText').textContent = 'Masukkan 6 digit.'; $('otpError').classList.add('show'); return; }
      if (entered !== otpCode) { $('otpErrorText').textContent = 'Kode salah.'; $('otpError').classList.add('show'); return; }
      const newUser = { ...pendingUser, verified: true, createdAt: new Date().toISOString() };
      await syncSaveUser(newUser);
      if (newUser.role === 'mahasiswa') {
        const newStudent = {
          id: 'm_' + Date.now(),
          nama: newUser.nama, nim: newUser.nim, email: newUser.email,
          warna: ['#1e4b7c','#7c3aed','#0891b2','#db2777','#ea580c','#16a34a','#9333ea','#0d9488'][Math.floor(Math.random() * 8)],
          aiStatus: 'bebas', prodi: newUser.prodi || '', angkatan: newUser.angkatan || '', kelas: newUser.kelas || '',
          sesi: {
            pretest: { status: 'belum', progress: 0, nilaiSoal: {}, rubrikSkor: {}, feedback: {}, emailSent: false, jawaban: {} },
            posttest: { status: 'belum', progress: 0, nilaiSoal: {}, rubrikSkor: {}, feedback: {}, emailSent: false, jawaban: {} }
          }
        };
        await syncSaveStudent(newStudent);
      }
      clearInterval(otpTimerInterval); $('otpModal').classList.remove('show');
      showToast('Verifikasi Berhasil!', `Akun ${newUser.nama} tersimpan.`, 'success', 6000);
      $('loginName').value = pendingUser.nim; $('loginPassword').value = '';
      if (pendingUser.role === 'dosen') $('roleDosenBtn').click(); else $('roleMahasiswaBtn').click();
      $('tabLogin').click(); pendingUser = null;
    });

    // ============================================================
    // REGISTER
    // ============================================================
    $('registerMahasiswaBtn').addEventListener('click', async () => {
      const name = $('mhsName').value.trim(), nim = $('mhsNim').value.trim();
      const prodi = $('mhsProdi').value, angkatan = $('mhsAngkatan').value.trim(), kelas = $('mhsKelas').value.trim();
      const email = $('mhsEmail').value.trim().toLowerCase(), pass = $('mhsPassword').value, passC = $('mhsPasswordConfirm').value;
      if (!name || !nim || !email || !pass || !passC || !angkatan) { showToast('Data Tidak Lengkap', 'Lengkapi semua field.', 'error'); return; }
      if (pass.length < 6) { showToast('Password Lemah', 'Min 6 karakter.', 'error'); return; }
      if (pass !== passC) { showToast('Password Tidak Cocok', 'Konfirmasi salah.', 'error'); return; }
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) { showToast('Email Tidak Valid', 'Format salah.', 'error'); return; }
      if (!/^\d{6,15}$/.test(nim)) { showToast('NIM Tidak Valid', 'Harus 6-15 digit.', 'error'); return; }
      showLoading(true); await fetchAllUsers(); showLoading(false);
      if (USERS_DATA.some(u => u.nim === nim && u.role === 'mahasiswa')) { showToast('NIM Terdaftar', `NIM ${nim} sudah digunakan.`, 'error'); return; }
      if (USERS_DATA.some(u => u.email === email)) { showToast('Email Terdaftar', `Email ${email} sudah digunakan.`, 'error'); return; }
      openOtpModal(email, { nama: name, nim, email, password: pass, role: 'mahasiswa', verified: false, prodi, angkatan, kelas });
    });
    $('registerDosenBtn').addEventListener('click', async () => {
      const name = $('dsnName').value.trim(), nidn = $('dsnNidn').value.trim();
      const jabatan = $('dsnJabatan').value, prodi = $('dsnProdi').value, matkul = $('dsnMatkul').value.trim();
      const email = $('dsnEmail').value.trim().toLowerCase(), pass = $('dsnPassword').value, passC = $('dsnPasswordConfirm').value;
      if (!name || !nidn || !email || !pass || !passC || !matkul) { showToast('Data Tidak Lengkap', 'Lengkapi semua field.', 'error'); return; }
      if (pass.length < 8) { showToast('Password Lemah', 'Min 8 karakter.', 'error'); return; }
      if (pass !== passC) { showToast('Password Tidak Cocok', 'Konfirmasi salah.', 'error'); return; }
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) { showToast('Email Tidak Valid', 'Format salah.', 'error'); return; }
      if (!/^\d{6,15}$/.test(nidn)) { showToast('NIDN Tidak Valid', 'Harus 6-15 digit.', 'error'); return; }
      showLoading(true); await fetchAllUsers(); showLoading(false);
      if (USERS_DATA.some(u => u.nim === nidn && u.role === 'dosen')) { showToast('NIDN Terdaftar', `NIDN ${nidn} sudah digunakan.`, 'error'); return; }
      if (USERS_DATA.some(u => u.email === email)) { showToast('Email Terdaftar', `Email ${email} sudah digunakan.`, 'error'); return; }
      openOtpModal(email, { nama: name, nim: nidn, email, password: pass, role: 'dosen', verified: false, jabatan, prodi, matkul });
    });

    // ============================================================
    // LOGIN
    // ============================================================
    $('loginBtn').addEventListener('click', async () => {
      const input = $('loginName').value.trim(), pass = $('loginPassword').value.trim();
      if (!input || !pass) { showToast('Data Tidak Lengkap', 'Isi kredensial.', 'error'); return; }
      $('loginBtn').disabled = true;
      $('loginBtn').innerHTML = '<i class="fas fa-spinner fa-spin"></i> Memuat...';
      showLoading(true);
      await fetchAllUsers(); await fetchAllStudents(); await fetchAllSoal();
      showLoading(false);
      $('loginBtn').disabled = false;
      if (currentRole === 'admin') $('loginBtn').className = 'btn-login admin-btn';
      else $('loginBtn').className = 'btn-login';
      $('loginBtn').innerHTML = '<i class="fas fa-arrow-right-to-bracket"></i> Masuk Portal';
      const user = USERS_DATA.find(u => (u.nim === input || u.nama.toLowerCase() === input.toLowerCase()) && u.password === pass && u.role === currentRole);
      if (!user) { showToast('Login Gagal', `Kredensial salah atau akun belum terdaftar sebagai ${currentRole}.`, 'error', 5000); return; }
      if (!user.verified) { showToast('Belum Terverifikasi', 'Verifikasi via OTP dulu.', 'error', 5000); return; }
      currentUser = user;
      $('loginSection').style.display = 'none';
      if (user.role === 'admin') {
        $('adminSection').style.display = 'block'; $('dosenSection').style.display = 'none';
        $('sesiPortalSection').style.display = 'none'; $('examSection').style.display = 'none';
        renderAdminUserList(); updateAdminStats();
        showToast('Login Berhasil', `Selamat datang, Admin ${user.nama}.`, 'success');
      } else if (user.role === 'dosen') {
        $('dosenSection').style.display = 'block'; $('adminSection').style.display = 'none';
        $('sesiPortalSection').style.display = 'none'; $('examSection').style.display = 'none';
        $('dosenNameDisplay').textContent = user.nama;
        if (STUDENTS_DATA.length > 0 && !selectedStudentId) selectedStudentId = STUDENTS_DATA[0].id;
        renderStudentList(); renderMahasiswaSelector(); renderSoalPenilaianSelector();
        if (selectedStudentId) updateSelectedStudentInfo(STUDENTS_DATA.find(s => s.id === selectedStudentId));
        updateRubrikUIForSoal(); renderDatabase(); renderSoalEditList(); renderLiveMonitor(); renderDeviceList();
        showToast('Login Berhasil', `Selamat datang, ${user.nama}. Multi-device monitoring aktif.`, 'success');
      } else {
        $('sesiPortalSection').style.display = 'block'; $('adminSection').style.display = 'none';
        $('dosenSection').style.display = 'none'; $('examSection').style.display = 'none';
        $('sesiUserName').textContent = user.nama;
        const jml = SOAL_DATA.length;
        if ($('pretestJumlahSoal')) $('pretestJumlahSoal').textContent = jml;
        if ($('posttestJumlahSoal')) $('posttestJumlahSoal').textContent = jml;
        showToast('Login Berhasil', `Selamat datang, ${user.nama}. Aktivitas Anda dipantau dari perangkat manapun.`, 'success');
      }
    });

    async function logout() {
      if (currentUser && currentSesi) {
        saveActivity({ studentId: currentUser.nim, nama: currentUser.nama, sesi: currentSesi, status: 'offline', currentSoal: null, currentSoalId: null, totalChars: 0, lastUpdate: new Date().toISOString() });
      }
      if (FIREBASE_READY && db && FB && MY_DEVICE_DOC) {
        try { await FB.updateDoc(FB.doc(db, 'devices', MY_DEVICE_DOC), { online: false, lastPing: new Date().toISOString() }); } catch(e) {}
      }
      if (notifUnsubscribe) { notifUnsubscribe(); notifUnsubscribe = null; }
      conflictShown = false; MY_DEVICE_DOC = null; NOTIFICATIONS_DATA = [];
      $('loginSection').style.display = 'block';
      $('sesiPortalSection').style.display = 'none'; $('examSection').style.display = 'none';
      $('dosenSection').style.display = 'none'; $('adminSection').style.display = 'none';
      $('loginName').value = ''; $('loginPassword').value = '';
      $('tabLogin').click(); currentUser = null; currentSesi = null;
      showToast('Logout', 'Anda telah keluar.', 'info');
    }
    $('logoutBtn').addEventListener('click', logout);
    $('logoutDosenBtn').addEventListener('click', logout);
    $('logoutSesiBtn').addEventListener('click', logout);
    $('logoutAdminBtn').addEventListener('click', logout);

    // ============================================================
    // ADMIN
    // ============================================================
    function updateAdminStats() {
      const users = USERS_DATA.length > 0 ? USERS_DATA : getLocal(LS_USERS);
      $('adminTotalUsers').textContent = users.length;
      $('adminTotalDosen').textContent = users.filter(u => u.role === 'dosen').length;
      $('adminTotalMhs').textContent = users.filter(u => u.role === 'mahasiswa').length;
      $('adminTotalSoal').textContent = SOAL_DATA.length;
    }

    function renderAdminUserList() {
      const list = $('adminUserList');
      list.innerHTML = '';
      const filterRole = $('adminFilterRole').value;
      const searchTerm = ($('adminSearchUser').value || '').toLowerCase().trim();
      let users = USERS_DATA.length > 0 ? [...USERS_DATA] : [...getLocal(LS_USERS)];
      if (filterRole !== 'all') users = users.filter(u => u.role === filterRole);
      if (searchTerm) users = users.filter(u => u.nama.toLowerCase().includes(searchTerm) || u.nim.toLowerCase().includes(searchTerm) || (u.email || '').toLowerCase().includes(searchTerm));
      const roleOrder = { admin: 0, dosen: 1, mahasiswa: 2 };
      users.sort((a, b) => (roleOrder[a.role] || 9) - (roleOrder[b.role] || 9));
      if (users.length === 0) {
        list.innerHTML = '<div style="padding:40px;text-align:center;color:#94a3b8;"><i class="fas fa-inbox" style="font-size:2.5rem;margin-bottom:12px;display:block;"></i>Tidak ada pengguna.</div>';
        updateAdminStats(); return;
      }
      users.forEach(u => {
        const row = document.createElement('div');
        row.className = 'user-row';
        const initials = u.nama.split(' ').map(n => n[0]).slice(0, 2).join('');
        const roleLabel = u.role === 'admin' ? 'Admin' : u.role === 'dosen' ? 'Dosen' : 'Mahasiswa';
        row.innerHTML = `
          <div class="avatar ${u.role}">${initials}</div>
          <div class="user-info">
            <div class="name">${u.nama}</div>
            <div class="nim">${u.nim}</div>
            <div class="email"><i class="fas fa-envelope"></i> ${u.email || '-'}</div>
          </div>
          <span class="user-role-badge ${u.role}">${roleLabel}</span>
          <div class="user-actions">
            <button class="btn-user-action primary" data-edit-user="${u.nim}_${u.role}"><i class="fas fa-edit"></i></button>
            <button class="btn-user-action danger" data-delete-user="${u.nim}_${u.role}"><i class="fas fa-trash"></i></button>
          </div>
        `;
        row.querySelector(`[data-edit-user]`).addEventListener('click', () => openEditUserAdmin(u.nim, u.role));
        row.querySelector(`[data-delete-user]`).addEventListener('click', () => openDeleteUserAdmin(u.nim, u.role, u.nama));
        list.appendChild(row);
      });
      updateAdminStats();
    }

    $('adminFilterRole').addEventListener('change', renderAdminUserList);
    $('adminSearchUser').addEventListener('input', renderAdminUserList);
    $('btnRefreshUsers').addEventListener('click', async () => {
      showToast('Refreshing...', 'Memuat ulang data', 'info', 2000);
      await fetchAllUsers(); renderAdminUserList(); updateAdminStats();
      showToast('Data Dimuat', `${USERS_DATA.length} pengguna`, 'success', 3000);
    });

    function openEditUserAdmin(nim, role) {
      const user = USERS_DATA.find(u => u.nim === nim && u.role === role);
      if (!user) return;
      editingUserId = { nim, role };
      $('adminEditNama').value = user.nama || '';
      $('adminEditNim').value = user.nim || '';
      $('adminEditEmail').value = user.email || '';
      $('adminEditRole').value = user.role;
      $('adminEditPassword').value = '';
      $('editUserAdminModal').classList.add('show');
    }
    $('editUserAdminClose').addEventListener('click', () => $('editUserAdminModal').classList.remove('show'));
    $('editUserAdminCancel').addEventListener('click', () => $('editUserAdminModal').classList.remove('show'));
    $('editUserAdminSave').addEventListener('click', async () => {
      if (!editingUserId) return;
      const newNama = $('adminEditNama').value.trim();
      const newNim = $('adminEditNim').value.trim();
      const newEmail = $('adminEditEmail').value.trim().toLowerCase();
      const newRole = $('adminEditRole').value;
      const newPass = $('adminEditPassword').value;
      if (!newNama || !newNim || !newEmail) { showToast('Data Tidak Lengkap', 'Lengkapi semua field.', 'error'); return; }
      const oldUser = USERS_DATA.find(u => u.nim === editingUserId.nim && u.role === editingUserId.role);
      if (!oldUser) return;
      if (newNim !== editingUserId.nim || newRole !== editingUserId.role) await syncDeleteUser(oldUser);
      const updatedUser = { ...oldUser, nama: newNama, nim: newNim, email: newEmail, role: newRole };
      if (newPass && newPass.length >= 6) updatedUser.password = newPass;
      await syncSaveUser(updatedUser);
      $('editUserAdminModal').classList.remove('show');
      editingUserId = null;
      showToast('Data Diperbarui', `${newNama} berhasil diupdate.`, 'success');
      await fetchAllUsers(); renderAdminUserList();
    });

    function openDeleteUserAdmin(nim, role, nama) {
      $('confirmTargetName').textContent = `${nama} (${role})`;
      $('confirmModal').classList.add('show');
      $('confirmDeleteBtn').dataset.deleteType = 'user';
      $('confirmDeleteBtn').dataset.deleteNim = nim;
      $('confirmDeleteBtn').dataset.deleteRole = role;
    }
    $('confirmCancelBtn').addEventListener('click', () => { $('confirmModal').classList.remove('show'); $('confirmDeleteBtn').dataset.deleteType = ''; });
    $('confirmDeleteBtn').addEventListener('click', async () => {
      const type = $('confirmDeleteBtn').dataset.deleteType;
      if (type === 'user') {
        const nim = $('confirmDeleteBtn').dataset.deleteNim;
        const role = $('confirmDeleteBtn').dataset.deleteRole;
        const user = USERS_DATA.find(u => u.nim === nim && u.role === role);
        if (user) {
          await syncDeleteUser(user);
          if (role === 'mahasiswa') {
            const student = STUDENTS_DATA.find(s => s.nim === nim);
            if (student) await syncDeleteStudent(student.id);
          }
          showToast('Pengguna Dihapus', `${user.nama} dihapus.`, 'success');
          await fetchAllUsers(); renderAdminUserList();
        }
      } else if (type === 'student') {
        const id = $('confirmDeleteBtn').dataset.deleteId;
        const student = STUDENTS_DATA.find(s => s.id === id);
        const nama = student ? student.nama : 'Peserta';
        await syncDeleteStudent(id);
        if (selectedStudentId === id) selectedStudentId = STUDENTS_DATA.length > 0 ? STUDENTS_DATA[0].id : null;
        renderStudentList(); renderMahasiswaSelector(); renderDatabase();
        showToast('Peserta Dihapus', `${nama} dihapus.`, 'success');
      } else if (type === 'soal') {
        const id = parseInt($('confirmDeleteBtn').dataset.deleteSoalId);
        await syncDeleteSoal(id);
        renderSoalEditList();
        showToast('Soal Dihapus', `Soal ${id} dihapus.`, 'success');
        if ($('pretestJumlahSoal')) $('pretestJumlahSoal').textContent = SOAL_DATA.length;
        if ($('posttestJumlahSoal')) $('posttestJumlahSoal').textContent = SOAL_DATA.length;
      } else if (type === 'db') {
        const studentId = $('confirmDeleteBtn').dataset.deleteStudentId;
        const sesi = $('confirmDeleteBtn').dataset.deleteSesi;
        const student = STUDENTS_DATA.find(s => s.id === studentId);
        if (student) {
          student.sesi[sesi] = { status: 'belum', progress: 0, nilaiSoal: {}, rubrikSkor: {}, feedback: {}, emailSent: false, jawaban: {} };
          await syncSaveStudent(student);
          showToast('Data Direset', `Data ${sesi} ${student.nama} direset.`, 'success');
          renderDatabase();
        }
      }
      $('confirmModal').classList.remove('show');
      $('confirmDeleteBtn').dataset.deleteType = '';
    });

    // ============================================================
    // PILIH SESI
    // ============================================================
    document.querySelectorAll('[data-sesi]').forEach(el => {
      el.addEventListener('click', (e) => { e.stopPropagation(); openSesiConfirm(el.dataset.sesi); });
    });
    function openSesiConfirm(sesi) {
      pendingSesi = sesi;
      const isPretest = sesi === 'pretest';
      $('sesiConfirmIcon').style.background = isPretest ? 'linear-gradient(135deg,#d97706,#f59e0b)' : 'linear-gradient(135deg,#15803d,#22c55e)';
      $('sesiConfirmIcon').innerHTML = isPretest ? '<i class="fas fa-clipboard-list"></i>' : '<i class="fas fa-graduation-cap"></i>';
      $('sesiConfirmTitle').textContent = `Mulai ${isPretest ? 'Pre-Test' : 'Post-Test'}?`;
      $('sesiConfirmDesc').textContent = `Aktivitas akan dipantau dosen realtime dari perangkat manapun.`;
      $('sesiStartBtn').style.background = isPretest ? 'linear-gradient(135deg,#d97706,#f59e0b)' : 'linear-gradient(135deg,#15803d,#22c55e)';
      $('sesiConfirmModal').classList.add('show');
    }
    $('sesiCancelBtn').addEventListener('click', () => { $('sesiConfirmModal').classList.remove('show'); pendingSesi = null; });
    $('sesiStartBtn').addEventListener('click', async () => {
      if (!pendingSesi) return;
      currentSesi = pendingSesi;
      $('sesiConfirmModal').classList.remove('show');
      showLoading(true);
      await fetchAllSoal(); await fetchAllStudents();
      showLoading(false);
      masukKeUjian(pendingSesi);
      pendingSesi = null;
    });
    function masukKeUjian(sesi) {
      $('sesiPortalSection').style.display = 'none';
      $('examSection').style.display = 'block';
      $('dosenSection').style.display = 'none';
      $('adminSection').style.display = 'none';
      const isPretest = sesi === 'pretest';
      $('examSesiTitle').textContent = `Ujian ${isPretest ? 'Pre-Test' : 'Post-Test'}`;
      $('displayUserName').textContent = currentUser.nama;
      renderExamSoal();
      loadPreviousAnswers();
      if (SOAL_DATA.length > 0) pindahSoal(SOAL_DATA[0].id);
      updateEmailPanel(currentUser);
      saveActivity({ studentId: currentUser.nim, nama: currentUser.nama, sesi: currentSesi, status: 'online', currentSoal: `Soal ${SOAL_DATA[0]?.id || 1}`, currentSoalId: SOAL_DATA[0]?.id || 1, totalChars: 0, lastUpdate: new Date().toISOString() });
      showToast('Ujian Dimulai', `${isPretest ? 'Pre-Test' : 'Post-Test'} · ${SOAL_DATA.length} soal · Multi-device monitoring aktif`, 'success', 4000);
    }

    // ============================================================
    // RENDER EXAM
    // ============================================================
    function renderExamSoal() {
      const nav = $('soalNavContainer');
      nav.innerHTML = '';
      const container = $('soalContentContainer');
      container.innerHTML = '';
      if (!SOAL_DATA || SOAL_DATA.length === 0) {
        container.innerHTML = '<div style="padding:40px; text-align:center; color:#94a3b8;"><i class="fas fa-inbox" style="font-size:3rem; margin-bottom:12px; display:block;"></i>Belum ada soal.</div>';
        return;
      }
      SOAL_DATA.forEach((s, idx) => {
        const btn = document.createElement('button');
        btn.className = 'soal-nav-btn' + (idx === 0 ? ' active' : '');
        btn.dataset.soal = s.id;
        btn.textContent = `Soal ${s.id}`;
        nav.appendChild(btn);
        const div = document.createElement('div');
        div.className = 'soal-content' + (idx === 0 ? ' active' : '');
        div.id = `soal${s.id}Content`;
        div.innerHTML = `
          <div class="case-box">
            <h3><i class="fas fa-file-alt"></i> ${s.judul}</h3>
            <p>${s.deskripsi}</p>
            <div class="pertanyaan-label">Pertanyaan:</div>
            <div class="soal-item">
              <p><strong>a.</strong> ${s.pertanyaanA}</p>
              <p><strong>b.</strong> ${s.pertanyaanB}</p>
              <p><strong>c.</strong> ${s.pertanyaanC}</p>
            </div>
          </div>
          ${['a','b','c'].map(sub => `
            <div class="sub-soal-group">
              <div class="sub-soal-header"><div class="sub-soal-label"><span class="badge-abcd">${sub}</span> Jawaban Soal ${s.id}${sub}</div><div class="lock-badge"><i class="fas fa-lock"></i> Anti-Paste · 🔴 Live</div></div>
              <textarea class="sub-soal-textarea" id="answer${s.id}${sub}" data-soal="${s.id}" data-sub="${sub}" placeholder="Tulis jawaban ${s.id}${sub}..."></textarea>
              <div class="paste-warning" id="pasteWarn${s.id}${sub}"><i class="fas fa-ban"></i> Paste diblokir!</div>
              <div class="char-counter" id="counter${s.id}${sub}">0 karakter</div>
            </div>
          `).join('')}
          <div class="ai-detection-panel">
            <div class="ai-status"><i class="fas fa-robot"></i><span>Deteksi AI:</span><span id="aiBadge${s.id}" class="badge">Belum diperiksa</span></div>
            <button class="btn-check-ai" data-soal="${s.id}"><i class="fas fa-search"></i> Periksa AI</button>
          </div>
          <div id="aiWarning${s.id}" class="warning-message"><i class="fas fa-exclamation-triangle"></i><span>Terdeteksi pola AI.</span></div>
          <button class="btn-submit" data-submit="${s.id}"><i class="fas fa-paper-plane"></i> Kirim & Nilai Soal ${s.id}</button>
          <div id="result${s.id}" class="result-panel">
            <h3><i class="fas fa-clipboard-check" style="color:#1e4b7c;"></i> Hasil Soal ${s.id}</h3>
            <div class="score-display" id="score${s.id}">0</div>
            <div class="score-label">dari 100 (sementara)</div>
            <ul class="feedback-list" id="feedback${s.id}"></ul>
          </div>
          <div class="feedback-dosen-panel" id="feedbackDosen${s.id}">
            <h3><i class="fas fa-comment-dots"></i> Feedback Dosen — Soal ${s.id}</h3>
            <div class="fb-dosen-meta"><span><i class="fas fa-user-tie"></i> Dosen</span><span><i class="fas fa-star"></i> Nilai: <span class="fb-nilai-badge" id="fbNilai${s.id}">—</span></span></div>
            <div class="fb-dosen-text" id="fbText${s.id}"></div>
          </div>
        `;
        container.appendChild(div);
      });
      attachExamListeners();
      initAntiPaste();
    }

    async function loadPreviousAnswers() {
      if (!currentUser || !currentSesi) return;
      const student = STUDENTS_DATA.find(s => s.nim === currentUser.nim);
      if (!student) return;
      const sesiData = getStudentSesi(student, currentSesi);
      SOAL_DATA.forEach(s => {
        const j = (sesiData.jawaban && sesiData.jawaban[s.id]) || {};
        ['a','b','c'].forEach(sub => {
          const ta = $(`answer${s.id}${sub}`);
          if (ta && j[sub]) {
            ta.value = j[sub];
            const counter = $(`counter${s.id}${sub}`);
            if (counter) { counter.textContent = `${j[sub].length} karakter`; counter.classList.add(j[sub].length >= 30 ? 'ok' : 'warn'); }
          }
        });
      });
      tampilkanFeedbackMahasiswa(currentUser.nim);
    }

    function attachExamListeners() {
      document.querySelectorAll('.soal-nav-btn').forEach(btn => btn.addEventListener('click', () => pindahSoal(btn.dataset.soal)));
      document.querySelectorAll('.btn-check-ai').forEach(btn => {
        btn.addEventListener('click', () => {
          const soal = btn.dataset.soal;
          const combined = ['a','b','c'].map(sub => $(`answer${soal}${sub}`).value).join('\n\n');
          if (!combined.trim()) { showToast('Kosong', 'Tulis jawaban dulu.', 'error'); return; }
          const result = detectAIContent(combined);
          const badge = $(`aiBadge${soal}`), warn = $(`aiWarning${soal}`);
          if (result.isAI) { badge.textContent = '⚠️ Terindikasi AI'; badge.style.background = '#fee2e2'; badge.style.color = '#b91c1c'; warn.style.display = 'flex'; }
          else { badge.textContent = '✅ Bebas AI'; badge.style.background = '#dcfce7'; badge.style.color = '#166534'; warn.style.display = 'none'; }
        });
      });
      document.querySelectorAll('[data-submit]').forEach(btn => {
        btn.addEventListener('click', async () => {
          if (!currentUser || !currentSesi) return;
          const soal = parseInt(btn.dataset.submit);
          const parts = ['a','b','c'].map(sub => $(`answer${soal}${sub}`).value.trim());
          const combined = parts.join('\n\n');
          if (combined.length < 30) { showToast('Jawaban Pendek', 'Min 30 karakter.', 'error'); return; }
          const ai = detectAIContent(combined);
          if (ai.isAI) {
            $(`aiWarning${soal}`).style.display = 'flex';
            const badge = $(`aiBadge${soal}`);
            badge.textContent = '⚠️ Terindikasi AI'; badge.style.background = '#fee2e2'; badge.style.color = '#b91c1c';
            showToast('Tidak Bisa Dikirim', 'Terindikasi AI.', 'error'); return;
          }
          const { score, feedback } = autoGrade(combined, soal);
          $(`score${soal}`).textContent = score;
          const fbList = $(`feedback${soal}`); fbList.innerHTML = '';
          feedback.forEach(fb => { const li = document.createElement('li'); const icon = document.createElement('i'); icon.className = `fas ${fb.icon}`; const span = document.createElement('span'); span.textContent = fb.text; li.appendChild(icon); li.appendChild(span); fbList.appendChild(li); });
          $(`result${soal}`).style.display = 'block';
          btn.disabled = true;
          const badge = $(`aiBadge${soal}`);
          badge.textContent = '✅ Bebas AI'; badge.style.background = '#dcfce7'; badge.style.color = '#166534';
          $(`aiWarning${soal}`).style.display = 'none';
          let student = STUDENTS_DATA.find(s => s.nim === currentUser.nim);
          if (student) {
            const sesiData = getStudentSesi(student, currentSesi);
            sesiData.jawaban[soal] = { a: parts[0], b: parts[1], c: parts[2] };
            if (sesiData.status === 'belum') sesiData.status = 'mengerjakan';
            const terisi = SOAL_DATA.filter(s => sesiData.jawaban[s.id] && sesiData.jawaban[s.id].a).length;
            sesiData.progress = Math.round((terisi / SOAL_DATA.length) * 100);
            await syncSaveStudent(student);
          }
          saveActivity({ studentId: currentUser.nim, nama: currentUser.nama, sesi: currentSesi, status: 'submitted', currentSoal: `Soal ${soal}`, currentSoalId: soal, totalChars: combined.length, lastUpdate: new Date().toISOString() });
          showToast('Terkirim', `Jawaban Soal ${soal} tersinkron ke cloud.`, 'success');
        });
      });

      // Auto-save
      document.querySelectorAll('.sub-soal-textarea').forEach(textarea => {
        textarea.addEventListener('input', () => {
          if (!currentUser || !currentSesi) return;
          clearTimeout(answerSaveTimeout);
          answerSaveTimeout = setTimeout(async () => {
            let totalChars = 0;
            SOAL_DATA.forEach(s => { ['a','b','c'].forEach(sub => { const ta = $(`answer${s.id}${sub}`); if (ta) totalChars += ta.value.length; }); });
            const activeSoalId = textarea.dataset.soal;
            const soalObj = SOAL_DATA.find(s => s.id == activeSoalId);
            await saveActivity({
              studentId: currentUser.nim, nama: currentUser.nama, sesi: currentSesi,
              status: 'writing',
              currentSoal: soalObj ? `Soal ${soalObj.id}` : '',
              currentSoalId: soalObj ? soalObj.id : null,
              totalChars: totalChars, lastUpdate: new Date().toISOString()
            });
          }, 1500);
        });
      });
    }

    function pindahSoal(soal) {
      currentSoal = parseInt(soal);
      document.querySelectorAll('.soal-nav-btn').forEach(b => b.classList.toggle('active', b.dataset.soal === String(soal)));
      document.querySelectorAll('.soal-content').forEach(c => c.classList.remove('active'));
      const el = $(`soal${soal}Content`);
      if (el) el.classList.add('active');
      if (currentUser && currentSesi) {
        const soalObj = SOAL_DATA.find(s => s.id === currentSoal);
        let totalChars = 0;
        SOAL_DATA.forEach(s => { ['a','b','c'].forEach(sub => { const ta = $(`answer${s.id}${sub}`); if (ta) totalChars += ta.value.length; }); });
        saveActivity({ studentId: currentUser.nim, nama: currentUser.nama, sesi: currentSesi, status: 'writing', currentSoal: soalObj ? `Soal ${soalObj.id}` : '', currentSoalId: soalObj ? soalObj.id : null, totalChars: totalChars, lastUpdate: new Date().toISOString() });
      }
    }

    function initAntiPaste() {
      document.querySelectorAll('.sub-soal-textarea').forEach(ta => {
        ta.addEventListener('paste', (e) => {
          e.preventDefault();
          const warnId = ta.id.replace('answer', 'pasteWarn');
          const warn = $(warnId);
          if (warn) { warn.classList.add('show'); ta.classList.add('paste-blocked'); setTimeout(() => { warn.classList.remove('show'); ta.classList.remove('paste-blocked'); }, 2000); }
          showToast('Paste Diblokir', 'Jawaban harus ditulis manual.', 'error', 2500);
        });
        ta.addEventListener('drop', (e) => { e.preventDefault(); showToast('Drop Diblokir', 'Jawaban harus ditulis manual.', 'error', 2500); });
        ta.addEventListener('input', () => {
          const counterId = ta.id.replace('answer', 'counter');
          const counter = $(counterId);
          if (counter) {
            const len = ta.value.length;
            counter.textContent = `${len} karakter`;
            counter.classList.remove('warn', 'ok');
            if (len >= 30) counter.classList.add('ok'); else if (len > 0) counter.classList.add('warn');
          }
        });
      });
    }

    function detectAIContent(text) {
      if (!text || text.trim().length < 20) return { isAI: false };
      const lower = text.toLowerCase();
      const phrases = ['sebagai ai','language model','saya tidak bisa','maaf, saya','berikut adalah langkah','pertama,','kedua,','ketiga,','dengan demikian','oleh karena itu','selain itu','namun demikian','pada dasarnya','secara umum','dapat disimpulkan'];
      let matchCount = 0; phrases.forEach(p => { if (lower.includes(p)) matchCount++; });
      const words = text.trim().split(/\s+/); const wordCount = words.length;
      const unique = new Set(words.map(w => w.toLowerCase().replace(/[^a-z]/g, '')));
      const diversity = unique.size / wordCount;
      const patterns = [/langkah\s+pertama/i,/langkah\s+kedua/i,/langkah\s+ketiga/i,/pertama,\s+/i,/kedua,\s+/i,/ketiga,\s+/i];
      let patternMatches = 0; patterns.forEach(p => { if (p.test(text)) patternMatches++; });
      let aiScore = 0;
      if (matchCount >= 4) aiScore += 40; else if (matchCount >= 2) aiScore += 20;
      if (patternMatches >= 3) aiScore += 30; else if (patternMatches >= 1) aiScore += 15;
      if (diversity < 0.5 && wordCount > 50) aiScore += 25;
      return { isAI: aiScore >= 45 };
    }

    function autoGrade(answer, soal) {
      const lower = answer.toLowerCase();
      const words = answer.trim().split(/\s+/).filter(w => w.length > 0);
      const wordCount = words.length;
      let score = 0; const feedback = [];
      let lengthScore = wordCount >= 150 ? 30 : wordCount >= 100 ? 24 : wordCount >= 60 ? 18 : wordCount >= 30 ? 10 : 5;
      score += lengthScore;
      feedback.push({ icon: lengthScore >= 24 ? 'fa-check-circle' : 'fa-info-circle', text: `Kedalaman: ${wordCount} kata (${lengthScore}/30)` });
      let keywords = [];
      if (soal === 1) keywords = ['evaluasi','kelayakan','revisi','ahli materi','ahli media','siswa','miskonsepsi','grafik','energi','perubahan energi'];
      else if (soal === 2) keywords = ['gelombang','bunyi','frekuensi','amplitudo','panjang gelombang','getaran','animasi','verifikasi','validasi','konten','alat','video'];
      else keywords = ['analisis','kebutuhan','observasi','wawancara','siswa','kesulitan','karakteristik','tujuan','materi','hukum newton','gaya','massa','percepatan','aksi-reaksi'];
      let matchCount = 0; keywords.forEach(k => { if (lower.includes(k)) matchCount++; });
      let keywordScore = Math.min(40, Math.round((matchCount / keywords.length) * 40));
      score += keywordScore;
      feedback.push({ icon: keywordScore >= 28 ? 'fa-check-circle' : 'fa-info-circle', text: `Kata kunci: ${matchCount}/${keywords.length} (${keywordScore}/40)` });
      let structureScore = 0;
      if (lower.includes('a.') || lower.includes('a)')) structureScore += 10;
      if (lower.includes('b.') || lower.includes('b)')) structureScore += 10;
      if (lower.includes('c.') || lower.includes('c)')) structureScore += 10;
      score += structureScore;
      feedback.push({ icon: structureScore >= 20 ? 'fa-check-circle' : 'fa-info-circle', text: `Struktur a/b/c: ${structureScore}/30` });
      const ai = detectAIContent(answer);
      if (ai.isAI) { score = Math.max(0, score - 30); feedback.push({ icon: 'fa-times-circle', text: 'Indikasi AI (-30)' }); }
      else feedback.push({ icon: 'fa-check-circle', text: 'Tidak terindikasi AI' });
      return { score: Math.min(100, Math.max(0, score)), feedback };
    }

    function tampilkanFeedbackMahasiswa(nim) {
      const student = STUDENTS_DATA.find(s => s.nim === nim);
      if (!student || !currentSesi) return;
      const sesiData = getStudentSesi(student, currentSesi);
      SOAL_DATA.forEach(s => {
        const panel = $(`feedbackDosen${s.id}`);
        if (!panel) return;
        const textEl = $(`fbText${s.id}`), nilaiEl = $(`fbNilai${s.id}`);
        const nilai = (sesiData.nilaiSoal && sesiData.nilaiSoal[s.id]) || null;
        const fb = (sesiData.feedback && sesiData.feedback[s.id]) || '';
        if (nilai !== null && fb) {
          panel.classList.add('show'); nilaiEl.textContent = nilai; textEl.textContent = fb;
        } else if (nilai !== null) {
          panel.classList.add('show'); nilaiEl.textContent = nilai; textEl.textContent = 'Dosen belum memberikan feedback tertulis.';
        } else panel.classList.remove('show');
      });
    }

    // ============================================================
    // EMAIL PANEL
    // ============================================================
    function updateEmailPanel(user) {
      if (!user || !currentSesi) return;
      const student = STUDENTS_DATA.find(s => s.nim === user.nim);
      if (!student) return;
      const sesiData = getStudentSesi(student, currentSesi);
      let totalNilai = 0, jumlahDinilai = 0;
      SOAL_DATA.forEach(s => {
        const n = sesiData.nilaiSoal && sesiData.nilaiSoal[s.id];
        if (typeof n === 'number') { totalNilai += n; jumlahDinilai++; }
      });
      const panel = $('emailNotifPanel');
      if (!panel) return;
      if (jumlahDinilai === SOAL_DATA.length && SOAL_DATA.length > 0) {
        const rataRata = Math.round(totalNilai / SOAL_DATA.length);
        $('emailSesi').textContent = currentSesi === 'pretest' ? 'Pre-Test' : 'Post-Test';
        $('emailTarget').textContent = user.email || '-';
        $('emailNamaMhs').textContent = user.nama || '-';
        $('emailNilaiAkhir').textContent = rataRata;
        panel.classList.add('show');
        if (sesiData.emailSent) {
          $('sendEmailBtn').disabled = true;
          $('sendEmailBtn').innerHTML = '<i class="fas fa-check-circle"></i> Sudah Terkirim';
        } else {
          $('sendEmailBtn').disabled = false;
          $('sendEmailBtn').innerHTML = '<i class="fas fa-paper-plane"></i> Kirim Nilai Akhir ke Email';
        }
      } else panel.classList.remove('show');
    }

    $('sendEmailBtn')?.addEventListener('click', async () => {
      if (!currentUser || !currentSesi) return;
      const student = STUDENTS_DATA.find(s => s.nim === currentUser.nim);
      if (!student) return;
      const sesiData = getStudentSesi(student, currentSesi);
      let totalNilai = 0, jumlahDinilai = 0;
      SOAL_DATA.forEach(s => {
        const n = sesiData.nilaiSoal && sesiData.nilaiSoal[s.id];
        if (typeof n === 'number') { totalNilai += n; jumlahDinilai++; }
      });
      const rataRata = Math.round(totalNilai / SOAL_DATA.length);
      $('sendEmailBtn').disabled = true;
      $('sendEmailBtn').innerHTML = '<i class="fas fa-spinner fa-spin"></i> Mengirim...';
      try {
        if (DEMO_MODE) {
          await new Promise(r => setTimeout(r, 1500));
          showToast('DEMO Mode', `Nilai ${rataRata} akan dikirim ke ${currentUser.email}`, 'info', 5000);
        } else {
          await emailjs.send(EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_ID, {
            to_email: currentUser.email, to_name: currentUser.nama, app_name: 'Portal Ujian UNM',
            tipe: 'nilai_akhir', sesi: currentSesi === 'pretest' ? 'Pre-Test' : 'Post-Test',
            nilai_akhir: rataRata, reply_to: 'noreply@unm.ac.id'
          });
          showToast('Email Terkirim!', `Nilai ${rataRata} dikirim ke ${currentUser.email}`, 'success', 6000);
        }
        sesiData.emailSent = true;
        await syncSaveStudent(student);
        $('sendEmailBtn').innerHTML = '<i class="fas fa-check-circle"></i> Sudah Terkirim';
      } catch (err) {
        showToast('Gagal Kirim', err.text || err.message, 'error', 5000);
        $('sendEmailBtn').disabled = false;
        $('sendEmailBtn').innerHTML = '<i class="fas fa-paper-plane"></i> Coba Lagi';
      }
    });

    // ============================================================
    // DOSEN: RENDER LIST
    // ============================================================
    function renderStudentList() {
      const list = $('studentList');
      if (!list) return;
      list.innerHTML = '';
      if (STUDENTS_DATA.length === 0) {
        list.innerHTML = '<div style="padding:40px;text-align:center;color:#94a3b8;"><i class="fas fa-user-slash" style="font-size:2.5rem;margin-bottom:12px;display:block;"></i>Belum ada peserta ujian.</div>';
        updateMonitoringStats(); return;
      }
      STUDENTS_DATA.forEach(student => {
        const row = document.createElement('div');
        row.className = 'student-row' + (selectedStudentId === student.id ? ' selected' : '');
        const act = ACTIVITIES_DATA.find(a => a.studentId === student.nim);
        const isWriting = act && act.status === 'writing';
        const isOnline = act && (act.status === 'writing' || act.status === 'online');
        if (isWriting) row.classList.add('writing-now');
        const initials = student.nama.split(' ').map(n => n[0]).slice(0, 2).join('');
        const sesiPre = getStudentSesi(student, 'pretest');
        const sesiPost = getStudentSesi(student, 'posttest');
        let activeSesi = 'pretest', statusLabel = 'Belum', statusClass = 'belum', nilaiAkhir = 0;
        if (sesiPost.status === 'selesai' || sesiPost.status === 'mengerjakan') activeSesi = 'posttest';
        const sesiData = activeSesi === 'pretest' ? sesiPre : sesiPost;
        if (sesiData.status === 'selesai') { statusLabel = 'Selesai'; statusClass = 'selesai'; }
        else if (sesiData.status === 'mengerjakan') { statusLabel = 'Mengerjakan'; statusClass = 'mengerjakan'; }
        let total = 0, count = 0;
        SOAL_DATA.forEach(s => { const n = sesiData.nilaiSoal && sesiData.nilaiSoal[s.id]; if (typeof n === 'number') { total += n; count++; } });
        nilaiAkhir = count > 0 ? Math.round(total / count) : 0;
        let scoreClass = '';
        if (nilaiAkhir >= 80) scoreClass = 'good'; else if (nilaiAkhir >= 60) scoreClass = 'warn'; else if (nilaiAkhir > 0) scoreClass = 'bad';
        row.innerHTML = `
          <div class="student-avatar" style="background:${student.warna || '#1e4b7c'};">${initials}${isWriting ? '<div class="writing-dot"></div>' : isOnline ? '<div class="online-dot"></div>' : ''}</div>
          <div class="student-info">
            <div class="name">${student.nama}${isWriting ? '<span class="writing-indicator"><span class="pulse-dot"></span>MENULIS</span>' : ''}</div>
            <div class="nim">${student.nim} · ${student.prodi || '-'}</div>
            <div class="email-mini"><i class="fas fa-envelope"></i> ${student.email || '-'}</div>
            ${act && act.currentSoal ? `<div class="sesi-mini"><i class="fas fa-pen"></i> ${act.currentSoal} (${act.totalChars || 0} char)</div>` : ''}
          </div>
          <div class="student-status">
            <span class="status-pill ${statusClass}">${statusLabel}</span>
            <span class="status-pill ${activeSesi}">${activeSesi === 'pretest' ? 'Pre-Test' : 'Post-Test'}</span>
            <div class="student-score-badge ${scoreClass}">${nilaiAkhir}</div>
          </div>
          <div style="display:flex; flex-direction:column; gap:6px;">
            <button class="btn-action-small" data-view-student="${student.id}"><i class="fas fa-eye"></i></button>
            <button class="btn-delete-student" data-del-student="${student.id}"><i class="fas fa-trash"></i></button>
          </div>
        `;
        row.addEventListener('click', (e) => {
          if (e.target.closest('button')) return;
          selectedStudentId = student.id;
          renderStudentList(); renderMahasiswaSelector();
          updateSelectedStudentInfo(student); updateRubrikUIForSoal();
        });
        row.querySelector('[data-view-student]').addEventListener('click', (e) => {
          e.stopPropagation();
          selectedStudentId = student.id;
          document.querySelectorAll('.dashboard-tab').forEach(t => t.classList.remove('active'));
          document.querySelectorAll('.dashboard-tab-content').forEach(c => c.classList.remove('active'));
          document.querySelector('[data-dashboard-tab="penilaian"]').classList.add('active');
          $('tabPenilaian').classList.add('active');
          renderStudentList(); renderMahasiswaSelector(); renderSoalPenilaianSelector();
          updateSelectedStudentInfo(student); updateRubrikUIForSoal();
        });
        row.querySelector('[data-del-student]').addEventListener('click', (e) => {
          e.stopPropagation();
          $('confirmTargetName').textContent = student.nama;
          $('confirmModal').classList.add('show');
          $('confirmDeleteBtn').dataset.deleteType = 'student';
          $('confirmDeleteBtn').dataset.deleteId = student.id;
        });
        list.appendChild(row);
      });
      updateMonitoringStats();
    }

    function updateMonitoringStats() {
      $('statTotal').textContent = STUDENTS_DATA.length;
      let mengerjakan = 0, selesai = 0, indikasiAI = 0;
      STUDENTS_DATA.forEach(s => {
        ['pretest', 'posttest'].forEach(sesi => {
          const sd = getStudentSesi(s, sesi);
          if (sd.status === 'mengerjakan') mengerjakan++;
          if (sd.status === 'selesai') selesai++;
        });
        if (s.aiStatus === 'terindikasi') indikasiAI++;
      });
      $('statMengerjakan').textContent = mengerjakan;
      $('statSelesai').textContent = selesai;
      $('statIndikasi').textContent = indikasiAI;
    }

    function renderMahasiswaSelector() {
      const sel = $('mahasiswaSelector');
      if (!sel) return;
      sel.innerHTML = '';
      if (STUDENTS_DATA.length === 0) {
        sel.innerHTML = '<span style="color:#94a3b8;font-size:0.9rem;">Belum ada mahasiswa.</span>';
        return;
      }
      STUDENTS_DATA.forEach(s => {
        const btn = document.createElement('button');
        btn.className = 'soal-penilaian-btn' + (selectedStudentId === s.id ? ' active' : '');
        btn.innerHTML = `<i class="fas fa-user"></i> ${s.nama.split(' ')[0]}`;
        btn.addEventListener('click', () => {
          selectedStudentId = s.id;
          renderMahasiswaSelector(); renderStudentList();
          updateSelectedStudentInfo(s); updateRubrikUIForSoal();
        });
        sel.appendChild(btn);
      });
    }

    function renderSoalPenilaianSelector() {
      const sel = $('soalPenilaianSelector');
      if (!sel) return;
      sel.innerHTML = '';
      SOAL_DATA.forEach(s => {
        const btn = document.createElement('button');
        btn.className = 'soal-penilaian-btn' + (currentDosenSoal === s.id ? ' active' : '');
        btn.dataset.soal = s.id;
        const student = STUDENTS_DATA.find(st => st.id === selectedStudentId);
        if (student) {
          const sesiData = getStudentSesi(student, 'posttest');
          const sudahDinilai = typeof (sesiData.nilaiSoal && sesiData.nilaiSoal[s.id]) === 'number';
          if (sudahDinilai) btn.classList.add('sudah-dinilai');
        }
        btn.innerHTML = `<span class="status-dot"></span> Soal ${s.id}`;
        btn.addEventListener('click', () => {
          currentDosenSoal = s.id;
          renderSoalPenilaianSelector();
          updateSelectedStudentInfo(STUDENTS_DATA.find(st => st.id === selectedStudentId));
          updateRubrikUIForSoal();
        });
        sel.appendChild(btn);
      });
    }

    function updateSelectedStudentInfo(student) {
      if (!student) return;
      $('selectedStudentName').textContent = student.nama;
      $('selectedStudentNim').textContent = student.nim;
      $('selectedStudentEmail').textContent = student.email || '-';
      const sesiPre = getStudentSesi(student, 'pretest');
      const sesiPost = getStudentSesi(student, 'posttest');
      const activeSesi = (sesiPost.status !== 'belum') ? 'posttest' : 'pretest';
      const sesiData = activeSesi === 'pretest' ? sesiPre : sesiPost;
      $('selectedStudentSesi').textContent = activeSesi === 'pretest' ? 'Pre-Test' : 'Post-Test';
      $('selectedStudentAI').textContent = student.aiStatus === 'terindikasi' ? '⚠️ Terindikasi' : '✅ Bebas';
      $('dosenSoalTitle').textContent = `Soal ${currentDosenSoal}`;
      const jawaban = (sesiData.jawaban && sesiData.jawaban[currentDosenSoal]) || {};
      const preview = $('dosenAnswerPreview');
      preview.innerHTML = '';
      if (!jawaban.a && !jawaban.b && !jawaban.c) {
        preview.innerHTML = '<div style="padding:20px;text-align:center;color:#94a3b8;"><i class="fas fa-inbox" style="font-size:2rem;margin-bottom:10px;display:block;"></i>Mahasiswa belum menjawab soal ini.</div>';
      } else {
        ['a', 'b', 'c'].forEach(sub => {
          const block = document.createElement('div');
          block.className = 'sub-answer-block';
          block.innerHTML = `<span class="sub-answer-label">Jawaban ${currentDosenSoal}${sub}:</span><div>${jawaban[sub] ? escapeHtml(jawaban[sub]).replace(/\n/g, '<br>') : '<em style="color:#94a3b8;">(kosong)</em>'}</div>`;
          preview.appendChild(block);
        });
      }
      const feedback = (sesiData.feedback && sesiData.feedback[currentDosenSoal]) || '';
      $('feedbackDosenInput').value = feedback;
    }

    function updateRubrikUIForSoal() {
      const container = $('rubrikAspectContainer');
      if (!container) return;
      container.innerHTML = '';
      if (!selectedStudentId) {
        container.innerHTML = '<div style="padding:40px;text-align:center;color:#94a3b8;"><i class="fas fa-user-slash" style="font-size:2.5rem;margin-bottom:12px;display:block;"></i>Pilih mahasiswa terlebih dahulu.</div>';
        return;
      }
      const student = STUDENTS_DATA.find(s => s.id === selectedStudentId);
      if (!student) return;
      const sesiPre = getStudentSesi(student, 'pretest');
      const sesiPost = getStudentSesi(student, 'posttest');
      const activeSesi = (sesiPost.status !== 'belum') ? 'posttest' : 'pretest';
      const sesiData = activeSesi === 'pretest' ? sesiPre : sesiPost;
      const rubrikSkor = (sesiData.rubrikSkor && sesiData.rubrikSkor[currentDosenSoal]) || {};
      currentRubrikSkor = { skills: rubrikSkor.skills ?? null, techniques: rubrikSkor.techniques ?? null, criteria: rubrikSkor.criteria ?? null };
      RUBRIK_DATA.forEach((aspek, idx) => {
        const block = document.createElement('div');
        block.className = 'rubrik-aspect-block';
        const currentSkor = currentRubrikSkor[aspek.id];
        let descriptorHTML = '';
        for (let skor = 0; skor <= 5; skor++) {
          const selected = currentSkor === skor;
          descriptorHTML += `
            <label class="rubrik-descriptor-item ${selected ? 'selected' : ''}" data-aspect="${aspek.id}" data-skor="${skor}">
              <div class="radio-wrapper">
                <input type="radio" name="rubrik_${aspek.id}" value="${skor}" ${selected ? 'checked' : ''}>
                <span class="skor-label">${skor}</span>
              </div>
              <div class="descriptor-text">${aspek.deskriptor[skor]}</div>
            </label>
          `;
        }
        block.innerHTML = `
          <div class="rubrik-aspect-header">
            <div class="aspect-title"><span class="aspect-number">${idx + 1}</span><div>${aspek.title}<small>${aspek.subtitle}</small></div></div>
            <div class="aspect-score-badge ${currentSkor !== null ? 'has-value' : ''}" id="badge_${aspek.id}">${currentSkor !== null ? currentSkor : '—'}</div>
          </div>
          <div class="rubrik-descriptor-list">${descriptorHTML}</div>
        `;
        container.appendChild(block);
      });
      container.querySelectorAll('.rubrik-descriptor-item').forEach(item => {
        item.addEventListener('click', () => {
          const aspect = item.dataset.aspect;
          const skor = parseInt(item.dataset.skor);
          currentRubrikSkor[aspect] = skor;
          container.querySelectorAll(`[data-aspect="${aspect}"]`).forEach(i => i.classList.remove('selected'));
          item.classList.add('selected');
          item.querySelector('input[type="radio"]').checked = true;
          const badge = $(`badge_${aspect}`);
          if (badge) { badge.textContent = skor; badge.classList.add('has-value'); }
          updateTotalNilaiRubrik();
        });
      });
      updateTotalNilaiRubrik();
    }

    function updateTotalNilaiRubrik() {
      const values = [currentRubrikSkor.skills, currentRubrikSkor.techniques, currentRubrikSkor.criteria];
      const dinilai = values.filter(v => v !== null).length;
      const total = values.reduce((sum, v) => sum + (v || 0), 0);
      const nilaiAkhir = dinilai === 3 ? Math.round((total / 15) * 100) : 0;
      $('totalNilaiRubrik').textContent = nilaiAkhir;
      const progressInfo = $('rubrikProgressInfo');
      progressInfo.textContent = `${dinilai} dari 3 aspek dinilai`;
      progressInfo.classList.toggle('complete', dinilai === 3);
      $('saveGradeBtn').disabled = dinilai < 3;
    }

    $('saveGradeBtn')?.addEventListener('click', async () => {
      if (!selectedStudentId) return;
      const student = STUDENTS_DATA.find(s => s.id === selectedStudentId);
      if (!student) return;
      const sesiPre = getStudentSesi(student, 'pretest');
      const sesiPost = getStudentSesi(student, 'posttest');
      const activeSesi = (sesiPost.status !== 'belum') ? 'posttest' : 'pretest';
      const sesiData = activeSesi === 'pretest' ? sesiPre : sesiPost;
      const total = currentRubrikSkor.skills + currentRubrikSkor.techniques + currentRubrikSkor.criteria;
      const nilaiAkhir = Math.round((total / 15) * 100);
      sesiData.nilaiSoal[currentDosenSoal] = nilaiAkhir;
      sesiData.rubrikSkor[currentDosenSoal] = { ...currentRubrikSkor };
      sesiData.feedback[currentDosenSoal] = $('feedbackDosenInput').value.trim();
      sesiData.status = 'selesai';
      await syncSaveStudent(student);
      $('gradeSavedNotif').style.display = 'flex';
      setTimeout(() => { $('gradeSavedNotif').style.display = 'none'; }, 3000);
      showToast('Nilai Tersimpan', `Soal ${currentDosenSoal} · Nilai ${nilaiAkhir}`, 'success');
      sendNotification(student.nim, 'mahasiswa', { title: 'Nilai Baru', desc: `Soal ${currentDosenSoal} telah dinilai: ${nilaiAkhir}`, type: 'success', icon: 'fa-star' });
      renderStudentList(); renderSoalPenilaianSelector(); renderDatabase();
    });

    // ============================================================
    // DATABASE
    // ============================================================
    function renderDatabase() {
      const tbody = $('dbTableBody');
      if (!tbody) return;
      tbody.innerHTML = '';
      const filterSesi = $('filterSesi').value;
      const filterStatus = $('filterStatus').value;
      const filterAI = $('filterAI').value;
      const search = ($('filterSearch').value || '').toLowerCase().trim();
      let rows = [];
      STUDENTS_DATA.forEach(student => {
        const sesiList = filterSesi === 'all' ? ['pretest', 'posttest'] : [filterSesi];
        sesiList.forEach(sesi => {
          const sd = getStudentSesi(student, sesi);
          if (filterStatus !== 'all' && sd.status !== filterStatus) return;
          if (filterAI !== 'all') {
            if (filterAI === 'bebas' && student.aiStatus === 'terindikasi') return;
            if (filterAI === 'terindikasi' && student.aiStatus !== 'terindikasi') return;
          }
          if (search) {
            const haystack = `${student.nama} ${student.nim} ${student.email || ''}`.toLowerCase();
            if (!haystack.includes(search)) return;
          }
          const nilaiPerSoal = {};
          SOAL_DATA.forEach(s => {
            const n = sd.nilaiSoal && sd.nilaiSoal[s.id];
            nilaiPerSoal[s.id] = typeof n === 'number' ? n : null;
          });
          const nilaiArr = Object.values(nilaiPerSoal).filter(v => v !== null);
          const rataRata = nilaiArr.length > 0 ? Math.round(nilaiArr.reduce((a,b) => a+b, 0) / nilaiArr.length) : 0;
          rows.push({ student, sesi, sd, nilaiPerSoal, rataRata });
        });
      });
      const nilaiAll = rows.map(r => r.rataRata).filter(v => v > 0);
      $('dbTotal').textContent = rows.length;
      $('dbRataRata').textContent = nilaiAll.length > 0 ? Math.round(nilaiAll.reduce((a,b) => a+b, 0) / nilaiAll.length) : 0;
      $('dbTertinggi').textContent = nilaiAll.length > 0 ? Math.max(...nilaiAll) : 0;
      $('dbTerendah').textContent = nilaiAll.length > 0 ? Math.min(...nilaiAll) : 0;
      if (rows.length === 0) {
        tbody.innerHTML = '<tr><td colspan="12" class="db-empty"><i class="fas fa-database"></i>Belum ada data.</td></tr>';
        return;
      }
      rows.forEach((r, idx) => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${idx + 1}</td>
          <td><span class="sesi-tag ${r.sesi}">${r.sesi === 'pretest' ? 'Pre-Test' : 'Post-Test'}</span></td>
          <td><strong>${r.student.nama}</strong></td>
          <td>${r.student.nim}</td>
          <td>${r.student.email || '-'}</td>
          <td class="aspek-score">${r.nilaiPerSoal[1] ?? '—'}</td>
          <td class="aspek-score">${r.nilaiPerSoal[2] ?? '—'}</td>
          <td class="aspek-score">${r.nilaiPerSoal[3] ?? '—'}</td>
          <td class="aspek-score"><strong>${r.rataRata}</strong></td>
          <td><span class="ai-tag ${r.student.aiStatus === 'terindikasi' ? 'indikasi' : 'bebas'}">${r.student.aiStatus === 'terindikasi' ? 'Indikasi' : 'Bebas'}</span></td>
          <td><span class="status-db ${r.sd.status}">${r.sd.status}</span></td>
          <td><button class="btn-delete-db" data-del-db="${r.student.id}_${r.sesi}"><i class="fas fa-trash"></i></button></td>
        `;
        tr.querySelector('[data-del-db]').addEventListener('click', () => {
          $('confirmTargetName').textContent = `${r.student.nama} - ${r.sesi === 'pretest' ? 'Pre-Test' : 'Post-Test'}`;
          $('confirmModal').classList.add('show');
          $('confirmDeleteBtn').dataset.deleteType = 'db';
          $('confirmDeleteBtn').dataset.deleteStudentId = r.student.id;
          $('confirmDeleteBtn').dataset.deleteSesi = r.sesi;
        });
        tbody.appendChild(tr);
      });
    }

    $('filterSesi')?.addEventListener('change', renderDatabase);
    $('filterStatus')?.addEventListener('change', renderDatabase);
    $('filterAI')?.addEventListener('change', renderDatabase);
    $('filterSearch')?.addEventListener('input', renderDatabase);

    $('downloadCsvBtn')?.addEventListener('click', () => {
      const rows = [['No', 'Sesi', 'Nama', 'NIM', 'Email', 'Soal 1', 'Soal 2', 'Soal 3', 'Rata-rata', 'AI', 'Status']];
      let no = 1;
      STUDENTS_DATA.forEach(student => {
        ['pretest', 'posttest'].forEach(sesi => {
          const sd = getStudentSesi(student, sesi);
          const nilaiPerSoal = {};
          SOAL_DATA.forEach(s => {
            const n = sd.nilaiSoal && sd.nilaiSoal[s.id];
            nilaiPerSoal[s.id] = typeof n === 'number' ? n : '';
          });
          const nilaiArr = Object.values(nilaiPerSoal).filter(v => v !== '');
          const rataRata = nilaiArr.length > 0 ? Math.round(nilaiArr.reduce((a,b) => a+b, 0) / nilaiArr.length) : 0;
          rows.push([no++, sesi === 'pretest' ? 'Pre-Test' : 'Post-Test', student.nama, student.nim, student.email || '', nilaiPerSoal[1] ?? '', nilaiPerSoal[2] ?? '', nilaiPerSoal[3] ?? '', rataRata, student.aiStatus === 'terindikasi' ? 'Indikasi' : 'Bebas', sd.status]);
        });
      });
      const csv = rows.map(r => r.map(c => `"${String(c).replace(/"/g, '""')}"`).join(',')).join('\n');
      const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `database_nilai_unm_${new Date().toISOString().slice(0,10)}.csv`;
      a.click();
      URL.revokeObjectURL(url);
      showToast('CSV Diunduh', 'Database nilai berhasil diunduh.', 'success');
    });

    // ============================================================
    // DOSEN: EDIT SOAL
    // ============================================================
    function renderSoalEditList() {
      const list = $('soalEditList');
      if (!list) return;
      list.innerHTML = '';
      if (SOAL_DATA.length === 0) {
        list.innerHTML = '<div style="padding:30px;text-align:center;color:#94a3b8;">Belum ada soal.</div>';
        return;
      }
      SOAL_DATA.forEach(s => {
        const item = document.createElement('div');
        item.className = 'soal-edit-item';
        item.innerHTML = `
          <div class="soal-title">${s.judul}<small>${(s.deskripsi || '').slice(0, 80)}...</small></div>
          <span class="soal-badge">Soal ${s.id}</span>
          <div class="soal-edit-actions">
            <button class="btn-edit-soal" data-edit-soal="${s.id}"><i class="fas fa-edit"></i> Edit</button>
            <button class="btn-delete-soal" data-del-soal="${s.id}"><i class="fas fa-trash"></i> Hapus</button>
          </div>
        `;
        item.querySelector('[data-edit-soal]').addEventListener('click', (e) => { e.stopPropagation(); openEditSoalForm(s.id); });
        item.querySelector('[data-del-soal]').addEventListener('click', (e) => { e.stopPropagation(); openDeleteSoalConfirm(s.id, s.judul); });
        list.appendChild(item);
      });
    }

    function openEditSoalForm(id) {
      const soal = SOAL_DATA.find(s => s.id === id);
      if (!soal) return;
      editingSoalId = id;
      $('editSoalJudul').value = soal.judul || '';
      $('editSoalDeskripsi').value = soal.deskripsi || '';
      $('editSoalPertanyaanA').value = soal.pertanyaanA || '';
      $('editSoalPertanyaanB').value = soal.pertanyaanB || '';
      $('editSoalPertanyaanC').value = soal.pertanyaanC || '';
      $('soalEditFormContainer').classList.remove('hidden');
      $('soalEditFormContainer').scrollIntoView({ behavior: 'smooth' });
    }

    $('addSoalBtn')?.addEventListener('click', () => {
      const newId = SOAL_DATA.length > 0 ? Math.max(...SOAL_DATA.map(s => s.id)) + 1 : 1;
      editingSoalId = null;
      $('editSoalJudul').value = `Soal ${newId} — `;
      $('editSoalDeskripsi').value = '';
      $('editSoalPertanyaanA').value = '';
      $('editSoalPertanyaanB').value = '';
      $('editSoalPertanyaanC').value = '';
      $('soalEditFormContainer').classList.remove('hidden');
      $('soalEditFormContainer').scrollIntoView({ behavior: 'smooth' });
    });

    $('cancelEditSoalBtn')?.addEventListener('click', () => { $('soalEditFormContainer').classList.add('hidden'); editingSoalId = null; });

    $('saveEditSoalBtn')?.addEventListener('click', async () => {
      const judul = $('editSoalJudul').value.trim();
      const deskripsi = $('editSoalDeskripsi').value.trim();
      const pertanyaanA = $('editSoalPertanyaanA').value.trim();
      const pertanyaanB = $('editSoalPertanyaanB').value.trim();
      const pertanyaanC = $('editSoalPertanyaanC').value.trim();
      if (!judul || !deskripsi || !pertanyaanA || !pertanyaanB || !pertanyaanC) { showToast('Data Tidak Lengkap', 'Lengkapi semua field.', 'error'); return; }
      let newId;
      if (editingSoalId) {
        newId = editingSoalId;
        const idx = SOAL_DATA.findIndex(s => s.id === editingSoalId);
        if (idx >= 0) SOAL_DATA[idx] = { id: newId, judul, deskripsi, pertanyaanA, pertanyaanB, pertanyaanC };
      } else {
        newId = SOAL_DATA.length > 0 ? Math.max(...SOAL_DATA.map(s => s.id)) + 1 : 1;
        SOAL_DATA.push({ id: newId, judul, deskripsi, pertanyaanA, pertanyaanB, pertanyaanC });
      }
      await syncSaveSoal(SOAL_DATA);
      $('soalEditFormContainer').classList.add('hidden');
      editingSoalId = null;
      renderSoalEditList();
      showToast('Soal Disimpan', `Soal ${newId} tersinkron ke cloud.`, 'success');
      if ($('pretestJumlahSoal')) $('pretestJumlahSoal').textContent = SOAL_DATA.length;
      if ($('posttestJumlahSoal')) $('posttestJumlahSoal').textContent = SOAL_DATA.length;
    });

    function openDeleteSoalConfirm(id, judul) {
      $('confirmTargetName').textContent = judul;
      $('confirmModal').classList.add('show');
      $('confirmDeleteBtn').dataset.deleteType = 'soal';
      $('confirmDeleteBtn').dataset.deleteSoalId = id;
    }

    // ============================================================
    // DASHBOARD TABS
    // ============================================================
    document.querySelectorAll('.dashboard-tab').forEach(tab => {
      tab.addEventListener('click', () => {
        const target = tab.dataset.dashboardTab;
        document.querySelectorAll('.dashboard-tab').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.dashboard-tab-content').forEach(c => c.classList.remove('active'));
        tab.classList.add('active');
        const content = $(`tab${target.charAt(0).toUpperCase() + target.slice(1)}`);
        if (content) content.classList.add('active');
        if (target === 'monitoring') { renderStudentList(); renderLiveMonitor(); renderDeviceList(); }
        else if (target === 'penilaian') { renderMahasiswaSelector(); renderSoalPenilaianSelector(); updateRubrikUIForSoal(); }
        else if (target === 'soal') { renderSoalEditList(); }
        else if (target === 'database') { renderDatabase(); }
      });
    });

    // ============================================================
    // RESET PASSWORD
    // ============================================================
    $('forgotPasswordBtn')?.addEventListener('click', () => {
      $('resetStep1').classList.remove('hidden');
      $('resetStep2').classList.add('hidden');
      $('resetStep3').classList.add('hidden');
      $('resetInput').value = '';
      $('resetError1').classList.remove('show');
      $('resetModal').classList.add('show');
    });
    $('resetCancelBtn1')?.addEventListener('click', () => $('resetModal').classList.remove('show'));
    $('resetCancelBtn3')?.addEventListener('click', () => $('resetModal').classList.remove('show'));

    $('resetSendOtpBtn')?.addEventListener('click', async () => {
      const input = $('resetInput').value.trim();
      if (!input) { $('resetErrorText1').textContent = 'Masukkan NIM/NIDN/email.'; $('resetError1').classList.add('show'); return; }
      showLoading(true); await fetchAllUsers(); showLoading(false);
      const user = USERS_DATA.find(u => u.nim === input || u.nim.toLowerCase() === input.toLowerCase() || (u.email || '').toLowerCase() === input.toLowerCase());
      if (!user) { $('resetErrorText1').textContent = 'Akun tidak ditemukan.'; $('resetError1').classList.add('show'); return; }
      resetUser = user;
      resetOtpCode = generateOTP();
      $('resetEmailDisplay').textContent = user.email;
      $('resetStep1').classList.add('hidden');
      $('resetStep2').classList.remove('hidden');
      $('resetError2').classList.remove('show');
      $('resetSuccess2').classList.remove('show');
      document.querySelectorAll('[data-reset-otp]').forEach(i => { i.value = ''; i.disabled = true; });
      $('resetVerifyOtpBtn').disabled = true;
      $('resetVerifyOtpBtn').innerHTML = '<i class="fas fa-spinner fa-spin"></i> Mengirim...';
      try {
        await kirimEmailOTP(user.email, user.nama, resetOtpCode, 'reset_password');
        document.querySelectorAll('[data-reset-otp]').forEach(i => i.disabled = false);
        $('resetVerifyOtpBtn').disabled = false;
        $('resetVerifyOtpBtn').innerHTML = '<i class="fas fa-check"></i> Verifikasi';
        if (DEMO_MODE) { $('resetSuccessText2').textContent = `DEMO: Kode OTP Anda ${resetOtpCode}`; $('resetSuccess2').classList.add('show'); }
        else showToast('Email Terkirim!', `Cek inbox ${user.email}`, 'success', 5000);
        resetOtpSecondsLeft = 300;
        clearInterval(resetOtpTimerInterval);
        const updateTimer = () => {
          const m = String(Math.floor(resetOtpSecondsLeft / 60)).padStart(2, '0');
          const s = String(resetOtpSecondsLeft % 60).padStart(2, '0');
          $('resetOtpTimer').textContent = `${m}:${s}`;
          if (resetOtpSecondsLeft <= 0) { $('resetOtpTimer').textContent = 'Kadaluarsa'; clearInterval(resetOtpTimerInterval); }
        };
        updateTimer();
        resetOtpTimerInterval = setInterval(() => { resetOtpSecondsLeft--; updateTimer(); }, 1000);
        setTimeout(() => document.querySelector('[data-reset-otp="0"]').focus(), 100);
      } catch (err) {
        $('resetErrorText2').textContent = 'Gagal kirim: ' + (err.text || err.message);
        $('resetError2').classList.add('show');
        $('resetVerifyOtpBtn').disabled = false;
        $('resetVerifyOtpBtn').innerHTML = '<i class="fas fa-check"></i> Verifikasi';
      }
    });

    document.querySelectorAll('[data-reset-otp]').forEach((inp, idx, all) => {
      inp.addEventListener('input', (e) => {
        e.target.value = e.target.value.replace(/\D/g, '').slice(0, 1);
        if (e.target.value && idx < all.length - 1) all[idx + 1].focus();
      });
      inp.addEventListener('keydown', (e) => { if (e.key === 'Backspace' && !e.target.value && idx > 0) all[idx - 1].focus(); });
    });

    $('resetBackBtn')?.addEventListener('click', () => {
      $('resetStep1').classList.remove('hidden');
      $('resetStep2').classList.add('hidden');
      clearInterval(resetOtpTimerInterval);
    });

    $('resetVerifyOtpBtn')?.addEventListener('click', () => {
      const entered = Array.from(document.querySelectorAll('[data-reset-otp]')).map(i => i.value).join('');
      if (entered.length < 6) { $('resetErrorText2').textContent = 'Masukkan 6 digit.'; $('resetError2').classList.add('show'); return; }
      if (entered !== resetOtpCode) { $('resetErrorText2').textContent = 'Kode salah.'; $('resetError2').classList.add('show'); return; }
      clearInterval(resetOtpTimerInterval);
      $('resetStep2').classList.add('hidden');
      $('resetStep3').classList.remove('hidden');
      $('resetError3').classList.remove('show');
      $('resetNewPassword').value = '';
      $('resetNewPasswordConfirm').value = '';
    });

    $('resetSavePasswordBtn')?.addEventListener('click', async () => {
      const pass = $('resetNewPassword').value;
      const passC = $('resetNewPasswordConfirm').value;
      if (!pass || pass.length < 6) { $('resetErrorText3').textContent = 'Min 6 karakter.'; $('resetError3').classList.add('show'); return; }
      if (pass !== passC) { $('resetErrorText3').textContent = 'Konfirmasi tidak cocok.'; $('resetError3').classList.add('show'); return; }
      resetUser.password = pass;
      await syncSaveUser(resetUser);
      $('resetModal').classList.remove('show');
      resetUser = null; resetOtpCode = '';
      showToast('Password Diubah', 'Silakan login dengan password baru.', 'success', 6000);
    });

    $('resetNewPassword')?.addEventListener('input', () => {
      const val = $('resetNewPassword').value;
      const fill = $('passwordStrengthFill');
      const text = $('passwordStrengthText');
      let strength = 0;
      if (val.length >= 6) strength++;
      if (val.length >= 10) strength++;
      if (/[A-Z]/.test(val)) strength++;
      if (/[0-9]/.test(val)) strength++;
      if (/[^A-Za-z0-9]/.test(val)) strength++;
      const colors = ['#dc2626', '#d97706', '#eab308', '#84cc16', '#16a34a'];
      const labels = ['Sangat Lemah', 'Lemah', 'Cukup', 'Kuat', 'Sangat Kuat'];
      if (val.length === 0) { fill.style.width = '0%'; text.textContent = ''; }
      else {
        const idx = Math.min(strength, 4);
        fill.style.width = ((idx + 1) * 20) + '%';
        fill.style.background = colors[idx];
        text.textContent = labels[idx];
        text.style.color = colors[idx];
      }
    });

    // ============================================================
    // INIT
    // ============================================================
    setTimeout(checkUrlPairing, 2000);
    setTimeout(() => updateAdminStats(), 2500);

    console.log('%c🎓 Portal Ujian Pendidikan Fisika UNM — Multi-Device', 'background:linear-gradient(135deg,#1e4b7c,#3b82f6);color:white;padding:12px 24px;border-radius:12px;font-size:16px;font-weight:bold;');
    console.log('%c🌐 Device ID: ' + DEVICE_ID, 'color:#7c3aed;font-weight:bold;');
    console.log('%c🖥️ Device Type: ' + getDeviceType(), 'color:#2563eb;font-weight:bold;');
    console.log('%c✅ Realtime Multi-Device: ' + (FIREBASE_READY ? 'ON' : 'OFF (Mode Lokal)'), 'color:#15803d;font-weight:bold;');

  })();
</script>
</body>
</html>
