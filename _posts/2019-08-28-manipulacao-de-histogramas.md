---
title: "Manipulação de histogramas"
date: 2019-08-28
tags: [python, opencv]
category: casual-code
header:
    teaser: /assets/images/histograms.png
---
Nesse tutorial vamos buscar entender o que são e para que podem ser usados histogramas de imagens.
<!--more-->


## Equalização

{% highlight python linenos %}
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'

def get_cum_hist(img):
    hist, _ = np.histogram(img.flatten(), bins=256, range=(0, 256))
    hist = np.cumsum(hist)
    return hist

while True:
    _, original = cap.read()
    r, g, b = cv2.split(original)
    
    histR = get_cum_hist(r)
    histG = get_cum_hist(g)
    histB = get_cum_hist(b)
    
    equalized = np.zeros_like(original)
    equalized[:, :, 0] = histB[original[:, :, 0]] * 255 / histB[-1]
    equalized[:, :, 1] = histG[original[:, :, 1]] * 255 / histG[-1]
    equalized[:, :, 2] = histR[original[:, :, 2]] * 255 / histR[-1]
    
    cv2.imshow('original', original)
    cv2.imshow('equalized', equalized)
    
    if cv2.waitKey(20) & 0xFF == 27:
        break
{% endhighlight %}

## Detecção de Movimentos

{% highlight python linenos %}
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
assert cap.isOpened(), 'UNAVAILABLE CAMERA'
    
nbins = 256
histw = nbins
histh = nbins // 2
threshold = 10.

histImg = np.zeros((3 * histh, histw, 3))
diffImg = np.zeros((3 * histh, histw, 3))

oldR = 0
oldG = 0
oldB = 0

def get_norm_hist(img):
    hist, _ = np.histogram(img.flatten(), bins=256, range=(0, 256))
    hist = cv2.normalize(src=hist, dst=None, alpha=0, beta=histh,
                         norm_type=cv2.NORM_MINMAX)
    return hist

def plot_hist(hist, color):
    img = np.zeros((histh, histw, 3))
    for i in range(nbins):
        cv2.line(img=img, pt1=(i, histh), pt2=(i, histh - hist[i]), 
                 color=color, thickness=1, lineType=cv2.LINE_8, shift=0)
    return img

while True:
    _, image = cap.read()
    r, g, b = cv2.split(image)
    
    histR = get_norm_hist(r)
    histG = get_norm_hist(g)
    histB = get_norm_hist(b)
    
    histImgR = plot_hist(histR, (0, 0, 255))
    histImgG = plot_hist(histG, (0, 255, 0))
    histImgB = plot_hist(histB, (255, 0, 0))
    
    histImg[       :  histh, :nbins] = histImgR
    histImg[  histh:2*histh, :nbins] = histImgG
    histImg[2*histh:3*histh, :nbins] = histImgB
    
    diffR = histR - oldR
    diffG = histG - oldG
    diffB = histB - oldB
    
    diffImgR = plot_hist(diffR, (0, 0, 255))
    diffImgG = plot_hist(diffG, (0, 255, 0))
    diffImgB = plot_hist(diffB, (255, 0, 0))
    
    mseR = np.mean(np.square(diffR))
    mseG = np.mean(np.square(diffG))
    mseB = np.mean(np.square(diffB))
    
    cv2.putText(diffImgR, '%.1f' % mseR, (0, histh//2), fontScale=2,
                fontFace=cv2.FONT_HERSHEY_SIMPLEX, thickness=2,
                color=(255, 255, 255), lineType=cv2.LINE_AA)
    cv2.putText(diffImgG, '%.1f' % mseG, (0, histh//2), fontScale=2,
                fontFace=cv2.FONT_HERSHEY_SIMPLEX, thickness=2,
                color=(255, 255, 255), lineType=cv2.LINE_AA)
    cv2.putText(diffImgB, '%.1f' % mseB, (0, histh//2), fontScale=2,
                fontFace=cv2.FONT_HERSHEY_SIMPLEX, thickness=2,
                color=(255, 255, 255), lineType=cv2.LINE_AA)
    
    diffImg[       :  histh, :nbins] = diffImgR
    diffImg[  histh:2*histh, :nbins] = diffImgG
    diffImg[2*histh:3*histh, :nbins] = diffImgB
    
    oldR = histR.copy()
    oldG = histG.copy()
    oldB = histB.copy()
    
    if mseR > threshold or mseG > threshold or mseB > threshold:
        cv2.putText(image, 'MOTION DETECTED', (50, 450), fontScale=2,
                    fontFace=cv2.FONT_HERSHEY_SIMPLEX, thickness=2,
                    color=(255, 255, 255), lineType=cv2.LINE_AA)
    
    cv2.imshow('image', image)
    cv2.imshow('histogram', histImg)
    cv2.imshow('differences', diffImg)
    
    if cv2.waitKey(20) & 0xFF == 27:
        break
```
