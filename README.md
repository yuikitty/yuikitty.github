<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pelota y Barra</title>
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Pacifico&display=swap');

        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #ffb6c1;
            flex-direction: column;
            position: relative;
        }

        .pantalla-inicio {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #ffb6c1;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 1;
            color: white;
            transform: translateY(-20px);
        }

        .pantalla-pausa {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(255, 182, 193, 0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 2;
            display: none;
        }

        button {
            font-family: 'Pacifico', cursive;
            font-size: 24px;
            padding: 10px 20px;
            border: none;
            border-radius: 10px;
            background-color: #ff4186;
            color: white;
            cursor: pointer;
            transition: background-color 0.3s;
            margin-bottom: 20px;
        }

        button:hover {
            background-color: #f41d85;
        }

        .imagenes-gatos {
            display: flex;
            gap: 30px;
        }

        img {
            width: 150px;
            height: auto;
        }

        canvas {
            background-color: #ffb6c1;
            border: none;
            border-radius: 10px;
            position: relative;
        }

        .pantalla-pausa h1 {
            font-family: 'Pacifico', cursive;
            font-size: 44px;
            color: white;
        }

        #ranking {
            position: absolute;
            left: 20px;
            top: 20px;
            color: white;
            font-family: 'Pacifico', cursive;
            font-size: 24px;
        }

        #limpiarRecords {
            position: absolute;
            top: 20px;
            right: 20px;
            color: white;
            display: none;
        }

        #adminPassword {
            display: none;
        }

        .contraseña {
            position: absolute;
            top: 20px;
            right: 20px;
            color: white;
            display: none;
        }

    </style>
</head>
<body>
    <div class="pantalla-inicio">
        <h1 style="font-family: 'Pacifico', cursive; font-size: 48px;">que</h1>
        <input type="text" id="nombreJugador" placeholder="Ingresa tu nombre" style="font-size: 24px; padding: 10px; border-radius: 10px;">
        <button id="iniciarJuego">Jugar ></button>
        <div class="imagenes-gatos">
            <img src="gato1.png" alt="Gato 1">
            <img src="gato2.png" alt="Gato 2">
            <img src="gato3.png" alt="Gato 3">
        </div>
    </div>
    
    <div class="pantalla-pausa">
        <h1>Pausado owo</h1>
    </div>
    
    <canvas id="juegoCanvas" width="800" height="600" style="display: none;"></canvas>
    <div id="ranking">Ranking:<br></div>
    <input type="password" id="adminPassword" placeholder="Contraseña admin" style="display:none;">
    <button id="limpiarRecords" style="display:none;">Limpiar Récords</button>

    <script>
        const canvas = document.getElementById("juegoCanvas");
        const ctx = canvas.getContext("2d");

        // Cargar la imagen de la pelota
        const imagenPelota = new Image();
        imagenPelota.src = "pelota.png";

        // Configuración de la puntuación
        let puntuacion = 0;
        let jugador = '';
        let juegoActivo = false; // Estado del juego
        let ranking = JSON.parse(localStorage.getItem("ranking")) || []; // Cargar el ranking desde localStorage

        // Configuración de la pelota
        const pelota = {
            x: canvas.width / 6,
            y: canvas.height / 8,
            dx: 4,
            dy: 4,
            radio: 15, // Para detectar colisiones
        };

        // Configuración de la barra
        const barra = {
            ancho: 100,
            alto: 20,
            x: canvas.width / 2 - 50,
            y: canvas.height - 30,
        };

        // Dibujar la puntuación
        function dibujarPuntuacion() {
            ctx.font = "24px Pacifico";
            ctx.fillStyle = "#fff";
            ctx.fillText("Puntuación: " + puntuacion, 20, 40);
        }

        // Dibujar la pelota como imagen
        function dibujarPelota() {
            ctx.drawImage(imagenPelota, pelota.x - pelota.radio, pelota.y - pelota.radio, pelota.radio * 3, pelota.radio * 3);
        }

        // Dibujar la barra en gris
        function dibujarBarra() {
            ctx.beginPath();
            ctx.rect(barra.x, barra.y, barra.ancho, barra.alto);
            ctx.fillStyle = "#808080"; // Color gris
            ctx.fill();
            ctx.closePath();
        }

        // Dibujar el borde del canvas con esquinas redondeadas
        function dibujarBorde() {
            ctx.beginPath();
            const bordeRadio = 10; // Radio de los bordes redondeados
            ctx.moveTo(bordeRadio, 0); // Esquina superior izquierda
            ctx.lineTo(canvas.width - bordeRadio, 0); // Esquina superior derecha
            ctx.arcTo(canvas.width, 0, canvas.width, bordeRadio, bordeRadio); // Esquina superior derecha
            ctx.lineTo(canvas.width, canvas.height - bordeRadio); // Esquina inferior derecha
            ctx.arcTo(canvas.width, canvas.height, canvas.width - bordeRadio, canvas.height, bordeRadio); // Esquina inferior derecha
            ctx.lineTo(bordeRadio, canvas.height); // Esquina inferior izquierda
            ctx.arcTo(0, canvas.height, 0, canvas.height - bordeRadio, bordeRadio); // Esquina inferior izquierda
            ctx.lineTo(0, bordeRadio); // Esquina superior izquierda
            ctx.arcTo(0, 0, bordeRadio, 0, bordeRadio); // Esquina superior izquierda
            ctx.closePath();
            ctx.lineWidth = 6; // Ancho del borde
            ctx.strokeStyle = "#fff"; // Color del borde
            ctx.stroke(); // Dibuja el borde
        }

        // Detectar colisiones
        function detectarColisiones() {
            if (pelota.x + pelota.radio > canvas.width || pelota.x - pelota.radio < 0) {
                pelota.dx = -pelota.dx;
            }

            if (pelota.y - pelota.radio < 0) {
                pelota.dy = -pelota.dy;
            }

            if (
                pelota.y + pelota.radio > barra.y &&
                pelota.x > barra.x &&
                pelota.x < barra.x + barra.ancho
            ) {
                pelota.dy = -pelota.dy;
                puntuacion++; // Incrementar la puntuación en 1 cuando la pelota rebota en la barra
                aumentarVelocidad(); // Aumentar velocidad si se alcanza un múltiplo de 5
            }

            if (pelota.y + pelota.radio > canvas.height) {
                pelota.x = canvas.width / 2;
                pelota.y = canvas.height / 4;
                pelota.dy = 4;
                actualizarRanking(jugador, puntuacion); // Actualizar el ranking
                puntuacion = 0; // Reiniciar la puntuación cuando la pelota cae
            }
        }

        // Aumentar la velocidad de la pelota
        function aumentarVelocidad() {
            if (puntuacion % 5 === 0) {
                pelota.dx *= 1.1; // Aumentar la velocidad horizontal
                pelota.dy *= 1.1; // Aumentar la velocidad vertical
            }
        }

        // Actualizar el ranking
        function actualizarRanking(nombre, score) {
            const index = ranking.findIndex(item => item.nombre === nombre); // Buscar si el jugador ya tiene un récord

            if (index === -1) {
                // Si el jugador no tiene un récord, agregarlo
                ranking.push({ nombre, score });
            } else {
                // Si el jugador ya tiene un récord, comprobar si es necesario actualizar
                if (ranking[index].score < score) {
                    ranking[index].score = score; // Reemplazar el puntaje
                }
            }

            ranking.sort((a, b) => b.score - a.score); // Ordenar el ranking de mayor a menor
            if (ranking.length > 5) ranking.pop(); // Limitar a 5 puestos
            localStorage.setItem("ranking", JSON.stringify(ranking)); // Guardar el ranking en localStorage
            mostrarRanking(); // Actualizar la visualización del ranking
        }

        // Mostrar el ranking en el DOM
        function mostrarRanking() {
            const rankingDiv = document.getElementById("ranking");
            rankingDiv.innerHTML = "Ranking:<br>";
            ranking.forEach(record => {
                rankingDiv.innerHTML += `${record.nombre} - ${record.score}<br>`;
            });
        }

        // Mover la barra con el mouse
        canvas.addEventListener("mousemove", function(event) {
            const rect = canvas.getBoundingClientRect();
            const mouseX = event.clientX - rect.left;
            barra.x = mouseX - barra.ancho / 2; // Centrar la barra en el mouse
            if (barra.x < 0) barra.x = 0; // Limitar movimiento a la izquierda
            if (barra.x + barra.ancho > canvas.width) barra.x = canvas.width - barra.ancho; // Limitar movimiento a la derecha
        });

        // Iniciar el juego
        document.getElementById("iniciarJuego").addEventListener("click", function() {
            jugador = document.getElementById("nombreJugador").value;
            if (jugador) {
                document.querySelector(".pantalla-inicio").style.display = "none";
                canvas.style.display = "block";
                juegoActivo = true; // Activar el juego
                puntuacion = 0; // Reiniciar puntuación
                loop(); // Iniciar el bucle del juego
            }
        });

        // Bucle del juego
        function loop() {
            if (juegoActivo) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                dibujarBorde();
                dibujarBarra();
                dibujarPelota();
                dibujarPuntuacion();
                detectarColisiones();

                pelota.x += pelota.dx;
                pelota.y += pelota.dy;

                requestAnimationFrame(loop);
            }
        }

        // Función para limpiar los récords
        document.getElementById("limpiarRecords").addEventListener("click", function() {
            const password = document.getElementById("adminPassword").value;
            if (password === "sombreroowo") {
                ranking = []; // Limpiar el ranking
                localStorage.removeItem("ranking"); // Eliminar el ranking del localStorage
                mostrarRanking(); // Actualizar la visualización del ranking
                alert("Récords limpiados.");
            } else {
                alert("Contraseña incorrecta.");
            }
        });

        // Mostrar el botón de limpiar récords y el campo de contraseña solo si hay récords
        window.addEventListener("load", function() {
            if (ranking.length > 0) {
                mostrarRanking(); // Mostrar el ranking al cargar
                document.getElementById("limpiarRecords").style.display = "block"; // Mostrar botón
                document.getElementById("adminPassword").style.display = "block"; // Mostrar campo de contraseña
            }
        });

        // Reiniciar la pelota cuando cae
        pelota.x = canvas.width / 6;
        pelota.y = canvas.height / 8;
    </script>
</body>
</html>
