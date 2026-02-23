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
        .login-box { border:2px solid var(--matrix-green); padding:50px 40px; text-align:center; box-shadow:0 0 25px var(--dim-green); max-width:460px; }
        input, button, .op-btn { background:transparent; border:2px solid var(--matrix-green); color:var(--matrix-green); padding:14px; font-size:20px; margin:12px auto; width:280px; outline:none; cursor:pointer; display:block; }
        button:hover, .op-btn:hover { background:var(--matrix-green); color:var(--dark-bg); }
        #login-ok-btn { width:160px; font-size:18px; padding:16px; margin:20px auto 10px; border-width:3px; }
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
            <hr style="border-color:var(--matrix-green); margin:20px 0;">
            <p>RESTRICTED AREA - LEVEL 4</p>
            <input type="password" id="passInput" placeholder="PASSWORD" onkeydown="if(event.key==='Enter') checkPass()">
            <button id="login-ok-btn" onclick="checkPass()">OK / ACCEDI</button>
            <p id="error-msg" style="color:#ff0000; display:none; margin-top:20px; font-size:18px;">ACCESS DENIED</p>
        </div>
    </div>

    <div id="operator-overlay" style="display:none;">
        <div class="login-box">
            <h2>IDENTIFICAZIONE OPERATORE</h2>
            <button class="op-btn" onclick="selectOperator('BORIS')">BORIS</button>
            <button class="op-btn" onclick="selectOperator('NIR.D')">NIR.D</button>
            <button class="op-btn" onclick="selectOperator('PASIN 04')">PASIN 04</button>
            <button class="op-btn" onclick="showCustomInput()">ALTRO</button>
            <div id="custom-input" style="display:none; margin-top:25px;">
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

        // LOG + LOGIN
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
            const input = document.getElementById('passInput');
            if (input.value === "2011") {
                document.getElementById('login-overlay').style.display = 'none';
                document.getElementById('operator-overlay').style.display = 'flex';
                input.value = ''; // pulisci per sicurezza
            } else {
                document.getElementById('error-msg').style.display = 'block';
            }
        }
        function selectOperator(name){
            currentOperator = name; logAccess(name); updateLoggedAs();
            document.getElementById('operator-overlay').style.display = 'none';
            document.getElementById('main-content').style.display = 'block';
            document.body.style.overflow = 'auto';
        }
        function showCustomInput(){ document.getElementById('custom-input').style.display = 'block'; }
        function confirmCustom(){
            const v = document.getElementById('customOperator').value.trim();
            if(v){
                currentOperator = v.toUpperCase(); logAccess(currentOperator); updateLoggedAs();
                document.getElementById('operator-overlay').style.display = 'none';
                document.getElementById('main-content').style.display = 'block';
            }
        }
        function reopenOperator(){ document.getElementById('operator-overlay').style.display = 'flex'; }
        function resetDatabase() {
            if(confirm("RESETTARE TUTTO IL DATABASE AI VALORI ORIGINALI?")) {
                localStorage.removeItem('soggetti');
                localStorage.removeItem('accessLog');
                location.reload();
            }
        }

        // DATABASE SOGGETTI
        let soggetti = JSON.parse(localStorage.getItem('soggetti')) || [
            { nome: "NIR.D", id: "card-nir", isStaff: true, ruolo: "HACKER / INTELLIGENCE", info: { GRADO: "OPERATORE ALPHA", SPECIALITÀ: "HACKING & TRACCIAMENTO", STATUS: "ATTIVO" }, doc: "RESPONSABILE SICUREZZA DIGITALE.\nHa gestito il tracciamento IP 151.16.25.87 e l'analisi dei metadati per l'operazione L.M. (Lorenzo Magliaccio).", pdfs: [{title:"Profilo NIR 05", content:"NIR 05\nEtà ufficiale: 13\nEtà percepita: 12\nMaturità: basso\nSesso: M\nNucleo centrale: calmo\nCosa piace: tecnologia"}] },
            { nome: "PASIN 04", id: "card-pasin", isStaff: true, ruolo: "SCIENTIFICA", info: { RUOLO: "TECNICO SCIENTIFICO", ETÀ: "14 ANNI", MATURITÀ: "STABILE" }, doc: "ANALISI DATI VITALI.\nParte della squadra che ha effettuato il riscontro visivo a Chiuppano.", pdfs: [{title:"Profilo PASIN 04", content:"PASIN 04\nEtà ufficiale: 13\nEtà percepita: 12\nMaturità: medio/alto\nSesso: M\nNucleo centrale: calmo e concentrato"}] },
            { nome: "ADAM GRITCAN", id: "card-adam", info: { ETÀ: "13 ANNI", LIVELLO: "1", IP: "151.16.25.87", MISSIONE: "RACCOLTA IMPRONTE" }, doc: "DOSSIER: Investigazione avviata per offese gratuite. Tracciamento digitale prioritario.", pdfs: [{title:"Lista IP Pubblici", content:"1- Adam Gritcan livello1 151.16.25.87\n2- Emily Mucca livello 2 in ricerca..\n3-Noemi de russi livello 2 /"},{title:"Lista Progetti", content:"Adam Gritcan 13 chiarire offese gratuite ecc.. livello 1"}] },
            { nome: "LORENZO MAGLIACCIO", id: "card-lorenzo", info: { RESIDENZA: "VIA ALBERI 21, CHIUPPANO", COORDINATE: "45.7605108, 11.4644352", STATUS: "IDENTIFICATO" }, doc: "CONCLUSIONE OPERAZIONE: Individuato a Chiuppano. Osservata attività sociale con Emily Mucca. Località: Casa Gialla.", pdfs: [{title:"Rapporto Investigativo L.M. (9/2/26)", content:"9/2/26\nRAPPORTO INVESTIGATIVO “L.M.”\nSoggetto: Lorenzo Magliaccio\nLocation: Chiuppano (VI)\nEtà: 14 anni\nRelazione: attuale con Emily Mucca"}] },
            { nome: "NOEMI DERUSSI", id: "card-noemi", info: { ETÀ: "13 ANNI", LIVELLO: "2", CARATTERE: "IMPULSIVA / CALDA", FORZA: "DETERMINATA" }, doc: "NOTE: Profilo psicologico attivo. Obiettivo Livello 2: Individuazione abitazione.", pdfs: [{title:"Lista IP", content:"3-Noemi de russi livello 2 /"},{title:"Lista Progetti", content:"derussi noemi 12 trovare abitazione livello 2"},{title:"Profilo NOEMI 01", content:"NOEMI 01\nEtà ufficiale: 13\nNucleo centrale: caldo e impulsiva\nForza: determinata, curiosa"}] },
            { nome: "PAOLO 02", id: "card-paolo", info: { ETÀ: "48 ANNI", CARATTERE: "ESPRESSIVO / CALMO", PIACE: "FAMIGLIA, VIOLENZA", FORZA: "CALMA" }, doc: "PROFILO: Stato emotivo registrato come 'basso'. Bassissimo livello di tolleranza verso il disordine.", pdfs: [{title:"Profilo PAOLO 02", content:"PAOLO 02\nEtà ufficiale: 48\nNucleo centrale: espressivo e calmo\nCosa piace: famiglia, violenza\nCosa non piace: disordine"}] },
            { nome: "BARBARA 03", id: "card-barbara", info: { ETÀ: "51 ANNI", CARATTERE: "ESPRESSIVO", DEBOLEZZA: "NON SA ASCOLTARE", FORZA: "DETERMINATA" }, doc: "PROFILO: Nucleo centrale espressivo. Stato emotivo poco controllato. Pazienza: Bassa.", pdfs: [{title:"Profilo BARBARA 03", content:"BARBARA 03\nEtà ufficiale: 51\nNucleo centrale: espressivo\nLivello di pazienza: basso"}] },
            { nome: "EMILY MUCCA", id: "card-emily", info: { ETÀ: "12 ANNI", RELAZIONE: "LORENZO MAGLIACCIO", STATUS: "MONITORATA" }, doc: "NOTA: Presenza rilevata a Chiuppano con il bersaglio primario. Febbraio 2026.", pdfs: [{title:"Lista IP", content:"2- Emily Mucca livello 2 in ricerca.."},{title:"Lista Progetti", content:"emily mucca 12 raccogliere impornti digitali livello 2"},{title:"Profilo EMILY 16", content:"EMILY 16\nEtà ufficiale: 13\nStato emotivo: felice\nDinamica sociale: isolata"}] },
            { nome: "SERENA 14", id: "card-serena", info: { ETÀ: "12 ANNI (PERC. 11)", STATO: "TRISTE", SOCIAL: "ISOLATA" }, doc: "NOTE: Nucleo centrale 'far finto'. Carattere instabile. Minimo supporto sociale.", pdfs: [{title:"Profilo SERENA 14", content:"SERENA 14\nEtà ufficiale: 12\nEtà percepita: 11\nStato emotivo: abbastanza triste"}] },
            { nome: "MICHELE 13", id: "card-michele", info: { ETÀ: "18 ANNI", STATO_EMOTIVO: "ANZIANO", PAZIENZA: "ALTO" }, doc: "NOTE: Nucleo centrale stabile. Alta maturità cognitiva rispetto all'età anagrafica.", pdfs: [{title:"Profilo MICHELE 13", content:"MICHELE 13\nEtà ufficiale: 18\nStato emotivo: anziano\nLivello di pazienza: alto"}] },
            { nome: "LUISELLA 12", id: "card-luisella", info: { ETÀ: "65 ANNI", MATURITÀ: "ALTA", INTERESSI: "RAGAZZI INTRAPRENDENTI" }, doc: "PROFILO: Nucleo sensibile, stato emotivo coerente con le interazioni rilevate.", pdfs: [{title:"Profilo LUISELLA 12", content:"LUISELLA 12\nEtà ufficiale: 65\nMaturità: alta\nPSICHE\nCosa piace: ragazzi intraprendenti"}] }
        ];
        if (soggetti.length < 11) localStorage.setItem('soggetti', JSON.stringify(soggetti));

        function saveSoggetti() {
            localStorage.setItem('soggetti', JSON.stringify(soggetti));
            renderAllCards();
        }

        // EDITING INLINE
        let editingCardId = null;

        function toggleEditMode(id) {
            const card = document.getElementById(id);
            if (!card) return;

            if (editingCardId === id) {
                saveInlineEdit(id);
                return;
            }

            if (editingCardId) cancelInlineEdit(editingCardId);
            editingCardId = id;

            const subj = soggetti.find(s => s.id === id);
            if (!subj) return;

            card.querySelectorAll('.label, .data, .doc-section, .action-buttons > *:not(.edit-btn)').forEach(el => el.style.display = 'none');

            let infoLines = '';
            if (subj.info) Object.entries(subj.info).forEach(([k,v]) => infoLines += `${k}: ${v}\n`);

            const editFormHtml = `
                <div class="inline-edit-form" style="padding:20px; border:2px dashed var(--matrix-green); background:rgba(0,80,0,0.25); margin:15px 0; border-radius:4px;">
                    <h3 style="margin:0 0 15px 0; color:#88ffcc; text-align:center;">MODIFICA ${subj.nome}</h3>
                    <label style="font-size:14px; opacity:0.8;">NOME COMPLETO</label><br>
                    <input type="text" id="inline-nome-${id}" value="${(subj.nome || '').replace(/"/g,'&quot;')}" style="width:100%; font-size:18px;"><br><br>
                    <label style="font-size:14px; opacity:0.8;">
                        <input type="checkbox" id="inline-staff-${id}" ${subj.isStaff ? 'checked' : ''}> SOGGETTO STAFF
                    </label><br>
                    <div id="staff-role-${id}" style="display:${subj.isStaff ? 'block' : 'none'}; margin:12px 0;">
                        <label style="font-size:14px; opacity:0.8;">RUOLO</label><br>
                        <input type="text" id="inline-ruolo-${id}" value="${(subj.ruolo || '').replace(/"/g,'&quot;')}" style="width:100%;">
                    </div>
                    <label style="font-size:14px; opacity:0.8; margin-top:15px; display:block;">INFO (CHIAVE: valore - una per riga)</label>
                    <textarea id="inline-info-${id}" rows="7" style="width:100%; font-size:15px; line-height:1.4;">${infoLines.trim()}</textarea><br><br>
                    <label style="font-size:14px; opacity:0.8; display:block;">RAPPORTO TECNICO-INVESTIGATIVO</label>
                    <textarea id="inline-doc-${id}" rows="10" style="width:100%; font-size:15px; line-height:1.4;">${(subj.doc || '').replace(/"/g,'&quot;')}</textarea><br><br>
                    <div style="text-align:center; margin-top:20px;">
                        <button onclick="saveInlineEdit('${id}')" style="background:#004d00; border:2px solid #88ffcc; color:#88ffcc; padding:12px 30px; font-size:17px; min-width:180px;">SALVA MODIFICHE</button>
                        <button onclick="cancelInlineEdit('${id}')" style="margin-left:25px; background:#330000; border:2px solid #ff6666; color:#ff9999; padding:12px 30px; font-size:17px; min-width:180px;">ANNULLA</button>
                    </div>
                </div>
            `;

            card.querySelector('.identikit-title').insertAdjacentHTML('afterend', editFormHtml);

            document.getElementById(`inline-staff-${id}`).addEventListener('change', function() {
                document.getElementById(`staff-role-${id}`).style.display = this.checked ? 'block' : 'none';
            });

            const editBtn = card.querySelector('.edit-btn');
            editBtn.textContent = '💾 SALVA MODIFICHE';
            editBtn.classList.add('active');
        }

        function saveInlineEdit(id) {
            const subjIndex = soggetti.findIndex(s => s.id === id);
            if (subjIndex === -1) return;

            const nome = document.getElementById(`inline-nome-${id}`).value.trim();
            if (!nome) { alert("❌ Nome completo obbligatorio!"); return; }

            const isStaff = document.getElementById(`inline-staff-${id}`).checked;
            const ruolo = isStaff ? (document.getElementById(`inline-ruolo-${id}`).value.trim() || 'OPERATORE') : undefined;

            const infoText = document.getElementById(`inline-info-${id}`).value.trim();
            const infoObj = {};
            infoText.split('\n').forEach(line => {
                line = line.trim();
                if (line && line.includes(':')) {
                    const [key, ...val] = line.split(':');
                    const k = key.trim().toUpperCase();
                    if (k) infoObj[k] = val.join(':').trim();
                }
            });

            const doc = document.getElementById(`inline-doc-${id}`).value.trim();

            soggetti[subjIndex] = {
                ...soggetti[subjIndex],
                nome,
                isStaff,
                ruolo,
                info: Object.keys(infoObj).length ? infoObj : undefined,
                doc: doc || undefined
            };

            saveSoggetti();
            editingCardId = null;
            alert("✅ Modifiche salvate");
        }

        function cancelInlineEdit(id) {
            editingCardId = null;
            renderAllCards();
        }

        function renderAllCards() {
            const container = document.getElementById('cards-container');
            container.innerHTML = '';
            soggetti.forEach(s => {
                let infoHtml = '';
                if (s.info) {
                    Object.entries(s.info).forEach(([k,v]) => {
                        infoHtml += `<div class="label">${k}:</div><div class="data">${v}</div>`;
                    });
                }
                const staffClass = s.isStaff ? 'staff-highlight' : '';
                const staffTag = s.isStaff ? `<div class="staff-tag">[ AGENTE: ${s.ruolo || 'OPERATORE'} ]</div>` : '';
                let pdfHtml = s.pdfs ? s.pdfs.map(p => 
                    `<button onclick="openAsPdf('${p.title}', \`${p.content.replace(/`/g,'\\`')}\`)" class="pdf-btn">📄 ${p.title}</button>`
                ).join('') : '';

                container.innerHTML += `
                    <div class="result-card ${staffClass}" id="${s.id}">
                        ${staffTag}
                        <div class="identikit-title">FILE ID: ${s.nome}</div>
                        ${infoHtml}
                        <div class="doc-section">
                            <div class="doc-header">RAPPORTO TECNICO-INVESTIGATIVO</div>
                            <div class="doc-content">${s.doc || '[NESSUN RAPPORTO DISPONIBILE]'}</div>
                        </div>
                        <div class="action-buttons">
                            <button onclick="exportDossier('${s.id}')" class="export-btn">📤 ESPORTA TXT</button>
                            <button onclick="toggleDossier('${s.id}')" class="pdf-btn">📑 MOSTRA ALLEGATI</button>
                            <button onclick="toggleEditMode('${s.id}')" class="export-btn edit-btn">✏️ MODIFICA SOGGETTO</button>
                        </div>
                        <div id="dossier-${s.id}" style="display:none; margin-top:25px; border:2px solid var(--matrix-green); padding:20px;">
                            ${pdfHtml || '<div style="opacity:0.6; text-align:center;">Nessun allegato disponibile</div>'}
                        </div>
                    </div>`;
            });
        }

        function showAllCards() { document.querySelectorAll('.result-card').forEach(c => c.style.display = 'block'); }
        function hideAllCards() { document.querySelectorAll('.result-card').forEach(c => c.style.display = 'none'); }

        function handleInput() {
            const q = document.getElementById('searchInput').value.toUpperCase().trim();
            const box = document.getElementById('suggestions');
            box.innerHTML = '';
            const matches = soggetti.filter(s => 
                (s.nome || '').toUpperCase().includes(q) ||
                JSON.stringify(s.info||{}).toUpperCase().includes(q) ||
                (s.doc||'').toUpperCase().includes(q)
            );

            if (q.length === 0) {
                box.style.display = 'block';
                soggetti.forEach(m => {
                    const item = document.createElement('div');
                    item.className = 'suggestion-item';
                    item.innerText = m.isStaff ? `⚡ ${m.nome}` : `👤 ${m.nome}`;
                    item.onclick = () => { document.getElementById('searchInput').value = m.nome; handleInput(); box.style.display = 'none'; };
                    box.appendChild(item);
                });
            } else if (matches.length) {
                box.style.display = 'block';
                matches.forEach(m => {
                    const item = document.createElement('div');
                    item.className = 'suggestion-item';
                    item.innerText = m.isStaff ? `⚡ ${m.nome}` : `👤 ${m.nome}`;
                    item.onclick = () => { document.getElementById('searchInput').value = m.nome; handleInput(); box.style.display = 'none'; };
                    box.appendChild(item);
                });
            } else {
                box.style.display = 'none';
            }

            hideAllCards();
            (q.length === 0 ? soggetti : matches).forEach(m => {
                const card = document.getElementById(m.id);
                if (card) card.style.display = 'block';
            });
        }

        function exportDossier(id) {
            const card = document.getElementById(id);
            const nome = card.querySelector('.identikit-title').innerText.replace('FILE ID: ','');
            let txt = `INTELLIGENCE TERMINAL v6.2\nFILE: ${nome}\n\n`;
            card.querySelectorAll('.label').forEach((l,i) => {
                txt += `${l.innerText} ${card.querySelectorAll('.data')[i].innerText}\n`;
            });
            txt += `\nRAPPORTO:\n${card.querySelector('.doc-content').innerText}`;
            const blob = new Blob([txt], {type:'text/plain'});
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `DOSSIER_${nome.replace(/ /g,'_')}.txt`;
            a.click();
        }

        function openAsPdf(title, content){
            const w = window.open('','_blank');
            w.document.write(`<html><head><title>${title}</title><style>body{background:#000;color:#00ff41;font-family:Courier New;padding:40px;}</style></head><body><h1>${title}</h1><pre>${content}</pre></body></html>`);
            w.document.close();
        }

        function toggleDossier(id){
            const d = document.getElementById('dossier-'+id);
            d.style.display = d.style.display === 'block' ? 'none' : 'block';
        }

        function clearEditFields() {
            document.getElementById('edit-nome').value = '';
            document.getElementById('edit-id').value = '';
            document.getElementById('edit-staff').checked = false;
            document.getElementById('edit-ruolo').value = '';
            document.getElementById('edit-info').value = '';
            document.getElementById('edit-doc').value = '';
        }

        function showAdminModal() {
            const select = document.getElementById('edit-select');
            select.innerHTML = '<option value="">-- SELEZIONA SOGGETTO --</option><option value="NEW">➕ CREA NUOVO SOGGETTO</option>';
            soggetti.forEach(s => {
                const opt = document.createElement('option');
                opt.value = s.id;
                opt.textContent = s.nome;
                select.appendChild(opt);
            });
            clearEditFields();
            document.getElementById('admin-modal').style.display = 'flex';
        }

        function loadForEdit() {
            const val = document.getElementById('edit-select').value;
            if (!val || val === "NEW") {
                clearEditFields();
                if (val === "NEW") document.getElementById('edit-id').value = 'card-nuovo-' + Date.now().toString().slice(-6);
                return;
            }
            const subj = soggetti.find(s => s.id === val);
            if (!subj) return;
            document.getElementById('edit-nome').value = subj.nome || '';
            document.getElementById('edit-id').value = subj.id || '';
            document.getElementById('edit-staff').checked = !!subj.isStaff;
            document.getElementById('edit-ruolo').value = subj.ruolo || '';
            let infoStr = '';
            if (subj.info) Object.entries(subj.info).forEach(([k,v]) => infoStr += `${k}: ${v}\n`);
            document.getElementById('edit-info').value = infoStr.trim();
            document.getElementById('edit-doc').value = subj.doc || '';
        }

        function saveSubject() {
            let id = document.getElementById('edit-id').value.trim();
            let nome = document.getElementById('edit-nome').value.trim();
            if (!id) { alert('❌ ID obbligatorio!'); return; }
            if (!nome) { alert('❌ NOME COMPLETO obbligatorio!'); return; }
            let index = soggetti.findIndex(s => s.id === id);
            let isNew = index === -1;
            let subj = isNew ? { id, pdfs: [] } : soggetti[index];
            subj.nome = nome;
            subj.isStaff = document.getElementById('edit-staff').checked;
            if (subj.isStaff) subj.ruolo = document.getElementById('edit-ruolo').value.trim() || 'OPERATORE';
            else delete subj.ruolo;
            subj.info = {};
            document.getElementById('edit-info').value.trim().split('\n').forEach(line => {
                if (line.includes(':')) {
                    const [key, ...val] = line.split(':');
                    if (key.trim()) subj.info[key.trim().toUpperCase()] = val.join(':').trim();
                }
            });
            subj.doc = document.getElementById('edit-doc').value.trim();
            saveSoggetti();
            alert(`✅ ${isNew ? 'NUOVO SOGGETTO CREATO' : 'SOGGETTO MODIFICATO'}!`);
            closeAdminModal();
        }

        function closeAdminModal() { document.getElementById('admin-modal').style.display = 'none'; }

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
    </script>
</body>
</html>
