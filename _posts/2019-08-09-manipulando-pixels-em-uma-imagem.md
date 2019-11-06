---
title: "Manipulando imagens do OpenCV usando arrays do NumPy"
date: 2019-08-09
tags: [python, opencv]
category: casual-code
header:
    teaser: /assets/images/regions.png
---

Esse é o primeiro post de uma série de exemplos práticos de processamento de imagens em Python usando OpenCV.<!--more--> Veremos como é simples realizar operações sobre arquivos de imagens.

Uma **imagem** é uma função f de duas variáveis (por exemplo x e y) que correspondem a duas dimensões espaciais (altura e largura). Uma **imagem digital** é obtida a partir dos processos de **amostragem** e **quantização**. Após esses processos, a imagem pode ser representada por uma matriz de dimensão M x N (M linhas e N colunas) em que cada elemento é um **pixel** (picture element) que apenas pode assumir uma quantidade finita L de níveis de quantização. Em geral, a representação da imagem se utiliza de inteiros entre 0 e L-1 para associar a intensidade de cada pixel ao nível correspondente. Na quantização usual de 8 bits, por exemplo, L = 256 níveis e cada pixel pode assumir valores entre 0 e 255, inclusive.

No caso de imagens coloridas, são superpostas três imagens digitais nos canais vermelho, verde e azul (RGB). Cada pixel passa então a corresponder a uma sequência de três valores inteiros. Por exemplo, um pixel de 8 bits com valores (0,255,0) corresponde ao verde puro, enquanto que (127,127,127) corresponde à intensidade de cinza 50%.

Ao utilizar as funcionalidades do OpenCV no Python, representamos imagens como *arrays* do NumPy. Se utilizarmos a representação inteira em 8 bits, convém que o tipo do array seja dado como `uint8`, mas durante as manipulações aritméticas os arrays poderão eventualmente assumir tipos de ponto flutuante para representação de números reais. A manipulação de pixels em Python é portanto idêntica à manipulação de elementos de um array multidimensional. Exploraremos isso nos exemplos simples abaixo.

## Negativo de imagens

O negativo de uma imagem $f$ é a imagem $g$ de mesmas dimensões com intensidade complementar a $f$:

\\[g(x,y)=(L-1)-f(x,y)\\]

No caso de imagens digitais coloridas, a operação acima é realizada em cada pixel de cada um dos três canais da imagem original $f$. Vamos ver como fazer essa operação básica em Python.

Em primeiro lugar, o OpenCV provê uma função de leitura de arquivos de imagens (`imread`) que recebe, além do nome do arquivo, uma flag indicando como deverá ser armazenada a imagem (por exemplo em escala de cinza ou em cores). Para ler um arquivo fornecido pela linha de comando na execução do programa, podemos fazer:

{% highlight python linenos %}
import sys
import cv2

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_COLOR)
{% endhighlight %}

Graças à capacidade do NumPy de realizar operações vetorizadas, podemos encontrar o negativo da imagem completa com uma única linha:

{% highlight python linenos %}
negative = 255 - image
{% endhighlight %}

Da mesma forma, caso queiramos realizar a operação exclusivamente em uma subregião retangular da figura, podemos usar a técnica de *slicing* para acessar apenas os elementos desejados do array. Por exemplo, considere que as coordenadas dos cantos superior esquerdo e inferior direito da região de interesse serão fornecidos pelo usuário durante a execução do programa. Podemos resolver esse problema em poucas linhas da seguinte forma:

{% highlight python linenos %}
import sys
import cv2

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
{% endhighlight %}

A função `imshow` abre uma tela identificada pelo título (fornecido no primeiro argumento) e mostra a imagem (segundo argumento) nessa tela. A função `waitKey`, como indica o nome, mantém o programa rodando (e, portanto, a tela aberta) até que uma tecla seja pressionada no teclado. Um possível resultado da execução do programa é o seguinte: 

```console
foo@bar:~$ python3 regions.py biel.png
pi: 100, 100
pj: 200, 200
```

<p align="center">
    <img src="../../assets/images/regions.png">
</p>

### Versão interativa

O OpenCV possibilita a interação em tempo real do usuário com as telas geradas pela interface (`imshow`) através de **eventos** que correspondem a ações realizadas no mouse ou no teclado. Podemos usar isso para definir interativamente, com o arrastar do mouse, a região retangular na qual queremos aplicar a operação de negativo. Para isso, precisamos definir uma função de *callback* que interprete os possíveis eventos realizados com o mouse e mostre o resultado em tempo real:

{% highlight python linenos %}
import sys
import cv2

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
    if event == cv2.EVENT_MOUSEMOVE:
        if drawing:
            xi, xj = sorted([xo, x])
            yi, yj = sorted([yo, y])
            negative[yi:yj, xi:xj] = 255 - original[yi:yj, xi:xj]
    if event == cv2.EVENT_LBUTTONUP:
        drawing = False
    if event == cv2.EVENT_RBUTTONDOWN:
        drawing = False
        negative = original.copy()
{% endhighlight %}

No código acima da função `invert`, analisamos quatro possíveis operações com o mouse: `EVENT_LBUTTONDOWN` corresponde ao pressionar do botão esquerdo do mouse, e marcará o ponto inicial da região retangular, (`xo`, `yo`). A flag booleana `drawing` indica que o arrastar do mouse (`EVENT_MOUSEMOVE`) serão consideradas como parte do desenhar do retângulo; se `drawing` for verdadeiro, estabelecemos a posição atual do mouse (`x`, `y`) e a posição inicial armazenada em (`xo`, `yo`) como os dois cantos do retângulo (precisamos garantir que estejam ordenados com a função `sorted`). No caso de soltarmos o botão esquerdo, `EVENT_LBUTTONUP` será emitido e `drawing` passará a ser falso. Além disso, ao pressionar o botão direito (`EVENT_RBUTTONDOWN`) restauramos a imagem ao seu estado original.

Para utilizarmos essa função de callback em uma tela do OpenCV, podemos criá-la ainda em branco com a função `namedWindow` e em seguinda utilizar `setMouseCallBack` da seguinte forma:

{% highlight python linenos %}
cv2.namedWindow('negative')
cv2.setMouseCallback('negative', invert)
{% endhighlight %}

Por fim, mantemos o programa em um loop infinito de `ìmshow` até que a tecla ESC (valor 27 na tabela ASCII) seja pressionada e identificada pela função `waitKey`. Logo, programa completo para a versão interativa do problema é algo da seguinte forma:

{% highlight python linenos %}
import sys
import cv2

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
    if event == cv2.EVENT_MOUSEMOVE:
        if drawing:
            xi, xj = sorted([xo, x])
            yi, yj = sorted([yo, y])
            negative[yi:yj, xi:xj] = 255 - original[yi:yj, xi:xj]
    if event == cv2.EVENT_LBUTTONUP:
        drawing = False
    if event == cv2.EVENT_RBUTTONDOWN:
        drawing = False
        negative = original.copy()

cv2.namedWindow('negative')
cv2.setMouseCallback('negative', invert)

while True:
    cv2.imshow('negative', negative)
    if cv2.waitKey(20) & 0xFF == 27:
        break

{% endhighlight %}

E um pequeno GIF que ilustra a execução do programa é mostrado abaixo.

<p align="center">
    <img src="../../assets/images/regions2.gif">
</p>

## Regiões de Interesse

Essas subregiões retangulares com as quais lidamos na seção anterior são conhecidas como **regiões de interesse** (ROIs). Um exemplo muito comum de manipulação de ROIs consiste na troca dos quadrantes de uma imagem. Se soubermos as dimensões da imagem em pixels, esse procedimento se torna meramente mais uma aplicação do slicing. 

{% highlight python linenos %}
import sys
import cv2

filename = sys.argv[1]
orig = cv2.imread(filename, cv2.IMREAD_COLOR)
M, N = orig.shape[:2]
{% endhighlight %}

O atributo `shape` de uma imagem digital colorida será uma tupla de três elementos da forma (M, N, 3), onde M e N correspondem ao número de linhas e colunas da imagem, respectivamente. Usando o operador de divisão inteira `//` e assumindo que M e N sejam inteiros pares para que todos os quatro quadrantes tenham dimensões idênticas, podemos atribuir diretamente os quadrantes da imagem original para uma imagem destino de mesmo tamanho:

{% highlight python linenos %}
dest = orig.copy()
dest[:M//2, :N//2] = orig[M//2:, N//2:]
dest[:M//2, N//2:] = orig[M//2:, :N//2]
dest[M//2:, :N//2] = orig[:M//2, N//2:]
dest[M//2:, N//2:] = orig[:M//2, :N//2]

cv2.imshow('original', orig)
cv2.imshow('swapped', dest)
cv2.waitKey()
{% endhighlight %}

Para ilustrar o resultado obtido com a troca de quadrantes de forma clara, eis um exemplo de `orig` e `dest`:

<p align="center">
    <img hspace="20" src="../../assets/images/abcd.jpeg">
    <img src="../../assets/images/trocaregioes.png">
</p>