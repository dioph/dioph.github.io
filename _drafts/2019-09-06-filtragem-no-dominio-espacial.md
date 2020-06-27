---
title: "Convolução digital e seus efeitos em imagens"
date: 2019-09-06
tags: [python, opencv]
category: casual-code
---
<!--more-->



## Convolução digital

A ideia de convolução entre sequências discretas pode ser estendida para qualquer número de dimensões. No caso bidimensional que nos interessa, podemos definir uma imagem $g$ a partir da convolução da imagem original com um determinado **núcleo de convolução**:

\\[
g(x,y) = f(x,y)*h(x,y) = \sum_{i=-\infty}^\infty \sum_{j=-\infty}^\infty h(i,j)f(x-i,y-j)
\\]

Se definirmos $h^T(x,y)=h(-x,-y)$, a operação de convolução acima pode ser escrita como 
\\[
g(x,y) = \sum_{i=-\infty}^\infty \sum_{j=-\infty}^\infty h^T(i,j)f(x+i,y+j) = f(x,y) \star h^T(x,y)
\\]

Aqui o símbolo da estrela de cinco pontas ($\star$) representa a operação de **correlação**, diferindo sutilmente da operação de convolução ($*$).

### O efeito de cada núcleo

Dependendo da escolha de $h$, o resultado da convolução digital pode ser dos mais diversos. Podemos diferenciar principalmente os efeitos de filtros **suavizadores** e filtros **aguçadores** (sim, análogos a filtros passa-baixa ou passa-alta no domínio da frequência).

Os filtros suavizadores causam o conhecido efeito de "borramento" da imagem, similar ao de fotos desfocadas. Eles conseguem esse efeito usando os somatórios de convolução para aproximar a intensidade dos pixels da imagem resultante por valores médios dos pixels de sua vizinhança, efetivamente reduzindo as transições abruptas. Os principais exemplos são o **filtro da média** e o **filtro gaussiano**.

Já os filtros aguçadores produzem o efeito oposto: usam o somatório de convolução para calcular diferenças na vizinhança dos pixels e realçar as transições abruptas, sendo muito utilizados como **detectores de borda**. Os **filtros de Sobel** são exemplos muito conhecidos de detecção de bordas em direções específicas (horizontal ou vertical), podendo inclusive ser usados para estimar módulo e direção de vetores gradiente em imagens. Outro exemplo clássico é o **filtro laplaciano**, que estima o operador laplaciano ($\nabla^2=\partial_x^2+\partial_y^2$) na vizinhança de cada pixel, detectando bordas em todas as direções.

Se representarmos os núcleos como matrizes quadradas de dimensões 3x3, eles são:

* Filtro da Média
\\[
h_m=\frac{1}{9}
\begin{pmatrix}
1 & 1 & 1 \\\
1 & 1 & 1 \\\
1 & 1 & 1
\end{pmatrix}
\\]
* Filtro Gaussiano
\\[
h_G=\frac{1}{16}
\begin{pmatrix}
1 & 2 & 1 \\\
2 & 4 & 2 \\\
1 & 2 & 1
\end{pmatrix}
\\]
* Filtro de Sobel
\\[
h_x=
\begin{pmatrix}
-1 & 0 & 1 \\\
-2 & 0 & 2 \\\
-1 & 0 & 1
\end{pmatrix},
\qquad
h_y=
\begin{pmatrix}
-1 & -2 & -1 \\\
0 & 0 & 0 \\\
1 & 2 & 1
\end{pmatrix}
\\]
* Filtro Laplaciano
\\[
h_\ell=
\begin{pmatrix}
0 & -1 & 0 \\\
-1 & 4 & -1 \\\
0 & -1 & 0
\end{pmatrix}
\\]

### Filtragem no OpenCV

No OpenCV, a função que vai nos permitir realizar convoluções digitais para filtrar imagens no domínio espacial é `filter2D`. **Mas cuidado**, o que `filter2D` calcula é a **correlação** entre duas matrizes fornecidas!

```python
filtered = cv2.filter2D(image, depth, kernel) 
```

Aqui, além da imagem `image`, os argumentos necessários são `depth` que especifica o tipo da imagem de saída (-1 para ser do mesmo tipo da entrada) e `kernel` é o array 2D representando a matriz $h^T(x,y)$ do filtro. Nos casos em que o núcleo é simétrico, $h(x,y)=h^T(x,y)$ e não precisamos nos preocupar muito com isso.

### Ativando múltiplos filtros simultaneamente

{% highlight python linenos %}
import cv2
import numpy as np

def menu():
    print("\npress a key to toggle a filter:"
          "\nm - mean"
          "\ng - gaussian"
          "\nv - vertical"
          "\nh - horizontal"
          "\nl - laplacian"
          "\nx - reset all"
          "\nesc - exit")
  
cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'
# definição dos núcleos
media = np.array([[1, 1, 1], [1, 1, 1], [1, 1, 1]]) / 9
gauss = np.array([[1, 2, 1], [2, 4, 2], [1, 2, 1]]) / 16
horiz = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]])
verti = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]])
lapla = np.array([[0, -1, 0], [-1, 4, -1], [0, -1, 0]])
masks = dict(m=media, g=gauss, h=horiz, v=verti, l=lapla)
activ = dict(m=False, g=False, h=False, v=False, l=False)

menu()
while True:
    _, image = cap.read()
    frame = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    cv2.imshow('original', frame)
    # aplicar todos os filtros ativos no momento sucessivamente
    for key in masks.keys():
        if activ[key]:
            frame = cv2.filter2D(frame, -1, masks[key])
    cv2.imshow('filtered', frame)
    # eventos de teclado
    key = cv2.waitKey(10)
    if key > 0:
        key = chr(key)
    # esc - exit
    if key == chr(27):
        break
    # x - reset all
    if key == 'x':
        menu()
        activ = dict.fromkeys(activ, False)
    # toggle filter
    if key in activ.keys():
        menu()
        activ[key] = not activ[key]
{% endhighlight %}

Os resultados obtidos foram os seguintes:

<figure class="half">
    <img src="../../assets/images/filt_m.png" width="256">
    <img src="../../assets/images/filt_g.png" width="256">
    <figcaption>Filtro da média (esquerda) vs filtro gaussiano (direita).</figcaption>
</figure>


<figure class="half">
    <img src="../../assets/images/filt_h.png" width="256">
    <img src="../../assets/images/filt_v.png" width="256">
    <figcaption>Sobel horizontal (esquerda) vs sobel vertical (direita).</figcaption>
</figure>


<figure class="half">
    <img src="../../assets/images/filt_l.png" width="256">
    <img src="../../assets/images/filt_gl.png" width="256">
    <figcaption>Filtro laplaciano (esquerda) vs laplaciano do gaussiano (direita).</figcaption>
</figure>
