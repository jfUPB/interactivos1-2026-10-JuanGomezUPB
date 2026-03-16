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


## Bitácora de reflexión
