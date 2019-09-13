---
title: "Manipulando pixels em uma imagem"
date: 2019-08-09
tags: [python, opencv]
category: casual-code
header:
    teaser: /assets/images/placeholder-300x225.png

---

Esse é o primeiro post de uma série de exemplos práticos de processamento de imagens em Python usando OpenCV.
<!--more-->
Veremos como é simples realizar operações sobre arquivos de imagens.
## Negativo de imagens

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

cv2.imshow('imagem', dest)
cv2.waitKey()
```

