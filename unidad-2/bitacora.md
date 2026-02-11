# Unidad 2

## Bitácora de proceso de aprendizaje
### Actividad 2
````
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)
class Semaforo:
    def __init__(self,_x,_y):
        self.redX = _x
        self.redY = _y
        self.event_queue = []
        self.timers = []
        self.myTimer= self.createTimer("Timeout",2000)
        self.estado_actual = None
        self.transicion_a(self.estado_WaitRed)
        
    def createTimer(self,event,duration):
            t = Timer(self, event, duration)
            self.timers.append(t)
            return t
        
    
    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def estado_WaitRed(self, ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY,9)
            self.myTimer.start(5000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY,0)
            self.transicion_a(self.estado_WaitGreen)
    
    def estado_WaitGreen(self, ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+2,9)
            self.myTimer.start(3000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)
    
    def estado_WaitYellow(self, ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+1,9)
            self.myTimer.start(500)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+1,0)
            self.transicion_a(self.estado_WaitRed)



semaforo1= Semaforo(2,1)

while True:
    semaforo1.update()
    utime.sleep_ms(20)
  ````

Código de semaforo nocturno
````
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)
class Semaforo:
    def __init__(self,_x,_y):
        self.redX = _x
        self.redY = _y
        self.event_queue = []
        self.timers = []
        self.myTimer= self.createTimer("Timeout",2000)
        self.estado_actual = None
        self.transicion_a(self.estado_WaitRed)
        
    def createTimer(self,event,duration):
            t = Timer(self, event, duration)
            self.timers.append(t)
            return t
        
    
    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def estado_WaitRed(self, ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY,9)
            self.myTimer.start(2000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY,0)
            self.transicion_a(self.estado_WaitGreen)
        if ev == "S":
            display.set_pixel(self.redX,self.redY,0)
            self.myTimer.stop()
            self.transicion_a(self.estado_WaitNightOn)
            
    
    def estado_WaitGreen(self, ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+2,9)
            self.myTimer.start(1000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)
        if ev == "A": #evento A
            self.myTimer.stop()
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)
        if ev == "S":
            display.set_pixel(self.redX,self.redY+2,0)
            self.myTimer.stop()
            self.transicion_a(self.estado_WaitNightOn)
    
    def estado_WaitYellow(self, ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+1,9)
            self.myTimer.start(500)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+1,0)
            self.transicion_a(self.estado_WaitRed)
        if ev == "S":
            display.set_pixel(self.redX,self.redY+1,0)
            self.myTimer.stop()
            self.transicion_a(self.estado_WaitNightOn)

    def estado_WaitNightOn(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+1,9)
            self.myTimer.start(1000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+1,0)
            self.transicion_a(self.estado_WaitNightOff)
            
    def estado_WaitNightOff(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+1,0)
            self.myTimer.start(1000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+1,9)
            self.transicion_a(self.estado_WaitNightOn)

            



semaforo1= Semaforo(2,1)

while True:

    
    if button_a.was_pressed(): #evento A
        semaforo1.post_event("A") #evento A

    if accelerometer.was_gesture("shake"):
        semaforo1.post_event("S")
    
    semaforo1.update()
    utime.sleep_ms(20)
````

## Bitácora de aplicación 
### Actividad 04
````
from microbit import *
import utime
import music


def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()
# Para mostrar usas display.show(FILL[n]) donde n será
# un valor de 0 a 25

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)
                
class Task:
    def __init__(self):
        self.event_queue = []
        self.timers = []
        # Personalizas el nombre del evento y la duración
        self.myTimer = self.createTimer("Timeout",1000)
        self.counter = 20
        self.estado_actual = None
        self.transicion_a(self.estado_config)

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def estado_config(self, ev):
        if ev == "ENTRY":
           self.counter = 20
           display.show(FILL[self.counter])
            
        if ev == "A":
            if self.counter<25:
              self.counter = self.counter + 1
              display.show(FILL[self.counter])     
        
        if ev == "B":
           if self.counter>15:
              self.counter = self.counter - 1
              display.show(FILL[self.counter])    
           
        if ev == "S":
            self.transicion_a(self.estado_timerStart)

    def estado_timerStart(self, ev):
        if ev == "ENTRY":
            self.myTimer.start(1000)
        if ev == "Timeout":
            if self.counter > 0: 
               self.counter = self.counter - 1
               display.show(FILL[self.counter])
               if self.counter == 0:     
                   self.transicion_a(self.estado_timerFinish)
               else:
                    self.myTimer.start(1000)
                   
    def estado_timerFinish(self, ev):
        if ev == "ENTRY":
            display.show(Image.SKULL)
            music.play(music.PRELUDE)
            
        if ev == "A":
           self.transicion_a(self.estado_config)        
                

task = Task()
while True:
    # Aquí generas los eventos de los botones y el gesto
    if button_a.was_pressed():
        task.post_event("A")
    if button_b.was_pressed():
        task.post_event("B")
    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
    utime.sleep_ms(20)

````
Código del diagrama
````
@startuml

title Game - UML State Machine

[*] --> estado_config : Game() (constructor)
estado_config --> estado_TimerStart : S 
estado_config: entry /\n self.counter = 20 \n if ev == "A": /\n self.counter + 1 \n if ev == "B": /\n self.counter - 1
estado_TimerStart --> estado_TimerFinish : Timeout 
estado_TimerStart : entry/\n self.myTimer.start(1000)
estado_TimerFinish --> estado_config : A 
estado_TimerFinish : entry/\n display.show(Image.SKULL) \n music.play(music.PRELUDE) 
@enduml
````

## Bitácora de reflexión






