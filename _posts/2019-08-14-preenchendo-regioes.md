---
title: "Preenchendo regiões"
date: 2019-08-14
tags: [python, opencv]
category: casual-code
excerpt: Veremos como utilizar o algoritmo flood-fill para rotular regiões conexas em uma imagem.
header:
    teaser: /assets/images/bolhas.png
---
<!--more-->

```python
import cv2
import sys

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_GRAYSCALE)

M, N = image.shape
```

![](/assets/images/bolhas.png)

```python
for i in range(M):
    for j in [0, N-1]:
        if image[i, j] == 255:
            cv2.floodFill(image, None, (j, i), 0)
for i in [0, M-1]:
    for j in range(N):
        if image[i, j] == 255:
            cv2.floodFill(image, None, (j, i), 0)
```

![](/assets/images/passo1.png)

```python
nobjects = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 255:
            nobjects += 1
            cv2.floodFill(image, None, (j, i), nobjects)
```

```python
cv2.floodFill(image, None, (0, 0), 255)
```

![](/assets/images/passo2.png)

```python
nburacos = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 0:
            if 0 < image[i, j-1] < 255:
                nburacos += 1
                cv2.floodFill(image, None, (j-1, i), 255)
            cv2.floodFill(image, None, (j, i), 255)
```

![](/assets/images/labeling.png)

```python
import cv2
import sys

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_GRAYSCALE)

M, N = image.shape

# remover as bolhas que tocam as bordas da imagem
for i in range(M):
    for j in [0, N-1]:
        if image[i, j] == 255:
            cv2.floodFill(image, None, (j, i), 0)
for i in [0, M-1]:
    for j in range(N):
        if image[i, j] == 255:
            cv2.floodFill(image, None, (j, i), 0)
            
# contar todas as bolhas
nobjects = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 255:
            nobjects += 1
            cv2.floodFill(image, None, (j, i), nobjects)

# background branco
cv2.floodFill(image, None, (0, 0), 255)

# contar as bolhas com buracos
nburacos = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 0:
            if 0 < image[i, j-1] < 255:
                nburacos += 1
                cv2.floodFill(image, None, (j-1, i), 255)
            cv2.floodFill(image, None, (j, i), 255)

print('TOTAL DE BOLHAS:', nobjects)
print('    COM BURACOS:', nburacos)
print('    SEM BURACOS:', nobjects-nburacos)

cv2.imwrite('labeling.png', image)
cv2.waitKey()
```

Saída:

```console
foo@bar:~$ python3 labeling.py bolhas.png
TOTAL DE BOLHAS: 21
    COM BURACOS: 7
    SEM BURACOS: 14
```
