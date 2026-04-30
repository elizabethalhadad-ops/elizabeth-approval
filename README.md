<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Panel de Aprobación — Elizabeth Alhadad</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #f3f4f6; min-height: 100vh; }
  .header { background: #1a1a2e; color: white; padding: 16px 20px; }
  .header h1 { font-size: 18px; font-weight: 600; }
  .header p { font-size: 12px; opacity: 0.7; margin-top: 2px; }
  .config { background: white; border-bottom: 1px solid #e5e7eb; padding: 16px 20px; }
  .config label { font-size: 12px; font-weight: 600; color: #6b7280; display: block; margin-bottom: 6px; }
  .config-row { display: flex; gap: 10px; align-items: center; }
  .config input { flex: 1; padding: 10px 12px; border: 1px solid #e5e7eb; border-radius: 8px; font-size: 13px; font-family: monospace; }
  .config button { padding: 10px 20px; background: #1a1a2e; color: white; border: none; border-radius: 8px; font-size: 13px; cursor: pointer; white-space: nowrap; }
  .config button:hover { background: #2d2d4e; }
  .container { max-width: 800px; margin: 20px auto; padding: 0 16px; }
  .status { padding: 12px 16px; border-radius: 10px; font-size: 13px; margin-bottom: 16px; }
  .status.loading { background: #eff6ff; color: #1d4ed8; border: 1px solid #bfdbfe; }
  .status.error { background: #fef2f2; color: #dc2626; border: 1px solid #fecaca; }
  .status.empty { background: #f0fdf4; color: #16a34a; border: 1px solid #bbf7d0; }
  .status.info { background: #fef9c3; color: #854d0e; border: 1px solid #fde68a; }
  .execution-card { background: white; border-radius: 12px; border: 1px solid #e5e7eb; margin-bottom: 12px; overflow: hidden; }
  .exec-header { padding: 14px 16px; border-bottom: 1px solid #f3f4f6; display: flex; align-items: center; justify-content: space-between; gap: 10px; }
  .exec-title { font-size: 14px; font-weight: 600; color: #111827; }
  .exec-time { font-size: 11px; color: #9ca3af; }
  .badge { padding: 3px 10px; border-radius: 20px; font-size: 11px; font-weight: 600; }
  .badge-lead { background: #fee2e2; color: #dc2626; }
  .badge-normal { background: #eff6ff; color: #2563eb; }
  .exec-body { padding: 14px 16px; }
  .comment-box { background: #f9fafb; border-radius: 8px; padding: 12px; margin-bottom: 12px; }
  .comment-label { font-size: 11px; color: #6b7280; font-weight: 600; margin-bottom: 4px; text-transform: uppercase; }
  .comment-text { font-size: 14px; color: #111827; line-height: 1.6; }
  .reply-box { background: #f0fdf4; border: 1px solid #bbf7d0; border-radius: 8px; padding: 12px; margin-bottom: 12px; }
  .reply-label { font-size: 11px; color: #16a34a; font-weight: 600; margin-bottom: 4px; text-transform: uppercase; }
  .reply-text { font-size: 14px; color: #15803d; line-height: 1.6; font-style: italic; }
  .meta-row { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 12px; }
  .meta-tag { padding: 3px 10px; border-radius: 20px; font-size: 11px; background: #f3f4f6; color: #6b7280; }
  .actions { display: flex; gap: 8px; }
  .btn-approve { flex: 1; padding: 10px; background: #16a34a; color: white; border: none; border-radius: 8px; font-size: 13px; font-weight: 600; cursor: pointer; }
  .btn-approve:hover { background: #15803d; }
  .btn-ignore { flex: 1; padding: 10px; background: #6b7280; color: white; border: none; border-radius: 8px; font-size: 13px; font-weight: 600; cursor: pointer; }
  .btn-ignore:hover { background: #4b5563; }
  .btn-approve:disabled, .btn-ignore:disabled { opacity: 0.5; cursor: not-allowed; }
  .refresh-btn { display: block; width: 100%; padding: 12px; background: white; border: 1px dashed #e5e7eb; border-radius: 10px; color: #6b7280; font-size: 13px; cursor: pointer; text-align: center; margin-bottom: 16px; }
  .refresh-btn:hover { background: #f9fafb; }
  .tip { font-size: 11px; color: #9ca3af; margin-top: 8px; }
</style>
</head>
<body>

<div class="header">
  <h1>Panel de Aprobación YouTube</h1>
  <p>@ElizabethAlhadad — Real Estate Agent</p>
</div>

<div class="config">
  <label>N8N API KEY (se guarda solo en tu navegador)</label>
  <div class="config-row">
    <input type="text" id="api-key-input" placeholder="Pega aquí tu n8n API key completa..." autocomplete="off" />
    <button onclick="saveAndLoad()">Cargar</button>
  </div>
  <p class="tip">La key se guarda en tu navegador. No se envía a ningún servidor externo.</p>
</div>

<div class="container">
  <button class="refresh-btn" onclick="loadExecutions()">🔄 Actualizar</button>
  <div id="status-area"></div>
  <div id="executions-area"></div>
</div>

<script>
const N8N_URL = 'https://primary-production-72d4.up.railway.app';
const PROXY = 'https://corsproxy.io/?';
let apiKey = '';

window.onload = function() {
  const saved = localStorage.getItem('ea_n8n_key');
  if (saved) {
    apiKey = saved;
    document.getElementById('api-key-input').value = saved;
    loadExecutions();
  } else {
    document.getElementById('status-area').innerHTML = '<div class="status info">👆 Pega tu n8n API key arriba y haz clic en Cargar</div>';
  }
};

function saveAndLoad() {
  const input = document.getElementById('api-key-input').value.trim();
  if (!input) {
    document.getElementById('status-area').innerHTML = '<div class="status error">Por favor pega tu API key primero</div>';
    return;
  }
  apiKey = input;
  localStorage.setItem('ea_n8n_key', apiKey);
  loadExecutions();
}

function extractData(exec) {
  try {
    const runData = exec.data?.resultData?.runData;
    if (!runData) return null;

    // Buscar en todas las posibles rutas
    const nodeNames = ['Procesar Respuesta Claude', 'Process Claude Response'];
    for (const nodeName of nodeNames) {
      const nodeData = runData[nodeName];
      if (!nodeData) continue;
      for (const attempt of nodeData) {
        const items = attempt?.data?.main?.[0];
        if (!items) continue;
        for (const item of items) {
          const json = item?.json;
          if (json && json.comment_text) return json;
        }
      }
    }

    // Buscar en cualquier nodo que tenga comment_text
    for (const nodeName of Object.keys(runData)) {
      const nodeData = runData[nodeName];
      for (const attempt of nodeData) {
        const items = attempt?.data?.main?.[0];
        if (!items) continue;
        for (const item of items) {
          if (item?.json?.comment_text) return item.json;
        }
      }
    }
  } catch(e) {}
  return null;
}

async function loadExecutions() {
  const statusArea = document.getElementById('status-area');
  const execArea = document.getElementById('executions-area');

  if (!apiKey) {
    statusArea.innerHTML = '<div class="status info">👆 Pega tu n8n API key arriba y haz clic en Cargar</div>';
    return;
  }

  statusArea.innerHTML = '<div class="status loading">⏳ Cargando comentarios pendientes...</div>';
  execArea.innerHTML = '';

  try {
    const url = `${N8N_URL}/api/v1/executions?status=waiting&limit=20&includeData=true`;
    const res = await fetch(PROXY + encodeURIComponent(url), {
      headers: { 'X-N8N-API-KEY': apiKey, 'x-requested-with': 'XMLHttpRequest' }
    });

    if (!res.ok) throw new Error(`Error ${res.status} — verifica tu API key`);

    const data = await res.json();
    const executions = data.data || [];

    if (executions.length === 0) {
      statusArea.innerHTML = '<div class="status empty">✅ No hay comentarios pendientes ahora mismo. El sistema revisa cada 30 minutos.</div>';
      return;
    }

    statusArea.innerHTML = `<div class="status loading">💬 ${executions.length} comentario(s) esperando tu aprobación</div>`;

    execArea.innerHTML = executions.map(exec => {
      const parsed = extractData(exec);
      const isLead = parsed?.es_lead || false;
      const commentText = parsed?.comment_text || 'Cargando...';
      const replyText = parsed?.respuesta_sugerida || 'Cargando...';
      const author = parsed?.author || 'Usuario de YouTube';
      const sentiment = parsed?.sentiment || '';
      const categoria = parsed?.categoria || '';
      const prioridad = parsed?.prioridad || '';
      const time = new Date(exec.startedAt).toLocaleString('es-CO');

      return `
        <div class="execution-card" id="card-${exec.id}">
          <div class="exec-header">
            <div>
              <div class="exec-title">💬 ${author}</div>
              <div class="exec-time">${time}</div>
            </div>
            <span class="badge ${isLead ? 'badge-lead' : 'badge-normal'}">${isLead ? '🔥 Lead Caliente' : 'Comentario Normal'}</span>
          </div>
          <div class="exec-body">
            <div class="meta-row">
              ${sentiment ? `<span class="meta-tag">😊 ${sentiment}</span>` : ''}
              ${categoria ? `<span class="meta-tag">📁 ${categoria}</span>` : ''}
              ${prioridad ? `<span class="meta-tag">⚡ prioridad ${prioridad}</span>` : ''}
            </div>
            <div class="comment-box">
              <div class="comment-label">Comentario recibido</div>
              <div class="comment-text">${commentText}</div>
            </div>
            <div class="reply-box">
              <div class="reply-label">✨ Respuesta sugerida por Claude</div>
              <div class="reply-text">${replyText}</div>
            </div>
            <div class="actions">
              <button class="btn-approve" onclick="respond('${exec.id}', 'aprobar', this)">✅ Aprobar y publicar</button>
              <button class="btn-ignore" onclick="respond('${exec.id}', 'ignorar', this)">⏭ Ignorar</button>
            </div>
          </div>
        </div>
      `;
    }).join('');

  } catch(e) {
    statusArea.innerHTML = `<div class="status error">❌ ${e.message}</div>`;
  }
}

async function respond(execId, action, btn) {
  const card = document.getElementById(`card-${execId}`);
  const buttons = card.querySelectorAll('button');
  buttons.forEach(b => b.disabled = true);
  btn.textContent = action === 'aprobar' ? '⏳ Publicando...' : '⏳ Procesando...';

  try {
    const webhookUrl = `${N8N_URL}/webhook-waiting/${execId}&response=${action}`;
    await fetch(PROXY + encodeURIComponent(webhookUrl));
    card.innerHTML = `<div style="padding:20px;text-align:center;color:${action === 'aprobar' ? '#16a34a' : '#6b7280'};font-weight:600;font-size:15px">
      ${action === 'aprobar' ? '✅ Respuesta publicada en YouTube' : '⏭ Comentario ignorado'}
    </div>`;
    setTimeout(() => card.remove(), 2000);
  } catch(e) {
    buttons.forEach(b => b.disabled = false);
    btn.textContent = action === 'aprobar' ? '✅ Aprobar y publicar' : '⏭ Ignorar';
  }
}

setInterval(loadExecutions, 60000);
</script>
</body>
</html>
