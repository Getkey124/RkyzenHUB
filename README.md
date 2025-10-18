<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Free Key — Zamas × Lennon</title>
  <style>
    :root {
      --accent: #ff7b00;
      --bg: #0d0d16;
      --card: rgba(20,20,30,0.9);
      --muted: #9aa0a6;
    }
    html,body { height: 100%; margin: 0; font-family: 'Poppins', system-ui, sans-serif; background: var(--bg); color: #fff; overflow: hidden; }

    /* Canvas (rain) */
    canvas { position: fixed; inset: 0; width: 100%; height: 100%; z-index: 0; }

    /* Center card */
    .key-box {
      position: absolute;
      left: 50%;
      bottom: 72px;
      transform: translateX(-50%);
      z-index: 3;
      background: var(--card);
      padding: 26px 36px;
      border-radius: 18px;
      box-shadow: 0 6px 30px rgba(0,0,0,0.6), 0 0 18px rgba(255,120,0,0.08);
      text-align: center;
      min-width: 300px;
      backdrop-filter: blur(6px);
    }
    .tag { color: #ffb347; font-weight: 600; letter-spacing: 0.6px; margin: 0; font-size: 0.95rem; }
    h1 { margin: 10px 0 14px; font-size: 1.4rem; color: #fff; }
    .key {
      display: inline-block;
      background: #12121a;
      padding: 10px 22px;
      border-radius: 10px;
      letter-spacing: 4px;
      font-weight: 600;
      margin-bottom: 12px;
      min-width: 180px;
    }
    .controls { margin-top: 10px; }
    .btn {
      display: inline-block;
      background: var(--accent);
      color: white;
      border: none;
      border-radius: 10px;
      padding: 10px 16px;
      margin: 6px 6px;
      font-size: 0.98rem;
      cursor: pointer;
      transition: transform .14s ease, filter .14s ease;
    }
    .btn:active { transform: translateY(1px) scale(.995); }
    .btn.secondary { background: transparent; border: 1px solid rgba(255,255,255,0.06); color: #ddd; }
    .footer { position: absolute; left: 0; right: 0; bottom: 12px; text-align:center; color: var(--muted); font-size: .88rem; z-index:2; }

    /* Toast */
    .toast {
      position: fixed;
      right: 18px;
      top: 18px;
      padding: 10px 14px;
      background: rgba(28,28,36,0.95);
      color: #fff;
      border-radius: 10px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.5);
      z-index: 70;
      transform: translateY(-6px);
      opacity: 0;
      transition: opacity .22s ease, transform .22s cubic-bezier(.2,.9,.3,1);
      pointer-events: none;
    }
    .toast.show { opacity: 1; transform: translateY(0); pointer-events: auto; }

    /* small responsive tweak */
    @media (max-width:420px) {
      .key-box { left: 50%; padding: 18px; min-width: 260px; bottom: 56px; border-radius: 14px; }
      .key { min-width: 150px; letter-spacing: 3px; }
    }
  </style>
</head>
<body>

  <canvas id="rain"></canvas>

  <div class="key-box" role="region" aria-label="Free key box">
    <p class="tag">🎃 HALLOWEEN</p>
    <h1>Free Key — Zamas × Lennon</h1>

    <div class="key" id="keyDisplay">••••••••</div>

    <div class="controls" aria-hidden="false">
      <button class="btn" id="getBtn" type="button">GET KEY</button>
      <button class="btn secondary" id="copyBtn" type="button">COPY KEY</button>
    </div>
  </div>

  <div class="footer">La selección y copia están bloqueadas.</div>

  <!-- Toast -->
  <div class="toast" id="toast">Link copied to clipboard</div>

  <script>
    // === CONFIG: your Roblox server link ===
    const ROBLOX_LINK = "https://www.roblox.com/share?code=9fa1135d2ef1754795fd9e5afdd41a72&type=Server";

    // GET KEY - generate fake key
    document.getElementById('getBtn').addEventListener('click', () => {
      const display = document.getElementById('keyDisplay');
      const key = Array.from({ length: 12 }, () => Math.random().toString(36).charAt(2)).join('').toUpperCase();
      display.textContent = key.match(/.{1,4}/g).join('-'); // show like XXXX-XXXX-XXXX
    });

    // COPY KEY - COPY ONLY (no redirect). Shows toast and changes button briefly.
    document.getElementById('copyBtn').addEventListener('click', async () => {
      const btn = document.getElementById('copyBtn');
      // Try clipboard API
      let copySucceeded = false;
      try {
        await navigator.clipboard.writeText(ROBLOX_LINK);
        copySucceeded = true;
        showToast("Link copied to clipboard");
      } catch (err) {
        // fallback: create temporary textarea
        const tmp = document.createElement('textarea');
        tmp.value = ROBLOX_LINK;
        document.body.appendChild(tmp);
        tmp.select();
        try {
          document.execCommand('copy');
          copySucceeded = true;
          showToast("Link copied to clipboard");
        } catch (e) {
          copySucceeded = false;
          showToast("Couldn't copy automatically — select and copy manually");
        }
        tmp.remove();
      }

      // Brief visual feedback on button
      const prevText = btn.textContent;
      if (copySucceeded) {
        btn.textContent = 'COPIED ✓';
      } else {
        btn.textContent = 'COPY FAILED';
      }
      btn.disabled = true;
      setTimeout(() => { btn.textContent = prevText; btn.disabled = false; }, 1600);
    });

    // Toast control
    const toastEl = document.getElementById('toast');
    let toastTimer = null;
    function showToast(msg = "") {
      toastEl.textContent = msg;
      toastEl.classList.add('show');
      clearTimeout(toastTimer);
      toastTimer = setTimeout(() => toastEl.classList.remove('show'), 2500);
    }

    // === Rain animation ===
    const canvas = document.getElementById('rain');
    const ctx = canvas.getContext('2d');
    let drops = [];
    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      // regenerate drops
      drops = [];
      const base = Math.floor(120 * (window.innerWidth / 1366));
      for (let i = 0; i < base; i++) {
        drops.push({
          x: Math.random() * canvas.width,
          y: Math.random() * canvas.height,
          l: Math.random() * 1 + 0.5,
          xs: -2 + Math.random() * 4,
          ys: Math.random() * 15 + 8
        });
      }
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.strokeStyle = 'rgba(150,150,255,0.22)';
      ctx.lineWidth = 1;
      ctx.beginPath();
      for (let i = 0; i < drops.length; i++) {
        const d = drops[i];
        ctx.moveTo(d.x, d.y);
        ctx.lineTo(d.x + d.l * d.xs, d.y + d.l * d.ys);
      }
      ctx.stroke();
      move();
    }

    function move() {
      for (let i = 0; i < drops.length; i++) {
        const d = drops[i];
        d.x += d.xs;
        d.y += d.ys;
        if (d.x > canvas.width || d.y > canvas.height) {
          d.x = Math.random() * canvas.width;
          d.y = -20;
        }
      }
    }

    setInterval(draw, 33);

    // accessibility: Enter key triggers click for focused buttons
    document.querySelectorAll('.btn').forEach(b => {
      b.addEventListener('keydown', (e) => { if (e.key === 'Enter') b.click(); });
    });
  </script>
</body>
</html>
