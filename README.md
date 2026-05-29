<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>معايرة الأيادي | Controller Calibration</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Cairo:wght@300;400;600;700&display=swap');

  :root {
    --ps-blue: #003791;
    --ps-light: #00439C;
    --xbox-green: #107C10;
    --xbox-light: #52B043;
    --neon-cyan: #00F5FF;
    --neon-green: #39FF14;
    --neon-orange: #FF6B35;
    --dark-bg: #050A14;
    --panel-bg: rgba(10,20,40,0.85);
    --text-main: #E8F4FF;
    --text-dim: #7A9CC4;
    --border: rgba(0,245,255,0.2);
  }

  * { margin:0; padding:0; box-sizing:border-box; }

  body {
    background: var(--dark-bg);
    color: var(--text-main);
    font-family: 'Cairo', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }

  body::before {
    content:'';
    position:fixed; inset:0;
    background-image:
      linear-gradient(rgba(0,245,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,245,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events:none;
    z-index:0;
  }

  body::after {
    content:'';
    position:fixed; inset:0;
    background:
      radial-gradient(ellipse 600px 400px at 10% 20%, rgba(0,67,156,0.25) 0%, transparent 70%),
      radial-gradient(ellipse 500px 350px at 90% 80%, rgba(16,124,16,0.2) 0%, transparent 70%);
    pointer-events:none;
    z-index:0;
  }

  .container {
    position:relative; z-index:1;
    max-width:900px; margin:0 auto;
    padding:20px;
  }

  header {
    text-align:center;
    padding:30px 0 20px;
  }
  .logo-badge {
    display:inline-flex; align-items:center; gap:10px;
    background: rgba(0,245,255,0.05);
    border:1px solid var(--border);
    border-radius:50px;
    padding:6px 20px;
    margin-bottom:16px;
    font-family:'Orbitron', monospace;
    font-size:11px;
    letter-spacing:3px;
    color:var(--neon-cyan);
    text-transform:uppercase;
  }
  .logo-badge span { width:6px;height:6px;border-radius:50%;background:var(--neon-cyan);animation:pulse 1.5s infinite; }
  @keyframes pulse { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:0.4;transform:scale(0.8)} }

  h1 {
    font-family:'Orbitron', monospace;
    font-size:clamp(22px,4vw,36px);
    font-weight:900;
    background: linear-gradient(135deg, #00F5FF 0%, #ffffff 50%, #52B043 100%);
    -webkit-background-clip:text; -webkit-text-fill-color:transparent;
    background-clip:text;
    letter-spacing:2px;
    margin-bottom:6px;
  }
  .subtitle { color:var(--text-dim); font-size:14px; }

  .tabs {
    display:flex; gap:8px;
    margin:24px 0;
    background: rgba(255,255,255,0.03);
    border:1px solid var(--border);
    border-radius:12px;
    padding:6px;
  }
  .tab {
    flex:1; padding:12px 8px;
    border:none; border-radius:8px;
    background:transparent;
    color:var(--text-dim);
    font-family:'Cairo',sans-serif;
    font-size:13px;
    font-weight:600;
    cursor:pointer;
    transition:all 0.3s;
    display:flex; align-items:center; justify-content:center; gap:6px;
  }
  .tab:hover { color:var(--text-main); background:rgba(255,255,255,0.05); }
  .tab.active.ps {
    background:linear-gradient(135deg, var(--ps-blue), var(--ps-light));
    color:#fff;
    box-shadow:0 0 20px rgba(0,55,145,0.5);
  }
  .tab.active.xbox {
    background:linear-gradient(135deg, var(--xbox-green), var(--xbox-light));
    color:#fff;
    box-shadow:0 0 20px rgba(16,124,16,0.5);
  }

  .subtype-row {
    display:flex; gap:8px; flex-wrap:wrap;
    margin-bottom:20px;
  }
  .subtype-btn {
    padding:8px 16px;
    border-radius:20px;
    border:1px solid var(--border);
    background:transparent;
    color:var(--text-dim);
    font-family:'Cairo',sans-serif;
    font-size:12px;
    cursor:pointer;
    transition:all 0.3s;
  }
  .subtype-btn:hover { border-color:var(--neon-cyan); color:var(--neon-cyan); }
  .subtype-btn.active { background:rgba(0,245,255,0.1); border-color:var(--neon-cyan); color:var(--neon-cyan); }

  .main-grid {
    display:grid;
    grid-template-columns:1fr 1fr;
    gap:16px;
    margin-bottom:16px;
  }
  @media(max-width:600px){ .main-grid { grid-template-columns:1fr; } }

  .panel {
    background:var(--panel-bg);
    border:1px solid var(--border);
    border-radius:16px;
    padding:20px;
    backdrop-filter:blur(10px);
  }
  .panel-title {
    font-family:'Orbitron',monospace;
    font-size:11px;
    letter-spacing:2px;
    color:var(--neon-cyan);
    text-transform:uppercase;
    margin-bottom:16px;
    display:flex; align-items:center; gap:8px;
  }
  .panel-title::before {
    content:'';
    width:16px; height:2px;
    background:var(--neon-cyan);
    box-shadow:0 0 8px var(--neon-cyan);
  }

  .joystick-container {
    display:flex; flex-direction:column; align-items:center; gap:8px;
  }
  .joystick-wrap {
    position:relative;
    width:130px; height:130px;
  }
  .joystick-bg {
    width:100%; height:100%;
    border-radius:50%;
    background:radial-gradient(circle, rgba(0,245,255,0.05) 0%, rgba(0,245,255,0.02) 60%, transparent 100%);
    border:2px solid rgba(0,245,255,0.15);
    position:relative;
  }
  .joystick-bg::before {
    content:'';
    position:absolute; inset:0;
    background:
      linear-gradient(transparent 49%, rgba(0,245,255,0.1) 49%, rgba(0,245,255,0.1) 51%, transparent 51%),
      linear-gradient(90deg, transparent 49%, rgba(0,245,255,0.1) 49%, rgba(0,245,255,0.1) 51%, transparent 51%);
    border-radius:50%;
  }
  .range-circle {
    position:absolute;
    border-radius:50%;
    border:1px dashed rgba(0,245,255,0.08);
    top:50%; left:50%;
    transform:translate(-50%,-50%);
  }
  .stick-dot {
    position:absolute;
    width:18px; height:18px;
    border-radius:50%;
    background:var(--neon-cyan);
    box-shadow:0 0 12px var(--neon-cyan);
    top:50%; left:50%;
    transform:translate(-50%,-50%);
    transition:all 0.05s;
    cursor:crosshair;
  }
  .stick-dot.ps { background:var(--ps-light); box-shadow:0 0 12px var(--ps-light); }
  .stick-dot.xbox { background:var(--xbox-light); box-shadow:0 0 12px var(--xbox-light); }

  .stick-coords {
    font-family:'Orbitron',monospace;
    font-size:10px;
    color:var(--text-dim);
    text-align:center;
  }
  .stick-label { font-size:12px; color:var(--text-dim); margin-top:4px; }

  .trigger-section { margin-top:8px; }
  .trigger-row {
    display:flex; align-items:center; gap:10px;
    margin-bottom:10px;
  }
  .trigger-label {
    font-size:11px; color:var(--text-dim);
    width:40px; text-align:center;
    font-family:'Orbitron',monospace;
  }
  .trigger-bar-bg {
    flex:1; height:12px;
    background:rgba(255,255,255,0.05);
    border:1px solid var(--border);
    border-radius:6px;
    overflow:hidden;
    position:relative;
  }
  .trigger-bar-fill {
    height:100%; width:0%;
    border-radius:6px;
    transition:width 0.05s;
    background:linear-gradient(90deg, var(--neon-cyan), var(--neon-green));
    box-shadow:0 0 8px rgba(0,245,255,0.4);
    position:relative;
  }
  .trigger-bar-fill.ps { background:linear-gradient(90deg, var(--ps-blue), #0070CC); box-shadow:0 0 8px rgba(0,112,204,0.4); }
  .trigger-bar-fill.xbox { background:linear-gradient(90deg, var(--xbox-green), var(--xbox-light)); box-shadow:0 0 8px rgba(82,176,67,0.4); }
  .trigger-val {
    width:36px; text-align:center;
    font-family:'Orbitron',monospace;
    font-size:10px; color:var(--text-dim);
  }

  .slider-row {
    display:flex; align-items:center; gap:10px;
    margin-bottom:10px;
  }
  .slider-label { font-size:12px; color:var(--text-dim); min-width:90px; }
  input[type=range] {
    flex:1; height:4px;
    -webkit-appearance:none;
    background:rgba(0,245,255,0.15);
    border-radius:2px;
    outline:none;
    cursor:pointer;
  }
  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance:none;
    width:14px; height:14px;
    border-radius:50%;
    background:var(--neon-cyan);
    box-shadow:0 0 8px var(--neon-cyan);
    cursor:pointer;
