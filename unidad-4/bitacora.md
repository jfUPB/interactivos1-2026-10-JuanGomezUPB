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

----
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
----


## Bitácora de reflexión

