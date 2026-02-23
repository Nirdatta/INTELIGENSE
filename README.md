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
        .login-box { border:2px solid var(--matrix-green); padding:40px; text-align:center; box-shadow:0 0 20px var(--dim-green); }
        input, button, .op-btn { background:transparent; border:1px solid var(--matrix-green); color:var(--matrix-green); padding:12px; font-size:20px; margin:10px; width:300px; outline:none; cursor:pointer; }
        button:hover, .op-btn:hover { background:var(--matrix-green); color:var(--dark-bg); }
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
        .data { font-size:20px; margin-bottom:5px; border-bottom:1px solid var(--dim-green); padding-bottom:5px; outline: none; }
        .data:focus { background: var(--highlight-bg); }
        .doc-section { margin-top:40px; padding:25px; border:1px dashed var(--matrix-green); background:rgba(0,59,0,0.1); }
        .doc-content { font-size:14px; line-height:1.5; white-space:pre-wrap; color:#a0ffa0; outline: none; }
        .doc-content:focus { background: var(--highlight-bg); }
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
        .quick-save-btn {
            background: var(--matrix-green) !important;
            color: black !important;
            font-weight: bold;
        }
        #cronologia { max-width:1000px; margin:40px auto; border:2px solid var(--matrix-green); padding:25px; background:rgba(0,59,0,0.1); }
        #admin-modal-content { background:var(--dark-bg); border:3px solid var(--matrix-green); padding:30px; width:90%; max-width:720px; }
        textarea, input { width:100%; background:var(--dark-bg); border:1px solid var(--matrix-green); color:var(--matrix-green); padding:10px; margin:8px 0; font-family:inherit; box-sizing:border-box; }
        #status-bar { position:fixed; bottom:0; left:0; width:100%; background:rgba(0,0,0,0.95); border-top:2px solid var(--matrix-green); padding:10px; text-align:center; font-size:13px; z-index:10000; }
    </style>
</head>
<body>
    <canvas id="matrix-bg"></canvas>

    <div id="login-overlay">
        <div class="login-box">
            <h2>INTELLIGENCE ARCHIVE SYSTEM v6.2</h2>
            <p>RESTRICTED AREA - LEVEL 4</p>
            <input type="password" id="passInput" placeholder="PASSWORD" onkeydown="if(event.key==='Enter') checkPass()">
            <p id="error-msg" style="color:#ff0000;display:none;">ACCESS DENIED</p>
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
            <h2>CRONOLOGIA ACCESSI E MODIFICHE</h2>
            <button onclick="exportLog()" class="export-btn">📤 ESPORTA TXT</button>
            <button onclick="clearLog()" class="export-btn" style="margin-left:10px;">🗑 CANCELLA</button>
            <div id="cronologia-list"></div>
        </div>
    </div>

    <div id="admin-modal" style="display:none;">
        <div id="admin-modal-content">
            <h2>🛠 ADMIN PANEL v6.2</h2>
            <select id="edit-select" onchange="loadForEdit()" style="width:100%; background:black; color:var(--matrix-green); padding:10px; border:1px solid var(--matrix-green);">
            </select><br><br>
            
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
        const chars = "01アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホ마미무메모ヤユヨ라릴레로완0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ█▓▒░";
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

        // DATABASE E LOGICA
        let soggetti = JSON.parse(localStorage.getItem('soggetti')) || [
            { nome: "NIR.D", id: "card-nir", isStaff: true, ruolo: "HACKER / INTELLIGENCE", info: { GRADO: "OPERATORE ALPHA", SPECIALITÀ: "HACKING & TRACCIAMENTO", STATUS: "ATTIVO" }, doc: "RESPONSABILE SICUREZZA DIGITALE.\nHa gestito il tracciamento IP 151.16.25.87 e l'analisi dei metadati.", pdfs: [{title:"Profilo NIR 05", content:"NIR 05\nMaturità: basso\nSesso: M"}] },
            { nome: "ADAM GRITCAN", id: "card-adam", info: { ETÀ: "13 ANNI", LIVELLO: "1", IP: "151.16.25.87", MISSIONE: "RACCOLTA IMPRONTE" }, doc: "DOSSIER: Investigazione avviata per offese gratuite. Tracciamento digitale prioritario.", pdfs: [{title:"Lista IP", content:"151.16.25.87"}] },
            { nome: "LORENZO MAGLIACCIO", id: "card-lorenzo", info: { RESIDENZA: "VIA ALBERI 21, CHIUPPANO", STATUS: "IDENTIFICATO" }, doc: "Località: Casa Gialla." }
        ];

        let accessLog = JSON.parse(localStorage.getItem('accessLog')) || [];
        let currentOperator = 'SCONOSCIUTO';

        function logAccess(msg) { 
            const now = new Date(); 
            const ts = now.toLocaleDateString('it-IT') + ' ' + now.toLocaleTimeString('it-IT');
            accessLog.unshift(`${ts} → ${msg}`); 
            if(accessLog.length>50) accessLog.pop(); 
            localStorage.setItem('accessLog',JSON.stringify(accessLog)); 
            renderCronologia(); 
        }

        function renderCronologia() { 
            document.getElementById('cronologia-list').innerHTML = accessLog.map(e=>`<div>${e}</div>`).join('') || '<div>Nessuna attività</div>'; 
        }

        function renderAllCards() {
            const container = document.getElementById('cards-container');
            container.innerHTML = '';
            soggetti.forEach(s => {
                let infoHtml = '';
                if (s.info) {
                    for (let [k, v] of Object.entries(s.info)) {
                        infoHtml += `<div class="label">${k}:</div><div class="data" contenteditable="true" data-subj="${s.id}" data-key="${k}">${v}</div>`;
                    }
                }
                const staffClass = s.isStaff ? 'staff-highlight' : '';
                const staffTag = s.isStaff ? `<div class="staff-tag">[ AGENTE: ${s.ruolo || 'OPERATORE'} ]</div>` : '';
                
                container.innerHTML += `
                    <div class="result-card ${staffClass}" id="${s.id}">
                        ${staffTag}
                        <div class="identikit-title">FILE ID: ${s.nome}</div>
                        ${infoHtml}
                        <div class="doc-section">
                            <div class="doc-header">RAPPORTO TECNICO-INVESTIGATIVO</div>
                            <div class="doc-content" contenteditable="true" id="doc-text-${s.id}">${s.doc || ''}</div>
                        </div>
                        <div class="action-buttons">
                            <button onclick="quickSave('${s.id}')" class="quick-save-btn">💾 SALVA MODIFICHE</button>
                            <button onclick="exportDossier('${s.id}')" class="export-btn">📤 ESPORTA</button>
                            <button onclick="editSubject('${s.id}')" class="export-btn edit-btn">✏️ ADMIN EDIT</button>
                        </div>
                    </div>`;
            });
        }

        // AGGIORNAMENTO STATO E SALVATAGGIO RAPIDO
        function quickSave(id) {
            const index = soggetti.findIndex(s => s.id === id);
            if(index === -1) return;

            // Leggi dati da campi editabili
            const card = document.getElementById(id);
            const dataFields = card.querySelectorAll(`.data[data-subj="${id}"]`);
            dataFields.forEach(f => {
                const key = f.getAttribute('data-key');
                soggetti[index].info[key] = f.innerText.trim();
            });

            // Leggi rapporto
            soggetti[index].doc = document.getElementById(`doc-text-${id}`).innerText.trim();

            // Salva e Notifica
            localStorage.setItem('soggetti', JSON.stringify(soggetti));
            logAccess(`MODIFICA DATI: ${soggetti[index].nome} (da ${currentOperator})`);
            
            // Update Threat Level visuale
            updateThreatStatus();

            const btn = card.querySelector('.quick-save-btn');
            btn.innerText = "✅ AGGIORNATO";
            setTimeout(() => { btn.innerText = "💾 SALVA MODIFICHE"; }, 2000);
        }

        function updateThreatStatus() {
            const tl = document.getElementById('threat-level');
            tl.innerText = "YELLOW (DATA UPDATE)";
            tl.style.color = "yellow";
            setTimeout(() => {
                tl.innerText = "GREEN";
                tl.style.color = "#00ff41";
            }, 4000);
        }

        // GESTIONE LOGIN
        function checkPass() { 
            if(document.getElementById('passInput').value==="2011"){ 
                document.getElementById('login-overlay').style.display='none'; 
                document.getElementById('operator-overlay').style.display='flex'; 
            } else { document.getElementById('error-msg').style.display='block'; }
        }

        function selectOperator(name){ 
            currentOperator=name; 
            logAccess(`ACCESSO OPERATORE: ${name}`); 
            document.getElementById('logged-as').innerText = `LOGGED AS: ${name}`;
            document.getElementById('operator-overlay').style.display='none'; 
            document.getElementById('main-content').style.display='block'; 
            document.body.style.overflow='auto'; 
            renderAllCards();
            showAllCards();
        }

        function confirmCustom(){
            const v = document.getElementById('customOperator').value.trim().toUpperCase();
            if(v) selectOperator(v);
        }

        function showCustomInput(){ document.getElementById('custom-input').style.display='block'; }

        // UTILITY DI RICERCA
        function handleInput() {
            const q = document.getElementById('searchInput').value.toUpperCase();
            document.querySelectorAll('.result-card').forEach(card => {
                card.style.display = card.innerText.toUpperCase().includes(q) ? 'block' : 'none';
            });
        }

        function showAllCards() { document.querySelectorAll('.result-card').forEach(c => c.style.display = 'block'); }
        
        function reopenOperator(){ location.reload(); }

        function resetDatabase() {
            if(confirm("RESET TOTALE?")) { localStorage.clear(); location.reload(); }
        }

        // ADMIN PANEL FUNZIONI
        function showAdminModal() {
            const sel = document.getElementById('edit-select');
            sel.innerHTML = '<option value="NEW">-- NUOVO SOGGETTO --</option>';
            soggetti.forEach(s => sel.innerHTML += `<option value="${s.id}">${s.nome}</option>`);
            document.getElementById('admin-modal').style.display = 'flex';
        }

        function closeAdminModal() { document.getElementById('admin-modal').style.display = 'none'; }

        function saveSubject() {
            const id = document.getElementById('edit-id').value || 'card-' + Date.now();
            const nome = document.getElementById('edit-nome').value;
            if(!nome) return alert("Nome mancante");
            
            const newObj = {
                id: id,
                nome: nome,
                isStaff: document.getElementById('edit-staff').checked,
                ruolo: document.getElementById('edit-ruolo').value,
                doc: document.getElementById('edit-doc').value,
                info: {}
            };
            
            const lines = document.getElementById('edit-info').value.split('\n');
            lines.forEach(l => {
                const parts = l.split(':');
                if(parts.length > 1) newObj.info[parts[0].trim().toUpperCase()] = parts[1].trim();
            });

            const idx = soggetti.findIndex(s => s.id === id);
            if(idx > -1) soggetti[idx] = newObj; else soggetti.push(newObj);

            localStorage.setItem('soggetti', JSON.stringify(soggetti));
            logAccess(`ADMIN: ARCHIVIO AGGIORNATO (${nome})`);
            location.reload();
        }

        function editSubject(id) {
            showAdminModal();
            const s = soggetti.find(x => x.id === id);
            if(s) {
                document.getElementById('edit-select').value = id;
                document.getElementById('edit-id').value = s.id;
                document.getElementById('edit-nome').value = s.nome;
                document.getElementById('edit-doc').value = s.doc;
                // info parsing...
            }
        }

        function exportDossier(id) {
            const s = soggetti.find(x => x.id === id);
            const blob = new Blob([JSON.stringify(s, null, 2)], {type: 'text/plain'});
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `DOSSIER_${s.nome}.txt`;
            a.click();
        }

        // TIME UPDATE
        setInterval(() => {
            document.getElementById('system-time').innerText = new Date().toLocaleTimeString('it-IT');
        }, 1000);

        renderCronologia();
    </script>
</body>
</html>
