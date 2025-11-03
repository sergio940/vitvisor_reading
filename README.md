<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device‑width, initial‑scale=1.0">
    <title>Vitvisor ‑ Valoración de Libros</title>
    <style>
      body { font-family: Arial, sans‑serif; background: #f2f2f2; margin:0; padding:0; }
      header { background: #2c3e50; color:#fff; padding:20px; text-align:center; }
      h1 { margin:0; font-size:2em; }
      .search-container { margin:20px auto; text-align:center; }
      .search-container input[type="text"] {
        padding:10px; width:60%; max-width:400px; border:1px solid #ccc; border-radius:5px;
      }
      .book-list { display:flex; flex-wrap:wrap; justify-content:center; gap:20px; padding:20px; }
      .book-card {
        background:#fff; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.2);
        width:200px; padding:15px; text-align:center;
      }
      .book-card img { width:100%; border-radius:5px; height: auto; }
      .book-card h3 { font-size:1.2em; margin:10px 0 5px 0; }
      .book-card p { font-size:0.9em; color:#555; }
      .rating { display:flex; justify-content:center; gap:5px; margin:10px 0; }
      .rating input[type="radio"] { display:none; }
      .rating label {
        font-size:1.5em; color:#ccc; cursor:pointer;
      }
      .rating input[type="radio"]:checked ~ label,
      .rating label:hover,
      .rating label:hover ~ label { color:#f39c12; }
      .comment-box { width:100%; margin-top:10px; }
      .comment-box textarea {
        width:100%; padding:5px; resize:none; border-radius:5px; border:1px solid #ccc;
      }
      .comment-box button {
        margin-top:5px; padding:8px 15px; background:#2c3e50; color:#fff; border:none; border-radius:5px; cursor:pointer;
      }
      .comment-box button:hover { background:#34495e; }
    </style>
</head>
<body>

<header><h1>Vitvisor ‑ Valoración de Libros</h1></header>

<div class="search-container">
  <input type="text" id="searchBook" placeholder="Buscar libro por título o autor...">
  <button id="btnSearch">Buscar</button>
</div>

<div class="book-list" id="bookList">
  <!-- Resultados aparecerán aquí -->
</div>

<script>
// Función para obtener libros de Google Books API
async function fetchBooks(query) {
  const url = `https://www.googleapis.com/books/v1/volumes?q=${encodeURIComponent(query)}&maxResults=10`;
  const resp = await fetch(url);
  const data = await resp.json();
  return data.items || [];
}

// Función para crear tarjeta de libro
function createBookCard(book, index) {
  const info = book.volumeInfo;
  const title = info.title || 'Título desconocido';
  const authors = (info.authors || ['Autor desconocido']).join(', ');
  const thumbnail = info.imageLinks?.thumbnail || 'https://via.placeholder.com/150x220.png?text=Sin+portada';

  const card = document.createElement('div');
  card.className = 'book-card';

  card.innerHTML = `
    <img src="${thumbnail}" alt="Portada de ${title}">
    <h3>${title}</h3>
    <p>Autor(es): ${authors}</p>
    <div class="rating">
      <input type="radio" id="star5-${index}" name="rating${index}" value="5"><label for="star5-${index}">&#9733;</label>
      <input type="radio" id="star4-${index}" name="rating${index}" value="4"><label for="star4-${index}">&#9733;</label>
      <input type="radio" id="star3-${index}" name="rating${index}" value="3"><label for="star3-${index}">&#9733;</label>
      <input type="radio" id="star2-${index}" name="rating${index}" value="2"><label for="star2-${index}">&#9733;</label>
      <input type="radio" id="star1-${index}" name="rating${index}" value="1"><label for="star1-${index}">&#9733;</label>
    </div>
    <div class="comment-box">
      <textarea placeholder="Escribe tu comentario..."></textarea>
      <button type="button">Enviar</button>
    </div>
  `;
  return card;
}

// Gestión del botón de búsqueda
const btnSearch = document.getElementById('btnSearch');
const inputSearch = document.getElementById('searchBook');
const bookList = document.getElementById('bookList');

btnSearch.addEventListener('click', async () => {
  const query = inputSearch.value.trim();
  if (!query) return;
  bookList.innerHTML = '';  // limpiar resultados anteriores
  const books = await fetchBooks(query);
  books.forEach((book, idx) => {
    const card = createBookCard(book, idx);
    bookList.appendChild(card);
  });
});
</script>

</body>
</html>
