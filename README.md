<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>App de Notas</title>
  <style>
    :root {
      --primary: #4a90e2;
      --bg: #f5f7fa;
      --card: #ffffff;
      --text: #333333;
      --border: #e0e0e0;
      --danger: #e74c3c;
    }
    @media (prefers-color-scheme: dark) {
      :root {
        --bg: #1a1a1a;
        --card: #2d2d2d;
        --text: #e0e0e0;
        --border: #444;
      }
    }
    * { margin:0; padding:0; box-sizing:border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      background: var(--bg);
      color: var(--text);
      padding: 1rem;
      min-height: 100vh;
    }
    header {
      text-align: center;
      margin-bottom: 1.5rem;
    }
    h1 {
      font-size: 1.8rem;
      color: var(--primary);
    }
    #new-note {
      width: 100%;
      padding: 0.8rem;
      border: 1px solid var(--border);
      border-radius: 8px;
      background: var(--card);
      color: var(--text);
      font-size: 1rem;
      margin-bottom: 1rem;
      resize: none;
      height: 100px;
    }
    #add-btn {
      background: var(--primary);
      color: white;
      border: none;
      padding: 0.8rem 1.2rem;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
      width: 100%;
      font-size: 1rem;
      transition: 0.2s;
    }
    #add-btn:hover { opacity: 0.9; }
    #notes-container {
      display: grid;
      gap: 1rem;
      margin-top: 1.5rem;
    }
    .note {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 1rem;
      position: relative;
      box-shadow: 0 2px 4px rgba(0,0,0,0.05);
    }
    .note-text {
      white-space: pre-wrap;
      word-break: break-word;
      margin-bottom: 0.5rem;
    }
    .note-date {
      font-size: 0.75rem;
      color: #888;
      margin-bottom: 0.5rem;
    }
    .delete-btn {
      background: var(--danger);
      color: white;
      border: none;
      padding: 0.4rem 0.8rem;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.8rem;
      position: absolute;
      top: 0.5rem;
      right: 0.5rem;
    }
    .empty {
      text-align: center;
      color: #888;
      font-style: italic;
      margin-top: 2rem;
    }
    @media (min-width: 600px) {
      #notes-container { grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); }
    }
  </style>
</head>
<body>
  <header>
    <h1>📝 Mis Notas</h1>
  </header>

  <textarea id="new-note" placeholder="Escribe tu nota aquí..."></textarea>
  <button id="add-btn">➕ Agregar Nota</button>

  <div id="notes-container">
    <p class="empty">Aún no hay notas. ¡Crea una!</p>
  </div>

  <script>
    const notesContainer = document.getElementById('notes-container');
    const newNoteInput = document.getElementById('new-note');
    const addBtn = document.getElementById('add-btn');

    // Cargar notas al iniciar
    loadNotes();

    // Agregar nota
    addBtn.addEventListener('click', addNote);
    newNoteInput.addEventListener('keypress', e => {
      if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        addNote();
      }
    });

    function addNote() {
      const text = newNoteInput.value.trim();
      if (!text) return;

      const note = {
        id: Date.now().toString(),
        text,
        date: new Date().toLocaleString('es-MX')
      };

      saveNote(note);
      renderNote(note);
      newNoteInput.value = '';
      checkEmptyState();
    }

    function renderNote(note) {
      const div = document.createElement('div');
      div.className = 'note';
      div.dataset.id = note.id;

      div.innerHTML = `
        <div class="note-text">${escapeHtml(note.text)}</div>
        <div class="note-date">${note.date}</div>
        <button class="delete-btn">🗑️ Eliminar</button>
      `;

      div.querySelector('.delete-btn').addEventListener('click', () => {
        deleteNote(note.id);
        div.remove();
        checkEmptyState();
      });

      notesContainer.insertBefore(div, notesContainer.firstChild);
    }

    function saveNote(note) {
      const notes = getNotes();
      notes.unshift(note);
      localStorage.setItem('notes', JSON.stringify(notes));
    }

    function deleteNote(id) {
      let notes = getNotes();
      notes = notes.filter(n => n.id !== id);
      localStorage.setItem('notes', JSON.stringify(notes));
    }

    function getNotes() {
      return JSON.parse(localStorage.getItem('notes') || '[]');
    }

    function loadNotes() {
      const notes = getNotes();
      if (notes.length === 0) {
        checkEmptyState();
        return;
      }
      notes.forEach(renderNote);
      checkEmptyState();
    }

    function checkEmptyState() {
      const hasNotes = notesContainer.children.length > 0 && 
                       !notesContainer.querySelector('.empty');
      if (!hasNotes && getNotes().length === 0) {
        if (!notesContainer.querySelector('.empty')) {
          const p = document.createElement('p');
          p.className = 'empty';
          p.textContent = 'Aún no hay notas. ¡Crea una!';
          notesContainer.appendChild(p);
        }
      } else if (hasNotes && notesContainer.querySelector('.empty')) {
        notesContainer.querySelector('.empty').remove();
      }
    }

    function escapeHtml(text) {
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }
  </script>
</body>
</html>
