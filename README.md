container.innerHTML += `
    <div class="result-card ${staffClass}" id="${s.id}">
        ${staffTag}
        <div class="identikit-title">FILE ID: ${s.nome}</div>
        ${infoHtml}
        <div class="doc-section">
            <div class="doc-header">RAPPORTO TECNICO-INVESTIGATIVO</div>
            <div class="doc-content">${s.doc}</div>
        </div>
        
        <div style="margin-top: 30px; text-align: center; display: flex; justify-content: center; gap: 15px; flex-wrap: wrap;">
            <button onclick="exportDossier('${s.id}')" class="export-btn" style="min-width: 160px;">
                📤 ESPORTA TXT
            </button>
            <button onclick="toggleDossier('${s.id}')" class="pdf-btn" style="min-width: 160px;">
                📑 MOSTRA ALLEGATI
            </button>
            <button onclick="editSubject('${s.id}')" class="export-btn" style="
                min-width: 160px;
                background: #004d00;
                border: 2px solid #00ff88;
                color: #00ff88;
                font-weight: bold;
                box-shadow: 0 0 12px rgba(0,255,136,0.5);
            ">
                ✏️ MODIFICA SOGGETTO
            </button>
        </div>
        
        <div id="dossier-${s.id}" style="display:none; margin-top:30px; border:2px solid var(--matrix-green); padding:20px;">
            ${pdfHtml}
        </div>
    </div>`;
        

       
