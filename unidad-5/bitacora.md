# Unidad 5
## Bitácora de proceso de aprendizaje
- ¿Qué es una **línea de empaquetado**? --> Línea 11 en el caso de la Actividad 1, donde se "guardan" o *empaquetan* los datos.
  
```` .py
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

while True:
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = button_a.is_pressed()
    bState = button_b.is_pressed()
    data = "{},{},{},{}\n".format(xValue, yValue, aState,bState)
    uart.write(data)
    sleep(100)
````
- Toda información se transmite en bytes, en este caso la info está transmitida en ASCII --> Ver *Aplicación de conexión serial*
- Para interpretar los datos, ver **tabla ASCII** --> https://juanferfranco.github.io/serialTerminal/ *una vez en el link traducir a texto*
- Se están mandando bytes, NO carácteres.
**HASTA ACÁ LO RELACIONADO CON LA UNIDAD 4**

  - *A tener en cuenta con los bytes*
    --> HEX 237 es igual a dos bytes: 02 37
  - Ya que trabajamos con binario y no trabajmos con ASCII es importante recoger el valor
  - **BIG ENDIAN**: primero se transmite el byte de mayor peso --> *nosotros vamos a transmitir en este*
      xValue = 567 --> 02 37
  - **LITTLE ENDIAN**: primero se transmite el byte de menor peso --> *esta es más común*
      xValue = 567 --> 14.082
    - Si es Big Endian o Little Endian, se le especifica a uno.
  - FFFF en BIG ENDIAN no va a ser 65.000 sino -1
  - aa: header (para saber donde empieza)
  - El checksum debe ser de un solo byte (carácter 0 a 255) --> el checksum verifica que el paquete este bien, por ejemplo, que el aa no se haya colado por ninguna parte.
## Bitácora de aplicación 
*Copié el códido de la unidad 4 y seguí a partir de ahí, porque los cambios no son estructurales sino de como se leen los datos*
````.js
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error {}

function parseBinaryPacket(buf) {
  if (buf.length !== 8) throw new ParseError("Invalid packet length");

  // Byte 0: header
  if (buf[0] !== 0xAA) throw new ParseError("Invalid header");

  // Datos
  const x = buf.readInt16BE(1);
  const y = buf.readInt16BE(3);
  const btnA = buf[5];
  const btnB = buf[6];
  const chk = buf[7];

  // Calcular checksum (bytes 1 a 6)
  let calcCHK = 0;
  for (let i = 1; i <= 6; i++) {
    calcCHK = (calcCHK + buf[i]) % 256;
  }

  if (calcCHK !== chk) throw new ParseError("Checksum mismatch");

  // Validaciones
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047)
    throw new ParseError("Out of range");

  if (![0, 1].includes(btnA) || ![0, 1].includes(btnB))
    throw new ParseError("Invalid button data");

  return {
    x,
    y,
    btnA: btnA === 1,
    btnB: btnB === 1,
  };
}

class MicrobitBinary2Adapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required");

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
        this.port.close((err) => (err ? reject(err) : resolve()));
      });
    }

    this.port = null;
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed");
  }

  _onChunk(chunk) {
    // Acumular bytes
    this.buf = Buffer.concat([this.buf, chunk]);

    while (this.buf.length >= 8) {
      // Buscar header 0xAA
      const start = this.buf.indexOf(0xAA);
      if (start < 0) {
        this.buf = Buffer.alloc(0);
        return;
      }

      // Si no hay suficientes bytes aún
      if (this.buf.length < start + 8) return;

      const packet = this.buf.slice(start, start + 8);
      this.buf = this.buf.slice(start + 8);

      try {
        const parsed = parseBinaryPacket(packet);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (this.verbose) console.log("Bad packet:", e.message);
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 4096) this.buf = Buffer.alloc(0);
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = Buffer.alloc(0);
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

module.exports = MicrobitBinary2Adapter;
````

Las diferencias con el código ASCII son las siguientes:
1.  Ya no se convierte a texto, sino que se acumulan buffers binarios reales.
````.js
this.buf = Buffer.concat([this.buf, chunk]);
````
2. Ya no se usan strings. Los datos se leen por la posición en la que llegan:
   - El primer byte es el **header (AA)**
   - El segundo y tercer bytes son la **x**
   - El cuarto y quinto son la **y**
   - El quinto es el **botón A**
   - El sexto es el **botón B**
   - El séptimo y ultimo es el **checksum**
3. Cada paquete inicia con un header, represntado por "AA".
````.js
const start = this.buf.indexOf(0xAA);
````
4. Para verificar, el checksum suma los bytes directamente y no los valores como se hacía en el ASCII.
````.js
  // Calcular checksum (bytes 1 a 6)
  let calcCHK = 0;
  for (let i = 1; i <= 6; i++) {
    calcCHK = (calcCHK + buf[i]) % 256;
  }
````


## Bitácora de reflexión
