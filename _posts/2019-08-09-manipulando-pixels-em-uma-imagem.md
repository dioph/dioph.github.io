---
title: "Manipulando pixels em uma imagem"
date: 2019-08-09
tags: [python, opencv]
category: casual-code
header:
    teaser: /assets/images/regions.png
---

Esse é o primeiro post de uma série de exemplos práticos de processamento de imagens em Python usando OpenCV.<!--more--> Veremos como é simples realizar operações sobre arquivos de imagens.

## Negativo de imagens

$$g(x,y)=255-f(x,y)$$

```python
import cv2
import sys

xi, yi = input('pi: ').split(',')
xi = int(xi)
yi = int(yi)

xj, yj = input('pj: ').split(',')
xj = int(xj)
yj = int(yj)

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_COLOR)
image[xi:xj, yi:yj] = 255 - image[xi:xj, yi:yj]
cv2.imshow('negative', image)
cv2.waitKey()
```

```shell
pi: 100, 100
pj: 200, 200
```

![](/assets/images/regions.png)

### Versão interativa

O OpenCV  

```python
import cv2
import sys

filename = sys.argv[1]
original = cv2.imread(filename, cv2.IMREAD_COLOR)
negative = original.copy()

drawing = False
xo, yo = -1, -1

def invert(event, x, y, flags, param):
    global drawing, xo, yo, negative
    if event == cv2.EVENT_LBUTTONDOWN:
        drawing = True
        xo, yo = x, y
    elif event == cv2.EVENT_MOUSEMOVE:
        if drawing:
            xi, xj = sorted([xo, x])
            yi, yj = sorted([yo, y])
            negative[yi:yj, xi:xj] = 255 - original[yi:yj, xi:xj]
    elif event == cv2.EVENT_LBUTTONUP:
        drawing = False
    elif event == cv2.EVENT_RBUTTONDOWN:
        drawing = False
        negative = original.copy()

cv2.namedWindow('negative')
cv2.setMouseCallback('negative', invert)

while(1):
    cv2.imshow('negative', negative)
    if cv2.waitKey(20) & 0xFF == 27:
        break

```

![](/assets/images/regions2.gif)

## Regiões de Interesse

```python
import cv2
import numpy as np
import sys

filename = sys.argv[1]
orig = cv2.imread(filename, cv2.IMREAD_COLOR)
N, M = orig.shape[:2]

dest = np.zeros_like(orig)
dest[:N//2, :M//2] = orig[N//2:, M//2:]
dest[:N//2, M//2:] = orig[N//2:, :M//2]
dest[N//2:, :M//2] = orig[:N//2, M//2:]
dest[N//2:, M//2:] = orig[:N//2, :M//2]

cv2.imshow('swapped', dest)
cv2.waitKey()
```
![](/assets/images/trocaregioes.png)
