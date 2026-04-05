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
import { BaseAdapter } from './BaseAdapter.js';

export class MicrobitBinaryAdapter extends BaseAdapter {
  constructor() {
    super();
    this.buffer = Buffer.alloc(0);
  }

  onSerialData(data) {
    // Acumular bytes
    this.buffer = Buffer.concat([this.buffer, data]);

    // Procesar mientras haya suficiente data
    while (this.buffer.length >= 8) {

      // Buscar header 0xAA
      let startIndex = this.buffer.indexOf(0xAA);

      if (startIndex === -1) {
        // No hay header → limpiar buffer
        this.buffer = Buffer.alloc(0);
        return;
      }

      // Si no hay suficientes bytes después del header, esperar
      if (this.buffer.length < startIndex + 8) {
        return;
      }

      // Extraer paquete de 8 bytes
      let packet = this.buffer.slice(startIndex, startIndex + 8);

      // Eliminar lo procesado del buffer
      this.buffer = this.buffer.slice(startIndex + 8);

      this.processPacket(packet);
    }
  }

  processPacket(packet) {
    try {
      // Verificar header
      if (packet[0] !== 0xAA) return;

      // Leer valores (Big Endian)
      let x = packet.readInt16BE(1);
      let y = packet.readInt16BE(3);

      let btnA = packet[5] === 1;
      let btnB = packet[6] === 1;

      let receivedChk = packet[7];

      // Calcular checksum
      let calcChk = (
        packet[1] +
        packet[2] +
        packet[3] +
        packet[4] +
        packet[5] +
        packet[6]
      ) % 256;

      if (calcChk !== receivedChk) {
        console.warn('Trama binaria corrupta');
        return;
      }

      // Emitir (MISMO CONTRATO)
      this.onData?.({
        x: x,
        y: y,
        btnA: btnA,
        btnB: btnB
      });

    } catch (error) {
      console.warn('Error procesando paquete binario:', error);
    }
  }
}
````

🧩 Código: constructor
constructor() {
  super();
  this.buffer = Buffer.alloc(0);
}
🔹 super();

👉 Llama al constructor de BaseAdapter
Es obligatorio porque estás heredando.

🔹 this.buffer = Buffer.alloc(0);

👉 Crea un buffer vacío de bytes

Antes era un string ("")
Ahora es un Buffer (porque trabajas con binario)

💡 Es donde vas guardando los bytes que llegan.

🔌 Código: onSerialData
onSerialData(data) {

👉 Esta función se ejecuta cada vez que llegan datos del puerto serial

this.buffer = Buffer.concat([this.buffer, data]);

👉 Une lo que ya tenías con lo nuevo que llegó

Ejemplo:

buffer viejo: [AA 01]
data nuevo:   [F4 02 0C]
resultado:    [AA 01 F4 02 0C]
while (this.buffer.length >= 8) {

👉 Mientras haya al menos 8 bytes (tamaño de un paquete), intenta procesar

let startIndex = this.buffer.indexOf(0xAA);

👉 Busca el byte 0xAA

💡 Ese es el inicio del paquete (header)

if (startIndex === -1) {
  this.buffer = Buffer.alloc(0);
  return;
}

👉 Si NO encuentra 0xAA:

Todo lo que tienes es basura ❌
Limpias el buffer
Paras la función
if (this.buffer.length < startIndex + 8) {
  return;
}

👉 Si aún no tienes los 8 bytes completos:

Esperas más datos
No haces nada todavía
let packet = this.buffer.slice(startIndex, startIndex + 8);

👉 Tomas exactamente 8 bytes desde el header

💡 Eso es UN paquete completo

this.buffer = this.buffer.slice(startIndex + 8);

👉 Eliminas del buffer lo que ya procesaste

this.processPacket(packet);

👉 Envías ese paquete a otra función para interpretarlo

🧠 Código: processPacket
processPacket(packet) {

👉 Aquí conviertes bytes → datos útiles

if (packet[0] !== 0xAA) return;

👉 Verifica que realmente empiece con el header

let x = packet.readInt16BE(1);

👉 Lee 2 bytes desde la posición 1 como número

Int16 = entero de 16 bits
BE = Big Endian

💡 Usa bytes 1 y 2

let y = packet.readInt16BE(3);

👉 Igual que arriba, pero desde posición 3 (bytes 3 y 4)

let btnA = packet[5] === 1;
let btnB = packet[6] === 1;

👉 Lee botones:

1 → true
0 → false
let receivedChk = packet[7];

👉 Último byte = checksum recibido

let calcChk = (
  packet[1] +
  packet[2] +
  packet[3] +
  packet[4] +
  packet[5] +
  packet[6]
) % 256;

👉 Calcula el checksum:

Suma bytes 1 a 6
Hace % 256
if (calcChk !== receivedChk) {
  console.warn('⚠️ Trama binaria corrupta');
  return;
}

👉 Si no coincide:

❌ descartas paquete
⚠️ muestras warning
this.onData?.({
  x: x,
  y: y,
  btnA: btnA,
  btnB: btnB
});

👉 Envías los datos al sistema

💡 Esto es lo más importante → el contrato

} catch (error) {
  console.warn('Error procesando paquete binario:', error);
}

👉 Si algo falla:

No rompe el programa
Solo muestra error
## Bitácora de reflexión
