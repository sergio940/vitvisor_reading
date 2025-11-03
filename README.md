<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor - Valoración de Libros Reales</title>
<style>
  body { font-family: Arial, sans-serif; background:#f2f2f2; margin:0; padding:0; }
  header { background:#2c3e50; color:#fff; padding:20px; text-align:center; }
  h1 { margin:0; font-size:2em; }
  .container { padding:20px; max-width:1000px; margin:auto; }
  .login-container, .search-container { margin-bottom:20px; text-align:center; }
  input { padding:10px; width:60%; max-width:300px; margin:5px; }
  button { padding:10px 15px; cursor:pointer; }
  .book-list { display:flex; flex-wrap:wrap; gap:20px; justify-content:center; }
  .book-card { background:#fff; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.2); width:220px; padding:15px; text-align:center; }
  .book-card img { width:100%; border-radius:5px; height:300px; object-fit:cover; }
  .book-card h3 { font-size:1.1em; margin:10px 0 5px 0; }
  .book-card p { font-size:0.85em; color:#555; }
  .rating { display:flex; justify-content:center; gap:5px; margin:10px 0; }
  .rating input[type="radio"] { display:none; }
  .rating label { font-size:1.5em; color:#ccc; cursor:pointer; }
  .rating input[type="radio"]:checked ~ label,
  .rating label:hover,
  .rating label:hover ~ label { color:#f39c12; }
  .comment-box { width:100%; margin-top:10px; }
  .comment-box textarea { width:100%; padding:5px; resize:none; border-radius:5px; border:1px solid #ccc; }
  .comment-box button { margin-top:5px; padding:8px 15px; background:#2c3e50; color:#fff; border:none; border-radius:5px; cursor:pointer; }
  .comment-box button:hover { background:#34495e; }
  .other-comments { font-size:0.8em; text-align:left; margin-top:5px; }
</style>
</head>
<body>

<header><h1>Vitvisor - Valoración de Libros Reales</h1></header>
<div class="container">

  <!-- LOGIN -->
  <div class="login-container">
    <input type="text" id="username" placeholder="Nombre de usuario">
    <button id="btnLogin">Iniciar Sesión</button>
    <p id="currentUser" style="margin-top:10px;"></p>
  </div>

  <!-- BUSCADOR -->
  <div class="search-container">
    <input type="text" id="searchBook" placeholder="Buscar libro por título o autor">
    <button id="btnSearch">Buscar</button>
  </div>

  <!-- LISTADO DE LIBROS -->
  <div class="book-list" id="bookList"></div>

</div>

<script>
// ---------- USUARIOS ----------
let currentUser = localStorage.getItem('currentUser') || null;
const currentUserDisplay = document.getElementById('currentUser');
function updateUserDisplay() {
  currentUserDisplay.textContent = currentUser ? `Usuario actual: ${currentUser}` : 'No has iniciado sesión';
}
updateUserDisplay();
document.getElementById('btnLogin').addEventListener('click', () => {
  const usernameInput = document.getElementById('username').value.trim();
  if(!usernameInput) return alert('Introduce un nombre de usuario.');
  currentUser = usernameInput;
  localStorage.setItem('currentUser', currentUser);
  updateUserDisplay();
});

// ---------- BÚSQUEDA GOOGLE BOOKS API ----------
async function fetchBooks(query){
  const url = `https://www.googleapis.com/books/v1/volumes?q=${encodeURIComponent(query)}&maxResults=15`;
  const resp = await fetch(url);
  const data = await resp.json();
  return data.items || [];
}

// ---------- CREAR TARJETA DE LIBRO ----------
function createBookCard(book){
  const info = book.volumeInfo;
  const id = book.id;
  const title = info.title || 'Título desconocido';
  const authors = (info.authors || ['Autor desconocido']).join(', ');
  const thumbnail = info.imageLinks?.thumbnail || 'https://via.placeholder.com/150x220.png?text=Sin+portada';
  const description = info.description ? info.description.slice(0, 150)+'...' : 'Sin descripción';

  const allRatings = JSON.parse(localStorage.getItem('ratings')) || {};
  const bookRatings = allRatings[id] || {};
  const userRating = bookRatings[currentUser]?.rating || 0;
  const userComment = bookRatings[currentUser]?.comment || '';

  const card = document.createElement('div');
  card.className = 'book-card';
  card.innerHTML = `
    <img src="${thumbnail}" alt="Portada de ${title}">
    <h3>${title}</h3>
    <p>Autor(es): ${authors}</p>
    <p>${description}</p>
    <div class="rating">
      ${[5,4,3,2,1].map(star => `
        <input type="radio" id="star${star}-${id}" name="rating${id}" value="${star}" ${userRating==star?'checked':''}>
        <label for="star${star}-${id}">&#9733;</label>
      `).join('')}
    </div>
    <div class="comment-box">
      <textarea placeholder="Escribe tu comentario...">${userComment}</textarea>
      <button type="button">Guardar</button>
    </div>
    <div class="other-comments">
      <h4>Otros usuarios:</h4>
      <ul id="comments-${id}"></ul>
    </div>
  `;

  // Mostrar comentarios de otros usuarios
  const commentsList = card.querySelector(`#comments-${id}`);
  for(const user in bookRatings){
    if(user!==currentUser){
      const li = document.createElement('li');
      li.textContent = `${user}: ${bookRatings[user].comment || ''} (${bookRatings[user].rating || '0'}★)`;
      commentsList.appendChild(li);
    }
  }

  // Guardar valoración
  card.querySelector('.comment-box button').addEventListener('click',()=>{
    if(!currentUser) return alert('Inicia sesión primero.');
    const rating = parseInt(card.querySelector(`input[name="rating${id}"]:checked`)?.value || 0);
    const comment = card.querySelector('textarea').value;
    if(!allRatings[id]) allRatings[id] = {};
    allRatings[id][currentUser] = {rating, comment};
    localStorage.setItem('ratings', JSON.stringify(allRatings));
    alert('Valoración guardada!');
  });

  return card;
}

// ---------- EVENTO DE BUSQUEDA ----------
document.getElementById('btnSearch').addEventListener('click', async () => {
  const query = document.getElementById('searchBook').value.trim();
  if(!query) return;
  const books = await fetchBooks(query);
  const bookList = document.getElementById('bookList');
  bookList.innerHTML = '';
  books.forEach(book => bookList.appendChild(createBookCard(book)));
});
</script>

</body>
</html>
