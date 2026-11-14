# Myhandwriting.html<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Meine Handschrift – mit Account</title>

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/10.14.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.14.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.14.0/firebase-firestore-compat.js"></script>

  <style>
    :root {
      --primary: #2563eb;
      --bg: #f1f5f9;
      --card: #ffffff;
      --border: #e2e8f0;
    }

    * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }

    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: var(--bg);
      margin: 0;
      padding: 16px;
      color: #1e293b;
      max-width: 900px;
      margin-inline: auto;
    }

    h1 { font-size: 1.5rem; margin: 0 0 6px 0; text-align: center; }
    .subtitle { text-align: center; color: #64748b; margin-bottom: 20px; font-size: 0.95rem; }

    .card {
      background: var(--card);
      border-radius: 16px;
      padding: 20px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.06);
      margin-bottom: 20px;
      border: 1px solid var(--border);
    }

    .section-title { font-weight: 600; margin-bottom: 12px; font-size: 1.05rem; }

    /* Auth */
    #auth-section { text-align: center; }
    .auth-form { display: flex; flex-direction: column; gap: 12px; max-width: 340px; margin: 0 auto; }
    input[type="email"], input[type="password"] {
      padding: 12px 14px;
      border: 2px solid var(--border);
      border-radius: 10px;
      font-size: 1rem;
    }
    .user-info {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: #f0f9ff;
      padding: 12px 16px;
      border-radius: 12px;
      margin-bottom: 16px;
    }

    /* Rest wie vorher */
    .char-grid { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 16px; }
    .char-btn {
      width: 42px; height: 42px;
      border: 2px solid var(--border);
      border-radius: 10px;
      background: white;
      font-size: 1.15rem; font-weight: 600;
      cursor: pointer; position: relative;
    }
    .char-btn.active { background: var(--primary); color: white; border-color: var(--primary); }
    .char-btn.has-drawing::after {
      content: ""; position: absolute; top: 3px; right: 3px;
      width: 8px; height: 8px; background: #22c55e; border-radius: 50%;
    }

    .canvas-wrapper {
      position: relative; background: #f1f5f9;
      border-radius: 12px; overflow: hidden;
      border: 2px dashed #cbd5e1; touch-action: none;
    }
    canvas { display: block; width: 100%; height: 220px; background: white; cursor: crosshair; }
    .canvas-hint {
      position: absolute; bottom: 10px; left: 0; right: 0;
      text-align: center; color: #94a3b8; font-size: 0.85rem; pointer-events: none;
    }

    .btn-row { display: flex; gap: 10px; margin-top: 14px; flex-wrap: wrap; }
    button {
      padding: 10px 18px; border-radius: 10px; border: none;
      font-weight: 600; font-size: 0.95rem; cursor: pointer;
    }
    .btn-primary { background: var(--primary); color: white; }
    .btn-secondary { background: #e2e8f0; color: #334155; }
    .btn-danger { background: #fee2e2; color: #b91c1c; }
    .btn-success { background: #22c55e; color: white; }

    textarea {
      width: 100%; min-height: 90px; padding: 14px;
      border: 2px solid var(--border); border-radius: 12px;
      font-size: 1.1rem; resize: vertical; font-family: inherit;
    }

    #result-container {
      background: white; border-radius: 12px; padding: 20px;
      min-height: 120px; border: 1px solid var(--border);
      overflow-x: auto; white-space: nowrap;
    }
    .letter-img { height: 64px; vertical-align: bottom; margin-right: 1px; }
    .space { display: inline-block; width: 18px; }
    .missing { display: inline-block; height: 64px; line-height: 64px; color: #ef4444; font-size: 1.4rem; margin: 0 2px; }

    .status { margin-top: 12px; font-size: 0.9rem; color: #64748b; }
    .progress { height: 6px; background: #e2e8f0; border-radius: 3px; margin-top: 10px; overflow: hidden; }
    .progress-bar { height: 100%; background: #22c55e; transition: width 0.3s; }

    .hidden { display: none !important; }
    .loading { opacity: 0.6; pointer-events: none; }
  </style>
</head>
<body>

  <h1>Meine Handschrift</h1>
  <p class="subtitle">Mit Account – deine Buchstaben werden in der Cloud gespeichert</p>

  <!-- ==================== ANMELDUNG ==================== -->
  <div class="card" id="auth-section">
    <div id="login-view">
      <div class="section-title">Anmelden / Registrieren</div>
      <div class="auth-form">
        <input type="email" id="email" placeholder="E-Mail-Adresse" />
        <input type="password" id="password" placeholder="Passwort (min. 6 Zeichen)" />
        <div class="btn-row" style="justify-content: center;">
          <button class="btn-primary" onclick="login()">Anmelden</button>
          <button class="btn-success" onclick="register()">Registrieren</button>
        </div>
        <p id="auth-error" style="color:#ef4444; font-size:0.9rem; min-height:20px;"></p>
      </div>
    </div>

    <div id="user-view" class="hidden">
      <div class="user-info">
        <div>
          Eingeloggt als:<br>
          <strong id="user-email"></strong>
        </div>
        <button class="btn-secondary" onclick="logout()">Abmelden</button>
      </div>
    </div>
  </div>

  <!-- ==================== HAUPTBEREICH (nur sichtbar wenn eingeloggt) ==================== -->
  <div id="main-content" class="hidden">

    <!-- 1. Buchstaben zeichnen -->
    <div class="card">
      <div class="section-title">1. Buchstaben zeichnen</div>
      <div class="char-grid" id="charGrid"></div>

      <div class="canvas-wrapper">
        <canvas id="drawCanvas"></canvas>
        <div class="canvas-hint" id="canvasHint">Zeichne den Buchstaben mit dem Apple Pencil</div>
      </div>

      <div class="btn-row">
        <button class="btn-secondary" onclick="clearCanvas()">Löschen</button>
        <button class="btn-primary" onclick="saveCurrentLetter()">Buchstaben speichern</button>
        <button class="btn-danger" onclick="deleteCurrentLetter()">Aus Speicher löschen</button>
      </div>

      <div class="status">
        Gespeicherte Buchstaben: <strong id="savedCount">0</strong>
        <div class="progress"><div class="progress-bar" id="progressBar" style="width:0%"></div></div>
      </div>
    </div>

    <!-- 2. Text umwandeln -->
    <div class="card">
      <div class="section-title">2. Text in Handschrift umwandeln</div>
      <textarea id="inputText" placeholder="Hier deinen Text eingeben..."></textarea>
      <div class="btn-row">
        <button class="btn-primary" onclick="renderHandwriting()">In Handschrift umwandeln</button>
        <button class="btn-secondary" onclick="downloadResult()">Als Bild speichern</button>
      </div>
      <div style="margin-top:18px;">
        <div class="section-title" style="font-size:0.95rem;">Ergebnis:</div>
        <div id="result-container">
          <span style="color:#94a3b8">Hier erscheint deine Handschrift...</span>
        </div>
      </div>
    </div>
  </div>

  <script>
    // ================== FIREBASE KONFIGURATION ==================
    // ←←← HIER DEINE FIREBASE-DATEN EINFÜGEN ←←←
    const firebaseConfig = {
      apiKey: "DEINE_API_KEY",
      authDomain: "DEIN_PROJEKT.firebaseapp.com",
      projectId: "DEIN_PROJEKT_ID",
      storageBucket: "DEIN_PROJEKT.appspot.com",
      messagingSenderId: "DEINE_SENDER_ID",
      appId: "DEINE_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const auth = firebase.auth();
    const db = firebase.firestore();

    // ================== STATE ==================
    const CHARACTERS = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789.,!?-()\"'";
    let currentChar = "A";
    let letters = {};
    let currentUser = null;
    let isDrawing = false;
    let lastX = 0, lastY = 0;

    const canvas = document.getElementById("drawCanvas");
    const ctx = canvas.getContext("2d");
    const dpr = window.devicePixelRatio || 1;

    // ================== AUTH ==================
    auth.onAuthStateChanged(async (user) => {
      currentUser = user;
      if (user) {
        document.getElementById("login-view").classList.add("hidden");
        document.getElementById("user-view").classList.remove("hidden");
        document.getElementById("main-content").classList.remove("hidden");
        document.getElementById("user-email").textContent = user.email;
        await loadLettersFromCloud();
        buildCharGrid();
        updateProgress();
      } else {
        document.getElementById("login-view").classList.remove("hidden");
        document.getElementById("user-view").classList.add("hidden");
        document.getElementById("main-content").classList.add("hidden");
        letters = {};
      }
    });

    async function register() {
      const email = document.getElementById("email").value.trim();
      const password = document.getElementById("password").value;
      try {
        await auth.createUserWithEmailAndPassword(email, password);
        showAuthError("");
      } catch (err) {
        showAuthError(err.message);
      }
    }

    async function login() {
      const email = document.getElementById("email").value.trim();
      const password = document.getElementById("password").value;
      try {
        await auth.signInWithEmailAndPassword(email, password);
        showAuthError("");
      } catch (err) {
        showAuthError(err.message);
      }
    }

    function logout() {
      auth.signOut();
    }

    function showAuthError(msg) {
      document.getElementById("auth-error").textContent = msg;
    }

    // ================== CLOUD SPEICHERN / LADEN ==================
    async function saveLettersToCloud() {
      if (!currentUser) return;
      try {
        await db.collection("users").doc(currentUser.uid).set({
          letters: letters,
          updatedAt: firebase.firestore.FieldValue.serverTimestamp()
        }, { merge: true });
      } catch (err) {
        console.error("Fehler beim Speichern:", err);
        alert("Speichern fehlgeschlagen: " + err.message);
      }
    }

    async function loadLettersFromCloud() {
      if (!currentUser) return;
      try {
        const doc = await db.collection("users").doc(currentUser.uid).get();
        if (doc.exists && doc.data().letters) {
          letters = doc.data().letters;
        } else {
          letters = {};
        }
      } catch (err) {
        console.error("Fehler beim Laden:", err);
      }
    }

    // ================== CANVAS ==================
    function resizeCanvas() {
      const rect = canvas.getBoundingClientRect();
      canvas.width = rect.width * dpr;
      canvas.height = rect.height * dpr;
      ctx.scale(dpr, dpr);
      ctx.lineCap = "round";
      ctx.lineJoin = "round";
      ctx.strokeStyle = "#1e293b";
      ctx.lineWidth = 3.5;
    }

    window.addEventListener("load", () => {
      resizeCanvas();
      buildCharGrid();
      selectChar("A");
    });
    window.addEventListener("resize", resizeCanvas);

    function getPos(e) {
      const rect = canvas.getBoundingClientRect();
      const clientX = e.clientX ?? e.touches?.[0]?.clientX;
      const clientY = e.clientY ?? e.touches?.[0]?.clientY;
      return { x: clientX - rect.left, y: clientY - rect.top };
    }

    function startDraw(e) {
      e.preventDefault();
      isDrawing = true;
      const pos = getPos(e);
      lastX = pos.x; lastY = pos.y;
      document.getElementById("canvasHint").style.opacity = "0";
    }

    function draw(e) {
      if (!isDrawing) return;
      e.preventDefault();
      const pos = getPos(e);
      ctx.beginPath();
      ctx.moveTo(lastX, lastY);
      ctx.lineTo(pos.x, pos.y);
      ctx.stroke();
      lastX = pos.x; lastY = pos.y;
    }

    function stopDraw() { isDrawing = false; }

    canvas.addEventListener("pointerdown", startDraw);
    canvas.addEventListener("pointermove", draw);
    canvas.addEventListener("pointerup", stopDraw);
    canvas.addEventListener("pointerleave", stopDraw);
    canvas.addEventListener("pointercancel", stopDraw);

    // ================== BUCHSTABEN ==================
    function buildCharGrid() {
      const grid = document.getElementById("charGrid");
      grid.innerHTML = "";
      for (const char of CHARACTERS) {
        const btn = document.createElement("button");
        btn.className = "char-btn";
        btn.textContent = char;
        btn.dataset.char = char;
        btn.onclick = () => selectChar(char);
        if (letters[char]) btn.classList.add("has-drawing");
        grid.appendChild(btn);
      }
    }

    function selectChar(char) {
      currentChar = char;
      document.querySelectorAll(".char-btn").forEach(btn => {
        btn.classList.toggle("active", btn.dataset.char === char);
      });
      clearCanvas(false);
      document.getElementById("canvasHint").textContent = `Zeichne jetzt „${char}“`;
      document.getElementById("canvasHint").style.opacity = "1";
    }

    function clearCanvas(showHint = true) {
      const rect = canvas.getBoundingClientRect();
      ctx.clearRect(0, 0, rect.width, rect.height);
      if (showHint) document.getElementById("canvasHint").style.opacity = "1";
    }

    async function saveCurrentLetter() {
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const hasInk = imageData.data.some((v, i) => i % 4 === 3 && v > 10);
      if (!hasInk) return alert("Bitte zuerst etwas zeichnen!");

      letters[currentChar] = canvas.toDataURL("image/png");
      document.querySelector(`.char-btn[data-char="${currentChar}"]`)?.classList.add("has-drawing");
      
      await saveLettersToCloud();
      updateProgress();

      // Nächsten Buchstaben vorschlagen
      const idx = CHARACTERS.indexOf(currentChar);
      if (idx < CHARACTERS.length - 1) selectChar(CHARACTERS[idx + 1]);
    }

    async function deleteCurrentLetter() {
      delete letters[currentChar];
      document.querySelector(`.char-btn[data-char="${currentChar}"]`)?.classList.remove("has-drawing");
      clearCanvas();
      await saveLettersToCloud();
      updateProgress();
    }

    function updateProgress() {
      const total = CHARACTERS.length;
      const saved = Object.keys(letters).length;
      document.getElementById("savedCount").textContent = `${saved} / ${total}`;
      document.getElementById("progressBar").style.width = `${(saved / total) * 100}%`;
    }

    // ================== TEXT → HANDSCHRIFT ==================
    function renderHandwriting() {
      const text = document.getElementById("inputText").value;
      const container = document.getElementById("result-container");
      container.innerHTML = "";

      if (!text.trim()) {
        container.innerHTML = '<span style="color:#94a3b8">Bitte Text eingeben</span>';
        return;
      }

      for (const char of text) {
        if (char === " ") {
          const space = document.createElement("span");
          space.className = "space";
          container.appendChild(space);
          continue;
        }
        if (letters[char]) {
          const img = document.createElement("img");
          img.src = letters[char];
          img.className = "letter-img";
          container.appendChild(img);
        } else {
          const missing = document.createElement("span");
          missing.className = "missing";
          missing.textContent = char;
          container.appendChild(missing);
        }
      }
    }

    function downloadResult() {
      const container = document.getElementById("result-container");
      if (!container.querySelector("img")) return alert("Zuerst Text umwandeln!");

      const tempCanvas = document.createElement("canvas");
      const tctx = tempCanvas.getContext("2d");
      const height = 80;
      tempCanvas.width = Math.max(container.scrollWidth + 40, 400) * dpr;
      tempCanvas.height = height * dpr;
      tctx.scale(dpr, dpr);
      tctx.fillStyle = "white";
      tctx.fillRect(0, 0, tempCanvas.width / dpr, height);

      let x = 10;
      container.querySelectorAll("img, .space, .missing").forEach(el => {
        if (el.tagName === "IMG") {
          tctx.drawImage(el, x, 8, el.naturalWidth * (64 / el.naturalHeight) || 40, 64);
          x += (el.naturalWidth * (64 / el.naturalHeight) || 40) + 1;
        } else if (el.classList.contains("space")) {
          x += 18;
        } else {
          tctx.fillStyle = "#ef4444";
          tctx.font = "40px system-ui";
          tctx.fillText(el.textContent, x, 55);
          x += 28;
        }
      });

      const link = document.createElement("a");
      link.download = "meine-handschrift.png";
      link.href = tempCanvas.toDataURL("image/png");
      link.click();
    }
  </script>
</body>
</html>
