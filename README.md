<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Переводчик</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: system-ui, -apple-system, sans-serif;
    background: linear-gradient(135deg, #667eea, #764ba2);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }

  .translator {
    background: #fff;
    padding: 30px;
    border-radius: 16px;
    box-shadow: 0 20px 50px rgba(0,0,0,0.25);
    width: 100%;
    max-width: 820px;
  }

  h1 { text-align: center; margin-bottom: 20px; color: #333; font-size: 24px; }

  .langs {
    display: flex;
    gap: 10px;
    align-items: center;
    margin-bottom: 15px;
  }

  .langs select {
    flex: 1;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 8px;
    font-size: 15px;
    background: #f9f9f9;
    cursor: pointer;
  }

  #swap {
    padding: 10px 14px;
    border: none;
    background: #667eea;
    color: #fff;
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
    transition: 0.2s;
  }
  #swap:hover { background: #556cd6; transform: rotate(180deg); }

  .panels {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    margin-bottom: 15px;
  }

  textarea {
    width: 100%;
    min-height: 170px;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 8px;
    font-size: 15px;
    resize: vertical;
    font-family: inherit;
    background: #fafafa;
  }

  textarea:focus { outline: 2px solid #667eea; border-color: transparent; background: #fff; }

  #translateBtn {
    width: 100%;
    padding: 14px;
    background: #667eea;
    color: #fff;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: 0.2s;
  }
  #translateBtn:hover { background: #556cd6; }
  #translateBtn:disabled { opacity: 0.6; cursor: not-allowed; }

  .bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 10px;
    font-size: 14px;
    color: #888;
    min-height: 22px;
  }

  .copy-btn {
    background: none;
    border: 1px solid #ddd;
    padding: 6px 12px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 13px;
    color: #555;
    transition: 0.2s;
  }
  .copy-btn:hover { background: #f0f0f0; }

  @media (max-width: 600px) {
    .panels { grid-template-columns: 1fr; }
    .translator { padding: 20px; }
  }
</style>
</head>
<body>
<div class="translator">
  <h1>🌐 Переводчик</h1>

  <div class="langs">
    <select id="from">
      <option value="ru">Русский</option>
      <option value="en" selected>Английский</option>
      <option value="de">Немецкий</option>
      <option value="fr">Французский</option>
      <option value="es">Испанский</option>
      <option value="it">Итальянский</option>
      <option value="zh">Китайский</option>
      <option value="ja">Японский</option>
      <option value="uk">Украинский</option>
      <option value="tr">Турецкий</option>
    </select>

    <button id="swap" title="Поменять языки">⇄</button>

    <select id="to">
      <option value="ru" selected>Русский</option>
      <option value="en">Английский</option>
      <option value="de">Немецкий</option>
      <option value="fr">Французский</option>
      <option value="es">Испанский</option>
      <option value="it">Итальянский</option>
      <option value="zh">Китайский</option>
      <option value="ja">Японский</option>
      <option value="uk">Украинский</option>
      <option value="tr">Турецкий</option>
    </select>
  </div>

  <div class="panels">
    <textarea id="input" placeholder="Введите текст... (Ctrl+Enter — перевести)"></textarea>
    <textarea id="output" placeholder="Перевод..." readonly></textarea>
  </div>

  <button id="translateBtn">Перевести</button>

  <div class="bottom">
    <span id="status"></span>
    <button class="copy-btn" id="copyBtn">📋 Копировать</button>
  </div>
</div>

<script>
  const fromSel = document.getElementById('from');
  const toSel   = document.getElementById('to');
  const input   = document.getElementById('input');
  const output  = document.getElementById('output');
  const btn     = document.getElementById('translateBtn');
  const swapBtn = document.getElementById('swap');
  const status  = document.getElementById('status');
  const copyBtn = document.getElementById('copyBtn');

  async function translate(text, from, to) {
    const url = `https://api.mymemory.translated.net/get?q=${encodeURIComponent(text)}&langpair=${from}|${to}`;
    const res = await fetch(url);
    if (!res.ok) throw new Error('Ошибка сети');
    const data = await res.json();
    if (data.responseStatus !== 200) throw new Error(data.responseDetails || 'Ошибка перевода');
    return data.responseData.translatedText;
  }

  btn.addEventListener('click', async () => {
    const text = input.value.trim();
    if (!text) return;

    btn.disabled = true;
    status.textContent = 'Перевод...';
    output.value = '';

    try {
      const result = await translate(text, fromSel.value, toSel.value);
      output.value = result;
      status.textContent = '';
    } catch (e) {
      status.textContent = '❌ ' + e.message;
    } finally {
      btn.disabled = false;
    }
  });

  swapBtn.addEventListener('click', () => {
    const f = fromSel.value;
    fromSel.value = toSel.value;
    toSel.value = f;

    const t = input.value;
    input.value = output.value;
    output.value = t;
  });

  input.addEventListener('keydown', (e) => {
    if (e.ctrlKey && e.key === 'Enter') btn.click();
  });

  copyBtn.addEventListener('click', () => {
    if (!output.value) return;
    navigator.clipboard.writeText(output.value).then(() => {
      copyBtn.textContent = '✅ Скопировано';
      setTimeout(() => copyBtn.textContent = '📋 Копировать', 1500);
    });
  });
</script>
</body>
</html>
