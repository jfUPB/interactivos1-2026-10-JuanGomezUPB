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
## Bitácora de reflexión
