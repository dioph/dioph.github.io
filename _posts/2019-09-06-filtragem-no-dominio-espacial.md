---
title: "Filtragem no domínio espacial"
date: 2019-09-06
tags: [python, opencv]
category: casual-code
---
<!--more-->

## Convolução digital

```python
import cv2
import numpy as np

def menu():
    print("\npress a key to toggle a filter:"
          "\na - absolute value"
          "\nm - mean"
          "\ng - gaussian"
          "\nv - vertical"
          "\nh - horizontal"
          "\nl - laplacian"
          "\nx - reset all"
          "\nesc - exit")
  
cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'

media = np.array([[1, 1, 1], [1, 1, 1], [1, 1, 1]]) / 9
gauss = np.array([[1, 2, 1], [2, 4, 2], [1, 2, 1]]) / 16
horiz = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]])
verti = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]])
lapla = np.array([[0, -1, 0], [-1, 4, -1], [0, -1, 0]])

masks = dict(m=media, g=gauss, h=horiz, v=verti, l=lapla)
activ = dict(a=False, m=False, g=False, h=False, v=False, l=False)

menu()
while True:
    _, image = cap.read()
    frame = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    frame = cv2.flip(frame, 1)
    cv2.imshow('original', frame)
    
    for key in masks.keys():
        if activ[key]:
            frame = cv2.filter2D(frame, -1, masks[key])
    if activ['a']:
        frame = np.abs(frame)
    cv2.imshow('filtered', frame)
    
    key = cv2.waitKey(10)
    if key > 0:
        key = chr(key)

    if key == chr(27):
        break
    if key == 'x':
        menu()
        activ = dict.fromkeys(activ, False)
    if key in activ.keys():
        menu()
        activ[key] = not activ[key]
        print([k for k in masks.keys() if activ[k]])
```

## Tilt-Shift
