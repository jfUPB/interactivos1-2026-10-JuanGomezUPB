# Unidad 6

## Bitácora de proceso de aprendizaje

## Bitácora de aplicación

**Cambios al código original**

**1.** ***bridgeServer.js***
````.js
const WebSocket = require('ws');
const osc = require('node-osc');
const StrudelAdapter = require('./StrudelAdapter'); // Importar el StrudelAdapter

// 1. CONFIGURACIÓN
const STRUDEL_PORT = 8080;   // Donde Strudel envía los datos
const P5JS_PORT = 8081;      // Donde p5.js escuchará
const adapter = new StrudelAdapter(); //Se crea el StrudelAdapter

// 2. SERVIDOR WEBSOCKET PARA STRUDEL (Puerto 8080)
const wssStrudel = new WebSocket.Server({ port: STRUDEL_PORT });

// 3. SERVIDOR WEBSOCKET PARA P5.JS (Puerto 8081)
const wssP5 = new WebSocket.Server({ port: P5JS_PORT });

console.log(`Escuchando a Strudel en ws://localhost:${STRUDEL_PORT}`);
console.log(`Transmitiendo a p5.js en ws://localhost:${P5JS_PORT}`);

wssStrudel.on('connection', (ws) => {
    console.log('Strudel conectado al Bridge');

    ws.on('message', (message) => {
        /*
        try {
            const msg = JSON.parse(message);
            // REENVIAR A P5.JS (WebSocket)
            const payload = JSON.stringify(msg);
            
            console.log('Reenviando mensaje a p5.js:', msg);
            wssP5.clients.forEach(client => {
                if (client.readyState === WebSocket.OPEN) {
                    client.send(payload);
                }
            });

        } catch (e) {
            console.error('Error procesando mensaje de Strudel:', e);
        }
            */
        adapter.handleMessage(message);
    });
});

adapter.onData = (data) => {
    const payload = JSON.stringify(data);

    wssP5.clients.forEach(client => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(payload);
        }
    });
};

wssP5.on('connection', (ws) => {
    console.log('p5.js se ha conectado al Bridge');
});
````
**a)**
````.js
adapter.handleMessage(message);

adapter.onData = (data) => {
    client.send(JSON.stringify(data));
};
````
Antes el bridge parseaba el mensaje y lo reenviaba directamente, es decir no solo enviaba el mensaje sino que además pensaba. Ahora delega el procesamiento al Adapter y se limita a reenviar datos, eso sí ya normalizados, manteniendo que no debe "pensar".

**2.** ***StrudelAdapter.js***
const BaseAdapter = require('./BaseAdapter');

class StrudelParseError extends Error {}

class StrudelAdapter extends BaseAdapter {
  constructor({ verbose = false } = {}) {
    super();
    this.verbose = verbose;
  }

  async connect() {
    this.connected = true;
    this.onConnected?.("strudel connected");
  }

  async disconnect() {
    this.connected = false;
    this.onDisconnected?.("strudel disconnected");
  }

  handleMessage(rawMsg) {
    try {
      const msg = JSON.parse(rawMsg);

      const parsed = this.parseStrudelMessage(msg);

      this.onData?.(parsed);

    } catch (e) {
      if (e instanceof StrudelParseError) {
        if (this.verbose) console.log("Bad Strudel message:", e.message);
      } else {
        this._fail(e);
      }
    }
  }

  parseStrudelMessage(msg) {
    // 🔹 Validaciones (igual que en U5)
    if (!msg.args || !msg.timestamp) {
      throw new StrudelParseError("Invalid message structure");
    }

    // 🔹 Convertir args → objeto
    const argsObj = {};
    for (let i = 0; i < msg.args.length; i += 2) {
      const key = msg.args[i];
      const value = msg.args[i + 1];
      argsObj[key] = value;
    }

    // 🔹 Validar datos importantes
    if (!argsObj.s) {
      throw new StrudelParseError("Missing sound (s)");
    }

    // 🔹 Normalización (CLAVE)
    return {
      type: "strudel",
      timestamp: Math.floor(msg.timestamp),
      payload: {
        eventType: "noteEvent",
        sound: this.normalizeSound(argsObj.s),
        delta: argsObj.delta || 0
      }
    };
  }

  normalizeSound(soundString) {
    if (!soundString) return "unknown";

    if (soundString.includes("bd")) return "bd";
    if (soundString.includes("sd") || soundString.includes("cp")) return "sd";
    if (soundString.includes("hh")) return "hh";

    return "other";
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
  }
}

module.exports = StrudelAdapter;

````.js
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
````
El Adapter toma el formato original que envía Strudel (como los args) y convertirlo en un objeto más claro y fácil de usar. Esto se hace para que el resto del sistema no tenga que entender ese formato más complejo y pueda trabajar con datos simples y consistentes.

**3.** ***sketch (frontend)***
````.js
  let eventQueue = [];
  let activeAnimations = []; // Animaciones que se están dibujando ahora
  const LATENCY_CORRECTION = 0; // Ajuste fino en ms (ej. 50 si el audio va lento)

  function setup() {
    createCanvas(windowWidth, windowHeight);
    rectMode(CENTER);
    noStroke();

    const socket = new WebSocket('ws://localhost:8081');

    socket.onmessage = (event) => {
        const msg = JSON.parse(event.data);

        console.log('Mensaje OSC recibido:', msg);
        /*
        let params = {};
        
        for (let i = 0; i < msg.args.length; i += 2) {
            params[msg.args[i]] = msg.args[i+1];
        }
        eventQueue.push({ 
          timestamp: msg.timestamp, 
          sound: params.s,
          delta: params.delta || 0.25,
          params: params });

        eventQueue.sort((a, b) => a.timestamp - b.timestamp);
        */
        if (msg.type === "strudel") {
          eventQueue.push({
              timestamp: msg.timestamp,
              sound: msg.payload.sound,
              delta: msg.payload.delta || 0.25
          });
      
          eventQueue.sort((a, b) => a.timestamp - b.timestamp);
      }
    };
  }


  function draw() {
    background(0, 30); 

    let now = Date.now() + LATENCY_CORRECTION;

    while (eventQueue.length > 0 && now >= eventQueue[0].timestamp) {
        let ev = eventQueue.shift();

        activeAnimations.push({
        startTime: ev.timestamp,
        duration: ev.delta * 1000, // Convertimos delta a milisegundos
        type: ev.sound,
        x: random(width * 0.2, width * 0.8), 
        y: random(height * 0.2, height * 0.8),
        color: getColorForSound(ev.sound)
        })        

    }
    
    for (let i = activeAnimations.length - 1; i >= 0; i--) {
      let anim = activeAnimations[i];
      
      // Calculamos el progreso rítmico (0.0 al inicio, 1.0 al final del delta)
      let elapsed = now - anim.startTime;
      let progress = elapsed / anim.duration;

      if (progress <= 1.0) {
        dibujarElemento(anim, progress);
      } else {
        // La animación terminó su ciclo rítmico, la eliminamos
        activeAnimations.splice(i, 1);
      }
    }
  }

  function dibujarElemento(anim, p) {
    push();
    const color = anim.color;
    
    switch (anim.type) {
      case 'bd':
        dibujarBombo(p, color);
        break;
    
      case 'sd':
        dibujarCaja(p, color);
        break;
    
      case 'hh':
        dibujarHat(anim, p, color);
        break;
    
      default:
        dibujarDefault(anim, p, color);
        break;
    }
    pop();
  }


  function dibujarBombo(p, c) {
    let d = lerp(100, 600, p);
    let alpha = lerp(255, 0, p);
    fill(c[0], c[1], c[2], alpha);
    circle(width / 2, height / 2, d);
  }

  function dibujarCaja(p, c) {
    let w = lerp(width, 0, p);
    let alpha = lerp(255, 0, p);
    fill(c[0], c[1], c[2], alpha);
    rect(width / 2, height / 2, w, 50);
  }

  function dibujarHat(anim, p, c) {
    let sz = lerp(40, 0, p);
    fill(c[0], c[1], c[2]);
    rect(anim.x, anim.y, sz, sz);
  }

  function dibujarDefault(anim, p, c) {
    // Un rombo que gira en una posición aleatoria (fijada al nacer la animación)
    // Usamos el progreso 'p' para la rotación y el tamaño
    let size = lerp(100, 0, p);
    let angle = p * TWO_PI;

    translate(anim.x, anim.y);
    rotate(angle);
    
    stroke(c[0], c[1], c[2]);
    strokeWeight(2);
    noFill();
    
    // Dibujamos un rombo o estrella simple
    rect(0, 0, size, size);
    line(-size, 0, size, 0);
    line(0, -size, 0, size);
    
    // Mostrar el nombre del sonido pequeño (opcional, útil para debug)
    noStroke();
    fill(255, 150);
    textSize(20);
    text(anim.type, 10, 10);
  }

  function getColorForSound(s) {
    const colors = {
      'bd': [255, 0, 80],
      'sd': [0, 200, 255],
      'hh': [255, 255, 0]
    };

    if (colors[s]) return colors[s];

    // Si el sonido no existe, generamos un color aleatorio pero consistente
    // basado en la primera letra del nombre del sonido
    let charCode = s.charCodeAt(0) || 0;
    let r = (charCode * 123) % 255;
    let g = (charCode * 456) % 255;
    let b = (charCode * 789) % 255;
    return [r, g, b];
  }  

  function windowResized() { resizeCanvas(windowWidth, windowHeight); }
````  
**a)**
````.js
if (msg.type === "strudel") {
    eventQueue.push({
        timestamp: msg.timestamp,
        sound: msg.payload.sound,
        delta: msg.payload.delta
    });
}
````
El frontend deja de interpretar el mensaje crudo (msg.args). En su lugar va a recibir e interpretar el mensaje que le llegue desde el Adapter. Esto simplifica la lógica en la capa visual.

**b)** 
````.js
case 'bd':
case 'sd':
case 'hh':
````
Los identificadores de sonido se simplifican porque el Adapter ya realizó la normalización. Los nombres como tr909bd ahora son reemplzados por bd, sd o hh. De necesitarse más sonidos solo se añadirían a estos casos. dado que ahora la aplicación funciona para canciones que reproduzcan estos sonidos ("bd","sd","hh") porque de lo contrario no los va a reconocer.

**c)**
````.js
const colors = {
  'bd': [255, 0, 80],
  'sd': [0, 200, 255],
  'hh': [255, 255, 0]
};
````
Se le va a asignar un color a cada sonido. 

## Bitácora de reflexión 


