# Phaser Juego 3 Balas 1 Jugador


### Variables globales
~~~
var w = 800;
var h = 400;
var jugador;
var fondo;

var bala, balaD = false, nave, nave2, bala2, balaD2 = false;
var nave3, bala3, balaD3 = false;

var salto;
var desplazo;
var menu;

var velocidadBala;
var despBala;
var estatusAire;

var velocidadBala2;
var despBala2;
var estatusDesp;

var velocidadBala3;
var despBala3;

var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = [];
var modoAuto = false, eCompleto = false;

var data_phaser = "";
~~~
### Creación del juego Phaser
~~~
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });
~~~
### Función para precargar los recursos
~~~
function preload() {
    juego.load.image('fondo', 'assets/game/fondo.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48);
    juego.load.image('nave', 'assets/game/ufo.png');
    juego.load.image('bala', 'assets/sprites/purple_ball.png');
    juego.load.image('menu', 'assets/game/menu.png');
}
~~~

### Función para crear los elementos del juego
~~~
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE);
    juego.time.desiredFps = 30;

    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
    nave = juego.add.sprite(w - 100, h - 70, 'nave');
    bala = juego.add.sprite(w - 100, h, 'bala');
    nave2 = juego.add.sprite(5, 0, 'nave');
    bala2 = juego.add.sprite(50, 0, 'bala');
    jugador = juego.add.sprite(50, h, 'mono');

    nave3 = juego.add.sprite(w - 100, 0, 'nave'); // Nueva nave en la esquina superior derecha
    bala3 = juego.add.sprite(50, 0, 'bala'); // Bala de la nueva nave

    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true;
    jugador.body.gravity.y = 800;
    var corre = jugador.animations.add('corre', [8, 9, 10, 11]);
    jugador.animations.play('corre', 10, true);

    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true;

    juego.physics.enable(bala2);
    bala2.body.collideWorldBounds = true;

    juego.physics.enable(bala3);
    bala3.body.collideWorldBounds = true;

    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);

    salto = juego.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
    desplazo = juego.input.keyboard.addKey(Phaser.Keyboard.RIGHT);

    nnNetwork = new synaptic.Architect.Perceptron(4, 8, 6, 6, 2);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
}
~~~
### Función para entrenar la red neuronal
~~~
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.0001, iterations: 30000, shuffle: true });
}
~~~
### Función para obtener datos de entrenamiento para la red neuronal
~~~
function datosDeEntrenamiento(param_entrada) {
    console.log("Entrada", param_entrada[0] + " " + param_entrada[1] + ' ' + param_entrada[2] + " " + param_entrada[3] + ' ' + param_entrada[4] + " " + param_entrada[5]);
    nnSalida = nnNetwork.activate(param_entrada);
    var aire = Math.round(nnSalida[0] * 100);
    var desp = Math.round(nnSalida[1] * 100);
    console.log("Valor ", "En el Aire %: " + aire + " En desplazamiento %: " + desp);
    var status = [false, false];
    if (aire > 50)
        status[0] = true;
    if (desp > 50)
        status[1] = true;
    return status;
}
~~~
### Función para pausar el juego
~~~
function pausa() {
    juego.paused = true;
    menu = juego.add.sprite(w / 2, h / 2, 'menu');
    menu.anchor.setTo(0.5, 0.5);
}
~~~

### Función para descargar los resultados en un archivo CSV
~~~
function descarga() {
    const enlaceDescarga = document.createElement('a');
    enlaceDescarga.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(data_phaser));
    enlaceDescarga.setAttribute('download', 'results.csv');
    enlaceDescarga.style.display = 'none';

    ### Inicia la descarga
    document.body.appendChild(enlaceDescarga);
    enlaceDescarga.click();
    document.body.removeChild(enlaceDescarga);
}
~~~
### Función para manejar la pausa del juego
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
                    descarga();
                    console.log("", "Entrenamiento " + datosEntrenamiento.length + " valores");
                    enRedNeural();
                    eCompleto = true;
                }
                modoAuto = true;
            }

            menu.destroy();
            resetVariables();
            resetVariablesB2();
            resetVariablesB3();
            jugador.position.x = 50;
            juego.paused = false;
        }
    }
}
~~~
### Función para resetear las variables del jugador y las balas
~~~
function resetVariables() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;
    bala.body.velocity.x = 0;
    bala.position.x = w - 100;
    balaD = false;
}

function resetVariablesB2() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;
    bala2.body.velocity.y = 0;
    bala2.position.y = 0;
    balaD2 = false;
}

function resetVariablesB3() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;
    bala3.body.velocity.x = 0;
    bala3.body.velocity.y = 0;
    bala3.position.x = w - 50;
    bala3.position.y = 0;
    balaD3 = false;
}
~~~
### Función para saltar
~~~
function saltar() {
    jugador.body.velocity.y = -270;
}
~~~
### Función para desplazar al jugador a la derecha
~~~
function desplazarDer() {
    if (jugador.position.x < 100)
        jugador.position.x += 5;
}
~~~
### Función para resetear la posición del jugador
~~~
function reset() {
    if (jugador.position.x > 50)
        jugador.position.x -= 5;
}
~~~
### Función de actualización del juego
~~~
function update() {
    fondo.tilePosition.x -= 1;

    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala2, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala3, jugador, colisionH, null, this);

    estatusAire = 0;
    estatusDesp = 0;

    if (!jugador.body.onFloor())
        estatusAire = 1;

    if (jugador.position.x != 50 && desplazo.isDown)
        estatusDesp = 1;

    despBala = Math.floor(jugador.position.x - bala.position.x);
    despBala2 = Math.floor(jugador.position.y - bala2.position.y);
    despBala3 = Math.floor(jugador.position.y - bala2.position.y); // Distancia en diagonal

    if (!modoAuto) {
        if (salto.isDown && jugador.body.onFloor())
            saltar();

        if (desplazo.isDown)
            desplazarDer();

        if (!desplazo.isDown && jugador.position.x != 50)
            reset();
    }

    if (modoAuto && (bala.position.x > 0 || bala2.position.y > 0 || bala3.position.x > 0)) {
        var resultConsulta = datosDeEntrenamiento([despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3]);
        if (resultConsulta[0] && jugador.body.onFloor()) {
            saltar();
        }

        if (resultConsulta[1])
            desplazarDer();
        else if (jugador.position.x != 50)
            reset();
    }

    if (balaD == false) {
        disparo();
    }

    if (balaD2 == false) {
        disparoVert();
    }

    if (balaD3 == false) {
        disparoDiag();
    }

    if (bala.position.x <= 0) {
        resetVariables();
    }

    if (bala2.body.onFloor()) {
        resetVariablesB2();
    }

    if (bala3.position.x <= 0 || bala3.body.onFloor()) {
        resetVariablesB3();
    }

    if (modoAuto == false && bala.position.x > 0) {
        datosEntrenamiento.push({
            'input': [despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3],
            'output': [estatusAire, estatusDesp]
        });

        console.log(despBala + ' ' + velocidadBala + ' ' + despBala2 + ' ' + velocidadBala2 + ' ' + despBala3 + ' ' + velocidadBala3 + ' ' + estatusAire + ' ' + estatusDesp);
        data_phaser += despBala + ' ' + velocidadBala + ' ' + despBala2 + ' ' + velocidadBala2 + ' ' + despBala3 + ' ' + velocidadBala3 + ' ' + estatusAire + ' ' + estatusDesp + "\n";
    }
}
~~~
### Función para disparar la bala horizontalmente
~~~
function disparo() {
    velocidadBala = -1 * velocidadRandom(150, 350);
    bala.body.velocity.y = 0;
    bala.body.velocity.x = velocidadBala;
    balaD = true;
}
~~~
### Función para disparar la bala verticalmente
~~~
function disparoVert() {
    velocidadBala2 = 100;
    bala2.body.velocity.x = 0;
    bala2.body.velocity.y = velocidadBala2;
    balaD2 = true;
}
~~~
### Función para disparar la bala diagonalmente
~~~
function disparoDiag() {
    bala3.body.velocity.x = -210;
    bala3.body.velocity.y = 100;
    balaD3 = true;
}
~~~
### Función para manejar la colisión del jugador con una bala
~~~
function colisionH() {
    pausa();
}
~~~
### Función para generar una velocidad aleatoria
~~~
function velocidadRandom(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
~~~
### Funcion para renderizar el juego
~~~
function render() {
    
}
~~~