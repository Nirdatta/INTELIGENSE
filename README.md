<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>INTELLIGENCE TERMINAL v6.2 - BORIS</title>
    <style>
        :root {
            --matrix-green: #00ff41;
            --dark-bg: #000000;
            --dim-green: #003b00;
            --highlight-bg: rgba(0,255,65,0.15);
            --alert-red: #ff0000;
        }
        * { box-sizing: border-box; }
        body {
            background: var(--dark-bg);
            color: var(--matrix-green);
            font-family: 'Courier New', monospace;
            margin: 0;
            padding: 10px;
            position: relative;
            overflow-x: hidden;
            min-height: 100vh;
        }
        #matrix-bg {
            position: fixed;
            top: 0; left: 0;
            width: 100%; height: 100%;
            z-index: -1;
            opacity: 0.09;
            pointer-events: none;
        }

        /* Login & Overlays */
        #login-overlay, #operator-overlay, #admin-modal {
            position: fixed;
            inset: 0;
            background: var(--dark-bg);
            z-index: 9999;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 15px;
        }
        .login-box {
            border: 2px solid var(--matrix-green);
            padding: 30px 20px;
            text-align: center;
            box-shadow: 0 0 25px var(--dim-green);
            width: 100%;
            max-width: 420px;
        }

        input, button, .op-btn {
            background: transparent;
            border: 2px solid var(--matrix-green);
            color: var(--matrix-green);
            padding: 14px;
            font-size: 18px;
            margin: 10px auto;
            width: 100%;
            max-width: 320px;
            outline: none;
            cursor: pointer;
            display: block;
            min-height: 48px; /* touch-friendly */
            touch-action: manipulation;
        }
        button:hover, .op-btn:hover {
            background: var(--matrix-green);
            color: var(--dark-bg);
        }
        #login-ok-btn {
            max-width: 180px;
            font-size: 18px;
            padding: 16px;
        }

        /* Main content */
        #main-content { display: none; }
        .header {
            border: 1px solid var(--matrix-green);
            padding: 15px;
            margin-bottom: 20px;
            text-align: center;
        }
        .header h1 { font-size: 1.6rem; margin: 0 0 10px; }
        .header p { margin: 5px 0; font-size: 1.1rem; }

        .search-box {
            position: relative;
            margin: 0 auto 25px;
            max-width: 100%;
        }
        #searchInput {
            width: 100%;
            padding: 14px;
            background: var(--dark-bg);
            border: 1px solid var(--matrix-green);
            color: var(--matrix-green);
            font-size: 1.1rem;
            text-transform: uppercase;
        }

        .result-card {
            display: none;
            background: var(--dark-bg);
            border: 1px solid var(--matrix-green);
            padding: 20px;
            margin: 0 auto 25px;
            max-width: 1000px;
        }
        @media (max-width: 768px) {
            .result-card { padding: 15px; }
            .identikit-title { font-size: 1.4rem; padding: 6px 12px; }
            .data { font-size: 1.05rem; }
            .doc-content { font-size: 0.95rem; }
            .action-buttons {
                flex-direction: column;
                gap: 12px;
            }
            button, .export-btn, .pdf-btn, .edit-btn {
                width: 100%;
                font-size: 1.05rem;
                padding: 14px;
            }
        }

        .action-buttons {
            margin: 25px 0 15px;
            display: flex;
            justify-content: center;
            gap: 15px;
            flex-wrap: wrap;
        }

        /* Inline edit form - più compatto su mobile */
        .inline-edit-form {
            padding: 15px;
            border: 2px dashed var(--matrix-green);
            background: rgba(0,80,0,0.25);
            margin: 12px 0;
        }
        .inline-edit-form textarea {
            font-size: 0.95rem;
            min-height: 80px;
        }
        @media (max-width: 768px) {
            .inline-edit-form h3 { font-size: 1.2rem; }
            .inline-edit-form textarea { font-size: 0.9rem; }
        }

        /* Status bar */
        #status-bar {
            position: fixed;
            bottom: 0; left: 0;
            width: 100%;
            background: rgba(0,0,0,0.95);
            border-top: 2px solid var(--matrix-green);
            padding: 8px;
            text-align: center;
            font-size: 0.85rem;
            z-index: 10000;
        }
        @media (max-width: 768px) {
            #status-bar { font-size: 0.75rem; padding: 6px; }
        }

        /* Altri stili invariati (staff-tag, doc-section, ecc.) */
        .staff-highlight { border: 4px double var(--matrix-green)!important; background: var(--highlight-bg)!important; }
        .staff-tag { background: var(--matrix-green); color: var(--dark-bg); padding: 5px 12px; font-weight: bold; margin-bottom: 12px; display: inline-block; }
        .doc-section { margin-top: 25px; padding: 15px; border: 1px dashed var(--matrix-green); background: rgba(0,59,0,0.1); }
        .suggestions { background: var(--dark-bg); border: 1px solid var(--matrix-green); max-height: 300px; overflow-y: auto; }
    </style>
</head>
<body>
    <canvas id="matrix-bg"></canvas>

    <!-- Login overlay -->
    <div id="login-overlay">
        <div class="login-box">
            <h2>INTELLIGENCE ARCHIVE SYSTEM v6.2</h2>
            <hr style="border-color:var(--matrix-green); margin:15px 0;">
            <p>RESTRICTED AREA - LEVEL 4</p>
            <input type="password" id="passInput" placeholder="PASSWORD" onkeydown="if(event.key==='Enter') checkPass()">
            <button id="login-ok-btn" onclick="checkPass()">OK / ACCEDI</button>
            <p id="error-msg" style="color:#ff0000; display:none; margin-top:15px; font-size:1.1rem;">ACCESS DENIED</p>
        </div>
    </div>

    <!-- Operator selection -->
    <div id="operator-overlay" style="display:none;">
        <div class="login-box">
            <h2>IDENTIFICAZIONE OPERATORE</h2>
            <button class="op-btn" onclick="selectOperator('BORIS')">BORIS</button>
            <button class="op-btn" onclick="selectOperator('NIR.D')">NIR.D</button>
            <button class="op-btn" onclick="selectOperator('PASIN 04')">PASIN 04</button>
            <button class="op-btn" onclick="showCustomInput()">ALTRO</button>
            <div id="custom-input" style="display:none; margin-top:20px;">
                <input type="text" id="customOperator" placeholder="NOME OPERATORE">
                <button class="op-btn" onclick="confirmCustom()">CONFERMA</button>
            </div>
        </div>
    </div>

    <!-- Main interface -->
    <div id="main-content">
        <div class="header">
            <h1>INTELLIGENCE TERMINAL v6.2 - BORIS MODE</h1>
            <p id="logged-as">LOGGED AS: ...</p>
            <button onclick="reopenOperator()" class="export-btn">REGISTRA NUOVO ACCESSO</button>
            <button onclick="showAdminModal()" class="export-btn">🛠 ADMIN PANEL</button>
            <button onclick="resetDatabase()" class="export-btn" style="background:#330000;">RESET DATABASE</button>
        </div>

        <div class="search-box">
            <input type="text" id="searchInput" placeholder="SEARCH IDENTITIES..." oninput="handleInput()" autocomplete="off">
            <div id="suggestions"></div>
        </div>

        <div id="cards-container"></div>

        <div id="cronologia">
            <h2>CRONOLOGIA ACCESSI</h2>
            <button onclick="exportLog()" class="export-btn">📤 ESPORTA TXT</button>
            <button onclick="clearLog()" class="export-btn" style="margin-left:10px;">🗑 CANCELLA</button>
            <div id="cronologia-list"></div>
        </div>
    </div>

    <!-- Admin modal (rimane quasi invariato) -->
    <div id="admin-modal" style="display:none;">
        <div id="admin-modal-content">
            <h2>🛠 ADMIN PANEL v6.2</h2>
            <select id="edit-select" onchange="loadForEdit()"></select><br><br>
            <input type="text" id="edit-nome" placeholder="NOME COMPLETO"><br>
            <input type="text" id="edit-id" placeholder="ID (es. card-nuovo)"><br>
            <label><input type="checkbox" id="edit-staff"> STAFF</label><br>
            <input type="text" id="edit-ruolo" placeholder="RUOLO (solo staff)"><br>
            <textarea id="edit-info" placeholder="INFO (una riga per campo)\nETÀ: 13\nIP: 151.16.25.87" rows="6"></textarea><br>
            <textarea id="edit-doc" placeholder="RAPPORTO TECNICO-INVESTIGATIVO" rows="8"></textarea><br>
            <div style="margin-top:25px; text-align:center;">
                <button onclick="saveSubject()" style="background:#004d00; border-color:#00ff88; color:#00ff88; font-size:1.1rem; padding:14px 30px;">INVIA / SALVA</button>
                <button onclick="closeAdminModal()" style="margin-left:15px; background:#330000; border-color:#ff3333; color:#ff8888; font-size:1.1rem; padding:14px 30px;">ANNULLA</button>
            </div>
        </div>
    </div>

    <div id="status-bar">
        CONNECTED TO: SECURE NODE CHIUPPANO-1912 | THREAT LEVEL: <span id="threat-level" style="color:#00ff41;">GREEN</span> | TIME: <span id="system-time"></span>
    </div>

    <script>
        // MATRIX RAIN (invariato)
        const canvas = document.getElementById('matrix-bg');
        const ctx = canvas.getContext('2d');
        function resizeCanvas() {
            canvas.height = window.innerHeight;
            canvas.width = window.innerWidth;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);
        const chars = "01アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワン0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ█▓▒░";
        const fontSize = 14;
        let drops = Array(Math.floor(canvas.width / fontSize)).fill(1);
        function drawMatrix() {
            ctx.fillStyle = 'rgba(0,0,0,0.05)';
            ctx.fillRect(0,0,canvas.width,canvas.height);
            ctx.fillStyle = '#00ff41';
            ctx.font = fontSize + 'px monospace';
            for (let i = 0; i < drops.length; i++) {
                const text = chars[Math.floor(Math.random() * chars.length)];
                ctx.fillText(text, i * fontSize, drops[i] * fontSize);
                if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) drops[i] = 0;
                drops[i]++;
            }
        }
        setInterval(drawMatrix, 35);

        // Il resto del JavaScript (login, soggetti, editing inline, renderAllCards, ecc.)
        // ... copia qui tutto il codice JavaScript dalla versione precedente che funzionava (quella con editing inline corretto)
        // Per brevità non lo riscrivo tutto qui, ma assicurati di tenere le versioni corrette di saveInlineEdit e renderAllCards che ti ho dato nell'ultimo messaggio

        // AVVIO
        renderAllCards();
        showAllCards();
        renderCronologia();

        function updateStatus(){
            setInterval(() => {
                document.getElementById('system-time').innerText = new Date().toLocaleTimeString('it-IT');
            }, 1000);
        }
        updateStatus();

        // Aggiunta resize per matrix rain su orientation change (mobile)
        window.addEventListener('orientationchange', resizeCanvas);
    </script>
</body>
</html>
