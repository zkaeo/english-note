# english-note
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Bilingual Editor Tool</title>
  <style>
    body { font-family: sans-serif; margin: 0; display: flex; height: 100vh; overflow: hidden; }
    #sidebar { width: 250px; background: #f4f4f4; border-right: 1px solid #ccc; overflow-y: auto; padding: 10px; }
    #main { flex: 1; display: flex; flex-direction: column; }
    .title-group { margin-bottom: 10px; }
    .title-group > div { cursor: pointer; padding: 5px; margin: 2px 0; border-radius: 5px; }
    .title-group > div:hover { background-color: #e0e0e0; }
    .active { background-color: #ddd; font-weight: bold; }
    #content-area { flex: 1; overflow-y: auto; padding: 10px; }
    .segment { display: flex; margin-bottom: 15px; border: 1px solid #ccc; padding: 5px; border-radius: 5px; }
    .en, .zh, .note { flex: 1; margin: 0 5px; }
    .zh { transition: max-width 0.3s ease-in-out; }
    .folded .zh { display: none; }
    .folded .en, .folded .note { flex: 1.5; }
    textarea { width: 100%; height: 80px; }
    button { margin: 5px; }
    .toolbar { border-top: 1px solid #ccc; padding: 5px; display: flex; justify-content: space-between; align-items: center; }
  </style>
</head>
<body>
  <div id="sidebar">
    <div><strong>文稿列表</strong></div>
    <div id="doc-list"></div>
    <button onclick="addDocument()">➕ 添加文稿</button>
    <hr>
    <button onclick="startReviewGame()">🎮 单词复习</button>
  </div>
  <div id="main">
    <div class="toolbar">
      <span id="doc-title" contenteditable="true" onblur="renameDocument()">默认文稿</span>
      <div>
        <button onclick="addSegment()">➕ 添加段落</button>
        <button onclick="toggleZh()">🔁 折叠中文</button>
        <button onclick="exportContent()">⬇️ 导出</button>
      </div>
    </div>
    <div id="content-area"></div>
  </div>

<script>
let currentDocId = 'doc-0';
let showZh = true;

function load() {
  const saved = JSON.parse(localStorage.getItem('docs') || '{}');
  if (Object.keys(saved).length === 0) {
    saved[currentDocId] = { title: '默认文稿', segments: [] };
    localStorage.setItem('docs', JSON.stringify(saved));
  }
  Object.entries(saved).forEach(([id, { title }]) => {
    const div = document.createElement('div');
    div.innerText = title;
    div.onclick = () => switchDoc(id);
    div.id = `title-${id}`;
    div.className = id === currentDocId ? 'active' : '';
    document.getElementById('doc-list').appendChild(div);
  });
  renderContent();
}

function renderContent() {
  const data = JSON.parse(localStorage.getItem('docs'))[currentDocId];
  document.getElementById('doc-title').innerText = data.title;
  const container = document.getElementById('content-area');
  container.innerHTML = '';
  data.segments.forEach((seg, idx) => {
    const div = document.createElement('div');
    div.className = 'segment' + (showZh ? '' : ' folded');
    div.innerHTML = `
      <textarea class="en" oninput="updateSegment(${idx}, 'en', this.value)">${seg.en}</textarea>
      <textarea class="zh" oninput="updateSegment(${idx}, 'zh', this.value)">${seg.zh}</textarea>
      <textarea class="note" oninput="updateSegment(${idx}, 'note', this.value)">${seg.note}</textarea>
    `;
    container.appendChild(div);
  });
}
