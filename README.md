<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>INTELLIGENCE TERMINAL v6.2 - BORIS</title>
    <style>
        :root { --matrix-green: #00ff41; --dark-bg: #000000; --dim-green: #003b00; --highlight-bg: rgba(0,255,65,0.15); --alert-red: #ff0000; }
        body { background:var(--dark-bg); color:var(--matrix-green); font-family:'Courier New',monospace; margin:0; padding:20px; position:relative; overflow:hidden; }
        #matrix-bg { position:fixed; top:0; left:0; width:100%; height:100%; z-index:-1; opacity:0.09; pointer-events:none; }
        #login-overlay, #operator-overlay, #admin-modal { position:fixed; top:0; left:0; width:100%; height:100%; background:var(--dark-bg); z-index:9999; display:flex; align-items:center; justify-content:center; }
        .login-box { border:2px solid var(--matrix-green); padding:40px; text-align:center; box-shadow:0 0 20px var(--dim-green); max-width:420px; }
        input, button, .op-btn { background:transparent; border:1px solid var(--matrix-green); color:var(--matrix-green); padding:12px; font-size:20px; margin:10px; width:300px; outline:none; cursor:pointer; }
        button:hover, .op-btn:hover { background:var(--matrix-green); color:var(--dark-bg); }
        #login-ok-btn { width:140px; font-size:18px; padding:14px; margin-top:15px; }
        #main-content { display:none; }
        .header { border:1px solid var(--matrix-green); padding:20px; margin-bottom:40px; text-align:center; }
        .search-box { position:relative; max-width:800px; margin:0 auto 40px; }
        #searchInput { width:100%; padding:15px; background:var(--dark-bg); border:1px solid var(--matrix-green); color:var(--matrix-green); font-size:20px; text-transform:uppercase; }
        #suggestions { position:absolute; width:100%; background:var(--dark-bg); border:1px solid var(--matrix-green); display:none; max-height:400px; overflow-y:auto; z-index:1001; }
        .suggestion-item { padding:15px; cursor:pointer; border-bottom:1px solid var(--dim-green); }
        .suggestion-item:hover { background:var(--matrix-green); color:var(--dark-bg); }
        .result-card { display:none; background:var(--dark-bg); border:1px solid var(--matrix-green); padding:40px; max-width:1000px; margin:0 auto 30px; }
        .staff-highlight { border:4px double var(--matrix-green)!important; background:var(--highlight-bg)!important; }
        .staff-tag { background:var(--matrix-green); color:var(--dark-bg); padding:5px 15px; font-weight:bold; margin-bottom:15px; display:inline-block; }
        .identikit-title { font-size:26px; font-weight:bold; margin-bottom:25px; text-transform:uppercase; background:var(--matrix-green); color:var(--dark-bg); padding:5px 15px; display:inline-block; }
        .label { color:var(--matrix-green); opacity:0.6; font-size:11px; margin-top:15px; text-transform:uppercase; }
        .data { font-size:20px; margin-bottom:5px; border-bottom:1px solid var(--dim-green); padding-bottom:5px; }
        .doc-section { margin-top:40px; padding:25px; border:1px dashed var(--matrix-green); background:rgba(0,59,0,0.1); }
        .doc-content { font-size:14px; line-height:1.5; white-space:pre-wrap; color:#a0ffa0; }
        button { background:transparent; border:2px solid var(--matrix-green); color:var(--matrix-green); padding:12px 25px; margin:8px 5px; font-size:16px; cursor:pointer; }
        button:hover { background:var(--matrix-green); color:var(--dark-bg); box-shadow:0 0 15px var(--matrix-green); }
        .action-buttons { margin: 35px 0 15px; display: flex; justify-content: center; gap: 20px; flex-wrap: wrap; }
        .export-btn, .pdf-btn { min-width: 160px; }
        .edit-btn {
            min-width: 180px;
            background: #004d00;
            border: 2px solid #00ff88;
            color: #00ff88;
            font-weight: bold;
            box-shadow: 0 0 12px rgba(0,255,136,0.4);
        }
        .edit-btn:hover { background: #006600; box-shadow: 0 0 20px rgba(0,255,136,0.7); }
        .edit-btn.active { background: #006600; border-color: #88ffcc; color: #88ffcc; }
        #cronologia { max-width:1000px; margin:40px auto; border:2px solid var(--matrix-green); padding:25px; background:rgba(0,59,0,0.1); }
        #admin-modal-content { background:var(--dark-bg); border:3px solid var(--matrix-green); padding:30px; width:90%; max-width:720px; }
        textarea, input { width:100%; background:var(--dark-bg); border:1px solid var(--matrix-green); color:var(--matrix-green); padding:10px; margin:8px 0; font-family:inherit; box-sizing:border-box; }
        select { width:100%; background:var(--dark-bg); border:1px solid var(--matrix-green); color:var(--matrix-green); padding:10px; margin:8px 0; }
        #status-bar { position:fixed; bottom:0; left:0; width:100%; background:rgba(0,0,0,0.95); border-top:2px solid var(--matrix-green); padding:10px; text-align:center; font-size:13px; z-index:10000; }
    </style>
</head>
<body>
    <canvas id="matrix-bg"></canvas>

    <div id="login-overlay">
        <div class="login-box">
            <h2>INTELLIGENCE ARCHIVE SYSTEM v6.2</h2>
            <hr style="border-color:var(--matrix-green); margin:15px 0;">
            <p>RESTRICTED AREA - LEVEL 4</p>
            <input type="password" id="passInput" placeholder="PASSWORD" onkeydown="if(event.key==='Enter') checkPass()">
            <button id="login-ok-btn" onclick="checkPass()">OK</button>
            <p id="error-msg" style="color:#ff0000; display:none; margin-top:15px;">ACCESS DENIED</p>
        </div>
    </div>

    <div id="operator-overlay" style="display:none;">
        <div class="login-box">
            <h2>IDENTIFICAZIONE OPERATORE</h2>
            <button class="op-btn" onclick="selectOperator('BORIS')">BORIS</button>
            <button class="op-btn" onclick="selectOperator('NIR.D')">NIR.D</button>
            <button class="op-btn" onclick="selectOperator('PASIN 04')">PASIN 04</button>
            <button class="op-btn" onclick="showCustomInput()">ALTRO</button>
            <div id="custom-input" style="display:none;margin-top:20px;">
                <input type="text" id="customOperator" placeholder="NOME OPERATORE">
                <button class="op-btn" onclick="confirmCustom()">CONFERMA</button>
            </div>
        </div>
    </div>

    <div id="main-content">
        <div class="header">
            <h1>INTELLIGENCE TERMINAL v6.2 - BORIS MODE</h1>
            <p id="logged-as">LOGGED AS: ...</p>
            <button onclick="reopenOperator()" class="export-btn">REGISTRA NUOVO ACCESSO</button>
            <button onclick="showAdminModal()" class="export-btn" style="margin-left:10px;">🛠 ADMIN PANEL</button>
            <button onclick="resetDatabase()" class="export-btn" style="margin-left:10px; background:#330000;">RESET DATABASE</button>
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
           
            <div class="admin-buttons" style="margin-top:30px; text-align:center;">
                <button onclick="saveSubject()" class="export-btn" style="background:#004d00; border-color:#00ff88; color:#00ff88; font-size:18px; padding:14px 40px;">INVIA / SALVA</button>
                <button onclick="closeAdminModal()" class="export-btn" style="margin-left:20px; background:#330000; border-color:#ff3333; font-size:18px; padding:14px 40px;">ANNULLA</button>
            </div>
        </div>
    </div>

    <div id="status-bar">
        CONNECTED TO: SECURE NODE CHIUPPANO-1912 | THREAT LEVEL: <span id="threat-level" style="color:#00ff41;">GREEN</span> | TIME: <span id="system-time"></span>
    </div>

    <script>
        // MATRIX RAIN
        const canvas = document.getElementById('matrix-bg');
        const ctx = canvas.getContext('2d');
        function resizeCanvas() { canvas.height = window.innerHeight; canvas.width = window.innerWidth; }
        resizeCanvas(); window.addEventListener('resize', resizeCanvas);
        const chars = "01アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワン0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ█▓▒░";
        const fontSize = 14; let drops = Array(Math.floor(canvas.width / fontSize)).fill(1);
        function drawMatrix() {
            ctx.fillStyle = 'rgba(0,0,0,0.05)'; ctx.fillRect(0,0,canvas.width,canvas.height);
            ctx.fillStyle = '#00ff41'; ctx.font = fontSize + 'px monospace';
            for (let i = 0; i < drops.length; i++) {
                const text = chars[Math.floor(Math.random() * chars.length)];
                ctx.fillText(text, i * fontSize, drops[i] * fontSize);
                if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) drops[i] = 0;
                drops[i]++;
            }
        }
        setInterval(drawMatrix, 35);

        // ... (il resto del codice JavaScript rimane identico a quello precedente)

        // LOG + LOGIN (parte modificata solo per chiarezza)
        let accessLog = JSON.parse(localStorage.getItem('accessLog')) || [];
        function logAccess(op) {
            const now = new Date();
            const ts = now.toLocaleDateString('it-IT',{day:'2-digit',month:'2-digit',year:'numeric'}) + ' ' + now.toLocaleTimeString('it-IT',{hour:'2-digit',minute:'2-digit'});
            accessLog.unshift(`${ts} → ${op}`);
            if(accessLog.length>50) accessLog.pop();
            localStorage.setItem('accessLog',JSON.stringify(accessLog));
            renderCronologia();
        }
        function renderCronologia() {
            document.getElementById('cronologia-list').innerHTML = accessLog.map(e=>`<div>${e}</div>`).join('') || '<div style="opacity:0.5;text-align:center;">Nessun accesso</div>';
        }
        function exportLog() {
            if(!accessLog.length) return;
            const blob=new Blob(["CRONOLOGIA v6.2 BORIS\n\n"+accessLog.join("\n")],{type:"text/plain"});
            const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download=`CRONOLOGIA_${new Date().toISOString().slice(0,10)}.txt`; a.click();
        }
        function clearLog() {
            if(confirm("CANCELLARE TUTTA LA CRONOLOGIA?")) { accessLog=[]; localStorage.setItem('accessLog',JSON.stringify(accessLog)); renderCronologia(); }
        }
        let currentOperator = '';
        function updateLoggedAs() { document.getElementById('logged-as').innerText = `LOGGED AS: ${currentOperator}`; }
        function checkPass() {
            if(document.getElementById('passInput').value==="2011"){
                document.getElementById('login-overlay').style.display='none';
                document.getElementById('operator-overlay').style.display='flex';
            } else {
                document.getElementById('error-msg').style.display='block';
            }
        }
        function selectOperator(name){
            currentOperator=name; logAccess(name); updateLoggedAs();
            document.getElementById('operator-overlay').style.display='none';
            document.getElementById('main-content').style.display='block';
            document.body.style.overflow='auto';
        }
        function showCustomInput(){ document.getElementById('custom-input').style.display='block'; }
        function confirmCustom(){
            const v=document.getElementById('customOperator').value.trim();
            if(v){ currentOperator=v.toUpperCase(); logAccess(currentOperator); updateLoggedAs();
            document.getElementById('operator-overlay').style.display='none';
            document.getElementById('main-content').style.display='block'; }
        }
        function reopenOperator(){ document.getElementById('operator-overlay').style.display='flex'; }
        function resetDatabase() {
            if(confirm("RESETTARE TUTTO IL DATABASE AI VALORI ORIGINALI?")) {
                localStorage.removeItem('soggetti');
                location.reload();
            }
        }

        // Il resto del codice (database soggetti, renderAllCards, toggleEditMode, saveInlineEdit, etc.) rimane esattamente come nella versione precedente
        // ... (copia-incolla qui tutto il codice JavaScript dalla versione precedente a partire da "let soggetti = ...")

        // AVVIO
        renderAllCards();
        showAllCards();
        renderCronologia();

        function updateStatus(){
            setInterval(()=>{
                document.getElementById('system-time').innerText = new Date().toLocaleTimeString('it-IT');
            },1000);
        }
        updateStatus();
    </script>
</body>
</html>
