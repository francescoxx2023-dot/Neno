<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Generador de scripts Lua para Delta</title>
<style>
  body { font-family: system-ui, sans-serif; max-width: 800px; margin: 1.5rem auto; padding: 0 1rem 2rem; background: #111; color: #eee; }
  h1 { font-size: 1.4rem; margin-bottom: .4rem; }
  h2 { font-size: 1.1rem; margin: 1.5rem 0 .5rem; }
  label { display: block; font-size: .9rem; margin: 1rem 0 .3rem; color: #bbb; }
  select, input[type=text], input[type=password], textarea, pre { width: 100%; box-sizing: border-box; padding: .7rem; border-radius: 8px; border: 1px solid #444; background: #1e1e1e; color: #ddd; font-family: monospace; font-size: .9rem; }
  textarea { height: 120px; resize: vertical; }
  pre { min-height: 220px; margin: 0; white-space: pre-wrap; word-break: break-word; }
  .fila { display: flex; gap: .5rem; align-items: center; flex-wrap: wrap; }
  .fila > input, .fila > select { flex: 1; min-width: 160px; width: auto; }
  .check { display: flex; align-items: center; gap: .5rem; font-size: .85rem; color: #bbb; margin: .5rem 0; }
  button { padding: .7rem 1.2rem; border: none; border-radius: 6px; background: #4f46e5; color: #fff; font-weight: bold; font-size: 1rem; cursor: pointer; }
  button.sec { background: #333; padding: .6rem .9rem; font-size: .9rem; }
  button:disabled { background: #333; cursor: wait; }
  .nota { font-size: .85rem; color: #f59e0b; margin-top: 1rem; line-height: 1.5; }
  .info { font-size: .8rem; color: #9ca3af; margin-top: .3rem; }
  .error { color: #f87171; }
  a { color: #93c5fd; }
</style>
</head>
<body>
  <h1>Generador de scripts Lua para Delta</h1>
  <p>Elegí una IA, poné tu API key y describí qué querés que haga el script. Te devuelve el código Lua listo para pegar en Delta.</p>

  <label for="proveedor">Proveedor de IA</label>
  <select id="proveedor"></select>
  <div class="info" id="linkKey"></div>

  <div id="cajaBase" style="display:none">
    <label for="base">URL base (compatible con OpenAI)</label>
    <input type="text" id="base" placeholder="https://api.ejemplo.com/v1" spellcheck="false">
  </div>

  <label for="apikey">API key</label>
  <div class="fila">
    <input type="password" id="apikey" placeholder="Pegá tu API key acá" autocomplete="off" spellcheck="false">
    <button type="button" class="sec" id="ver">Ver</button>
  </div>
  <div class="check"><input type="checkbox" id="recordar"> <span>Recordar las keys en este navegador (una por proveedor)</span></div>

  <label for="modelo">Modelo (escribí el que quieras o elegí de la lista)</label>
  <div class="fila">
    <input type="text" id="modelo" list="listaModelos" spellcheck="false" autocomplete="off">
    <button type="button" class="sec" id="cargar">Cargar modelos</button>
  </div>
  <datalist id="listaModelos"></datalist>
  <div class="info" id="infoModelos"></div>

  <label for="prompt">Qué tiene que hacer el script</label>
  <textarea id="prompt" placeholder="Ej: Un script que haga que el jugador corra rápido (speed) y salte alto (jump power) al tocar un bloque."></textarea>
  <div class="check"><input type="checkbox" id="modificar"> <span>Modificar el script actual en vez de crear uno nuevo</span></div>

  <div class="fila" style="margin-top:.6rem">
    <button id="btn" type="button">Generar script Lua</button>
    <button type="button" class="sec" id="copiar">Copiar</button>
    <button type="button" class="sec" id="descargar">Descargar .lua</button>
    <label class="check" style="margin:0"><input type="checkbox" id="wrap" checked> Envolver en loadstring</label>
  </div>

  <h2>Resultado:</h2>
  <pre id="output">Acá aparecerá el script...</pre>
  <div class="info" id="estado"></div>

  <div class="nota">
    ⚠️ Las keys se guardan solo en tu navegador (si tildás “Recordar”) y se envían únicamente al proveedor que elijas. No uses “Recordar” en un dispositivo compartido.<br>
    ⚠️ Cada proveedor tiene sus propios límites y costos. Algunos pueden bloquear llamadas directas desde el navegador (CORS); en ese caso probá con OpenRouter.<br>
    ⚠️ Los nombres de modelos cambian seguido: si uno da error, usá “Cargar modelos”.<br>
    ⚠️ Usá los scripts bajo tu responsabilidad. Delta es un ejecutor de terceros; su uso puede violar los Términos de Servicio de Roblox.
  </div>

<script>
  // tipo: 'openai' (compatible con OpenAI), 'gemini' o 'anthropic'
  const P = {};
  function add(id, nombre, tipo, base, link, modelos) { P[id] = { nombre, tipo, base, link, modelos: modelos || [] }; }

  add('openrouter', 'OpenRouter (cientos de modelos con una sola key)', 'openai', 'https://openrouter.ai/api/v1', 'https://openrouter.ai/keys', ['anthropic/claude-sonnet-4.5', 'openai/gpt-4o-mini', 'google/gemini-2.5-flash', 'deepseek/deepseek-chat', 'meta-llama/llama-3.3-70b-instruct']);
  add('gemini', 'Google Gemini', 'gemini', '', 'https://aistudio.google.com/apikey', ['gemini-2.5-flash', 'gemini-2.5-pro', 'gemini-2.0-flash']);
  add('openai', 'OpenAI', 'openai', 'https://api.openai.com/v1', 'https://platform.openai.com/api-keys', ['gpt-4o', 'gpt-4o-mini', 'gpt-4.1']);
  add('anthropic', 'Anthropic (Claude)', 'anthropic', '', 'https://console.anthropic.com/settings/keys', ['claude-sonnet-5-5', 'claude-opus-5-5', 'claude-haiku-5-5']);
  add('groq', 'Groq', 'openai', 'https://api.groq.com/openai/v1', 'https://console.groq.com/keys', ['llama-3.3-70b-versatile', 'llama-3.1-8b-instant']);
  add('mistral', 'Mistral', 'openai', 'https://api.mistral.ai/v1', 'https://console.mistral.ai/api-keys', ['mistral-large-latest', 'codestral-latest']);
  add('deepseek', 'DeepSeek', 'openai', 'https://api.deepseek.com/v1', 'https://platform.deepseek.com/api_keys', ['deepseek-chat', 'deepseek-reasoner']);
  add('together', 'Together AI', 'openai', 'https://api.together.xyz/v1', 'https://api.together.ai/settings/api-keys', ['meta-llama/Llama-3.3-70B-Instruct-Turbo']);
  add('fireworks', 'Fireworks AI', 'openai', 'https://api.fireworks.ai/inference/v1', 'https://fireworks.ai/account/api-keys', ['accounts/fireworks/models/llama-v3p3-70b-instruct']);
  add('perplexity', 'Perplexity', 'openai', 'https://api.perplexity.ai', 'https://www.perplexity.ai/settings/api', ['sonar']);
  add('xai', 'xAI (Grok)', 'openai', 'https://api.x.ai/v1', 'https://console.x.ai', []);
  add('cerebras', 'Cerebras', 'openai', 'https://api.cerebras.ai/v1', 'https://cloud.cerebras.ai', ['llama-3.3-70b']);
  add('sambanova', 'SambaNova', 'openai', 'https://api.sambanova.ai/v1', 'https://cloud.sambanova.ai/apis', []);
  add('nvidia', 'NVIDIA NIM', 'openai', 'https://integrate.api.nvidia.com/v1', 'https://build.nvidia.com', ['meta/llama-3.3-70b-instruct']);
  add('hyperbolic', 'Hyperbolic', 'openai', 'https://api.hyperbolic.xyz/v1', 'https://app.hyperbolic.xyz/settings', []);
  add('deepinfra', 'DeepInfra', 'openai', 'https://api.deepinfra.com/v1/openai', 'https://deepinfra.com/dash/api_keys', []);
  add('novita', 'Novita AI', 'openai', 'https://api.novita.ai/v3/openai', 'https://novita.ai/settings/key-management', []);
  add('cohere', 'Cohere', 'openai', 'https://api.cohere.ai/compatibility/v1', 'https://dashboard.cohere.com/api-keys', []);
  add('moonshot', 'Moonshot (Kimi)', 'openai', 'https://api.moonshot.ai/v1', 'https://platform.moonshot.ai', []);
  add('dashscope', 'Alibaba Qwen (DashScope)', 'openai', 'https://dashscope-intl.aliyuncs.com/compatible-mode/v1', 'https://bailian.console.alibabacloud.com', []);
  add('siliconflow', 'SiliconFlow', 'openai', 'https://api.siliconflow.com/v1', 'https://cloud.siliconflow.com/account/ak', []);
  add('nebius', 'Nebius AI Studio', 'openai', 'https://api.studio.nebius.com/v1', 'https://studio.nebius.com', []);
  add('huggingface', 'Hugging Face (router)', 'openai', 'https://router.huggingface.co/v1', 'https://huggingface.co/settings/tokens', []);
  add('github', 'GitHub Models', 'openai', 'https://models.inference.ai.azure.com', 'https://github.com/settings/tokens', []);
  add('ollama', 'Ollama (local, en tu PC)', 'openai', 'http://localhost:11434/v1', '', ['llama3.2']);
  add('lmstudio', 'LM Studio (local, en tu PC)', 'openai', 'http://localhost:1234/v1', '', []);
  add('custom', 'Personalizado (cualquier URL compatible con OpenAI)', 'openai', '', '', []);

  const $ = id => document.getElementById(id);
  const selProv = $('proveedor'), apiInput = $('apikey'), baseInput = $('base'), modeloInput = $('modelo'),
        recordar = $('recordar'), btn = $('btn'), output = $('output'), wrap = $('wrap');
  let codigo = '';

  // ---------- Almacenamiento (con try/catch) ----------
  function leer(k, def) { try { const v = localStorage.getItem(k); return v === null ? def : JSON.parse(v); } catch (e) { return def; } }
  function guardar(k, v) { try { localStorage.setItem(k, JSON.stringify(v)); } catch (e) {} }
  function borrar(k) { try { localStorage.removeItem(k); } catch (e) {} }

  let keys = leer('lua_keys', {});
  let modelosGuardados = leer('lua_modelos', {});
  let bases = leer('lua_bases', {});
  recordar.checked = Object.keys(keys).length > 0;

  for (const [id, p] of Object.entries(P)) {
    const o = document.createElement('option'); o.value = id; o.textContent = p.nombre; selProv.appendChild(o);
  }
  const provGuardado = leer('lua_prov', 'openrouter');
  selProv.value = P[provGuardado] ? provGuardado : 'openrouter';

  function actualizarProveedor() {
    const id = selProv.value, p = P[id];
    apiInput.value = keys[id] || '';
    modeloInput.value = modelosGuardados[id] || p.modelos[0] || '';
    $('cajaBase').style.display = id === 'custom' ? 'block' : 'none';
    baseInput.value = bases[id] || '';
    const esLocal = id === 'ollama' || id === 'lmstudio';
    $('linkKey').innerHTML = p.link ? 'Conseguí tu key en <a href="' + p.link + '" target="_blank" rel="noopener">' + p.link.replace('https://', '') + '</a>' :
      (esLocal ? 'Modelo local: no hace falta key. Tiene que estar corriendo en tu PC.' : 'Poné la URL base y la key (si el servicio la pide).');
    const dl = $('listaModelos'); dl.innerHTML = '';
    p.modelos.forEach(m => { const o = document.createElement('option'); o.value = m; dl.appendChild(o); });
    $('infoModelos').classList.remove('error');
    $('infoModelos').textContent = 'Tocá “Cargar modelos” para traer la lista completa del servicio.';
  }
  selProv.addEventListener('change', actualizarProveedor);
  actualizarProveedor();

  $('ver').addEventListener('click', () => {
    const oculto = apiInput.type === 'password';
    apiInput.type = oculto ? 'text' : 'password';
    $('ver').textContent = oculto ? 'Ocultar' : 'Ver';
  });

  function baseDe(id) { return (id === 'custom' ? baseInput.value.trim() : P[id].base).replace(/\/+$/, ''); }
  function sinKeyOk(id) { return id === 'custom' || id === 'ollama' || id === 'lmstudio'; }

  // ---------- Cargar modelos ----------
  $('cargar').addEventListener('click', async () => {
    const id = selProv.value, p = P[id], key = apiInput.value.trim();
    const info = $('infoModelos'); info.textContent = 'Cargando modelos...'; info.classList.remove('error');
    try {
      let ids = [];
      if (p.tipo === 'gemini') {
        if (!key) throw new Error('Poné la API key primero.');
        const r = await fetch('https://generativelanguage.googleapis.com/v1beta/models?pageSize=200', { headers: { 'x-goog-api-key': key } });
        const d = await r.json(); if (!r.ok) throw new Error(d.error?.message || 'Error al listar modelos');
        ids = (d.models || []).filter(m => (m.supportedGenerationMethods || []).includes('generateContent')).map(m => m.name.replace('models/', ''));
      } else if (p.tipo === 'anthropic') {
        if (!key) throw new Error('Poné la API key primero.');
        const r = await fetch('https://api.anthropic.com/v1/models?limit=100', { headers: { 'x-api-key': key, 'anthropic-version': '2023-06-01', 'anthropic-dangerous-direct-browser-access': 'true' } });
        const d = await r.json(); if (!r.ok) throw new Error(d.error?.message || 'Error al listar modelos');
        ids = (d.data || []).map(m => m.id);
      } else {
        const base = baseDe(id); if (!base) throw new Error('Poné la URL base primero.');
        const r = await fetch(base + '/models', { headers: key ? { Authorization: 'Bearer ' + key } : {} });
        const d = await r.json(); if (!r.ok) throw new Error(d.error?.message || 'Error al listar modelos');
        ids = (Array.isArray(d) ? d : (d.data || d.models || [])).map(m => m.id || m.name).filter(Boolean);
      }
      ids.sort();
      const dl = $('listaModelos'); dl.innerHTML = '';
      ids.forEach(m => { const o = document.createElement('option'); o.value = m; dl.appendChild(o); });
      info.textContent = ids.length + ' modelos cargados. Escribí en el campo para filtrarlos.';
    } catch (err) {
      info.textContent = '❌ ' + (err instanceof TypeError ? 'No se pudo conectar (¿CORS, URL incorrecta o sin internet?).' : err.message);
      info.classList.add('error');
    }
  });

  // ---------- Llamadas a cada tipo de API ----------
  async function pedir(id, key, modelo, prompt) {
    const p = P[id];
    let r, d;
    if (p.tipo === 'gemini') {
      r = await fetch('https://generativelanguage.googleapis.com/v1beta/models/' + encodeURIComponent(modelo) + ':generateContent', {
        method: 'POST', headers: { 'Content-Type': 'application/json', 'x-goog-api-key': key },
        body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }], generationConfig: { temperature: 0.2, maxOutputTokens: 8192 } })
      });
      d = await r.json(); if (!r.ok) throw new Error(d.error?.message || 'Error en la API');
      return (d.candidates?.[0]?.content?.parts || []).map(x => x.text || '').join('');
    }
    if (p.tipo === 'anthropic') {
      r = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'x-api-key': key, 'anthropic-version': '2023-06-01', 'anthropic-dangerous-direct-browser-access': 'true' },
        body: JSON.stringify({ model: modelo, max_tokens: 8192, messages: [{ role: 'user', content: prompt }] })
      });
      d = await r.json(); if (!r.ok) throw new Error(d.error?.message || 'Error en la API');
      return (d.content || []).map(x => x.text || '').join('');
    }
    const base = baseDe(id); if (!base) throw new Error('Poné la URL base.');
    const headers = { 'Content-Type': 'application/json' };
    if (key) headers.Authorization = 'Bearer ' + key;
    r = await fetch(base + '/chat/completions', { method: 'POST', headers, body: JSON.stringify({ model: modelo, messages: [{ role: 'user', content: prompt }] }) });
    d = await r.json(); if (!r.ok) throw new Error(d.error?.message || (typeof d.error === 'string' ? d.error : 'Error en la API'));
    return d.choices?.[0]?.message?.content || '';
  }

  function extraer(t) {
    const m = t.match(/```(?:lua|luau)?\s*\n([\s\S]*?)```/i);
    return (m ? m[1] : t.replace(/```(?:lua|luau)?/gi, '')).trim();
  }
  function textoFinal() {
    return wrap.checked && codigo ? 'loadstring([==[\n' + codigo + '\n]==])()' : codigo;
  }
  function mostrar() { output.textContent = textoFinal() || 'Acá aparecerá el script...'; }
  wrap.addEventListener('change', () => { if (codigo) mostrar(); });

  function aviso(boton, texto) { const o = boton.textContent; boton.textContent = texto; setTimeout(() => (boton.textContent = o), 1500); }
  $('copiar').addEventListener('click', async () => {
    if (!codigo) return;
    try { await navigator.clipboard.writeText(textoFinal()); aviso($('copiar'), 'Copiado'); } catch (e) { aviso($('copiar'), 'Copiá a mano'); }
  });
  $('descargar').addEventListener('click', () => {
    if (!codigo) return;
    const a = document.createElement('a');
    a.href = URL.createObjectURL(new Blob([textoFinal()], { type: 'text/plain' }));
    a.download = 'script.lua';
    a.click();
  });

  // ---------- Generar ----------
  btn.addEventListener('click', async () => {
    const id = selProv.value;
    const key = apiInput.value.trim(), modelo = modeloInput.value.trim(), desc = $('prompt').value.trim();
    if (!key && !sinKeyOk(id)) return alert('Pegá tu API key primero.');
    if (!modelo) return alert('Elegí o escribí un modelo.');
    if (!desc) return alert('Describí el script primero.');

    keys[id] = key; modelosGuardados[id] = modelo; if (id === 'custom') bases[id] = baseInput.value.trim();
    guardar('lua_prov', id); guardar('lua_modelos', modelosGuardados); guardar('lua_bases', bases);
    if (recordar.checked) guardar('lua_keys', keys); else borrar('lua_keys');

    const reglas = 'Sos un experto en scripting de Roblox (Luau) para ejecutores como Delta. Generá únicamente el código Lua del script, listo para pegar y ejecutar. ' +
      'No agregues explicaciones fuera del código; podés usar comentarios dentro del script. Usá game:GetService para los servicios de Roblox y no dependas de librerías externas.';
    const prompt = ($('modificar').checked && codigo)
      ? reglas + '\n\nScript actual:\n' + codigo + '\n\nCambios pedidos: ' + desc + '\n\nDevolvé el script completo actualizado.'
      : reglas + '\n\nDescripción del script: ' + desc;

    btn.disabled = true; $('estado').classList.remove('error'); $('estado').textContent = 'Generando...'; output.textContent = 'Generando...';
    try {
      const texto = await pedir(id, key, modelo, prompt);
      if (!texto) throw new Error('La IA no devolvió texto.');
      codigo = extraer(texto);
      mostrar();
      $('estado').textContent = 'Listo.';
    } catch (err) {
      output.textContent = 'Acá aparecerá el script...';
      $('estado').textContent = '❌ ' + (err instanceof TypeError ? 'No se pudo conectar (¿CORS, URL incorrecta o sin internet?). Probá con OpenRouter.' : err.message);
      $('estado').classList.add('error');
    } finally { btn.disabled = false; }
  });
</script>
</body>
</html>
