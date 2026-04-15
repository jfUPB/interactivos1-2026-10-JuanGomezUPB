# Unidad 6

## Bitácora de proceso de aprendizaje

## Bitácora de reflexión

**Cambios al código original**

**1.** ***bridgeServer.js***
.js
adapter.handleMessage(message);

adapter.onData = (data) => {
    client.send(JSON.stringify(data));
};

Antes el bridge parseaba el mensaje y lo reenviaba directamente, es decir no solo enviaba el mensaje sino que además pensaba. Ahora delega el procesamiento al Adapter y se limita a reenviar datos, eso sí ya normalizados, manteniendo que no debe "pensar".

**2.** ***StrudelAdapter.js***
.js
const msg = JSON.parse(rawMsg);

const parsed = {
  type: "strudel",
  timestamp: Math.floor(msg.timestamp),
  payload: {
    sound: this.normalizeSound(argsObj.s),
    delta: argsObj.delta || 0
  }
};

this.onData?.(parsed);

El Adapter toma el formato original que envía Strudel (como los args) y convertirlo en un objeto más claro y fácil de usar. Esto se hace para que el resto del sistema no tenga que entender ese formato más complejo y pueda trabajar con datos simples y consistentes.

**3.** ***sketch (frontend)***

a)
.js
if (msg.type === "strudel") {
    eventQueue.push({
        timestamp: msg.timestamp,
        sound: msg.payload.sound,
        delta: msg.payload.delta
    });
}
El frontend deja de interpretar el mensaje crudo (msg.args). En su lugar va a recibir e interpretar el mensaje que le llegue desde el Adapter. Esto simplifica la lógica en la capa visual.

b) 
.js
case 'bd':
case 'sd':
case 'hh':

Los identificadores de sonido se simplifican porque el Adapter ya realizó la normalización. Los nombres como tr909bd ahora son reemplzados por bd, sd o hh. De necesitarse más sonidos solo se añadirían a estos casos. dado que ahora la aplicación funciona para canciones que reproduzcan estos sonidos ("bd","sd","hh") porque de lo contrario no los va a reconocer.

c)
.js
const colors = {
  'bd': [255, 0, 80],
  'sd': [0, 200, 255],
  'hh': [255, 255, 0]
};

Se le va a asignar un color a cada sonido. 

## Bitácora de aplicación 


