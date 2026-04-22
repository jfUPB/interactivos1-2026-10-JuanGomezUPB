# Unidad 7

## Bitácora de proceso de aprendizaje
**Superficies de control**
- hexler.net/touchosc --> *de pago*
- Open Stage Control --> La que vamos a utilizar

**Backend**
- BridgeServer.js --> Se le ponen unos puertos: **Adapters**
- Adapters --> No importa la tecnología, se debe poder leer
- La fase de **bridge** hace la traducción
- En la **unidad 6** se conecto STRUDEL al adapter: STRUDEL --> Adapter
- En la **unidad 7** ser va a poder tener activos dos adapters.
- El segundo adapter será el de Open Stage Control --> Va  a permitir controlar en tiempo real los parámetros de Open Stage Control
**Frontend**
- Visuales --> p5.js *en este caso*

### Actividad 1
- https://github.com/juanferfranco/openStageControl-sfi1-2026-10
- Protocolo udp: comúnica dos aplicaciones
- No se pueden utilizar los mismos puertos incluso si son protocolos distintos (8080, 8081, 8081, 9000)
- Node birdge.js y node bridgeOSC.js respectivamente
  *OpenStageController*
- Se abre opennstagecontrol (launcher) --> lanza automaticamente un servidor y un cliente (aplicaciones locales corriendo en el computador)
- send: dirección local (en este caso debe terminart en 9000) --> el puerto donde el bridge esta escuchando.
- port: 8086 en este caso
- Se lanza una configuración..> en este caso launcher.config

## Bitácora de aplicación 



## Bitácora de reflexión
