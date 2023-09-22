---
layout: post
title: Practica 1 - Basic Vacuum Cleaner
---

# Primer contacto

Al ser la primera vez escribiendo codigo en python para controlar un robot usando ros, y al no conocer el API, el
primer modelo  que hice fue lo siguiente:

- Va recto hasta que se encuentra un objeto a 1 metro.
- Para, imprime al terminal STOP y sale del programa.

![Primera prueba gif](../images/primera_prueba.gif)

```python
from GUI import GUI
from HAL import HAL

# Enter sequential code!

while True:
    # Enter iterative code!

    print(HAL.getLaserData().values[90])
    if HAL.getLaserData().values[90] > 1 :
        HAL.setV(3)
    else :
      print("STOP")
      HAL.setV(0)
      exit(10)
```

# Implementado maquinas de estado

Para este segundo modelo he creado una máquina de estados sencilla con dos estados: hacia delante y giro.

Por defecto empieza hacia delante y cuando encuentra un objeto delante a menos de 0.5 metros gira durante 3 segundos para luego volver a ir recto.

![Segunda prueba](../images/segunda_prueba.gif)

```python
from GUI import GUI
from HAL import HAL
import datetime

FORWARD=0
TURN=1

current_state = FORWARD

def is_object_near(value):
    return value < 0.5
    
def has_time_finished(start, now):
    after_3_sec = start + datetime.timedelta(seconds = 3)
    return now > after_3_sec

while True:

    if current_state == FORWARD:
        HAL.setV(3)
        HAL.setW(0)
        if is_object_near(HAL.getLaserData().values[90]):
            current_state = TURN
            start_turn = datetime.datetime.now()

    elif current_state == TURN:
        HAL.setV(0)
        HAL.setW(3)
        if has_time_finished(start_turn, datetime.datetime.now()):
          current_state = FORWARD

```

# Implementando aleatoriedad y espirales

En este modelo he aumentado la complejidad de la máquina de estados con un estado más, el del movimiento en espiral.

También en esta iteración he añadido aleatoriedad al sentido de los giros usando random

![Tercer modelo](../images/tercera_prueba.gif)

```python
from GUI import GUI
from HAL import HAL
import datetime
import random

FORWARD=0
TURN=1
SPIRAL=2

turn_direction = 1

spiral_direction = 1
spiral_lin_speed_base = 1.5
spiral_lin_speed = spiral_lin_speed_base
spiral_lin_speed_increment = 0.1

current_state = FORWARD

def is_object_near(laser_data):
    # Iterate in a range of 15 around the center
    for i in range(30):
        if laser_data.values[75 + i] < 0.5:
            return True
    return False
    
def has_time_finished(start, now, delay):
    after_delay = start + datetime.timedelta(seconds = delay)
    return now > after_delay

def go_to_turn():
    global current_state, TURN, turn_direction, start_turn
    current_state = TURN
    turn_direction = random.choice([-1,1])
    start_turn = datetime.datetime.now()
    
def go_to_spiral():
    global current_state, SPIRAL, spiral_direction, \
        spiral_lin_speed, spiral_lin_speed_base
    
    current_state = SPIRAL
    spiral_lin_speed = spiral_lin_speed_base
    spiral_direction = random.choice([-1,1])

while True:

    if current_state == FORWARD:
        HAL.setV(3)
        HAL.setW(0)
        if is_object_near(HAL.getLaserData()):
            go_to_turn()
    elif current_state == TURN:
        HAL.setV(0)
        HAL.setW(3 * turn_direction)
        if has_time_finished(start_turn, datetime.datetime.now(), 3):
          go_to_spiral()
    elif current_state == SPIRAL:
        spiral_lin_speed = spiral_lin_speed + spiral_lin_speed_increment
        HAL.setV(spiral_lin_speed)
        HAL.setW(3 * spiral_direction)
        if has_time_finished(start_turn, datetime.datetime.now(), 10):
          spiral_lin_speed = spiral_lin_speed_base
          current_state = FORWARD
        elif is_object_near(HAL.getLaserData()):
            go_to_turn()
```