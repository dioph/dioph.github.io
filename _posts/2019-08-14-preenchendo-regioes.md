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

<p align="center">
    <img src="../../assets/images/bolhas.png">
</p>

Não estamos interessados em bolhas tocando as bordas da imagem, uma vez que não podemos determinar com certeza se elas apresentam buracos ou não. A primeira coisa a se fazer será portanto preencher todos os objetos na borda da imagem com o tom de fundo `0`. Em outras palavras, basta invocar `floodFill` nos pixels da borda da imagem (colunas 0 e N-1, linhas 0 e M-1) que possuam tom de objeto `255`:

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

<p align="center">
    <img src="../../assets/images/bolhas1.png">
</p>

Todas as bolhas podem então ser rotuladas com o `floodFill` com um inteiro diferente, possibilitando a contagem do total de objetos na imagem:

```python
nobjects = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 255:
            nobjects += 1
            cv2.floodFill(image, None, (j, i), nobjects)
```

Agora ao usar `floodFill` em um dos pixels da borda podemos "pintar" o fundo de branco. Qualquer pixel que ainda possua tom de fundo `0` corresponderá aos buracos internos às bolhas:

```python
cv2.floodFill(image, None, (0, 0), 255)
```

<p align="center">
    <img src="../../assets/images/bolhas2.png">
</p>

Agora é fácil contar o número de bolhas com buracos preenchendo cada um deles com `floodFill`. Para evitar contagem repetida em bolhas com mais de um buraco, também preenchemos a bolha simultaneamente, de forma que qualquer buraco subsequente pode ser preenchido sem ser contabilizado.

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

<p align="center">
    <img src="../../assets/images/labeling.png">
</p>

O código final e seu exemplo de execução seriam algo do tipo:

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
```

Saída:

```console
foo@bar:~$ python3 labeling.py bolhas.png
TOTAL DE BOLHAS: 21
    COM BURACOS: 7
    SEM BURACOS: 14
```
