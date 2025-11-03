<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor - Libros en Tiempo Real</title>
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
  .other-comments { font-size:0.8em; text-align:left; margin-top:5px; max-height:100px; overflow-y:auto; }
</style>
</head>
<body>

<header><h1>Vitvisor - Libros en Tiempo Real</h1></header>
<div class="container">

  <!-- LOGIN -->
  <div class="login-container">
    <input type="email" id="email" placeholder="Email">
    <input type="password" id="password" placeholder="Contraseña">
    <button id="btnRegister">Registrarse</button>
    <button id="btnLogin">Iniciar Sesión</button>
    <p id="currentUser"></p>
  </div>

  <!-- BUSCADOR -->
  <div class="search-container">
    <input type="text" id="searchBook" placeholder="Buscar libro por título o autor">
  </div>

  <!-- LISTADO DE LIBROS -->
  <div class="book-list" id="bookList"></div>

</div>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<script>
  // ----- CONFIGURACIÓN DE FIREBASE -----
  const firebaseConfig = {
    apiKey: "TU_API_KEY",
    authDomain: "TU_PROJECT_ID.firebaseapp.com",
    projectId: "TU_PROJECT_ID",
    storageBucket: "TU_PROJECT_ID.appspot.com",
    messagingSenderId: "TU_SENDER_ID",
    appId: "TU_APP_ID"
  };
  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.firestore();

  // ----- GESTIÓN DE USUARIOS -----
  const currentUserDisplay = document.getElementById('currentUser');
  function updateUserDisplay(user) {
    currentUserDisplay.textContent = user ? `Usuario actual: ${user.email}` : 'No has iniciado sesión';
  }

  document.getElementById('btnRegister').addEventListener('click', async () => {
    const email = document.getElementById('email').value;
    const password = document.getElementById('password').value;
    try {
      await auth.createUserWithEmailAndPassword(email,password);
    } catch(e){ alert(e.message); }
  });

  document.getElementById('btnLogin').addEventListener('click', async () => {
    const email = document.getElementById('email').value;
    const password = document.getElementById('password').value;
    try {
      await auth.signInWithEmailAndPassword(email,password);
    } catch(e){ alert(e.message); }
  });

  auth.onAuthStateChanged(user => {
    updateUserDisplay(user);
  });

  // ----- BÚSQUEDA DE LIBROS EN TIEMPO REAL -----
  const searchInput = document.getElementById('searchBook');
  const bookList = document.getElementById('bookList');
  let timeout = null;

  searchInput.addEventListener('input', () => {
    clearTimeout(timeout);
    timeout = setTimeout(async () => {
      const query = searchInput.value.trim();
      if(!query) { bookList.innerHTML = ''; return; }
      const books = await fetchBooks(query);
      bookList.innerHTML = '';
      books.forEach(book => bookList.appendChild(createBookCard(book)));
    }, 400); // retraso para evitar demasiadas consultas
  });

  async function fetchBooks(query){
    const url = `https://www.googleapis.com/books/v1/volumes?q=${encodeURIComponent(query)}&maxResults=15`;
    const resp = await fetch(url);
    const data = await resp.json();
    return data.items || [];
  }

  // ----- CREAR TARJETA DE LIBRO -----
  function createBookCard(book){
    const info = book.volumeInfo;
    const id = book.id;
    const title = info.title || 'Título desconocido';
    const authors = (info.authors || ['Autor desconocido']).join(', ');
    const thumbnail = info.imageLinks?.thumbnail || 'https://via.placeholder.com/150x220.png?text=Sin+portada';
    const description = info.description ? info.description.slice(0,150)+'...' : 'Sin descripción';

    const card = document.createElement('div');
    card.className = 'book-card';
    card.innerHTML = `
      <img src="${thumbnail}" alt="Portada de ${title}">
      <h3>${title}</h3>
      <p>Autor(es): ${authors}</p>
      <p>${description}</p>
      <div class="rating">
        ${[5,4,3,2,1].map(star=>`
          <input type="radio" id="star${star}-${id}" name="rating${id}" value="${star}">
          <label for="star${star}-${id}">&#9733;</label>
        `).join('')}
      </div>
      <div class="comment-box">
        <textarea placeholder="Escribe tu comentario..."></textarea>
        <button>Guardar</button>
      </div>
      <div class="other-comments">
        <h4>Otros usuarios:</h4>
        <ul id="comments-${id}"></ul>
      </div>
    `;

    const commentList = card.querySelector(`#comments-${id}`);

    // Escuchar cambios en tiempo real
    db.collection('books').doc(id).onSnapshot(doc=>{
      commentList.innerHTML = '';
      const comments = doc.data()?.comments || [];
      comments.forEach(c=>{
        const li = document.createElement('li');
        li.textContent = `${c.user}: ${c.comment} (${c.rating}★)`;
        commentList.appendChild(li);
      });
    });

    // Guardar comentario
    card.querySelector('button').addEventListener('click', async ()=>{
      const user = auth.currentUser;
      if(!user) return alert('Inicia sesión primero.');
      const rating = parseInt(card.querySelector(`input[name="rating${id}"]:checked`)?.value || 0);
      const comment = card.querySelector('textarea').value;
      const ref = db.collection('books').doc(id);
      await ref.set({
        comments: firebase.firestore.FieldValue.arrayUnion({
          user: user.email,
          rating,
          comment,
          timestamp: Date.now()
        })
      }, {merge:true});
      card.querySelector('textarea').value = '';
    });

    return card;
  }

</script>
</body>
</html>
