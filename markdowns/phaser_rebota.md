
### Definición de variables globales
~~~
var w = 800; // Ancho del juego
var h = 400; // Alto del juego
var jugador, fondo, bala, nave; // Variables para los sprites del jugador, fondo, bala y nave
var balaD = false; // Estado de la bala disparada
var izquierda, derecha, arriba, abajo; // Teclas de movimiento
var menu, exportarBtn; // Variables para el menú y botón de exportar
var velocidadBala, despBala; // Variables para la velocidad y desplazamiento de la bala
var estatusAire, estatusSuelo; // Estado del jugador en el aire y en el suelo
var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = []; // Variables para la red neuronal y datos de entrenamiento
var modoAuto = false, eCompleto = false; // Variables para el modo automático y estado de entrenamiento completo
~~~
### Creación del juego con Phaser
~~~
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });
~~~
### Precarga de recursos del juego
~~~
function preload() {
    juego.load.image('fondo', 'assets/game/fondo.jpg'); // Carga la imagen del fondo
    juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48); // Carga el sprite del jugador
    juego.load.image('nave', 'assets/game/ufo.png'); // Carga la imagen de la nave
    juego.load.image('bala', 'assets/sprites/purple_ball.png'); // Carga la imagen de la bala
    juego.load.image('menu', 'assets/game/menu.png'); // Carga la imagen del menú
}
~~~
### Inicialización del juego
~~~
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE); // Inicia el sistema de física
    juego.physics.arcade.gravity.y = 800; // Configura la gravedad
    juego.time.desiredFps = 30; // Configura los FPS deseados

    // Añade el fondo, nave, bala y jugador al juego
    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
    nave = juego.add.sprite(w - 100, h - 70, 'nave');
    bala = juego.add.sprite(w / 2, h / 2, 'bala');
    jugador = juego.add.sprite(w / 2, h / 2, 'mono');

    // Habilita la física para el jugador y añade animación
    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true;
    var corre = jugador.animations.add('corre', [8, 9, 10, 11]);
    jugador.animations.play('corre', 10, true);

    // Habilita la física para la bala y configura el rebote
    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true;
    bala.body.bounce.set(1);

    // Añade texto de pausa y habilita eventos de pausa
    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);

    // Configura las teclas de movimiento
    izquierda = juego.input.keyboard.addKey(Phaser.Keyboard.A);
    derecha = juego.input.keyboard.addKey(Phaser.Keyboard.D);
    arriba = juego.input.keyboard.addKey(Phaser.Keyboard.W);
    abajo = juego.input.keyboard.addKey(Phaser.Keyboard.S);

    // Inicializa la red neuronal
    nnNetwork = new synaptic.Architect.Perceptron(8, 16, 4); // Ajustado para 8 entradas y 4 salidas
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);

    // Añade botón de exportar
    exportarBtn = juego.add.text(20, 20, 'Exportar', { font: '20px Arial', fill: '#fff' });
    exportarBtn.inputEnabled = true;
    exportarBtn.events.onInputUp.add(exportarDatos, self);
}
~~~
### Función para entrenar la red neuronal
~~~
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.01, iterations: 20000, shuffle: true, cost: synaptic.Trainer.cost.CROSS_ENTROPY });
}
~~~
### Función para obtener los datos de entrenamiento y calcular la salida de la red neuronal
~~~
function datosDeEntrenamiento(param_entrada) {
    nnSalida = nnNetwork.activate(param_entrada);
    var aire = Math.round(nnSalida[0] * 100);
    var piso = Math.round(nnSalida[1] * 100);
    var movimientoX = Math.round(nnSalida[2] * 100);
    var movimientoY = Math.round(nnSalida[3] * 100);

    console.log("Valor ", "En el Aire %: " + aire + " En el suelo %: " + piso + " Movimiento X %: " + movimientoX + " Movimiento Y %: " + movimientoY);
    
    if (aire >= piso && aire >= movimientoX && aire >= movimientoY) {
        return 'arriba';
    } else if (piso >= aire && piso >= movimientoX && piso >= movimientoY) {
        return 'abajo';
    } else if (movimientoX >= aire && movimientoX >= piso && movimientoX >= movimientoY) {
        return 'derecha';
    } else {
        return 'izquierda';
    }
}
~~~
### Función para pausar el juego y mostrar el menú
~~~
function pausa() {
    juego.paused = true;
    menu = juego.add.sprite(w / 2, h / 2, 'menu');
    menu.anchor.setTo(0.5, 0.5);
}
~~~
### Función para manejar el menú de pausa
~~~
function mPausa(event) {
    if (juego.paused) {
        var menu_x1 = w / 2 - 270 / 2, menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2, menu_y2 = h / 2 + 180 / 2;

        var mouse_x = event.x,
            mouse_y = event.y;

        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
                eCompleto = false;
                datosEntrenamiento = [];
                modoAuto = false;
            } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
                if (!eCompleto) {
                    console.log("", "Entrenamiento " + datosEntrenamiento.length + " valores");
                    enRedNeural();
                    eCompleto = true;
                }
                modoAuto = true;
            }

            menu.destroy();
            resetVariables();
            juego.paused = false;
        }
    }
}
~~~
### Función para resetear las variables del juego
~~~
function resetVariables() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;
    bala.body.velocity.x = velocidadRandom(300, 800);
    bala.body.velocity.y = velocidadRandom(300, 800);
    bala.position.x = w / 2;
    bala.position.y = h / 2;
    jugador.position.x = w / 2;
    jugador.position.y = h / 2;
    balaD = false;
}
~~~
### Funciones para mover el jugador
~~~
function moverIzquierda() {
    jugador.body.velocity.x = -150;
    jugador.body.velocity.y = -30;
    if (jugador.body.blocked.left) {
        jugador.body.velocity.y = -150; // Salto en caso de bloqueo lateral
    }
}

function moverDerecha() {
    jugador.body.velocity.x = 150;
    jugador.body.velocity.y = -30;
    if (jugador.body.blocked.right) {
        jugador.body.velocity.y = -150; // Salto en caso de bloqueo lateral
    }
}

function moverArriba() {
    jugador.body.velocity.y = -150;
}

function moverAbajo() {
    jugador.body.velocity.y = 150;
}

function detenerMovimiento() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = -30; // Detiene el movimiento vertical también
}
~~~
### Función para actualizar el estado del juego en cada frame
~~~
function update() {
    fondo.tilePosition.x -= 1; // Mueve el fondo
    juego.physics.arcade.collide(bala, jugador, colisionH, null, this); // Verifica colisiones entre bala y jugador

    // Actualiza los estados del jugador
    estatusSuelo = jugador.body.onFloor() ? 1 : 0;
    estatusAire = !jugador.body.onFloor() ? 1 : 0;
    despBala = Math.floor(jugador.position.x - bala.position.x);

    if (!modoAuto) {
        // Movimiento manual
        if (izquierda.isDown) {
            moverIzquierda();
        } else if (derecha.isDown) {
            moverDerecha();
        } else if (arriba.isDown) {
            moverArriba();
        } else if (abajo.isDown) {
            moverAbajo();
        } else {
            detenerMovimiento();
        }

        // Recolectar datos de entrenamiento
        if (bala.position.x > 0) {
            datosEntrenamiento.push({
                input: [despBala, bala.body.velocity.x, jugador.body.velocity.x, jugador.body.velocity.y, jugador.position.x, jugador.position.y, bala.position.x, bala.position.y],
                output: [estatusAire, estatusSuelo, jugador.body.velocity.x, jugador.body.velocity.y]
            });

            console.log("Desplazamiento Bala, Velocidad Bala, Velocidad Jugador, Estatus Aire, Estatus Suelo, Posiciones: ",
                despBala + " " + bala.body.velocity.x + " " + jugador.body.velocity.x + " " + estatusAire + " " + estatusSuelo + " " + jugador.position.x + " " + jugador.position.y);
        }
    } else if (modoAuto && bala.position.x > 0) {
        // Movimiento automático
        var accion = datosDeEntrenamiento([despBala, bala.body.velocity.x, jugador.body.velocity.x, jugador.body.velocity.y, jugador.position.x, jugador.position.y, bala.position.x, bala.position.y]);
        if (accion === 'arriba') {
            moverArriba();
        } else if (accion === 'izquierda') {
            moverIzquierda();
        } else if (accion === 'derecha') {
            moverDerecha();
        } else if (accion === 'abajo') {
            moverAbajo();
        } else {
            detenerMovimiento();
        }
    }

    if (!balaD) {
        disparo();
    }
}
~~~
### Función para disparar la bala
~~~
function disparo() {
    velocidadBala = -1 * velocidadRandom(300, 800);
    bala.body.velocity.y = velocidadRandom(300, 800);
    bala.body.velocity.x = velocidadBala;
    balaD = true;
}
~~~
### Función que maneja la colisión entre la bala y el jugador
~~~
function colisionH() {
    pausa(); // Pausa el juego en caso de colisión
}
~~~

### Función para generar una velocidad aleatoria entre un rango
~~~
function velocidadRandom(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
~~~
### Función para exportar datos de entrenamiento a un archivo Excel
~~~
function exportarDatos() {
    // Crear una nueva hoja de cálculo
    var wb = XLSX.utils.book_new();
    var ws_data = [["despBala", "velocidadBala", "velocidadJugador", "estatusAire", "estatusSuelo"]]; // encabezados
    datosEntrenamiento.forEach(function (dato) {
        ws_data.push([dato.input[0], dato.input[1], dato.input[2], dato.input[3], dato.input[4], dato.input[5], dato.input[6], dato.input[7]]);
    });
    var ws = XLSX.utils.aoa_to_sheet(ws_data);
    XLSX.utils.book_append_sheet(wb, ws, "Datos de Entrenamiento");

    // Generar archivo Excel y descargarlo
    XLSX.writeFile(wb, "datos_entrenamiento.xlsx");
}
~~~
### Función render, actualmente vacía
~~~
function render() {
}
~~~