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

```python
import cv2
import numpy as np
import sys

height_slider_max = 100
decay_slider_max = 100
offset_slider_max = 100

height_slider_name = "Region size (%)"
decay_slider_name = "Decay (%)"
offset_slider_name = "Center height (%)"

filename = sys.argv[1]
original = cv2.imread(filename)
M, N = original.shape[:2]

blur = np.ones((5, 5)) / 25
filtered = cv2.filter2D(original, -1, blur)

mask = np.ones_like(original, dtype=float)
mesh_horiz, mesh_vert = np.meshgrid(np.arange(M), np.arange(N))

window_name = "tiltshift"
cv2.namedWindow(window_name, cv2.WINDOW_AUTOSIZE)

def alpha(x, d, l1, l2):
    return 0.5 * (np.tanh((x - l1) / d) - np.tanh((x - l2) / d))

def on_change(val):
    height = cv2.getTrackbarPos(height_slider_name, window_name)
    decay = cv2.getTrackbarPos(decay_slider_name, window_name)
    offset = cv2.getTrackbarPos(offset_slider_name, window_name)

    d = decay * M / 100
    l1 = (offset - height / 2) * M / 100
    l2 = (offset + height / 2) * M / 100
    mask = alpha(mesh_vert, d, l1, l2)
    mask = np.atleast_3d(mask)

    blended = mask * original + (1 - mask) * filtered
    blended = np.array(blended, np.uint8)
    cv2.imshow(window_name, blended)

cv2.createTrackbar(height_slider_name, window_name, 0,
                   height_slider_max, on_change)

cv2.createTrackbar(decay_slider_name, window_name, 0,
                   decay_slider_max, on_change)

cv2.createTrackbar(offset_slider_name, window_name, 0,
                   offset_slider_max, on_change)

on_change(0)
cv2.waitKey(0)
```

### Análise de vídeo

