<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Panel de Aprobación — Elizabeth Alhadad</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #f3f4f6; min-height: 100vh; }
  .header { background: #1a1a2e; color: white; padding: 16px 20px; display: flex; align-items: center; gap: 12px; }
  .header h1 { font-size: 18px; font-weight: 600; }
  .header p { font-size: 12px; opacity: 0.7; margin-top: 2px; }
  .config { background: white; border-bottom: 1px solid #e5e7eb; padding: 16px 20px; }
  .config-row { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
  .config input { flex: 1; min-width: 200px; padding: 8px 12px; border: 1px solid #e5e7eb; border-radius: 8px; font-size: 13px; }
  .config button { padding: 8px 16px; background: #1a1a2e; color: white; border: none; border-radius: 8px; font-size: 13px; cursor: pointer; white-space: nowrap; }
  .config button:hover { background: #2d2d4e; }
  .container { max-width: 800px; margin: 20px auto; padding: 0 16px; }
  .status { padding: 12px 16px; border-radius: 10px; font-size: 13px; margin-bottom: 16px; }
  .status.loading { background: #eff6ff; color: #1d4ed8; border: 1px solid #bfdbfe; }
  .status.error { background: #fef2f2; color: #dc2626; border: 1px solid #fecaca; }
  .status.empty { background: #f0fdf4; color: #16a34a; border: 1px solid #bbf7d0; }
  .execution-card { background: white; border-radius: 12px; border: 1px solid #e5e7eb; margin-bottom: 12px; overflow: hidden; }
  .exec-header { padding: 14px 16px; border-bottom: 1px solid #f3f4f6; display: flex; align-items: center; justify-content: space-between; gap: 10px; }
  .exec-title { font-size: 14px; font-weight: 600; color: #111827; }
  .exec-time { font-size: 11px; color: #9ca3af; }
  .badge { padding: 3px 10px; border-radius: 20px; font-size: 11px; font-weight: 600; }
  .badge-waiting { background: #fef3c7; color: #d97706; }
  .badge-lead { background: #fee2e2; color: #dc2626; }
  .badge-normal { background: #eff6ff; color: #2563eb; }
  .exec-body { padding: 14px 16px; }
  .comment-box { background: #f9fafb; border-radius: 8px; padding: 12px; margin-bottom: 12px; }
  .comment-label { font-size: 11px; color: #6b7280; font-weight: 600; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
  .comment-text { font-size: 14px; color: #111827; line-height: 1.6; }
  .reply-box { background: #f0fdf4; border: 1px solid #bbf7d0; border-radius: 8px; padding: 12px; margin-bottom: 12px; }
  .reply-label { font-size: 11px; color: #16a34a; font-weight: 600; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
  .reply-text { font-size: 14px; color: #15803d; line-height: 1.6; font-style: italic; }
  .actions { display: flex; gap: 8px; flex-wrap: wrap; }
  .btn-approve { flex: 1; padding: 10px; background: #16a34a; color: white; border: none; border-radius: 8px; font-size: 13px; font-weight: 600; cursor: pointer; transition: background 0.15s; }
  .btn-approve:hover { background: #15803d; }
  .btn-ignore { flex: 1; padding: 10px; background: #6b7280; color: white; border: none; border-radius: 8px; font-size: 13px; font-weight: 600; cursor: pointer; transition: background 0.15s; }
  .btn-ignore:hover { background: #4b5563; }
  .btn-approve:disabled, .btn-ignore:disabled { opacity: 0.5; cursor: not-allowed; }
  .exec-id { font-size: 10px; color: #d1d5db; margin-top: 8px; }
  .refresh-btn { display: block; width: 100%; padding: 12px; background: white; border: 1px dashed #e5e7eb; border-radius: 10px; color: #6b7280; font-size: 13px; cursor: pointer; text-align: center; margin-bottom: 16px; }
  .refresh-btn:hover { background: #f9fafb; }
  .saved-key { font-size: 11px; color: #16a34a; margin-top: 6px; }
</style>
</head>
<body>

<div class="header">
  <div>
    <h1>Panel de Aprobación YouTube</h1>
    <p>@ElizabethAlhadad — Real Estate Agent</p>
  </div>
</div>

<div class="config">
  <div class="config-row">
    <input type="password" id="api-key-input" placeholder="Pega tu n8n API key aquí..." />
    <button onclick="saveAndLoad()">Cargar comentarios</button>
  </div>
  <div id="key-status"></div>
</div>

<div class="container">
  <button class="refresh-btn" onclick="loadExecutions()">🔄 Actualizar comentarios pendientes</button>
  <div id="status-area"></div>
  <div id="executions-area"></div>
</div>

<script>
const N8N_URL = 'https://primary-production-72d4.up.railway.app';
let apiKey = localStorage.getItem('elizabeth_n8n_key') || '';

if (apiKey) {
  document.getElementById('api-key-input').value = '••••••••••••••••';
  document.getElementById('key-status').innerHTML = '<div class="saved-key">✓ API key guardada</div>';
  loadExecutions();
}

function saveAndLoad() {
  const input = document.getElementById('api-key-input').value.trim();
  if (input && input !== '••••••••••••••••') {
    apiKey = input;
    localStorage.setItem('elizabeth_n8n_key', apiKey);
    document.getElementById('api-key-input').value = '••••••••••••••••';
    document.getElementById('key-status').innerHTML = '<div class="saved-key">✓ API key guardada</div>';
  }
  loadExecutions();
}

async function loadExecutions() {
  const statusArea = document.getElementById('status-area');
  const execArea = document.getElementById('executions-area');
  
  if (!apiKey) {
    statusArea.innerHTML = '<div class="status error">Ingresa tu n8n API key para continuar</div>';
    return;
  }

  statusArea.innerHTML = '<div class="status loading">⏳ Cargando comentarios pendientes...</div>';
  execArea.innerHTML = '';

  try {
    const res = await fetch(`${N8N_URL}/api/v1/executions?status=waiting&limit=20`, {
      headers: { 'X-N8N-API-KEY': apiKey }
    });

    if (!res.ok) throw new Error('Error de autenticación');
    
    const data = await res.json();
    const executions = data.data || [];

    if (executions.length === 0) {
      statusArea.innerHTML = '<div class="status empty">✓ No hay comentarios pendientes de aprobación ahora mismo</div>';
      return;
    }

    statusArea.innerHTML = `<div class="status loading">${executions.length} comentario(s) esperando tu aprobación</div>`;
    
    execArea.innerHTML = executions.map(exec => {
      const isLead = exec.data?.resultData?.runData?.['Procesar Respuesta Claude']?.[0]?.data?.main?.[0]?.[0]?.json?.es_lead;
      const commentText = exec.data?.resultData?.runData?.['Procesar Respuesta Claude']?.[0]?.data?.main?.[0]?.[0]?.json?.comment_text || 'Comentario no disponible';
      const replyText = exec.data?.resultData?.runData?.['Procesar Respuesta Claude']?.[0]?.data?.main?.[0]?.[0]?.json?.respuesta_sugerida || 'Respuesta no disponible';
      const author = exec.data?.resultData?.runData?.['Procesar Respuesta Claude']?.[0]?.data?.main?.[0]?.[0]?.json?.author || 'Usuario';
      const time = new Date(exec.startedAt).toLocaleString('es-CO');

      return `
        <div class="execution-card" id="card-${exec.id}">
          <div class="exec-header">
            <div>
              <div class="exec-title">💬 ${author}</div>
              <div class="exec-time">${time}</div>
            </div>
            <span class="badge ${isLead ? 'badge-lead' : 'badge-normal'}">${isLead ? '🔥 Lead Caliente' : 'Comentario'}</span>
          </div>
          <div class="exec-body">
            <div class="comment-box">
              <div class="comment-label">Comentario</div>
              <div class="comment-text">${commentText}</div>
            </div>
            <div class="reply-box">
              <div class="reply-label">Respuesta sugerida por Claude</div>
              <div class="reply-text">${replyText}</div>
            </div>
            <div class="actions">
              <button class="btn-approve" onclick="respond('${exec.id}', 'aprobar', this)">✅ Aprobar y publicar</button>
              <button class="btn-ignore" onclick="respond('${exec.id}', 'ignorar', this)">⏭ Ignorar</button>
            </div>
            <div class="exec-id">ID: ${exec.id}</div>
          </div>
        </div>
      `;
    }).join('');

  } catch(e) {
    statusArea.innerHTML = `<div class="status error">Error: ${e.message} — Verifica tu API key</div>`;
  }
}

async function respond(execId, action, btn) {
  btn.disabled = true;
  btn.textContent = action === 'aprobar' ? '⏳ Publicando...' : '⏳ Ignorando...';
  
  try {
    const webhookUrl = `${N8N_URL}/webhook-waiting/${execId}?response=${action}`;
    const res = await fetch(webhookUrl, { method: 'GET' });
    
    const card = document.getElementById(`card-${execId}`);
    if (res.ok || res.status === 200) {
      card.style.opacity = '0.5';
      card.innerHTML = `<div style="padding:16px;text-align:center;color:#16a34a;font-weight:600">${action === 'aprobar' ? '✅ Respuesta publicada en YouTube' : '⏭ Comentario ignorado'}</div>`;
      setTimeout(() => card.remove(), 2000);
    } else {
      const data = await res.json();
      card.querySelector('.actions').insertAdjacentHTML('afterend', `<div style="color:#dc2626;font-size:12px;margin-top:8px">Error: ${data.message}</div>`);
      btn.disabled = false;
      btn.textContent = action === 'aprobar' ? '✅ Aprobar y publicar' : '⏭ Ignorar';
    }
  } catch(e) {
    btn.disabled = false;
    btn.textContent = action === 'aprobar' ? '✅ Aprobar y publicar' : '⏭ Ignorar';
  }
}

setInterval(loadExecutions, 60000);
</script>
</body>
</html>
