<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vitvisor - Valoración de Libros</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f2f2f2;
            margin: 0;
            padding: 0;
        }

        header {
            background-color: #2c3e50;
            color: #fff;
            padding: 20px;
            text-align: center;
        }

        h1 {
            margin: 0;
            font-size: 2em;
        }

        .search-container {
            margin: 20px auto;
            text-align: center;
        }

        .search-container input[type="text"] {
            padding: 10px;
            width: 60%;
            max-width: 400px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }

        .book-list {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 20px;
            padding: 20px;
        }

        .book-card {
            background-color: #fff;
            border-radius: 10px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
            width: 200px;
            padding: 15px;
            text-align: center;
        }

        .book-card img {
            width: 100%;
            border-radius: 5px;
        }

        .book-card h3 {
            font-size: 1.2em;
            margin: 10px 0 5px 0;
        }

        .book-card p {
            font-size: 0.9em;
            color: #555;
        }

        .rating {
            display: flex;
            justify-content: center;
            gap: 5px;
            margin: 10px 0;
        }

        .rating input[type="radio"] {
            display: none;
        }

        .rating label {
            font-size: 1.5em;
            color: #ccc;
            cursor: pointer;
        }

        .rating input[type="radio"]:checked ~ label,
        .rating label:hover,
        .rating label:hover ~ label {
            color: #f39c12;
        }

        .comment-box {
            width: 100%;
            margin-top: 10px;
        }

        .comment-box textarea {
            width: 100%;
            padding: 5px;
            resize: none;
            border-radius: 5px;
            border: 1px solid #ccc;
        }

        .comment-box button {
            margin-top: 5px;
            padding: 8px 15px;
            background-color: #2c3e50;
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        .comment-box button:hover {
            background-color: #34495e;
        }
    </style>
</head>
<body>

<header>
    <h1>Vitvisor - Valoración de Libros</h1>
</header>

<div class="search-container">
    <input type="text" id="searchBook" placeholder="Buscar libro por título...">
</div>

<div class="book-list">
    <!-- Libro ejemplo -->
    <div class="book-card">
        <img src="https://via.placeholder.com/150x220.png?text=Libro+1" alt="Libro 1">
        <h3>Libro 1</h3>
        <p>Autor: Juan Pérez</p>
        <div class="rating">
            <input type="radio" id="star5-1" name="rating1" value="5"><label for="star5-1">&#9733;</label>
            <input type="radio" id="star4-1" name="rating1" value="4"><label for="star4-1">&#9733;</label>
            <input type="radio" id="star3-1" name="rating1" value="3"><label for="star3-1">&#9733;</label>
            <input type="radio" id="star2-1" name="rating1" value="2"><label for="star2-1">&#9733;</label>
            <input type="radio" id="star1-1" name="rating1" value="1"><label for="star1-1">&#9733;</label>
        </div>
        <div class="comment-box">
            <textarea placeholder="Escribe tu comentario..."></textarea>
            <button>Enviar</button>
        </div>
    </div>

    <!-- Otro libro ejemplo -->
    <div class="book-card">
        <img src="https://via.placeholder.com/150x220.png?text=Libro+2" alt="Libro 2">
        <h3>Libro 2</h3>
        <p>Autor: Ana Gómez</p>
        <div class="rating">
            <input type="radio" id="star5-2" name="rating2" value="5"><label for="star5-2">&#9733;</label>
            <input type="radio" id="star4-2" name="rating2" value="4"><label for="star4-2">&#9733;</label>
            <input type="radio" id="star3-2" name="rating2" value="3"><label for="star3-2">&#9733;</label>
            <input type="radio" id="star2-2" name="rating2" value="2"><label for="star2-2">&#9733;</label>
            <input type="radio" id="star1-2" name="rating2" value="1"><label for="star1-2">&#9733;</label>
        </div>
        <div class="comment-box">
            <textarea placeholder="Escribe tu comentario..."></textarea>
            <button>Enviar</button>
        </div>
    </div>

    <!-- Puedes duplicar book-card para más libros -->
</div>

<script>
    // Filtrado básico por título
    const searchInput = document.getElementById('searchBook');
    const books = document.querySelectorAll('.book-card');

    searchInput.addEventListener('input', function() {
        const filter = this.value.toLowerCase();
        books.forEach(book => {
            const title = book.querySelector('h3').textContent.toLowerCase();
            book.style.display = title.includes(filter) ? 'block' : 'none';
        });
    });
</script>

</body>
</html>
