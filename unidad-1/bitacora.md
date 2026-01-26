# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01

Este es  mi primer texto de la actividad 1

## Bitácora de aplicación 



## Bitácora de reflexión




### Actividad 05
# código p5.js
let port;
let connectBtn;
let movex;


function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    let sendBtn = createButton('Send Love');
    sendBtn.position(220, 300);
    fill('white');
    movex= width / 2;
    ellipse(width / 2, height / 2, 100, 100);
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){
          movex= movex - 6;
        }
        else if(dataRx == 'B'){
          movex= movex + 6;
        }
      background(220);
          ellipse(movex, height / 2, -100, 100);
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}

# Explicación
- se crea una variable para la posición en x. Todo lo demás se deja como en el código anterior: la primera parte sirve para crear el circulo y la posición inicial de este.
- function.draw: cada vez que se presiona una tecla se crea un elipse nuevo con las mismas cordeenadas del incial excepto por las cordeenadas en el eje x, que son reemplazadas por la variable anteriormente creada (movex).
- los botones B y A le suman o le restan 6 de posición a la ultima coordenada respectivamente. Es decir, si se presiona el botón A, por ejemplo, el círculo se mueve 6 hacia la izquierda desde el inicio y así sucesivamente. 


