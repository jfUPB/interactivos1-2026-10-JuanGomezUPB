# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad 01
#### ¿Qué es un sistema físico interactivo?
Trata de la aplicación de un programa que junte al mundo análogo con el mundo digital, usando sensores para captar información del mundo real y utilaizarla en el programa y que este a su vez responda a dichos sensores.
#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
Esto abre un espacio para trabajar con distintas empresas que busquen llevar su producto o publicitar su producto a tráves de la interacción con las personas.
### Actividad 02
#### ¿Qué es el diseño/arte generativo?
El diseño o arte generativo consiste en la creación de diseños en sistemas computacionales con información del mundo real. Es decir, el diseño, que ya esta programado, no se basa en la intuición o las imágenes preconcebidas del diseñador sobre la marca, sino sobre eventos medibles.
#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
Lo más factible es que el diseño generativo se aplique para empresas que busquen estrategias de marketing basadas en los datos que manejan y quieren resaltar.
### Actividad 03
### Actividad 04
#### código


```js
  let port;
  let connectBtn;
  let connectionInitialized = false;

  function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton("Connect to micro:bit");
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
  }

  function draw() {
    background(220);

    if (port.opened() && !connectionInitialized) {
      port.clear();
      connectionInitialized = true;
    }

    if (port.availableBytes() > 0) {
      let dataRx = port.read(1);
      if (dataRx == "A") {
        fill("red");
      } else if (dataRx == "N") {
        fill("green");
      }
    }

    rectMode(CENTER);
    rect(width / 2, height / 2, 50, 50);

    if (!port.opened()) {
      connectBtn.html("Connect to micro:bit");
    } else {
      connectBtn.html("Disconnect");
    }
  }

  function connectBtnClick() {
    if (!port.opened()) {
      port.open("MicroPython", 115200);
      connectionInitialized = false;
    } else {
      port.close();
    }
  }

```


## Bitácora de aplicación 
### Actividad 05
#### código p5.js

```js
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

```

#### código micro.bit

```py
from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:
    if button_a.is_pressed():
        uart.write('A')
        sleep(500)
    if button_b.is_pressed():
        uart.write('B')
        sleep(500)
```

#### Explicación
- se crea una variable para la posición en x. Todo lo demás se deja como en el código anterior: la primera parte sirve para crear el circulo y la posición inicial de este.
- function.draw: cada vez que se presiona una tecla se crea un elipse nuevo con las mismas cordeenadas del incial excepto por las cordeenadas en el eje x, que son reemplazadas por la variable anteriormente creada (movex).
- los botones B y A le suman o le restan 6 de posición a la ultima coordenada respectivamente. Es decir, si se presiona el botón A, por ejemplo, el círculo se mueve 6 hacia la izquierda desde el inicio y así sucesivamente. 




## Bitácora de reflexión
### Actividad 06
#### Regresa a la actividad 4 y trata de explicar en tus propias palabras de la manera más detallada que puedas cómo funciona el sistema físico interactivo. 
  En la Actividad 04 el fin del sistema es que cuando se presione el botón A del micro:bit, el cuadrado en pantalla pase de ser color verde a color rojo.
  
  En el códidgo del programa p5.js el function setup crea el canvas, el fondo y el botón para conectar el programa con el micro:bit. Una vez se tiene el canvas sobre el cual trabajar hay que crear el cuadrado, por lo que se crea un function: draw. Se escribe una condición con un if, de modo que si el micro:bit manda una señal de "A", el cuadrado se pinte rojo. Por el contrario, si llega una señal "N", el cuadrado se pinta de verde. Las siguientes líneas definen el tamaña del cuadrado y su posición en el canvas (centrado): "rect(width / 2, height / 2, 50, 50);".
  
  En el programa del micro:bit se trabaja solo con un if. Si  el botón A se presiona ("if button_a.is_pressed():"), entonces se envía la señal "A" ("uart.write('A')") al programa p5.js. Si se presiona cualquier botón que no sea el A, o si directamente no se presiona ningún botón. Es decir, si se lleva a cabo cualquier otra acción ("else:"), entonces se envía la señal "N" ("uart.write('N')").
  
  Una vez se ejevcuta el programa, como el usuario no presiona ningún botón el cuadrado permeanece verde hasta que el usuario presione el botón A.








