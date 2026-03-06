[index.html.html](https://github.com/user-attachments/files/25785237/index.html.html)
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <link rel="icon" type="image/png" href="favicon.png?v=3">
    <link rel="apple-touch-icon" href="favicon.png?v=3">
    <link rel="shortcut icon" href="favicon.png?v=3" type="image/png">

    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>BioFuel Pro - Analisador</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        :root { --primary: #003366; --success: #28a745; --danger: #dc3545; --bg: #f4f7f6; }
        * { box-sizing: border-box; font-family: sans-serif; }
        body { background: var(--bg); margin: 0; padding: 10px 10px 80px 10px; display: flex; flex-direction: column; align-items: center; }
        
        .container { background: white; width: 100%; max-width: 500px; padding: 20px; border-radius: 16px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        h2 { color: var(--primary); text-align: center; margin-top: 0; border-bottom: 2px solid #eee; padding-bottom: 10px; }

        .input-group { margin-bottom: 12px; }
        label { display: block; font-size: 0.8rem; font-weight: bold; color: #555; margin-bottom: 4px; }
        input, select { width: 100%; height: 48px; padding: 10px; border: 2px solid #ddd; border-radius: 10px; font-size: 16px; }

        .btn-calc { background: var(--primary); color: white; width: 100%; height: 55px; border: none; border-radius: 10px; font-size: 1rem; font-weight: bold; margin-top: 10px; }
        .btn-save { background: var(--success); color: white; width: 100%; height: 50px; border: none; border-radius: 10px; font-size: 1rem; font-weight: bold; margin-top: 15px; display: none; }

        /* Estilo do Laudo */
        #laudo-area { display: none; background: white; padding: 20px; border: 2px solid #1a1a1a; margin-top: 20px; }
        .header-laudo { text-align: center; border-bottom: 2px solid #333; padding-bottom: 10px; margin-bottom: 15px; }
        .info-linha { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #eee; font-size: 0.9rem; }
        .resultado-box { margin-top: 15px; padding: 15px; text-align: center; font-size: 1.1rem; font-weight: 800; border-radius: 8px; }

        /* BARRA DE CONFIGURAÇÃO INFERIOR */
        .bottom-config-bar {
            position: fixed; bottom: 0; left: 0; width: 100%; background: #fff;
            box-shadow: 0 -2px 10px rgba(0,0,0,0.1); padding: 10px; border-top: 1px solid #ddd;
            z-index: 100;
        }
        #panel-config { display: none; padding: 15px; background: #fff; max-width: 500px; margin: 0 auto; }
        .config-toggle { width: 100%; max-width: 500px; margin: 0 auto; display: block; background: #666; color: white; height: 40px; border-radius: 8px; border: none; font-weight: bold; }
        .config-row { display: grid; grid-template-columns: 2fr 1fr 1fr; gap: 8px; margin-bottom: 10px; align-items: center; font-size: 0.85rem; }
        .config-row input { height: 35px; border: 1px solid #ccc; font-size: 14px; text-align: center; }
    </style>
</head>
<body>

<div class="container">
    <h2>BioFuel Analisador</h2>
    
    <div class="input-group">
        <label>Combustível</label>
        <select id="combustivel">
            <option value="Gasolina">Gasolina (Comum/Aditivada)</option>
            <option value="Etanol">Etanol (Comum/Aditivado)</option>
            <option value="Diesel">Diesel (S10/Aditivado)</option>
        </select>
    </div>

    <div class="input-group">
        <label>Temperatura Observada (°C)</label>
        <input type="number" id="temp" step="0.1" inputmode="decimal" placeholder="00.0">
    </div>

    <div class="input-group">
        <label>Densidade Lida (kg/m³)</label>
        <input type="number" id="dens" step="0.1" inputmode="decimal" placeholder="000.0">
    </div>

    <button class="btn-calc" onclick="gerarLaudo()">GERAR LAUDO TÉCNICO</button>

    <div id="laudo-area">
        <div class="header-laudo"><strong>LAUDO TÉCNICO DE CONFORMIDADE</strong></div>
        <div class="info-linha"><span>Data:</span> <span id="l-data"></span></div>
        <div class="info-linha"><span>Hora:</span> <span id="l-hora"></span></div>
        <div class="info-linha"><span>Produto:</span> <span id="l-prod"></span></div>
        <div class="info-linha"><span>Temp. Coleta:</span> <span id="l-temp"></span> °C</div>
        <div class="info-linha" style="border-top: 2px solid #333; margin-top:5px; background:#f9f9f9;">
            <span>DENSIDADE A 20°C:</span> <strong id="l-dens20"></strong>
        </div>
        <div id="varedito" class="resultado-box"></div>
    </div>

    <button class="btn-save" id="btnSave" onclick="baixarImagem()">💾 SALVAR LAUDO EM IMAGEM</button>
</div>

<div class="bottom-config-bar">
    <div id="panel-config">
        <div style="display:flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
            <strong>Limites da ANP (kg/m³)</strong>
            <button onclick="toggleConfig()" style="width:30px; height:30px; padding:0; background:#eee; color:#333; margin:0;">X</button>
        </div>
        <div class="config-row"><span>Produto</span><span>Mín</span><span>Máx</span></div>
        <div class="config-row"><span>Gasolinas</span><input id="min_gas" value="715"> <input id="max_gas" value="775"></div>
        <div class="config-row"><span>Etanóis</span><input id="min_eta" value="807.6"> <input id="max_eta" value="811.0"></div>
        <div class="config-row"><span>Diesels</span><input id="min_die" value="820"> <input id="max_die" value="850"></div>
        <button onclick="saveSettings()" style="height:40px; background:var(--primary); color:white; margin-top:5px;">Aplicar e Salvar</button>
    </div>
    <button class="config-toggle" onclick="toggleConfig()">⚙️ Ajustar Parâmetros Técnicos</button>
</div>

<script>
    let limites = JSON.parse(localStorage.getItem('biofuel_limits')) || {
        gas: { min: 715, max: 775 },
        eta: { min: 807.6, max: 811 },
        die: { min: 820, max: 850 }
    };

    function toggleConfig() {
        const p = document.getElementById('panel-config');
        p.style.display = p.style.display === 'block' ? 'none' : 'block';
    }

    function saveSettings() {
        limites.gas.min = parseFloat(document.getElementById('min_gas').value);
        limites.gas.max = parseFloat(document.getElementById('max_gas').value);
        limites.eta.min = parseFloat(document.getElementById('min_eta').value);
        limites.eta.max = parseFloat(document.getElementById('max_eta').value);
        limites.die.min = parseFloat(document.getElementById('min_die').value);
        limites.die.max = parseFloat(document.getElementById('max_die').value);
        localStorage.setItem('biofuel_limits', JSON.stringify(limites));
        alert("Parâmetros atualizados!");
        toggleConfig();
    }

    function gerarLaudo() {
        const prod = document.getElementById('combustivel').value;
        const t = parseFloat(document.getElementById('temp').value);
        const d = parseFloat(document.getElementById('dens').value);

        if (isNaN(t) || isNaN(d)) return alert("Preencha os dados de campo.");

        let fator = (prod === "Etanol") ? 0.75 : 0.72; 
        const d20 = (d + (t - 20) * fator).toFixed(1);

        // Captura Data e Hora atual
        const agora = new Date();
        const dataFormatada = agora.toLocaleDateString('pt-BR');
        const horaFormatada = agora.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' });

        document.getElementById('l-data').innerText = dataFormatada;
        document.getElementById('l-hora').innerText = horaFormatada;
        document.getElementById('l-prod').innerText = prod;
        document.getElementById('l-temp').innerText = t;
        document.getElementById('l-dens20').innerText = d20 + " kg/m³";

        const box = document.getElementById('varedito');
        let lim = (prod === "Gasolina") ? limites.gas : (prod === "Etanol" ? limites.eta : limites.die);
        const ok = (d20 >= lim.min && d20 <= lim.max);

        box.innerText = ok ? "✅ DENTRO DA NORMA" : "❌ FORA DE ESPECIFICAÇÃO";
        box.style.backgroundColor = ok ? "#d4edda" : "#f8d7da";
        box.style.color = ok ? "#155724" : "#721c24";

        document.getElementById('laudo-area').style.display = 'block';
        document.getElementById('btnSave').style.display = 'block';
        window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
    }

    function baixarImagem() {
        html2canvas(document.getElementById('laudo-area'), { scale: 3 }).then(canvas => {
            const link = document.createElement('a');
            link.download = `Relatorio_${Date.now()}.png`;
            link.href = canvas.toDataURL("image/png");
            link.click();
        });
    }
</script>
</body>
</html>
