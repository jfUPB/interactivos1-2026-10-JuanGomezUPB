# Unidad 4

## Bitácora de proceso de aprendizaje

### Actividad 01
- ¿Cómo hago las instrucciones con el microbit?

### Actividad 02
- Backend
- Frontend

**Comandos**
- npm
- *npm install*
- ctrl + C = mandar la señal para terminar el proceso
- code . = abrimos el proyecto en visual studio code (code= vsiual srtudio) ( . = abrir la carpeta del proyecto)

**n**
node bridgeServer.js --device microbit = Abrir él que funciona con los comandos del microbit

## Bitácora de aplicación 
### Actividad 02
En esa parte del código se están procesando los datos que llegan desde el microbit. La función toma la línea de texto que llega por el puerto y la divide usando **|** como separador. A continuación, lo que sucede es que el código verificar que realmente hayan llegado los 6 campos esperados (T, X, Y, A, B, CHK). De no ser así, se lanza un error porque significa que está incompleta o corrupta.

Luego se extrae cada valor separando por **:** para obtener solo el número asociado a cada variable (por ejemplo, de X:-245 se obtiene -245). Estos valores se convierten a números para poder trabajar con ellos.

Ahora se realizan algunas validaciones:
- **x** y **y** deben ser números válidos.
- Deben estar dentro del rango esperado del acelerómetro.
- Los botones solo deben teneer valores de **0** o **1**.

Después se calcula el checksum, que consiste en sumar el valor absoluto de x y y, más los estados de los botones A y B. Este valor se compara con el checksum que viene en la trama. Si no coinciden, se lanza un error porque significa que los datos pudieron corromperse. Finalmente, y si todo es correcto, la función devuelve un objeto con los valores de x, y y los botones convertidos a booleanos, que es el formato que espera el resto del sistema.

'''' .py
function parseCsvLine(line) {
  const values = line.trim().split("|");

  if (values.length !== 6) throw new ParseError(`Expected 6 values, got ${values.length}`);

  // $T:tiempo|X:acel_x|Y:acel_y|A:estado_a|B:estado_b|CHK:checksum\n

  const t = Number(values[0].split(":")[1]);
  const x = Number(values[1].split(":")[1]);
  const y = Number(values[2].split(":")[1]);
  const btnA = Number(values[3].split(":")[1]);
  const btnB = Number(values[4].split(":")[1]);
  const chk = Number(values[5].split(":")[1]); 

  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (![0, 1].includes(btnA) || ![0, 1].includes(btnB)) throw new ParseError("Invalid button data");
  
  const calc = Math.abs(x) + Math.abs(y) + btnA + btnB;
  if (calc !== chk) throw new ParseError("Hola, checksum mismatch");


  return { x: x | 0, y: y | 0, btnA: btnA === 1, btnB: btnB === 1 };
}
''''

En esta parte se programa el microbit para enviar continuamente los datos al computador mediante comunicación serial. Primero se inicializa el puerto UART a 115200 baudios y se enciende un píxel en la pantalla para indicar que el programa está activo. Luego, dentro de un ciclo infinito, se obtienen el tiempo desde que el dispositivo se encendió, los valores del acelerómetro en los ejes X y Y, y el estado de los botones A y B, representados como 1 si están presionados o 0 si no. Después se calcula un checksum sumando el valor absoluto de X y Y más el estado de los botones, lo que permite verificar la integridad de los datos. Finalmente se construye la trama con el formato definido y se envía por el puerto serial cada 100 milisegundos, es decir, aproximadamente a una frecuencia de 10 Hz.

```` .py
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

while True:
    t = running_time()
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = 1 if button_a.is_pressed() else 0
    bState = 1 if button_b.is_pressed() else 0
    chk = abs(xValue) + abs(yValue) + aState + bState
    data = "$T:{}|X:{}|Y:{}|A:{}|B:{}|CHK:{}\n".format(t, xValue, yValue, aState, bState, chk
    )
    uart.write(data)
    sleep(100) # Envia datos a 10 Hz    
````


En esta parte del proyecto se integra la información que llega del microbit con la lógica gráfica del programa en p5.js, respetando la arquitectura del sistema. Para hacerlo, solo se modifican dos funciones del objeto painter: updateLogic(data) y drawRunning(), además de agregar dos nuevas variables al objeto: circleResolution y radius.

En el constructor se inicializan estas variables con valores por defecto para asegurar que el programa tenga parámetros válidos antes de recibir datos del hardware.
`````.py
this.circleResolution = 5;
this.radius = 100;
````

Luego, dentro de updateLogic(data), se actualizan estas variables utilizando los valores del acelerómetro que llegan desde el microbit. Para adaptar los rangos del sensor al tamaño del canvas se utiliza la función map(), que transforma el rango original del acelerómetro (-2048 a 2047) en valores útiles para el dibujo.
```` .py
this.circleResolution = int(map(data.y, -2048, 2047, 2, 10));
this.radius = map(data.x, -2048, 2047, -width/2, width/2);
````

De esta manera, el eje Y controla la resolución del círculo (es decir, cuántos vértices tiene la figura), mientras que el eje X controla el radio del círculo.

Finalmente, en la función drawRunning() se utiliza esa información para dibujar la figura en el canvas. Dependiendo del estado del botón B, el círculo puede dibujarse con relleno o solo con su contorno.

## Explicación

En esta parte se define la función `drawRunning()`, que es la encargada de dibujar la figura en el canvas usando los datos que llegan desde el microbit. Primero se obtiene el objeto `mb`, que contiene la información recibida del dispositivo, y se verifica que los datos estén listos antes de continuar.

Si el botón **A** está presionado, el programa comienza a dibujar. Para hacerlo, se mueve el origen del canvas al centro usando `translate(width / 2, height / 2)`, lo que permite que la figura se dibuje alrededor del centro de la pantalla.

Luego se calcula el ángulo entre cada vértice del polígono usando `TAU / painter.circleResolution`. Esto determina cómo se distribuyen los puntos alrededor del círculo. Dependiendo del estado del botón **B**, la figura se dibuja con relleno o sin relleno. Después se usa `beginShape()` y un ciclo `for` para calcular la posición de cada vértice utilizando funciones trigonométricas (`cos` y `sin`) multiplicadas por el radio del círculo. Cada punto se agrega con `vertex()`, y finalmente `endShape()` cierra la figura.

---

```javascript
function drawRunning() {
  let mb = painter.rxData;

  if (!mb.ready) return;

  if (mb.btnA) {
    push();
    translate(width / 2, height / 2);

    let angle = TAU / painter.circleResolution;
    if (mb.btnB) {
      fill(34, 45, 122, 50);
    } else {
      noFill();
    }
    stroke(0);
    beginShape();
    for (let i = 0; i <= painter.circleResolution; i++) {
      let x = cos(angle * i) * painter.radius;
      let y = sin(angle * i) * painter.radius;

      vertex(x, y);
    }
    endShape();
    pop();
  }
}
```

En esta función se realiza el dibujo de la figura en el canvas utilizando los datos que llegan desde el microbit. Primero se revisa que los datos estén listos y luego se verifica si el botón A está presionado, ya que este botón controla cuándo se dibuja la figura. Después se mueve el origen del dibujo al centro del canvas para que la figura se genere desde allí. Con el valor de `circleResolution` se calcula el ángulo entre cada vértice del polígono, y dependiendo del estado del botón B se decide si la figura se dibuja con relleno o solo con su contorno. Luego, mediante un ciclo `for`, se calculan las posiciones de los vértices usando funciones trigonométricas y el radio definido, agregando cada punto con `vertex()` hasta completar la figura.

## Bitácora de reflexión

