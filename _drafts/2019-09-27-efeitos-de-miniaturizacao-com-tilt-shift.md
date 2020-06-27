---
title: "Efeitos de miniaturização com tilt-shift"
date: 2019-09-27
tags: [python, opencv]
category: casual-code
---
<!--more-->

## Tilt-Shift

\\[
\frac{1}{2} \left\[ \tanh\left(\frac{x-\ell_1}{d}\right) - \tanh\left(\frac{x-\ell_2}{d}\right) \right\]
\\]

### Usando sliders no OpenCV

```python
cv2.createTrackbar(slider_name, window_name, 0, slider_max, on_change)
```

{% highlight python linenos %}
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
_, mesh = np.meshgrid(np.arange(N), np.arange(M))

blur = np.ones((5, 5)) / 25
filtered = cv2.filter2D(original, -1, blur)

window_name = "tilt-shift"
cv2.namedWindow(window_name, cv2.WINDOW_AUTOSIZE)

def alpha(x, d, l1, l2):
    return 0.5 * (np.tanh((x - l1) / d) - np.tanh((x - l2) / d))

def tiltshift(d, l1, l2):
    mask = alpha(mesh, d, l1, l2)
    mask = np.atleast_3d(mask)
    blended = mask * original + (1 - mask) * filtered
    blended = blended.astype(np.uint8)
    return blended

def on_change(val):
    height = cv2.getTrackbarPos(height_slider_name, window_name)
    decay = cv2.getTrackbarPos(decay_slider_name, window_name)
    offset = cv2.getTrackbarPos(offset_slider_name, window_name)
    d = decay * M / 100
    l1 = (offset - height / 2) * M / 100
    l2 = (offset + height / 2) * M / 100
    blended = tiltshift(d, l1, l2)
    cv2.imshow(window_name, blended)

cv2.createTrackbar(height_slider_name, window_name, 0,
                   height_slider_max, on_change)
cv2.createTrackbar(decay_slider_name, window_name, 0,
                   decay_slider_max, on_change)
cv2.createTrackbar(offset_slider_name, window_name, 0,
                   offset_slider_max, on_change)
on_change(0)
cv2.waitKey(0)
{% endhighlight %}

<p align="center">
    <img src="../../assets/images/tiltshift_mask.gif">
</p>

<figure class="half">
    <img src="../../assets/images/fruits.png">
    <img src="../../assets/images/tilted.png">
    <figcaption>Figura original (esquerda) e com foco no plano das uvas (direita).</figcaption>
</figure>

## Análise de vídeo

### Passando argumentos na linha de comando com `argparse`

{% highlight python linenos %}
import cv2
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("filename", help="input video file path")
parser.add_argument("-o", "--output", default="output.avi",
                    help="output file path (default: ouput.avi)")
parser.add_argument("-s", "--size", type=float, required=True,
                    help="region size in %% of image height")
parser.add_argument("-d", "--decay", type=float, required=True,
                    help="decay rate in %% of image height")    
parser.add_argument("-c", "--center", type=float, required=True,
                    help="center height in %% of image height")
parser.add_argument("-f", '--framedrop', type=int, default=1,
                    help="frame dropping rate")
args = vars(parser.parse_args())
{% endhighlight %}

### Lendo arquivos de vídeo frame a frame

{% highlight python linenos %}
cap = cv2.VideoCapture(args['filename'])
_, first_frame = cap.read()
M, N = first_frame.shape[:2]
_, mesh = np.meshgrid(np.arange(N), np.arange(M))
d = args['decay'] * M / 100
l1 = (args['center'] - args['size'] / 2) * M / 100
l2 = (args['center'] + args['size'] / 2) * M / 100

def alpha(x, d, l1, l2):
    return 0.5 * (np.tanh((x - l1) / d) - np.tanh((x - l2) / d))
{% endhighlight %}

{% highlight python linenos %}
mask = alpha(mesh, d, l1, l2)
mask = np.atleast_3d(mask)
blur = np.ones((5, 5)) / 25

def tiltshift(original):
    filtered = cv2.filter2D(original, -1, blur)
    blended = mask * original + (1 - mask) * filtered
    blended = blended.astype(np.uint8)
    return blended
{% endhighlight %}

### Escrevendo arquivos de vídeo com OpenCV

{% highlight python linenos %}
fourcc = cv2.VideoWriter_fourcc(*'MJPG')
out = cv2.VideoWriter(args['output'], fourcc, 24., (N, M))
{% endhighlight %}


{% highlight python linenos %}
framecount = 0
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    framecount = framecount + 1
    if framecount % args['framedrop'] > 0:
        continue
    blended = tiltshift(frame)
    out.write(blended)
{% endhighlight %}

### Resultado final

{% highlight python linenos %}
import cv2
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("filename", help="input video file path")
parser.add_argument("-o", "--output", default="output.avi",
                    help="output file path (default: ouput.avi)")
parser.add_argument("-s", "--size", type=float, required=True,
                    help="region size in %% of image height")
parser.add_argument("-d", "--decay", type=float, required=True,
                    help="decay rate in %% of image height")    
parser.add_argument("-c", "--center", type=float, required=True,
                    help="center height in %% of image height")
parser.add_argument("-f", '--framedrop', type=int, default=1,
                    help="frame dropping rate")
args = vars(parser.parse_args())

cap = cv2.VideoCapture(args['filename'])
_, first_frame = cap.read()
M, N = first_frame.shape[:2]
_, mesh = np.meshgrid(np.arange(N), np.arange(M))
d = args['decay'] * M / 100
l1 = (args['center'] - args['size'] / 2) * M / 100
l2 = (args['center'] + args['size'] / 2) * M / 100

def alpha(x, d, l1, l2):
    return 0.5 * (np.tanh((x - l1) / d) - np.tanh((x - l2) / d))

mask = alpha(mesh, d, l1, l2)
mask = np.atleast_3d(mask)
blur = np.ones((5, 5)) / 25

def tiltshift(original):
    filtered = cv2.filter2D(original, -1, blur)
    blended = mask * original + (1 - mask) * filtered
    blended = blended.astype(np.uint8)
    return blended

fourcc = cv2.VideoWriter_fourcc(*'MJPG')
out = cv2.VideoWriter(args['output'], fourcc, 24., (N, M))

framecount = 0
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    framecount = framecount + 1
    if framecount % args['framedrop'] > 0:
        continue
    blended = tiltshift(frame)
    out.write(blended)
{% endhighlight %}
