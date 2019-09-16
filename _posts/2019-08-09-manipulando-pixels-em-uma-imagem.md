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

p1x, p1y = input('p1: ').split(',')
p1x = int(p1x)
p1y = int(p1y)

p2x, p2y = input('p2: ').split(',')
p2x = int(p2x)
p2y = int(p2y)

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_COLOR)
image[p1x:p2x, p1y:p2y] = 255 - image[p1x:p2x, p1y:p2y]
cv2.imshow('negativo', image)
cv2.waitKey()
```

### Versão interativa

O OpenCV  

```python
import cv2
import sys
import numpy as np

drawing = False
xo, yo = -1, -1

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_COLOR)
img = image.copy()

def invert(event, x, y, flags, param):
    global drawing, xo, yo, img
    if event == cv2.EVENT_LBUTTONDOWN:
        drawing = True
        xo, yo = x, y
    elif event == cv2.EVENT_MOUSEMOVE:
        if drawing == True:
            xi = min(xo, x)
            xj = max(xo, x)
            yi = min(yo, y)
            yj = max(yo, y)
            img[yi:yj, xi:xj] = 255 - image[yi:yj, xi:xj]
    elif event == cv2.EVENT_LBUTTONUP:
        drawing = False
    elif event == cv2.EVENT_RBUTTONDOWN:
        drawing = False
        img = image.copy()

cv2.namedWindow('regions')
cv2.setMouseCallback('regions', invert)

while(1):
    cv2.imshow('regions', img)
    if cv2.waitKey(20) & 0xFF == 27:
        break
cv2.destroyAllWindows()

```

<div style="text-align:center"><img src="/assets/gifs/regions2.gif" /></div>

## Manipulação de quadrantes

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
