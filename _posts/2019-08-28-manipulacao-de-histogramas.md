---
title: "Manipulação de histogramas"
date: 2019-08-28
tags: [python, opencv]
category: casual-code
---
<!--more-->
## Equalização

```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)

assert cap.isOpened(), 'UNAVAILABLE CAMERA'

def get_hist(src):
    hist, _ = np.histogram(src.flatten(), bins=256, range=(0, 256))
    hist = np.cumsum(hist)
    return hist

while(1):
    _, image = cap.read()
    r, g, b = cv2.split(image)
    
    histR = get_hist(r)
    histG = get_hist(g)
    histB = get_hist(b)
    
    newImg = np.zeros_like(image)
    newImg[:, :, 0] = histB[image[:, :, 0]] * 255 / histB[-1]
    newImg[:, :, 1] = histG[image[:, :, 1]] * 255 / histG[-1]
    newImg[:, :, 2] = histR[image[:, :, 2]] * 255 / histR[-1]
    
    cv2.imshow('image', image)
    cv2.imshow('new', newImg)
    
    if cv2.waitKey(20) & 0xFF == 27:
        break
```

## Detecção de Movimentos

```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)

assert cap.isOpened(), 'UNAVAILABLE CAMERA'
    
nbins = 256
histw = nbins
histh = nbins // 2
motion_threshold = 10.

histImg = np.zeros((3*histh, histw, 3))
delta = np.zeros((3*histh, histw, 3))

oldR = 0
oldG = 0
oldB = 0

def get_hist(src):
    hist, _ = np.histogram(src.flatten(), bins=256, range=(0, 256))
    hist = cv2.normalize(hist, None, 0, histh, norm_type=cv2.NORM_MINMAX)
    return hist

def plot_hist(hist, color):
    img = np.zeros((histh, histw, 3))
    for i in range(nbins):
        cv2.line(img=img, pt1=(i, histh), pt2=(i, histh - hist[i]), 
                 color=color, thickness=1, lineType=8, shift=0)
    return img

while(1):
    _, image = cap.read()
    r, g, b = cv2.split(image)
    
    histR = get_hist(r)
    histG = get_hist(g)
    histB = get_hist(b)
    
    histImgR = plot_hist(histR, (0, 0, 255))
    histImgG = plot_hist(histG, (0, 255, 0))
    histImgB = plot_hist(histB, (255, 0, 0))
    
    histImg[       :  histh, :nbins] = histImgR
    histImg[  histh:2*histh, :nbins] = histImgG
    histImg[2*histh:3*histh, :nbins] = histImgB
    
    deltaR = histR - oldR
    deltaG = histG - oldG
    deltaB = histB - oldB
    
    dImgR = plot_hist(deltaR, (0, 0, 255))
    dImgG = plot_hist(deltaG, (0, 255, 0))
    dImgB = plot_hist(deltaB, (255, 0, 0))
    
    mseR = np.mean(np.square(deltaR))
    mseG = np.mean(np.square(deltaG))
    mseB = np.mean(np.square(deltaB))
    
    cv2.putText(dImgR, '{:.1f}'.format(mseR), (0, histh//2), 
                cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 2, cv2.LINE_AA)
    cv2.putText(dImgG, '{:.1f}'.format(mseG), (0, histh//2), 
                cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 2, cv2.LINE_AA)
    cv2.putText(dImgB, '{:.1f}'.format(mseB), (0, histh//2), 
                cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 2, cv2.LINE_AA)
    
    delta[       :  histh, :nbins] = dImgR
    delta[  histh:2*histh, :nbins] = dImgG
    delta[2*histh:3*histh, :nbins] = dImgB
    
    oldR = np.copy(histR)
    oldG = np.copy(histG)
    oldB = np.copy(histB)
    
    if mseR > motion_threshold or mseG > motion_threshold or mseB > motion_threshold:
        cv2.putText(image, 'MOTION DETECTED', (50, 450), 
                    cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 2, cv2.LINE_AA)
    
    cv2.imshow('image', image)
    cv2.imshow('histogram', histImg)
    cv2.imshow('differences', delta)
    
    if cv2.waitKey(20) & 0xFF == 27:
        break
```
