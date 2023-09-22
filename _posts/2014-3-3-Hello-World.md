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

El código es:
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