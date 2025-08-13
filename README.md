# cleartrade-sentinel-demo
Formulario online
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>ClearTrade Sentinel — Demo</title>
  <style>
    body { font-family: Arial, sans-serif; background:#f6f7fb; margin:0; padding:24px; }
    .wrap { max-width: 640px; margin: 0 auto; background:#fff; border-radius:12px; padding:20px; box-shadow: 0 4px 14px rgba(0,0,0,.08); }
    h1 { margin:0 0 8px; font-size: 20px; }
    p { margin:0 0 16px; color:#444; }
    label { display:block; margin:12px 0 6px; font-weight:600; }
    input, select, button { width:100%; padding:12px; border:1px solid #d9d9d9; border-radius:8px; font-size:16px; }
    button { background:#0a66ff; color:#fff; border:none; margin-top:14px; }
    button:active { opacity:.9; }
    .result { margin-top:16px; padding:14px; border-left:6px solid; border-radius:8px; background:#f1f5ff; }
    .low { border-color:#2e7d32; background:#edf7ed; }
    .mid { border-color:#f57c00; background:#fff4e5; }
    .high { border-color:#c62828; background:#ffebee; }
    .hint { font-size:12px; color:#666; margin-top:8px; }
  </style>
</head>
<body>
  <div class="wrap">
    <h1>ClearTrade Sentinel — Demo</h1>
    <p>Ingresa un producto y un precio de referencia del importador real. El demo estima si hay <b>riesgo de competencia desleal</b> basado en reglas simples (sin conexión a datos externos).</p>

    <form id="form">
      <label>Producto</label>
      <input id="prod" placeholder="Ej. Pasta Barilla 360g caja individual" required />

      <label>Presentación</label>
      <input id="pres" placeholder="Ej. Caja de 12 unidades" required />

      <label>Precio de referencia (USD)</label>
      <input id="ref" type="number" step="0.01" placeholder="Ej. 18.50" required />

      <label>Ubicación / Región</label>
      <input id="loc" placeholder="Ej. Guatemala" required />

      <button type="submit">Analizar riesgo</button>
      <div class="hint">Nota: Este demo funciona 100% en el navegador (sin servidor). El análisis es heurístico de ejemplo.</div>
    </form>

    <div id="out" class="result" style="display:none;"></div>
  </div>

  <script>
    const form = document.getElementById('form');
    const out  = document.getElementById('out');

    form.addEventListener('submit', (e) => {
      e.preventDefault();
      const prod = document.getElementById('prod').value.trim();
      const pres = document.getElementById('pres').value.trim();
      const ref  = parseFloat(document.getElementById('ref').value);
      const loc  = document.getElementById('loc').value.trim();

      if (!isFinite(ref) || ref <= 0) {
        out.style.display = 'block';
        out.className = 'result high';
        out.innerHTML = 'Por favor, ingresa un precio de referencia válido mayor a 0.';
        return;
      }

      // Heurística offline de ejemplo:
      // - Si detectamos "promo", "2x1", "sin factura", "mercado gris" en la descripción simulada (no scraping)
      // - O si el "precio implícito" cae por debajo del 60% del precio de referencia => Riesgo Alto
      // - Por debajo del 80% => Riesgo Medio
      // - En caso contrario => Riesgo Bajo

      // Simulación: tomamos 3 "ofertas" ficticias basadas en el precio de referencia
      const offers = [
        ref * 0.95,  // cercana
        ref * 0.78,  // con descuento notable
        ref * 0.55   // sospechosa
      ];

      let risk = 'low';
      for (const p of offers) {
        if (p < ref * 0.6) { risk = 'high'; break; }
        if (p < ref * 0.8) { risk = (risk === 'high') ? 'high' : 'mid'; }
      }

      let msg = `<b>Producto:</b> ${prod} (${pres})<br><b>Ubicación:</b> ${loc}<br><b>Referencia:</b> $${ref.toFixed(2)}<br><br>`;
      if (risk === 'high') {
        out.className = 'result high';
        msg += '⚠️ <b>Riesgo Alto</b>: Se observan señales de precios de liquidación o mercado gris frente al precio de referencia. Recomendación: <b>no promover</b>, vigilar canales y <b>notificar a fábrica</b>.';
      } else if (risk === 'mid') {
        out.className = 'result mid';
        msg += '🟠 <b>Riesgo Medio</b>: Existen ofertas por debajo del margen esperado. Recomendación: <b>monitoreo</b> y validar condiciones comerciales locales.';
      } else {
        out.className = 'result low';
        msg += '✅ <b>Riesgo Bajo</b>: No hay señales de dumping/mercado gris con la referencia indicada.';
      }
      out.innerHTML = msg;
      out.style.display = 'block';
    });
  </script>
</body>
</html>
