<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Programação Diária de Frota</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f2f5; padding: 20px; }
        .container { max-width: 1600px; margin: 0 auto; background: white; border-radius: 12px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); overflow: hidden; }
        .header { background: linear-gradient(135deg,#2c3e50,#34495e); color: white; padding: 30px; text-align: center; }
        .header h1 { font-size: 2em; margin-bottom: 5px; }
        .content { padding: 30px; }
        .section { background: #f8f9fa; padding: 25px; border-radius: 8px; margin-bottom: 20px; border-left: 4px solid #2c3e50; }
        .section-title { font-size: 1.3em; color: #2c3e50; margin-bottom: 20px; font-weight: 600; }
        .destination-tabs { display:flex; gap:10px; margin-bottom:20px; flex-wrap:wrap; }
        .tab-btn { padding:12px 24px; border:2px solid #2c3e50; background:white; border-radius:6px; cursor:pointer; font-weight:600; transition:all .3s; color:#2c3e50; }
        .tab-btn:hover{ background:#34495e; color:white; }
        .tab-btn.active{ background:#2c3e50; color:white; }
        .destination-content { display:none; }
        .destination-content.active { display:block; }
        .schedule-table { width:100%; border-collapse:collapse; background:white; margin-bottom:20px; }
        .schedule-table th { background:#2c3e50; color:white; padding:12px; text-align:left; font-weight:600; font-size:.95em; }
        .schedule-table td { padding:8px; border-bottom:1px solid #e0e0e0; }
        .schedule-table input, .schedule-table select { width:100%; padding:8px; border:2px solid #e0e0e0; border-radius:4px; font-size:.9em; box-sizing:border-box; }
        .schedule-table input:focus, .schedule-table select:focus { outline:none; border-color:#2c3e50; }
        .schedule-table input[readonly] { background:#f8f9fa; cursor:not-allowed; }
        .btn { padding:10px 20px; border:none; border-radius:6px; font-size:.95em; font-weight:600; cursor:pointer; transition:all .3s; }
        .btn-add { background:#27ae60; color:white; margin-bottom:15px; }
        .btn-remove { background:#e74c3c; color:white; padding:6px 12px; font-size:.85em; }
        .btn-primary { background:#2c3e50; color:white; }
        .btn-success { background:#27ae60; color:white; }
        .btn-group { display:flex; gap:15px; margin-top:20px; }
        .alert { padding:15px; border-radius:6px; margin-bottom:20px; display:none; }
        .alert-success { background:#d4edda; color:#155724; border:1px solid #c3e6cb; }
        .alert-info { background:#d1ecf1; color:#0c5460; border:1px solid #bee5eb; }
        .file-input-wrapper input[type=file]{ display:none; }
        .file-input-label { display:inline-block; padding:12px 30px; background:#3498db; color:white; border-radius:6px; cursor:pointer; font-weight:600; transition:all .3s; }
        .empty-state{ text-align:center; padding:40px; color:#95a5a6; }
        .obs-area { margin-top:20px; padding:15px; background:white; border-radius:6px; border:1px solid #e0e0e0; }
        .obs-area label{ font-weight:600; color:#495057; margin-bottom:10px; display:block; }
        .obs-area textarea { width:100%; min-height:80px; padding:10px; border:2px solid #e0e0e0; border-radius:4px; resize:vertical; }
        .inline-controls { display:flex; gap:10px; align-items:center; flex-wrap:wrap; margin-bottom:10px; }
        .small-btn { padding:8px 12px; border-radius:6px; font-size:.9em; }
        /* Ensure placa inputs have room and do not clip text */
        input[id*="placaImplemento"] { text-overflow: ellipsis; white-space: nowrap; overflow: hidden; }
        @media (max-width:1200px){ .schedule-table { font-size:.85em; } }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📋 Programação Diária de Frota</h1>
            <p>Sistema de Controle e Gerenciamento de Transportes</p>
        </div>

        <div class="content">
            <div id="alertSuccess" class="alert alert-success"></div>
            <div id="alertInfo" class="alert alert-info"></div>

            <!-- Bases Fixas -->
            <div class="section">
                <div class="section-title">1️⃣ Configurar Bases Fixas</div>
                <p style="margin-bottom:15px;color:#6c757d;">Importe os arquivos conforme abaixo:</p>

                <div style="margin-bottom:20px;">
                    <label style="display:block;font-weight:600;margin-bottom:10px;color:#2c3e50;">
                        a) Base de Motoristas (Formato esperado)
                    </label>
                    <p style="margin-bottom:10px;color:#6c757d;font-size:.9em;">
                        Coluna A: ID (ex: 6553) — Coluna B: Nome — Coluna C: Cargo
                    </p>
                    <div class="file-input-wrapper" style="margin-bottom:10px;">
                        <input type="file" id="driversFile" accept=".xlsx,.xls" />
                        <label for="driversFile" class="file-input-label">👨‍✈️ Importar Motoristas</label>
                    </div>
                    <span id="driversFileName" style="color:#3498db;font-weight:600;"></span>
                    <div id="driversStatus" style="font-size:.9em;color:#27ae60;margin-top:5px;"></div>
                </div>

                <div style="margin-bottom:20px;">
                    <label style="display:block;font-weight:600;margin-bottom:10px;color:#2c3e50;">
                        b) Base de Placas (Formato esperado)
                    </label>
                    <p style="margin-bottom:10px;color:#6c757d;font-size:.9em;">
                        Coluna A: Frota (ex: 2019 ou 6553) — Coluna B: Placa (ex: ABC-1234) — ou formato legado com Cavalo/Implemento (4 colunas)
                    </p>
                    <div class="file-input-wrapper" style="margin-bottom:10px;">
                        <input type="file" id="platesFile" accept=".xlsx,.xls" />
                        <label for="platesFile" class="file-input-label">🚛 Importar Placas</label>
                    </div>
                    <span id="platesFileName" style="color:#3498db;font-weight:600;"></span>
                    <div id="platesStatus" style="font-size:.9em;color:#27ae60;margin-top:5px;"></div>
                </div>

                <div style="margin-top:20px;padding:15px;background:#e8f4fd;border-radius:6px;">
                    <p style="color:#0c5460;margin:0;">
                        <strong>ℹ️ Observação:</strong> Após importar, digite o ID (ou o nome) no campo "Colaborador" — o nome e o cargo serão preenchidos automaticamente. Digite o número da frota no campo "Cavalo" e as placas do cavalo/implemento serão preenchidas automaticamente.
                    </p>
                </div>
            </div>

            <!-- Programação do dia anterior -->
            <div class="section">
                <div class="section-title">📅 Programação do Dia Anterior</div>
                <div id="previousSchedule" class="empty-state">Carregando...</div>
            </div>

            <!-- Nova Programação e controles por data -->
            <div class="section">
                <div class="section-title">2️⃣ Nova Programação</div>

                <div style="margin-bottom:10px;">
                    <label style="font-weight:600;margin-bottom:10px;display:block;">Data da Programação</label>
                    <div class="inline-controls">
                        <input type="date" id="scheduleDate" style="padding:10px;border:2px solid #e0e0e0;border-radius:6px;font-size:1em;" />
                        <button class="small-btn btn-primary" onclick="loadScheduleBySelectedDate()">🔎 Carregar Programação</button>
                        <button class="small-btn btn-success" onclick="saveSchedule()">💾 Salvar Programação</button>
                        <button class="small-btn" style="background:#f39c12;color:white;border-radius:6px;" onclick="duplicateSchedule()">📄 Duplicar (copiar para outra data)</button>
                        <button class="small-btn" style="background:#c0392b;color:white;border-radius:6px;" onclick="deleteScheduleBySelectedDate()">🗑️ Excluir Programação</button>
                        <span id="savedIndicator" style="color:#2c3e50;font-weight:600;margin-left:10px;"></span>
                    </div>
                </div>

                <div class="destination-tabs">
                    <button class="tab-btn active" onclick="switchTab('fabrica', this)">Fábrica</button>
                    <button class="tab-btn" onclick="switchTab('uberaba', this)">Uberaba</button>
                    <button class="tab-btn" onclick="switchTab('frutal', this)">Frutal</button>
                    <button class="tab-btn" onclick="switchTab('iturama', this)">Iturama</button>
                    <button class="tab-btn" onclick="switchTab('patos', this)">Patos</button>
                </div>

                <!-- Fábrica -->
                <div id="fabrica-content" class="destination-content active">
                    <h3 style="margin-bottom:15px;color:#2c3e50;">🏭 Fábrica</h3>
                    <button class="btn btn-add" onclick="addRow('fabrica')">➕ Adicionar Linha</button>
                    <table class="schedule-table">
                        <thead>
                            <tr>
                                <th style="width:100px;">Cavalo</th>
                                <th style="width:150px;">Implemento</th>
                                <th style="width:200px;">Colaborador</th>
                                <th style="width:200px;">Cargo</th>
                                <th style="width:170px;">Placa Cavalo</th>
                                <th style="width:220px;">Placa Implemento</th>
                                <th style="width:200px;">Programação</th>
                                <th style="width:80px;">Ação</th>
                            </tr>
                        </thead>
                        <tbody id="fabrica-tbody"></tbody>
                    </table>
                    <div class="obs-area">
                        <label>📝 Observações</label>
                        <textarea id="fabrica-obs" placeholder="Digite observações sobre a programação da Fábrica..."></textarea>
                    </div>
                </div>

                <!-- Uberaba -->
                <div id="uberaba-content" class="destination-content">
                    <h3 style="margin-bottom:15px;color:#2c3e50;">📍 Uberaba</h3>
                    <button class="btn btn-add" onclick="addRow('uberaba')">➕ Adicionar Linha</button>
                    <table class="schedule-table">
                        <thead><tr>
                            <th style="width:100px;">Cavalo</th>
                            <th style="width:150px;">Implemento</th>
                            <th style="width:200px;">Colaborador</th>
                            <th style="width:200px;">Cargo</th>
                            <th style="width:170px;">Placa Cavalo</th>
                            <th style="width:220px;">Placa Implemento</th>
                            <th style="width:200px;">Programação</th>
                            <th style="width:80px;">Ação</th>
                        </tr></thead>
                        <tbody id="uberaba-tbody"></tbody>
                    </table>
                    <div class="obs-area"><label>📝 Observações</label><textarea id="uberaba-obs" placeholder="..."></textarea></div>
                </div>

                <!-- Frutal -->
                <div id="frutal-content" class="destination-content">
                    <h3 style="margin-bottom:15px;color:#2c3e50;">📍 Frutal</h3>
                    <button class="btn btn-add" onclick="addRow('frutal')">➕ Adicionar Linha</button>
                    <table class="schedule-table">
                        <thead><tr>
                            <th style="width:100px;">Cavalo</th>
                            <th style="width:150px;">Implemento</th>
                            <th style="width:200px;">Colaborador</th>
                            <th style="width:200px;">Cargo</th>
                            <th style="width:170px;">Placa Cavalo</th>
                            <th style="width:220px;">Placa Implemento</th>
                            <th style="width:200px;">Programação</th>
                            <th style="width:80px;">Ação</th>
                        </tr></thead>
                        <tbody id="frutal-tbody"></tbody>
                    </table>
                    <div class="obs-area"><label>📝 Observações</label><textarea id="frutal-obs" placeholder="..."></textarea></div>
                </div>

                <!-- Iturama -->
                <div id="iturama-content" class="destination-content">
                    <h3 style="margin-bottom:15px;color:#2c3e50;">📍 Iturama</h3>
                    <button class="btn btn-add" onclick="addRow('iturama')">➕ Adicionar Linha</button>
                    <table class="schedule-table">
                        <thead><tr>
                            <th style="width:100px;">Cavalo</th>
                            <th style="width:150px;">Implemento</th>
                            <th style="width:200px;">Colaborador</th>
                            <th style="width:200px;">Cargo</th>
                            <th style="width:170px;">Placa Cavalo</th>
                            <th style="width:220px;">Placa Implemento</th>
                            <th style="width:200px;">Programação</th>
                            <th style="width:80px;">Ação</th>
                        </tr></thead>
                        <tbody id="iturama-tbody"></tbody>
                    </table>
                    <div class="obs-area"><label>📝 Observações</label><textarea id="iturama-obs" placeholder="..."></textarea></div>
                </div>

                <!-- Patos -->
                <div id="patos-content" class="destination-content">
                    <h3 style="margin-bottom:15px;color:#2c3e50;">📍 Patos</h3>
                    <button class="btn btn-add" onclick="addRow('patos')">➕ Adicionar Linha</button>
                    <table class="schedule-table">
                        <thead><tr>
                            <th style="width:100px;">Cavalo</th>
                            <th style="width:150px;">Implemento</th>
                            <th style="width:200px;">Colaborador</th>
                            <th style="width:200px;">Cargo</th>
                            <th style="width:170px;">Placa Cavalo</th>
                            <th style="width:220px;">Placa Implemento</th>
                            <th style="width:200px;">Programação</th>
                            <th style="width:80px;">Ação</th>
                        </tr></thead>
                        <tbody id="patos-tbody"></tbody>
                    </table>
                    <div class="obs-area"><label>📝 Observações</label><textarea id="patos-obs" placeholder="..."></textarea></div>
                </div>

                <div class="btn-group" style="margin-top:18px;">
                    <button class="btn btn-success" onclick="exportToExcel()">📊 Exportar para Excel</button>
                </div>
            </div>
        </div>
    </div>

    <datalist id="driversList"></datalist>

    <script>
        // Data stores
        let driversById = {};
        let driversByNormalizedName = {};
        let platesByFleet = {};
        let platesNormalized = {};
        let schedulesByDate = {}; // date (YYYY-MM-DD) -> schedule object
        let currentSchedule = null;
        let currentScheduleDate = null;
        let rowCounters = { fabrica: 0, uberaba: 0, frutal: 0, iturama: 0, patos: 0 };

        const destinationRoutes = {
            fabrica: ['FÁBRICA','ALB/FAB','FOLGA','BANCO DE HORAS','TREINAMENTO'],
            uberaba: ['URA/POTY/URA','POTY/URA','BANCO DE HORAS','FABRICA','URA TARDE'],
            frutal: ['ALB/FRU','FRU/ALB','FRU/POTY/FRU','FOLGA','BANCO DE HORAS'],
            iturama: ['ITM/ALB','ALB/ITM','ITM/POTY/ITM'],
            patos: ['PTS/ALB/PTS','PTS/ALB','ALB/PTS','FOLGA','BANCO DE HORAS']
        };

        // Normalization helpers
        function removeDiacritics(text) { return String(text).normalize('NFD').replace(/[\u0300-\u036f]/g,''); }
        function normalizeString(text) {
            if (text === undefined || text === null) return '';
            return removeDiacritics(String(text)).toUpperCase().replace(/\s+/g,' ').trim();
        }
        function normalizeFleetKey(text) { return normalizeString(text).replace(/[^A-Z0-9]/g,''); }
        function normalizePlateKey(cavalo, implemento) {
            const c = normalizeFleetKey(cavalo);
            let imp = normalizeString(implemento).replace(/[^A-Z0-9\/]/g,'');
            imp = imp.split('/').filter(Boolean).join('/');
            return `${c}/${imp}`;
        }

        window.onload = () => {
            loadSavedData();
            setTodayDate();
            checkBasesStatus();
            // ensure there's at least one empty row in the active tab
            addRow('fabrica');
        };

        function setTodayDate() {
            const today = new Date().toISOString().split('T')[0];
            const el = document.getElementById('scheduleDate');
            if (el) el.value = today;
        }

        function checkBasesStatus() {
            let msg = '';
            if (Object.keys(driversById).length === 0) msg += '⚠️ Base de motoristas não configurada. ';
            else msg += `✅ ${Object.keys(driversById).length} motoristas carregados. `;
            if (Object.keys(platesByFleet).length === 0 && Object.keys(platesNormalized).length === 0) msg += '⚠️ Base de placas não configurada. ';
            else msg += '✅ Base de placas carregada. ';
            if (msg) showAlert('alertInfo', msg);
        }

        function loadSavedData() {
            const sDriversById = localStorage.getItem('driversById');
            const sDriversByName = localStorage.getItem('driversByNormalizedName');
            const sPlatesFleet = localStorage.getItem('platesByFleet');
            const sPlatesNorm = localStorage.getItem('platesNormalized');
            const sSchedules = localStorage.getItem('schedulesByDate');
            const sCurrent = localStorage.getItem('currentSchedule');
            const sCurrentDate = localStorage.getItem('currentScheduleDate');

            if (sDriversById) driversById = JSON.parse(sDriversById);
            if (sDriversByName) driversByNormalizedName = JSON.parse(sDriversByName);
            if (sPlatesFleet) platesByFleet = JSON.parse(sPlatesFleet);
            if (sPlatesNorm) platesNormalized = JSON.parse(sPlatesNorm);
            if (sSchedules) schedulesByDate = JSON.parse(sSchedules);

            if (Object.keys(driversById).length) {
                document.getElementById('driversStatus').innerText = `✅ ${Object.keys(driversById).length} motoristas carregados`;
                refreshDriverDatalist();
            }
            if (Object.keys(platesByFleet).length || Object.keys(platesNormalized).length) {
                document.getElementById('platesStatus').innerText = `✅ ${Object.keys(platesByFleet).length + Object.keys(platesNormalized).length} placas carregadas`;
            }

            // Load schedule for today if exists, else if currentSchedule saved load it
            const today = new Date().toISOString().split('T')[0];
            if (schedulesByDate[today]) {
                currentSchedule = schedulesByDate[today];
                currentScheduleDate = today;
                loadScheduleData();
            } else if (sCurrent && sCurrentDate) {
                currentSchedule = JSON.parse(sCurrent);
                currentScheduleDate = sCurrentDate;
                // make date input reflect it
                const dateEl = document.getElementById('scheduleDate');
                if (dateEl) dateEl.value = sCurrentDate;
                loadScheduleData();
            } else {
                currentSchedule = null;
                currentScheduleDate = null;
                updateSavedIndicator();
            }

            // display previous day's schedule
            displayPreviousSchedule();
        }

        function displayPreviousSchedule() {
            const container = document.getElementById('previousSchedule');
            const yesterday = new Date(Date.now() - 86400000).toISOString().split('T')[0];
            const schedule = schedulesByDate[yesterday] || null;
            if (!schedule) {
                container.innerHTML = '<div class="empty-state">Nenhuma programação registrada no dia anterior</div>';
                return;
            }

            let html = `<div style="background:white;padding:20px;border-radius:8px;"><h4 style="margin-bottom:15px;">Data: ${yesterday}</h4>`;
            Object.keys(schedule.destinations).forEach(dest => {
                const destData = schedule.destinations[dest];
                if (destData.rows.length) {
                    html += `<h5 style="margin-top:20px;margin-bottom:10px;color:#2c3e50;text-transform:uppercase;">${dest}</h5>`;
                    html += '<table class="schedule-table"><thead><tr><th>Cavalo</th><th>Implemento</th><th>Colaborador</th><th>Cargo</th><th>Placa Cavalo</th><th>Placa Implemento</th><th>Programação</th></tr></thead><tbody>';
                    destData.rows.forEach(r => {
                        const pc = r.placaCavalo || r.placa || '';
                        const pi = r.placaImplemento || '';
                        html += `<tr><td>${r.cavalo||'-'}</td><td>${r.implemento||'-'}</td><td>${r.colaborador||'-'}</td><td>${r.cargo||'-'}</td><td>${pc||'-'}</td><td>${pi||'-'}</td><td>${r.route||'-'}</td></tr>`;
                    });
                    html += '</tbody></table>';
                    if (destData.obs) html += `<p style="margin-top:10px;"><strong>Observações:</strong> ${destData.obs}</p>`;
                }
            });
            html += '</div>';
            container.innerHTML = html;
        }

        // File inputs
        document.getElementById('driversFile').addEventListener('change', e => processFixedFile(e, 'drivers'));
        document.getElementById('platesFile').addEventListener('change', e => processFixedFile(e, 'plates'));

        function processFixedFile(e, type) {
            const file = e.target.files[0];
            if (!file) return;
            const fnEl = (type==='drivers') ? document.getElementById('driversFileName') : document.getElementById('platesFileName');
            const statusEl = (type==='drivers') ? document.getElementById('driversStatus') : document.getElementById('platesStatus');
            fnEl.textContent = `📄 ${file.name}`;
            const reader = new FileReader();
            reader.onload = evt => {
                try {
                    const data = new Uint8Array(evt.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
                    const jsonData = XLSX.utils.sheet_to_json(firstSheet, { header: 1 });

                    if (type === 'drivers') {
                        driversById = {};
                        driversByNormalizedName = {};
                        for (let i = 1; i < jsonData.length; i++) {
                            const row = jsonData[i] || [];
                            const id = row[0] !== undefined ? String(row[0]).trim() : '';
                            const name = row[1] !== undefined ? String(row[1]).trim() : '';
                            const cargo = row[2] !== undefined ? String(row[2]).trim() : '';
                            if (id || name) {
                                if (id) driversById[id] = { id, name, cargo };
                                if (name) driversByNormalizedName[normalizeString(name)] = { id, name, cargo };
                            }
                        }
                        localStorage.setItem('driversById', JSON.stringify(driversById));
                        localStorage.setItem('driversByNormalizedName', JSON.stringify(driversByNormalizedName));
                        statusEl.innerText = `✅ ${Object.keys(driversById).length} motoristas importados (IDs) e ${Object.keys(driversByNormalizedName).length} nomes registrados`;
                        refreshDriverDatalist();
                        refreshAllCollaboratorInputs();
                        showAlert('alertSuccess','✅ Base de motoristas atualizada!');
                    } else {
                        platesByFleet = {};
                        platesNormalized = {};
                        for (let i = 1; i < jsonData.length; i++) {
                            const row = jsonData[i] || [];
                            const fleet = row[0] !== undefined ? String(row[0]).trim() : '';
                            const placa = row[1] !== undefined ? String(row[1]).trim() : '';
                            if (fleet && placa) {
                                platesByFleet[normalizeFleetKey(fleet)] = placa;
                            } else {
                                const cavalo = row[0] !== undefined ? String(row[0]).trim() : '';
                                const placaCavalo = row[1] !== undefined ? String(row[1]).trim() : '';
                                const implemento = row[2] !== undefined ? String(row[2]).trim() : '';
                                const placaImplemento = row[3] !== undefined ? String(row[3]).trim() : '';
                                if (cavalo && implemento) {
                                    platesNormalized[normalizePlateKey(cavalo, implemento)] = { placaCavalo: placaCavalo||'', placaImplemento: placaImplemento||'' };
                                }
                            }
                        }
                        localStorage.setItem('platesByFleet', JSON.stringify(platesByFleet));
                        localStorage.setItem('platesNormalized', JSON.stringify(platesNormalized));
                        statusEl.innerText = `✅ ${Object.keys(platesByFleet).length} placas (por frota) e ${Object.keys(platesNormalized).length} entradas legadas`;
                        refreshAllPlateInputs();
                        showAlert('alertSuccess','✅ Base de placas atualizada!');
                    }
                    checkBasesStatus();
                } catch (err) {
                    console.error('Erro ao processar arquivo:', err);
                    alert('Erro ao processar o arquivo. Verifique o formato e tente novamente.');
                }
            };
            reader.readAsArrayBuffer(file);
        }

        function refreshDriverDatalist() {
            const dl = document.getElementById('driversList');
            dl.innerHTML = '';
            const names = Object.values(driversByNormalizedName).map(x => x.name).filter(Boolean).sort();
            names.forEach(n => {
                const opt = document.createElement('option');
                opt.value = n;
                dl.appendChild(opt);
            });
        }

        function refreshAllCollaboratorInputs() {
            document.querySelectorAll('input[id$="-colaborador-"]').forEach(inp => {
                inp.setAttribute('list','driversList');
                updateCargoFromInputElement(inp);
            });
        }

        function refreshAllPlateInputs() {
            document.querySelectorAll('input[id$="-cavalo-"]').forEach(cavInput => {
                const idParts = cavInput.id.split('-');
                const dest = idParts[0];
                const rowId = idParts[idParts.length - 1];
                updatePlacas(dest, rowId);
            });
        }

        function addRow(destination) {
            const tbody = document.getElementById(`${destination}-tbody`);
            const rowId = rowCounters[destination]++;
            const routes = destinationRoutes[destination];
            const row = document.createElement('tr');
            row.id = `${destination}-row-${rowId}`;
            row.innerHTML = `
                <td><input type="text" id="${destination}-cavalo-${rowId}" oninput="updatePlacas('${destination}', ${rowId})" placeholder="Ex: 6553"></td>
                <td><input type="text" id="${destination}-implemento-${rowId}" oninput="updatePlacas('${destination}', ${rowId})" placeholder="Ex: 2509/2609 (opcional)"></td>
                <td><input type="text" id="${destination}-colaborador-${rowId}" list="driversList" oninput="updateCargo('${destination}', ${rowId})" placeholder="Digite ID ou Nome"></td>
                <td><input type="text" id="${destination}-cargo-${rowId}" readonly placeholder="Cargo será preenchido automaticamente"></td>
                <td><input type="text" id="${destination}-placaCavalo-${rowId}" readonly placeholder="Placa do Cavalo"></td>
                <td><input type="text" id="${destination}-placaImplemento-${rowId}" readonly placeholder="Placa do Implemento"></td>
                <td>
                    <select id="${destination}-route-${rowId}">
                        <option value="">Selecione</option>
                        ${routes.map(r => `<option value="${r}">${r}</option>`).join('')}
                    </select>
                </td>
                <td><button class="btn btn-remove" onclick="removeRow('${destination}', ${rowId})">🗑️</button></td>
            `;
            tbody.appendChild(row);
            // attach datalist and initial updates
            const colInput = document.getElementById(`${destination}-colaborador-${rowId}`);
            if (colInput) colInput.setAttribute('list','driversList');
        }

        function removeRow(destination, rowId) {
            const row = document.getElementById(`${destination}-row-${rowId}`);
            if (row) row.remove();
        }

        function updatePlacas(destination, rowId) {
            const cavaloInput = document.getElementById(`${destination}-cavalo-${rowId}`);
            const implementoInput = document.getElementById(`${destination}-implemento-${rowId}`);
            const placaCavInput = document.getElementById(`${destination}-placaCavalo-${rowId}`);
            const placaImpInput = document.getElementById(`${destination}-placaImplemento-${rowId}`);
            if (!cavaloInput || !implementoInput || !placaCavInput || !placaImpInput) return;

            const cavalo = cavaloInput.value || '';
            const implemento = implementoInput.value || '';

            // placa do Cavalo: busca por frota
            const normFleetCav = normalizeFleetKey(cavalo);
            if (normFleetCav && platesByFleet[normFleetCav]) {
                placaCavInput.value = platesByFleet[normFleetCav];
            } else {
                placaCavInput.value = '';
            }

            // placa do Implemento: pode ser múltiplos implementos separados por '/'
            let placaImpValue = '';
            const parts = (implemento || '').split('/').map(p => p.trim()).filter(Boolean);
            if (parts.length) {
                const foundParts = [];
                for (const part of parts) {
                    const nf = normalizeFleetKey(part);
                    if (nf && platesByFleet[nf]) {
                        foundParts.push(platesByFleet[nf]);
                    } else {
                        // tente legacy key cavalo/part
                        const legacyKey = normalizePlateKey(cavalo, part);
                        if (legacyKey && platesNormalized[legacyKey] && platesNormalized[legacyKey].placaImplemento) {
                            foundParts.push(platesNormalized[legacyKey].placaImplemento);
                        }
                    }
                }
                placaImpValue = foundParts.join(' | ');
            } else {
                // se não tem '/', tente lookup direto com implemento como frota
                const normImp = normalizeFleetKey(implemento);
                if (normImp && platesByFleet[normImp]) {
                    placaImpValue = platesByFleet[normImp];
                } else {
                    const key = normalizePlateKey(cavalo, implemento);
                    if (key && platesNormalized[key] && platesNormalized[key].placaImplemento) {
                        placaImpValue = platesNormalized[key].placaImplemento;
                        if (!placaCavInput.value && platesNormalized[key].placaCavalo) {
                            placaCavInput.value = platesNormalized[key].placaCavalo;
                        }
                    }
                }
            }
            placaImpInput.value = placaImpValue;
        }

        function isIdString(s) {
            if (!s) return false;
            return /^[0-9A-Z\-]+$/i.test(s.trim()) && !/\s/.test(s.trim());
        }

        function updateCargo(destination, rowId) {
            const colaboradorInput = document.getElementById(`${destination}-colaborador-${rowId}`);
            const cargoInput = document.getElementById(`${destination}-cargo-${rowId}`);
            if (!colaboradorInput || !cargoInput) return;
            updateCargoFromInputElement(colaboradorInput, cargoInput);
        }

        function updateCargoFromInputElement(inputEl, cargoEl) {
            const val = (inputEl.value || '').trim();
            if (!cargoEl) {
                const parts = inputEl.id.split('-');
                const dest = parts[0];
                const rowId = parts[parts.length - 1];
                cargoEl = document.getElementById(`${dest}-cargo-${rowId}`);
            }
            if (!val) { if (cargoEl) cargoEl.value = ''; return; }

            if (isIdString(val) && driversById[val]) {
                const found = driversById[val];
                inputEl.value = found.name || val;
                if (cargoEl) cargoEl.value = found.cargo || '';
                return;
            }

            const norm = normalizeString(val);
            if (driversByNormalizedName[norm]) {
                const found = driversByNormalizedName[norm];
                inputEl.value = found.name;
                if (cargoEl) cargoEl.value = found.cargo || '';
                return;
            }

            for (const k of Object.keys(driversByNormalizedName)) {
                if (k.startsWith(norm) && norm.length > 0) {
                    const found = driversByNormalizedName[k];
                    inputEl.value = found.name;
                    if (cargoEl) cargoEl.value = found.cargo || '';
                    return;
                }
            }
            if (cargoEl) cargoEl.value = '';
        }

        function switchTab(destination, btn) {
            try {
                document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                document.querySelectorAll('.destination-content').forEach(c => c.classList.remove('active'));
                if (!btn) btn = document.querySelector(`.tab-btn[onclick*="${destination}"]`);
                if (btn) btn.classList.add('active');
                const content = document.getElementById(`${destination}-content`);
                if (content) content.classList.add('active');
            } catch (err) {
                console.error('Erro em switchTab', err);
            }
        }

        // Save schedule: stores in schedulesByDate keyed by date
        function saveSchedule() {
            const date = document.getElementById('scheduleDate').value;
            if (!date) { alert('Selecione uma data antes de salvar.'); return; }

            const scheduleData = { date, destinations: {} };
            Object.keys(destinationRoutes).forEach(dest => {
                const rows = [];
                const tbody = document.getElementById(`${dest}-tbody`);
                const trs = tbody.getElementsByTagName('tr');
                for (let tr of trs) {
                    const inputs = tr.getElementsByTagName('input');
                    const select = tr.getElementsByTagName('select')[0];
                    rows.push({
                        cavalo: inputs[0] ? inputs[0].value : '',
                        implemento: inputs[1] ? inputs[1].value : '',
                        colaborador: inputs[2] ? inputs[2].value : '',
                        cargo: inputs[3] ? inputs[3].value : '',
                        placaCavalo: inputs[4] ? inputs[4].value : '',
                        placaImplemento: inputs[5] ? inputs[5].value : '',
                        route: select ? select.value : ''
                    });
                }
                scheduleData.destinations[dest] = { rows, obs: document.getElementById(`${dest}-obs`).value };
            });

            // store by date
            schedulesByDate[date] = scheduleData;
            localStorage.setItem('schedulesByDate', JSON.stringify(schedulesByDate));

            // also update currentSchedule/currentScheduleDate for UI and export
            currentSchedule = scheduleData;
            currentScheduleDate = date;
            localStorage.setItem('currentSchedule', JSON.stringify(currentSchedule));
            localStorage.setItem('currentScheduleDate', currentScheduleDate);

            updateSavedIndicator();
            showAlert('alertSuccess', '✅ Programação salva com sucesso!');
            // update previous schedule display (yesterday)
            displayPreviousSchedule();
        }

        function updateSavedIndicator() {
            const indicator = document.getElementById('savedIndicator');
            const date = document.getElementById('scheduleDate').value;
            if (schedulesByDate[date]) {
                indicator.textContent = `📌 Salvo para ${date}`;
            } else {
                indicator.textContent = '';
            }
        }

        // Load schedule by the date currently selected in the date picker
        function loadScheduleBySelectedDate() {
            const date = document.getElementById('scheduleDate').value;
            if (!date) { alert('Selecione uma data para carregar.'); return; }
            loadScheduleByDate(date);
        }

        function loadScheduleByDate(date) {
            if (!date) return;
            const schedule = schedulesByDate[date];
            if (!schedule) {
                // if none exist, clear UI and notify
                clearAllTables();
                currentSchedule = null;
                currentScheduleDate = null;
                localStorage.removeItem('currentSchedule');
                localStorage.removeItem('currentScheduleDate');
                updateSavedIndicator();
                showAlert('alertInfo', `⚠️ Não existe programação salva para ${date}. Você pode criar e salvar uma nova.`);
                return;
            }

            // load into UI
            currentSchedule = schedule;
            currentScheduleDate = date;
            localStorage.setItem('currentSchedule', JSON.stringify(currentSchedule));
            localStorage.setItem('currentScheduleDate', currentScheduleDate);

            // Populate UI
            Object.keys(currentSchedule.destinations).forEach(dest => {
                const destData = currentSchedule.destinations[dest];
                const tbody = document.getElementById(`${dest}-tbody`);
                tbody.innerHTML = '';
                rowCounters[dest] = 0;
                destData.rows.forEach((row, i) => {
                    addRow(dest);
                    const rowId = i;
                    const cav = document.getElementById(`${dest}-cavalo-${rowId}`);
                    const imp = document.getElementById(`${dest}-implemento-${rowId}`);
                    const col = document.getElementById(`${dest}-colaborador-${rowId}`);
                    const car = document.getElementById(`${dest}-cargo-${rowId}`);
                    const pc = document.getElementById(`${dest}-placaCavalo-${rowId}`);
                    const pi = document.getElementById(`${dest}-placaImplemento-${rowId}`);
                    const rou = document.getElementById(`${dest}-route-${rowId}`);
                    if (cav) cav.value = row.cavalo || '';
                    if (imp) imp.value = row.implemento || '';
                    if (col) { col.value = row.colaborador || ''; col.setAttribute('list','driversList'); }
                    if (car) car.value = row.cargo || '';
                    if (pc) pc.value = row.placaCavalo || row.placa || '';
                    if (pi) pi.value = row.placaImplemento || '';
                    if (rou) rou.value = row.route || '';
                    updatePlacas(dest, rowId);
                    updateCargo(dest, rowId);
                });
                const obsEl = document.getElementById(`${dest}-obs`);
                if (obsEl) obsEl.value = destData.obs || '';
            });
            refreshAllCollaboratorInputs();
            updateSavedIndicator();
            showAlert('alertSuccess', `✅ Programação de ${date} carregada.`);
        }

        function clearAllTables() {
            Object.keys(destinationRoutes).forEach(dest => {
                document.getElementById(`${dest}-tbody`).innerHTML = '';
                rowCounters[dest] = 0;
                document.getElementById(`${dest}-obs`).value = '';
            });
            // add one empty row for convenience
            addRow('fabrica');
        }

        function deleteScheduleBySelectedDate() {
            const date = document.getElementById('scheduleDate').value;
            if (!date) { alert('Selecione uma data para excluir.'); return; }
            if (!schedulesByDate[date]) { alert(`Não existe programação salva para ${date}`); return; }
            if (!confirm(`Tem certeza que deseja excluir a programação de ${date}?`)) return;
            delete schedulesByDate[date];
            localStorage.setItem('schedulesByDate', JSON.stringify(schedulesByDate));
            // if it was current, clear
            if (currentScheduleDate === date) {
                currentSchedule = null;
                currentScheduleDate = null;
                localStorage.removeItem('currentSchedule');
                localStorage.removeItem('currentScheduleDate');
                clearAllTables();
            }
            updateSavedIndicator();
            showAlert('alertSuccess', `🗑️ Programação de ${date} removida.`);
            displayPreviousSchedule();
        }

        // Duplicate (copy) schedule from current selected date into another date
        function duplicateSchedule() {
            const sourceDate = document.getElementById('scheduleDate').value;
            if (!sourceDate || !schedulesByDate[sourceDate]) {
                alert('Coloque uma data que já tenha programação salva para duplicar (source).');
                return;
            }
            const target = prompt('Digite a data de destino (YYYY-MM-DD) para copiar a programação:');
            if (!target) return;
            // basic date format check
            if (!/^\d{4}-\d{2}-\d{2}$/.test(target)) { alert('Formato de data inválido. Use YYYY-MM-DD.'); return; }
            // deep copy
            schedulesByDate[target] = JSON.parse(JSON.stringify(schedulesByDate[sourceDate]));
            schedulesByDate[target].date = target;
            localStorage.setItem('schedulesByDate', JSON.stringify(schedulesByDate));
            showAlert('alertSuccess', `✅ Programação copiada para ${target}`);
            displayPreviousSchedule();
        }

        function exportToExcel() {
            if (!currentSchedule) { alert('Salve ou carregue uma programação antes de exportar!'); return; }
            const wb = XLSX.utils.book_new();
            const colaboradores = new Set();
            Object.values(currentSchedule.destinations).forEach(dest => {
                dest.rows.forEach(row => {
                    if (row.colaborador) {
                        const nomeCompleto = row.cargo ? `${row.colaborador} - ${row.cargo}` : row.colaborador;
                        colaboradores.add(nomeCompleto);
                    }
                });
            });
            const colaboradoresData = [['COLABORADORES']];
            Array.from(colaboradores).sort().forEach(c => colaboradoresData.push([c]));
            const wsCol = XLSX.utils.aoa_to_sheet(colaboradoresData);
            XLSX.utils.book_append_sheet(wb, wsCol, 'Colaboradores');

            Object.keys(currentSchedule.destinations).forEach(dest => {
                const destData = currentSchedule.destinations[dest];
                if (!destData.rows.length) return;
                const sheetData = [['PROGRAMAÇÃO - ' + dest.toUpperCase()], ['Data:', currentSchedule.date], [], ['Cavalo','Implemento','Colaborador','Cargo','Placa Cavalo','Placa Implemento','Programação']];
                destData.rows.forEach(r => {
                    if (r.cavalo || r.implemento) sheetData.push([r.cavalo||'', r.implemento||'', r.colaborador||'', r.cargo||'', r.placaCavalo||r.placa||'', r.placaImplemento||'', r.route||'']);
                });
                if (destData.obs) { sheetData.push([]); sheetData.push(['OBSERVAÇÕES:']); sheetData.push([destData.obs]); }
                const ws = XLSX.utils.aoa_to_sheet(sheetData);
                // set column widths so "Placa Implemento" column is wider in exported file
                ws['!cols'] = [{wch:12},{wch:20},{wch:25},{wch:20},{wch:20},{wch:30},{wch:25}];
                XLSX.utils.book_append_sheet(wb, ws, dest.charAt(0).toUpperCase() + dest.slice(1));
            });

            const date = currentSchedule.date || new Date().toISOString().split('T')[0];
            XLSX.writeFile(wb, `Programacao_Frota_${date}.xlsx`);
            showAlert('alertSuccess','✅ Arquivo Excel exportado com sucesso!');
        }

        function showAlert(id, message) {
            const el = document.getElementById(id);
            if (!el) return;
            el.textContent = message;
            el.style.display = 'block';
            setTimeout(()=> el.style.display = 'none', 5000);
        }
    </script>
</body>
</html># Programa-o-Di-ria-2.0
