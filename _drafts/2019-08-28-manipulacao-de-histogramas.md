---
title: "Histogramas de imagens e pra que servem"
date: 2019-08-28
tags: [python, opencv]
category: casual-code
header:
    teaser: /assets/images/histogram.png
---
Nesse tutorial vamos buscar entender o que são e para que podem ser usados histogramas de imagens.
<!--more-->

Histogramas são nossos conhecidos de estatística e consistem basicamente na contagem da *frequência de ocorrências* de um certo evento. No nosso caso, estamos interessados na distribuição de brilho de uma imagem digital. Um histograma de imagem associa a cada valor de brilho o número de pixels da imagem com aquele dado valor.

A aplicação mais imediata dos histogramas de imagens é a operação de **limiarização** da imagem: através da análise da distribuição de brilho, as maiores concentrações de pixels claros/escuros são identificadas e um limiar é estabelecido a partir do qual a imagem pode ser binarizada. Mas não é disso que vou tratar nesse post; em vez disso, investigaremos outras aplicações de histogramas.

## Equalização

A operação de **equalização** de uma imagem consiste em um tipo especial de ajuste de *contraste*. A intenção aqui é tornar a distribuição de brilho da imagem uma distribuição (aproximadamente) uniforme.

Estatisticamente podemos associar o histograma da imagem $f(x,y)$ a uma *função massa de probabilidade* \\[p_f(i)=p(f=i)=\frac{n_i}{n}\\]
em que $n_i$ pixels de um total de $n$ possuem intensidade igual a $i$. A *distribuição acumulada* é definida como \\[P_f(i)=\sum_{j=0}^ip_f(j)\\]
Para tornar a distribuição da imagem uniforme, sua distribuição acumulada deve ser linear; basta fazer então \\[g(x,y)=P_f(f(x,y)) \times (L-1)\\]
e a imagem equalizada $g(x,y)$ terá intensidade zero (preto puro) nos pontos de intensidade mínima de $f(x,y)$ e intensidade $L-1$ (branco puro) nos pontos de intensidade máxima de $f(x,y)$.

### Capturando vídeo pela câmera do computador

Já temos condições de implementar nosso próprio algoritmo de equalização de imagens em Python. Antes disso vamos ver como capturar vídeo em tempo real usando o OpenCV com dispositivos conectados ao computador. Isso é feito através da classe `VideoCapture`, que cria um objeto de captura de vídeo a partir de um parâmetro identificador da câmera em questão. Essa classe possui um método `isOpened` que pode ser usada para testar o sucesso ao abrir a conexão com a câmera. Além disso usaremos o método `read` para capturar um frame da câmera. Tudo isso pode ser feito dentro de um loop até que a tecla `ESC` seja pressionada:

{% highlight python linenos %}
import cv2

cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'
while True:
    ret, frame = cap.read()
    cv2.imshow('camera video', frame)
    if cv2.waitKey(20) & 0xFF == 27:
        cv2.destroyAllWindows()
        break
{% endhighlight %}

O trecho de código acima apenas captura e mostra em tempo real os frames da forma como são capturados. Ao final da utilização, o objeto `VideoCapture` pode ser liberado através da função `release`:

```python
cap.release()
```

### Calculando o histograma: NumPy vs OpenCV

O cálculo do histograma a partir da contagem de ocorrências dos níveis de intensidade da imagem é uma operação simples que pode ser implementada de maneiras triviais. No entanto, essas implementações muito ingênuas podem levar a tempos de execução desnecessariamente elevados, o que não é ideal (principalmente para realizar operações em tempo real).

O NumPy possui uma função chamada `histogram` que recebe como parâmetros um array unidimensional, o número de bins e o intervalo inferior e superior dos bins. Para uma imagem em tons de cinza de 256 níveis de intensidade, seu histograma poderia ser calculado diretamente da seguinte forma:

```python
hist, bins = np.histogram(image.ravel(), bins=256, range=(0, 256))
```
em que `hist` é o array com as contagens que desejamos e `bins` é simplesmente um array de limites entre cada bin (0, 1, ..., 256). Para o caso de histogramas unidimensionais (como os que estamos usando aqui) é mais eficiente utilizar a função `bincount` com o parâmetro `minlength` igual a 256:

```python
hist = np.bincount(image.ravel(), minlength=256)
```

Em comparação a `histogram`, a execução de `bincount` pode ser até **10x** mais rápida. Entretanto, o próprio OpenCV possui uma função para o cálculo de histogramas, cerca de **40x** mais eficiente que a versão do NumPy! Isso em geral é verdade sobre a maioria das funcionalidades implementadas no OpenCV, que, além de otimizadas, estão implementadas em C/C++ por trás das cortinas. A função do OpenCV que vai nos interessar é a `calcHist`, recebendo como parâmetros a imagem, o canal da imagem no qual se quer calcular o histograma, uma máscara de uma região particular da imagem que se queira calcular o histograma, o número de bins e o intervalo:

```python
hist = cv2.calcHist([image], [0], None, [256], [0, 256])
```

A sintaxe acima deve ser utilizada exatamente dessa forma, com cada argumento sendo uma lista delimitada por colchetes, ou o OpenCV retornará um `SystemError` afirmando que a função retornou `NULL`.

### Usando o OpenCV para plotar os histogramas na própria imagem

Para uma melhor visualização, podemos plotar os histogramas juntamente com cada frame. Para isso podemos nos valer da função `line` do OpenCV para desenhar linhas verticais e traçar o gráfico completo do histograma em uma imagem de 256 pixels de largura:

{% highlight python linenos %}
def plot_hist(hist, color):
    norm_hist = hist * (height / np.max(hist))
    norm_hist = norm_hist.astype(int)
    img = np.zeros((height, 256, 3))
    for i in range(256):
        cv2.line(img, (i, height), (i, height - norm_hist[i]), color)
    return img
{% endhighlight %}

No código acima, a variável `height` define a altura em pixels (número de linhas) da imagem que vai conter o plot do histograma, e a variável `norm_hist`normaliza o intervalo de variação do histograma entre zero e `height`. Depois disso, para cada valor de intensidade `i` entre 0 e 255 é traçada uma reta vertical do ponto `(i, height)` na última linha da imagem, até o ponto correspondente `norm_hist[i]` pixels acima. Ao testar essa função com o histograma de uma imagem aleatória com distribuição gaussiana, obtemos:

{% highlight python linenos %}
height = 192
noise = np.random.normal(loc=100, scale=20, size=(640, 480)).astype(np.uint8)
hist = cv2.calcHist([noise], [0], None, [256], [0, 256])
hist_img = plot_hist(hist, (0, 255, 0))
cv2.imwrite('histogram.png', hist_img)
{% endhighlight %}

<p align="center">
    <img src="../../assets/images/histogram.png" width="320">
</p>

### Simplificando o processo: `cv2.equalizeHist`

Naturalmente o OpenCV já dispõe de uma implementação do processo de equalização completo através da função `equalizeHist`. Essa função basicamente recebe como parâmetro de entrada a imagem original e retorna a imagem já equalizada:

```python
equalized = cv2.equalizeHist(original)
```

No entanto, ela apenas funciona com imagens em escala de cinza, já que opera com o histograma de um único canal. Podemos contornar isso considerando os três canais de uma imagem colorida como sendo independentes e equalizando cada um deles individualmente.

{% highlight python linenos %}
_, original = cap.read()
r, g, b = cv2.split(original)
eqr = cv2.equalizeHist(r)
eqg = cv2.equalizeHist(g)
eqb = cv2.equalizeHist(b)
equalized = cv2.merge([eqr, eqg, eqb])
{% endhighlight%}

A função `split` basicamente transforma a imagem colorida representada por um array de formato (M, N, 3) em três arrays de formato (M, N), enquanto que `merge` realiza o processo inverso.

### Juntando tudo

Finalmente, podemos usar tudo isso que discutimos e criar um programa que:

* capture frames em tempo real da câmera,
* equalize a imagem original,
* calcule os histogramas da imagem original e da imagem equalizada, e
* plote os histogramas em regiões retangulares das próprias imagens. 

{% highlight python linenos %}
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'

def get_hist(img):
    hist = cv2.calcHist([img], [0], None, [256], [0, 256])
    return hist.ravel()

height = 128
def plot_hist(hist, color):
    norm_hist = hist * (height / np.max(hist))
    norm_hist = norm_hist.astype(int)
    img = np.zeros((height, 256, 3))
    for i in range(256):
        cv2.line(img, (i, height), (i, height - norm_hist[i]), color)
    return img

while True:
    _, original = cap.read()
    r, g, b = cv2.split(original)
    eqr = cv2.equalizeHist(r)
    eqg = cv2.equalizeHist(g)
    eqb = cv2.equalizeHist(b)
    equalized = cv2.merge([eqr, eqg, eqb])
    histR = get_hist(r)
    histG = get_hist(g)
    histB = get_hist(b)
    original[        :  height, :256] = plot_hist(histR, (0, 0, 255))
    original[  height:2*height, :256] = plot_hist(histG, (0, 255, 0))
    original[2*height:3*height, :256] = plot_hist(histB, (255, 0, 0))
    eq_histR = get_hist(eqr)
    eq_histG = get_hist(eqg)
    eq_histB = get_hist(eqb)
    equalized[        :  height, :256] = plot_hist(eq_histR, (0, 0, 255))
    equalized[  height:2*height, :256] = plot_hist(eq_histG, (0, 255, 0))
    equalized[2*height:3*height, :256] = plot_hist(eq_histB, (255, 0, 0))
    cv2.imshow('original', original)
    cv2.imshow('equalized', equalized)
    if cv2.waitKey(20) & 0xFF == 27:
        break
{% endhighlight %}

O resultado obtido deve ser algo parecido com

<figure class="half">
    <img src="../../assets/images/placeholder-300x225.png">
    <img src="../../assets/images/placeholder-300x225.png">
    <figcaption>Resultado da execução do código de equalização acima. Imagem original (esquerda) e equalizada (direita).</figcaption>
</figure>

## Detecção de Movimentos

Uma aplicação menos usual de histogramas de imagens pode ser na detecção de movimentos. Afinal de contas, enquanto comparações entre imagens consecutivas sempre terão diferenças devido a ruído branco e a pequenas variações na posição da câmera, a distribuição de brilho da imagem não é tão sensível a esses efeitos. Assim, a comparação de histogramas de imagens consecutivas pode ser uma estratégia efetiva de detecção de movimentos.

### Criando uma métrica de comparação entre histogramas consecutivos

Para decidir se realmente houve movimento ou não, é preciso estabelecer uma forma de *quantificar* as diferenças entre dois histogramas consecutivos. Uma maneira seria, por exemplo, calcular o **erro médio quadrático**:

```python
mse = np.mean(np.square(hist - previous))
```

A partir daí poderia ser calibrado um valor *limiar* de erro, acima do qual o movimento seria detectado.

### Usando `cv2.compareHist` e suas métricas

O OpenCV nos dá uma função que realiza a comparação entre dois histogramas chamada `compareHist`. Além dos histogramas em questão, essa função pede como argumento qual a *métrica* de comparação que deve ser aplicada. Em outras palavras, `compareHist` retorna o resultado da aplicação de uma "função distância" $d$ sobre os dois histogramas $h_1$ e $h_2$. As principais funções de métricas são:

* Correlação (`HISTCMP_CORREL`) 
\\[d(h_1,h_2) = \frac{\sum_i (h_1(i)-\bar{h_1})(h_2(i)-\bar{h_2})}{\sqrt{\sum_i (h_1(i)-\bar{h_1})^2 \sum_i (h_2(i)-\bar{h_2})^2}}\\]
* Qui-Quadrado (`HISTCMP_CHISQR`) 
\\[d(h_1,h_2) = \sum_i\frac{(h_1(i) - h_2(i))^2}{h_1(i)}\\]
* Intersecção (`HISTCMP_INTERSECT`) 
\\[d(h_1,h_2) = \sum_i\min(h_1(i), h_2(i))\\]

Também há outras métricas mais sofisticadas, como por exemplo a [distância de Hellinger](https://en.wikipedia.org/wiki/Hellinger_distance) (associada com o [coeficiente de Bhattacharyya](https://en.wikipedia.org/wiki/Bhattacharyya_distance)) ou a [divergência de Kullback-Leibler](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence), mas para nossos propósitos podemos escolher uma dentre essas três mais simples.

As mudanças no coeficiente de correlação entre quadros consecutivos são em geral muito pequenas, tornando difícil a estimação de um limiar. Da mesma forma, como os histogramas mudam pouco entre quadros, a intersecção se mantém sempre em níveis elevados. Assim, para essa nossa aplicação vamos escolher usar a métrica de qui-quadrado para comparar histogramas consecutivos.

### Resultado final

Para facilitar nossa vida, vamos lidar com a imagem em escala de cinza. Assim, teremos de nos preocupar com as variações ao longo de apenas um histograma.  Também vamos aproveitar as funções que definimos acima para plotar o histograma normalizado no canto superior esquerdo da imagem, juntamente com um "histograma diferença" entre histogramas consecutivos.

Para registrar a diferença calculada entre o histograma atual e o anterior, usaremos a função `putText` do OpenCV para escrever o valor sobre o histograma. Também vamos usar `putText` para escrever a mensagem `MOTION DETECTED` sempre que a distância estiver além de um certo threshold que podemos escolher heuristicamente.

{% highlight python linenos %}
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'

height = 128
def plot_hist(hist, color):
    hist = hist.astype(int)
    img = np.zeros((height, 256))
    for i in range(256):
        cv2.line(img, (i, height), (i, height - hist[i]), color)
    return img
    
threshold = 1000
previous = np.zeros((256, 1), np.float32)
while True:
    _, frame = cap.read()
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    hist = cv2.calcHist([gray], [0], None, [256], [0, 256])
    diff = hist - previous
    norm_hist = hist * (height / np.max(hist))
    norm_diff = diff * (height / np.max(hist))
    gray[      :  height, :256] = plot_hist(norm_hist.ravel(), 127)
    gray[height:2*height, :256] = plot_hist(norm_diff.ravel(), 127)
    dist = cv2.compareHist(hist, previous, cv2.HISTCMP_CHISQR)
    previous = hist.copy()
    cv2.putText(gray, '%.2f' % dist, (0, 3*height//2), fontScale=2,
                fontFace=cv2.FONT_HERSHEY_SIMPLEX, thickness=2,
                color=255, lineType=cv2.LINE_AA)
    if dist > threshold:
        cv2.putText(gray, 'MOTION DETECTED', (50, 450), fontScale=2,
                    fontFace=cv2.FONT_HERSHEY_SIMPLEX, thickness=2,
                    color=255, lineType=cv2.LINE_AA)
    cv2.imshow('frame', gray)
    if cv2.waitKey(20) & 0xFF == 27:
        break
{% endhighlight %}

Aqui está o detector funcionando:

<p align="center">
    <img src="../../assets/images/placeholder-300x225.png">
</p>