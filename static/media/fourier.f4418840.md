---
title: "二維離散傅里葉頻譜（正逆變換）"
date: 2018-11-23
type: ["笔记"]
weight: 7
tags: ["PYTHON","OPENCV","图像处理"]
categories: ["笔记","最近"]
description: "数字图像处理的结课课题任务，附带前后学习笔记与实验代码"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fourier/fourier.webp"
---

# 相關知識
## 傅里葉

理論基礎：任何函數都可以表示成正弦函數的線性組合形式。
應用簡介：將信號從時域或空域轉換到頻域，使用頻域關係來保存對信號的描述。音頻應用方面可用於分頻，均衡器、濾波等。二維離散（圖像）引用方面可以用於圖像分析、增強、降噪等。

簡單的傅里葉音頻解析原理圖

![簡單的傅里葉音頻解析原理圖](https://upload.wikimedia.org/wikipedia/commons/7/72/Fourier_transform_time_and_frequency_domains_%28small%29.gif)


在圖像應用上，RGB色彩模式下，圖像亮度與數值成正比，通過計算一定範圍內的一系列數值，來計算光暗變化的頻率，進而將全圖的頻率掃描映射到頻域，并將映射的結果保存在頻譜圖像上。

二維離散傅里葉變換公式

![二維離散傅里葉變換公式](https://upload-images.jianshu.io/upload_images/4305587-b722ae259c0016cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)[^0]

![引用博客圖](http://my.csdn.net/uploads/201206/01/1338526304_4991.jpg)[^1]

在變換出來的圖像中，每一個像素點代表一個頻率值（縱橫方向），亮度代表頻率出現的多少。最中間亮點代表直流分量（不隨空間變化的量，頻率為0），可以看到圖像聚焦于中間一橫。其表達意義在於：圖像僅有在橫向上存在變化的變化率，在垂直方向上變化率幾乎為0。

[更多關於頻譜圖的解讀](https://zhuanlan.zhihu.com/p/29442552)

圖像應用上用於解析、修復條紋：
![圖像傅里葉應用](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fourier/01.png)[^2]

頻譜圖的亮度分佈越集中，畫面越柔和，則原圖畫面越平緩。若亮點四散分佈，則畫面又較強條幅、亮度差異等情況。

# 工具使用
## python

- opencv-python (import: cv2)

- numpy (快速運算矩陣)

> opencv 基本使用
>
> ``cv2.imshow('imageName1',cv2.imread("./img/cloud.jpg"))``顯示圖像
>
> ``cv2.waitKey()``imshow配套使用，阻塞線程
>
> ``cv2.imwrite('imageName.png',img)``寫入圖像

導入包：
```python
import math
import cv2;# package: opencv-python
import numpy as np;
```

傅里葉變換：
```python
def Fourier(originImg):
# this function offer a O(n^4) method to load gray pixel one-by-one
# to calculate Fourier image of param image
# : param originImg: offer an image read by gray uint8(default)
    width, height = originImg.shape
    # acquire image size
    complexMartixReal = [[0 for i in range(width)]for i in range(height)]
    complexMartixImag = [[0 for i in range(width)]for i in range(height)]
    # create two complex martix to save(real one and imag one)
    newimg = np.zeros([width, height, 1], np.uint8)
    # create new image to return
    for u in range(width):
        for v in range(height):
            tmpx = 0;tmpy = 0;
            for x in range(width):
                for y in range(height):
                    arg=-2*math.pi*(u*x/width+v*y/height)
                    tmpx+=(pow(-1,x+y)*img[x][y]*math.cos(arg))
                    tmpy+=(pow(-1,x+y)*img[x][y]*math.sin(arg))
            # save calculation result of complex
            complexMartixReal[u][v] = tmpx
            complexMartixImag[u][v] = tmpy
            newimg[u,v] = np.abs(complex(tmpx, tmpy))/math.sqrt(width*height)
            # move to new central
    return complexMartixReal,complexMartixImag,newimg
```

傅里葉逆變換：
```python
def InverseFourier(originImg,complexMartixReal,complexMartixImag):
# this function offer a O(n^4) method to load gray image which had
# been  returned by Fourier(), to replay origin image
# : param originImg: offer an image which had been returned by Fourier()
# : param complexMartixReal: an argument returned by Fourier() loading complex
#                            martix of real-part while calculating image
# : param complexMartixImag: an argument returned by Fourier() loading complex
#                            martix of imag-part while calculating image
    # acquire image size
    width, height = originImg.shape
    newimg = np.zeros([width, height, 1], np.uint8)
    # create new image to return
    for u in range(width):
        for v in range(height):
            tmp=0
            for x in range(width):
                for y in range(height):
                    arg=2*math.pi*(u*x/width+v*y/height)
                    euler=complexMartixReal[x][y]* math.cos(arg)
                    euler-=complexMartixImag[x][y]*math.sin(arg)
                    # calculate euler result from complex martix
                    tmp+=int(euler/(width*height))
            newimg[u][v]=abs(tmp)
    return newimg
```

測試代碼：
```python
# test code :
originImg = cv2.imread("./img/cloud.jpg", cv2.IMREAD_GRAYSCALE)
# load image
complexMartixReal,complexMartixImag,fourierImg=Fourier(originImg);
# tmp of complex martix, fourier img to load
fourierInverseImg=InverseFourier(fourierImg,complexMartixReal,complexMartixImag);

cv2.imshow("origin",originImg)
cv2.imshow("fourier",fourierImg)
cv2.imshow("new",fourierInverseImg)
cv2.waitKey()
```

## 測試結果

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fourier/01.png)

## 寫在最後

1. numpy有提供fft、fft2（專門用於圖像變換的**快速傅里葉變換**的函數庫）

2. 測試代碼時候不要使用太大的圖片（建議100px^2^以內），時間複雜度導致運算時間增長驚人

3. 傅里葉變換的思路（特征分頻過濾）在抵抗DOS方面是否有可以借鑒的思路

# 相關資料

1. [傅里叶分析之掐死教程（完整版）](http://gr.xjtu.edu.cn/c/document_library/get_file?p_l_id=1722675&folderId=2083076&name=DLFE-58002.pdf)

2. [3Blue1Brown Fourier visual introduction](https://www.youtube.com/watch?v=spUNpyF58BY)

3. [3Blue1Brown Fourier visual introduction 中文版](https://www.bilibili.com/video/av19141078)

4. [傅里葉變換 wiki](https://www.wikiwand.com/en/Fourier_transform)

5. [歐拉公式 wiki](https://www.wikiwand.com/zh/%E6%AC%A7%E6%8B%89%E5%85%AC%E5%BC%8F)

6. [opencv及相關資料](http://wiki.opencv.org.cn/index.php/%E9%A6%96%E9%A1%B5)

7. [課本教材：數字圖像處理第三版](https://www.amazon.cn/dp/B00513FBZK)


[^0]:[傅里葉公式集 簡介(簡書)](https://www.jianshu.com/p/7dfe02fa34a9)

[^1]:[資料引用出處1](https://blog.csdn.net/abcjennifer/article/details/7622228)

[^2]:[資料引用出處2](https://www.zhihu.com/question/20460630/answer/105888045)
