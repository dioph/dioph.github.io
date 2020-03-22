---
title: "Usando o algoritmo flood-fill para rotular objetos em imagens binárias"
date: 2019-08-14
tags: [python, opencv]
category: casual-code
header:
    teaser: /assets/images/bolhas.png
excerpt: Uma das principais etapas em visão computacional é a segmentação da imagem. Nesse tutorial vamos explorar o processo de rotulagem dos pixels.
---
<!--more-->

## Segmentação de imagens

Uma das principais etapas em visão computacional é a **segmentação** da imagem. Nessa etapa a imagem é subdividida em regiões conexas, que podem representar objetos individuais, por exemplo. Para isso é preciso **rotular** os pixels da imagem de maneira que pixels de mesmo rótulo sejam agrupados em conjuntos de acordo com uma propriedade de interesse.

Nesse tutorial consideraremos que algum pré-processamento (por exemplo, uma limiarização) já foi feito na imagem de maneira a torná-la **binária**, isto é, apenas com pixels pretos e brancos. Para isso vamos usar a imagem abaixo, criada artificialmente para simular uma imagem de microscópio de várias "células" ou "organismos" em formato de bolhas:

<p align="center">
    <img src="../../assets/images/bolhas.png">
</p>

Observando a imagem acima, percebemos que cada bolha individual é uma região conexa de pixels brancos. Como contar o número de bolhas da imagem? Bom, se cada uma delas estivesse associada a rótulos inteiros consecutivos, a última bolha a ser rotulada teria como rótulo a quantidade total de bolhas. A partir dessa ideia geral, podemos contar indiscriminadamente as regiões brancas conexas de qualquer imagem binária.

### Preenchendo regiões: entra em cena o flood-fill

E como pode ser feita essa rotulagem? Entra em cena o algoritmo **flood-fill**. A ideia dele é bastante simples e se assemelha à funcionalidade do "balde de tinta" de editores de imagens; a partir de um pixel de origem, todos os seus vizinhos contidos na mesma região conexa são preenchdos com o mesmo valor. Isso pode ser facilmente implementado através de algoritmos usuais de buscas em grafos como a busca em largura (BFS) ou em profundidade (DFS).

No OpenCV, esse algoritmo está implementado na função `floodFill`, que recebe como argumentos a imagem `image`, uma máscara opcional `mask` com obstáculos intransponíveis para o preenchimento, as coordenadas de uma semente inicial `seedPoint` e o valor `newVal` a ser utilizado no preenchimento. Opcionalmente, também poderíamos especificar valores de diferenças máximas entre a intensidade dos pixels para a definição `loDiff` e `upDiff`, ou `flags` que especifiquem o tipo de conectividade a ser considerado, mas para o nosso caso (imagem binária) esses parâmetros não serão úteis. Sabendo disso, em poucas linhas podemos obter a quantidade de objetos na imagem:

{% highlight python linenos %}
import sys
import cv2

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_GRAYSCALE)
M, N = image.shape

nobjects = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 255:
            nobjects += 1
            cv2.floodFill(image, None, (j, i), nobjects)
{% endhighlight %}

>**Cuidado!**
> Sorrateiramente estamos assumindo que, como o número de bolhas parece ser bem menor que 256, não teremos problemas de ambiguidade ao procurar pelas bolhas ainda não rotuladas (pixels de intensidade 255); se precisássemos levar isso em consideração, bastaria alterar o tipo de dados da imagem original e atribuir inicialmente a todos os pixels brancos um valor "inalcançável" (valores negativos, fracionários, ou muito altos, por exemplo). Não há problema algum em fazer isso uma vez que já estamos em um nível de abstração abaixo da visualização da imagem propriamente dita!

### Aprimorando a contagem: com ou sem buraco?

Um dos problemas usuais envolvendo segmentação de imagens envolve a diferenciação dos elementos da cena. Em outras palavras, às vezes contar as ocorrências de *todos* os objetos indiscriminadamente não ajuda a responder nossa pergunta. Podemos aprimorar nosso algoritmo de contagem para o mesmo exemplo das bolhas microscópicas, mas agora buscando contar separadamente o número de bolhas com ou sem "buraco" interno.

Não estamos interessados em bolhas tocando as bordas da imagem, uma vez que não podemos determinar com certeza se elas apresentam buracos ou não. A primeira coisa a se fazer será portanto preencher todos os objetos na borda da imagem com o tom de fundo `0`. Em outras palavras, basta invocar `floodFill` nos pixels da borda da imagem (colunas 0 e N-1, linhas 0 e M-1) que possuam tom de objeto `255`:

{% highlight python linenos %}
for i in range(M):
    for j in [0, N-1]:
        if image[i, j] == 255:
            cv2.floodFill(image, None, (j, i), 0)
for i in [0, M-1]:
    for j in range(N):
        if image[i, j] == 255:
            cv2.floodFill(image, None, (j, i), 0)
{% endhighlight %}

<p align="center">
    <img src="../../assets/images/bolhas1.png">
</p>

Com as bolhas restantes, vamos rotular todas usando exatamente o mesmo código anterior. Agora ao usar `floodFill` em um dos pixels da borda podemos "pintar" o fundo de branco. Qualquer pixel que ainda possua tom de fundo `0` corresponderá aos buracos internos às bolhas:

{% highlight python linenos %}
cv2.floodFill(image, None, (0, 0), 255)
{% endhighlight %}

<p align="center">
    <img src="../../assets/images/bolhas2.png">
</p>

Note na visualização acima que cada bolha possui um tom de cinza bem escuro, já que estamos rotulando com inteiros consecutivos. Agora é fácil contar o número de bolhas com buracos simplesmente contando os buracos! Vamos preencher cada um deles com `floodFill` usando mais uma vez o mesmo processo. No entanto, para evitar contagem repetida em bolhas que eventualmente tenham mais de um buraco, vamos preencher o buraco em conjunto com sua bolha correspondente, de forma que qualquer buraco subsequente pode ser preenchido sem ser contabilizado. Em outras palavras:

{% highlight python linenos %}
nburacos = 0
for i in range(M):
    for j in range(N):
        if image[i, j] == 0:
            cv2.floodFill(image, None, (j, i), 255)
            if 0 < image[i, j-1] < 255:
                nburacos += 1
                cv2.floodFill(image, None, (j-1, i), 255)
{% endhighlight %}

O código acima procura por buracos (com tom de fundo preto igual a `0`) preenchendo eles com o novo tom de fundo branco `255`. No caso em que esse buraco pertence a uma bolha ainda não contabilizada, o vizinho imediatamente anterior do primeiro pixel preto identificado deve ser um tom de cinza intermediário (de objeto rotulado), então também invocamos `floodFill` nesse pixel com `255`; se esse não for o caso, significa que a bolha já foi contabilizada e não precisamos fazer mais nada. O resultado desse procedimento deixa somente as bolhas sem buraco com tons diferentes do branco que foi usado para "apagar" as bolhas com buraco conforme iam sendo contadas.

<p align="center">
    <img src="../../assets/images/labeling.png">
</p>

### Juntando tudo

O código final e seu exemplo de execução seriam algo do tipo:

{% highlight python linenos %}
import sys
import cv2

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
{% endhighlight %}

Saída:

```console
foo@bar:~$ python labeling.py bolhas.png
TOTAL DE BOLHAS: 21
    COM BURACOS: 7
    SEM BURACOS: 14
```
