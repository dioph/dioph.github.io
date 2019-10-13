---
title: "Filtragem no domínio da frequência"
date: 2019-10-11
tags: [python, opencv]
category: casual-code
---
<!--more-->

## Filtragem homomórfica

```python
import sys
import cv2
import numpy as np

gamma_h_slider_max = 100
gamma_l_slider_max = 100
c_slider_max = 100
d0_slider_max = 100

gamma_h_slider_name = "\u03B3h (%)"
gamma_l_slider_name = "\u03B3l (%)"
c_slider_name = "c"
d0_slider_name = "D0"

filename = sys.argv[1]
image = cv2.imread(filename, cv2.IMREAD_GRAYSCALE)
logimage = np.log(image + 1.)

M, N = image.shape
v, u = np.meshgrid(np.arange(N), np.arange(M))
d = np.hypot(u-M/2, v-N/2)

dft_log_image = np.fft.fft2(logimage)
dft_log_image = np.fft.fftshift(dft_log_image)

window_name = "filtered"
cv2.namedWindow(window_name, cv2.WINDOW_AUTOSIZE)

def on_change(val):
    gamma_h = cv2.getTrackbarPos(gamma_h_slider_name, window_name) / 100
    gamma_l = cv2.getTrackbarPos(gamma_l_slider_name, window_name) / 100
    c = cv2.getTrackbarPos(c_slider_name, window_name)
    d0 = cv2.getTrackbarPos(d0_slider_name, window_name)

    mask = (gamma_h - gamma_l) * (1 - np.exp(-c * (d/d0)**2)) + gamma_l
    dft_log_filtered = mask * dft_log_image
    dft_log_filtered = np.fft.ifftshift(dft_log_filtered)
    log_filtered = np.fft.ifft2(dft_log_filtered)
    filtered = np.exp(np.abs(log_filtered))
    filtered = cv2.normalize(filtered, None, 0, 1, norm_type=cv2.NORM_MINMAX)

    cv2.imshow(window_name, filtered)

cv2.createTrackbar(gamma_h_slider_name, window_name, 0,
                   gamma_h_slider_max, on_change)

cv2.createTrackbar(gamma_l_slider_name, window_name, 0,
                   gamma_l_slider_max, on_change)

cv2.createTrackbar(c_slider_name, window_name, 0,
                   c_slider_max, on_change)

cv2.createTrackbar(d0_slider_name, window_name, 0,
                   d0_slider_max, on_change)

on_change(0)
cv2.waitKey(0)
```