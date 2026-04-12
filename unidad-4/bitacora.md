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
**Parte 1**
*En esta primera parte voy a explicar el código que aparece en la página de la unidad 4 con el fin de entender bien que hace cada parte*

En esa parte del código se están procesando los datos que llegan desde el microbit. Se verifica si lo que llego esta incompleto o corrupto.
````.js
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
  if (calc !== chk) throw new ParseError("Checksum mismatch");


  return { x: x | 0, y: y | 0, btnA: btnA === 1, btnB: btnB === 1 };
}
````
- En la siguiente parte del código se dividen los valores que llegarón en las distintas constantes: t, x, y, btnA, btnB, chk. Con **values[n]** se accede a el valor en la posición n. Es decir, values[0] va a ser el primer valor, en este caso t. El **.split(":")** va a dividir los valores usando **":"** :
```` .py
const t = Number(values[0].split(":")[1]);
const x = Number(values[1].split(":")[1]);
const y = Number(values[2].split(":")[1]);
const btnA = Number(values[3].split(":")[1]);
const btnB = Number(values[4].split(":")[1]);
const chk = Number(values[5].split(":")[1]);
````
- Las siguientes lineas representan los posibles errores en los datos que llegaron. Por ejemplo, si el botón A o el botón B tienen valores distintos de 0 o 1:
```` .js
if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
if (![0, 1].includes(btnA) || ![0, 1].includes(btnB)) throw new ParseError("Invalid button data");
````
- Hay que verificar que sean correctos los datos, por lo que se suman los valores absolutos de x, y, btnA y btnB. De ser distintos, se muestra un error:
```` .js
const calc = Math.abs(x) + Math.abs(y) + btnA + btnB;
if (calc !== chk) throw new ParseError("Hola, checksum mismatch");
````
- Finalmente, y si todo es correcto, la función devuelve un objeto con los valores de x, y y los botones convertidos a booleanos, que es el formato que espera el resto del sistema.
```` .js
return { x: x | 0, y: y | 0, btnA: btnA === 1, btnB: btnB === 1 };
````

**Parte 2**
- Capa de transporte --> "invisible" para mí.

**Parte 3**
- En el sketch.js solo se modifican dos funciones que son drawRunning() y updateLogic(data) del objeto painter, y se agregan dos variables a este mismo objeto, circleResolution y radius, las cuales serán modificadas por los valores de X y Y del microbit respectivamente.
- En el constructor se agregan estas dos variables para inicializarlas con valores por defecto:
````.js
this.circleResolution = 5;
this.radius = 100;
````
  **updateLogic**:
- En **updateLogic**  van a aterrizar los datos provenientes del hardware.
```` .js
updateLogic(data) {
        this.rxData.ready = true;
        this.rxData.x = map(data.x,-2048,2047,0,width);
        this.rxData.y = map(data.y,-2048,2047,0,height);
        this.rxData.btnA = data.btnA;
        this.rxData.btnB = data.btnB;

        if (this.rxData.btnA && !this.prevA) {
            this.lineSize = random(50, 160);
            this.clickPosX = this.rxData.x;
            this.clickPosY = this.rxData.y;
            console.log("A pressed");
        }

        if (!this.rxData.btnB && this.prevB) {
            this.c = color(random(255), random(255), random(255), random(80, 100));
            console.log("B released");
        }

        this.prevA = this.rxData.btnA;
        this.prevB = this.rxData.btnB;

        this.circleResolution = int(map(data.y, -2048, 2047, 2, 10));
        this.radius = map(data.x, -2048, 2047, -width/2, width/2);
    }

````

- En la primera parte se actualizan los datos que llegan del micro:bit. **this.rxData** es un objeto donde se van a guardar los datos procesados. El **ready = true:** indica que ya llegaron datos. El **map(...):** que nos recomiendan utilizar en la unidad transforma un rango a otro. En este caso, convierte data.x y data.y (que van de -2048 a 2047) a coordenadas de pantalla, 0 a width y 0 a height respectivamente.
```` .js
this.rxData.ready = true;
this.rxData.x = map(data.x,-2048,2047,0,width);
this.rxData.y = map(data.y,-2048,2047,0,height);
this.rxData.btnA = data.btnA;
this.rxData.btnB = data.btnB;
````

- Luego se verifica si el botón A esta siendo presionado. **this.rxData.btnA** significa que A está presionado ahora, **!this.prevA** significa que antes NO estaba presionado.
```` .js
if (this.rxData.btnA && !this.prevA) {
````

- Cuando A esta presionado, se crea un tamaño aleatorio (**lineSize**) y se guarda la posición de pantalla con **ClickPosY** y **ClickPosX**.
````.js
this.lineSize = random(50, 160);
this.clickPosX = this.rxData.x;
this.clickPosY = this.rxData.y;
console.log("A pressed");
````

- Luego se detecta si B se soltó:
```` .js
if (!this.rxData.btnB && this.prevB) {
````

- Se crea un color aleatorio y se guarda this.c:
```` .js
this.c = color(random(255), random(255), random(255), random(80, 100));
console.log("B released");
````

- Finalmente se agrega:
```` .js
  this.circleResolution = int(map(data.y, -2048, 2047, 2, 10));
  this.radius = map(data.x, -2048, 2047, -width/2, width/2);
````
- En **this.circleResolution**, **map(...):** transforma data.y de un rango [-2048, 2047] a un nuevo rango [2, 10] e **int(...):** convierte el resultado a un número entero.
- En **this.Radius** se convierte data.x al rango [-width/2, width/2].

  **drawRunning**
- Se dibuja en pantalla solo cuando se mantiene presionado el botón A.
```` .js
function drawRunning() {
    let mb = painter.rxData;

    if (!mb.ready) return;

    if (mb.btnA) {
        let x = mb.x;
        let y = mb.y;
        push();
        translate(x, y);
        rotate(radians(painter.angle));
        stroke(painter.c);
        line(0, 0, painter.lineSize, painter.lineSize);
        painter.angle += 1;
        pop();
    }
}
````

- Primero se accede a los datos:
```` .js
let mb = painter.rxData;
````

- Para evitar errores o datos invalidos, si ready es false, la función se detiene:
```` .js
if (!mb.ready) return;
````

- Se dibuja solo si el botón A esta siendo presionado:
  ````.js
  if (mb.btnA) {
  ````
- Ahora se utilizan las coordenas del dibujo guardadas antes:
  ```` .js
  let x = mb.x;
  let y = mb.y;
  ````
- *Dibujo*:
````.js
push();
translate(x, y);
rotate(radians(painter.angle));
stroke(painter.c);
line(0, 0, painter.lineSize, painter.lineSize);
painter.angle += 1;
pop();
````
- **push() / pop()**: Guardan y restauran el estado del dibujo. Evitan que transformaciones afecten otras partes.
- **translate(x, y)**: Mueve el origen del dibujo a la posición (x, y). A partir de aquí, todo se dibuja relativo a ese punto.
- **rotate(radians(painter.angle))**: Rota el sistema de coordenadas. angle está en grados, radians() lo convierte a radianes. Hace que la línea gire.
- **stroke(painter.c)**: Define el color de la línea.
- **line(0, 0, painter.lineSize, painter.lineSize)**: Dibuja una línea desde (0,0) hasta (lineSize, lineSize). Como el sistema está trasladado y rotado, la línea: aparece en (x, y) y gira con el ángulo.
- **painter.angle += 1**: Incrementa el ángulo en cada frame. Produce animación (la línea gira continuamente).

**-------------------------------------------------------------------------------------------------------**

**Parte 2**
***Código del MicrobitASCII2Adapter*** *(El código nuevo)*
````.js
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error { }

//$T:tiempo|X:acel_x|Y:acel_y|A:estado_a|B:estado_b|CHK:checksum\n

function parseCsvLine(line) {
  const values = line.trim().split("|");
  if (values.length !== 6) throw new ParseError(`Expected 6 values, got ${values.length}`);

  const t = Number(values[0].split(":")[1]);
  const x = Number(values[1].split(":")[1]);
  const y = Number(values[2].split(":")[1]);
  const btnA = Number(values[3].split(":")[1]);
  const btnB = Number(values[4].split(":")[1]);
  const CHK = Number(values[5].split(":")[1]) % 1000;
  const calcCHK = Math.abs(x) + Math.abs(y) + btnA + btnB;
  if (calcCHK !== CHK)
  throw new ParseError("Checksum mismatch");

  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (![1, 0].includes(btnA) || ![1, 0].includes(btnB)) throw new ParseError("Invalid button data");

  return { x: x | 0, y: y | 0, btnA: btnA === 1, btnB: btnB === 1 };
}


class MicrobitAscii2Adapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = "";
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required for microbit device mode");

    this.port = new SerialPort({
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });

    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", (chunk) => this._onChunk(chunk));
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;

    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close((err) => {
          if (err) reject(err);
          else resolve();
        });
      });
    }
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    this.buf += chunk.toString("utf8");

    let idx;
    while ((idx = this.buf.indexOf("\n")) >= 0) {
      const line = this.buf.slice(0, idx).trim();
      this.buf = this.buf.slice(idx + 1);

      if (!line) continue;

      try {
        const parsed = parseCsvLine(line);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (this.verbose) console.log("Bad data:", e.message, "raw:", line);
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 4096) this.buf = "";
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  async handleCommand(cmd) {
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      await this.writeLine(`LED,${x},${y},${v}\n`);
    }
  }
}

module.exports = MicrobitAscii2Adapter;
````
**Explicación del código nuevo:**
````.js
class ParseError extends Error { }
````
Define un error propio para distinguir errores de datos mal formados.

````.js
function parseCsvLine(line) {
  const values = line.trim().split("|");
  if (values.length !== 6) throw new ParseError(`Expected 6 values, got ${values.length}`);

  const t = Number(values[0].split(":")[1]);
  const x = Number(values[1].split(":")[1]);
  const y = Number(values[2].split(":")[1]);
  const btnA = Number(values[3].split(":")[1]);
  const btnB = Number(values[4].split(":")[1]);
  const CHK = Number(values[5].split(":")[1]) % 1000;
  const calcCHK = Math.abs(x) + Math.abs(y) + btnA + btnB;
  if (calcCHK !== CHK)
  throw new ParseError("Checksum mismatch");

  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (![1, 0].includes(btnA) || ![1, 0].includes(btnB)) throw new ParseError("Invalid button data");

  return { x: x | 0, y: y | 0, btnA: btnA === 1, btnB: btnB === 1 };
````
Esta función es importante porque se encarga de leer las líneas que llegan de la siguiente forma:

**a)** Separa los datos
````.js
 const values = line.trim().split("|");
````
Divide los datos que llegan por "|"

**b)** Validar cantidad
````.js
if (values.length !== 6) throw new ParseError(`Expected 6 values, got ${values.length}`);
````
Los datos deben llegar en seis partes, de lo contrario se muestra en pantalla el siguiente error: "Expected 6 values, got ${values.length}"

**c)** Extraer valores
````.js
  const t = Number(values[0].split(":")[1]);
  const x = Number(values[1].split(":")[1]);
  const y = Number(values[2].split(":")[1]);
  const btnA = Number(values[3].split(":")[1]);
  const btnB = Number(values[4].split(":")[1]);
  const CHK = Number(values[5].split(":")[1]) % 1000;
````
Divide los valores que llegan por ":" y toma el valor numérico.

**d)** Verificar checksum
````.js
const calcCHK = Math.abs(x) + Math.abs(y) + btnA + btnB;
if (calcCHK !== CHK)
throw new ParseError("Checksum mismatch");
````
Hace una suma entre todos los valores a la que se le llama Checksum, si es igual al Check original entonces está bien, de lo contrario es un error y se muestra en pantalla como: "Checksum mismatch".

**e)** Validaciones extra
````.js
  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (![1, 0].includes(btnA) || ![1, 0].includes(btnB)) throw new ParseError("Invalid button data");
````
Se verifica que sean números válidos. que estén en rango esperado y que los botones sean 0 o 1, de lo contrario se muestra en pantalla su respectivo error.

**f)** Resultado final
````.js
return { x: x | 0, y: y | 0, btnA: btnA === 1, btnB: btnB === 1 };
````
Convierte: x, y en enteros y los botones en booleanos.

**Código del microbit**
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
