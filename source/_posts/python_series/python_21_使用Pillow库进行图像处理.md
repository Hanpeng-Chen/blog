---
title: Python从小白到攻城狮(21)——使用Pillow库进行图像处理
urlname: python-pillow-deal-images
tags:
  - python教程
  - Pillow库
  - 图像处理
categories:
  - Python
abbrlink: 4398
date: 2020-01-20 00:00:00
---

用程序来处理图像和办公文档经常出现在实际开发中，Python的标准库中虽然没有直接支持这些操作的模块，但我们可以通过Python的第三方模块来完成这些操作。

本篇文章我们一起来学习如何用Python对图像进行处理。

# 计算机图像相关知识
在介绍Python操作图像之前，我们要先来了解一下计算机图像的相关知识。

## 颜色
在计算机中，我们可以将红、绿、蓝三种色光以不同的比例叠加来组合成其他的颜色，因此这三种颜色就是色光三原色，所以我们通常会将一个颜色表示为一个RGB值或者RGBA值，其中的A表示Alpha通道，它决定了透过这个图像的像素，也就是透明度。

常见的几个颜色的RGBA值如下：


名称 |	RGBA值	| 名称	| RGBA值
-|-|-|-
White |	(255, 255, 255, 255) |	Red |	(255, 0, 0, 255)
Green |	(0, 255, 0, 255) |	Blue |	(0, 0, 255, 255)
Gray |	(128, 128, 128, 255) |	Yellow |	(255, 255, 0, 255)
Black |	(0, 0, 0, 255) |	Purple |	(128, 0, 128, 255)

## 像素
对于一个由数字序列表示的图像来说，最小的单位就是图像上单一颜色的小方格，这些小方格都有一个明确的位置和被分配的色彩数值，而这些一小方格的颜色和位置决定了该图像最终呈现出来的样子，它们是不可分割的单位，我们通常称之为像素(pixel)。每一个图像都包含了一定量的像素，这些像素决定图像在屏幕上所呈现的大小。

# 用Pillow操作图像
Pillow是由著名的Python图像处理库PIL发展出来的一个分支，如今已经发展成为比PIL本身更具活力的图像处理库。pillow可以说已经取代了PIL。通过Pillow可以实现图像压缩和图像处理等各种操作。

## 安装Pillow
可以使用下面的命令安装Pillow。
```
pip install pillow
```

## 图片的读写操作

### 读取图像
Pillow中最为重要的是Image类，读取和处理图像都要通过这个类来完成。要想以文件的方式读取图片，可以使用`Image`模块的`open()`方法：
```python
from PIL import Image

# 打开图片文件
image = Image.open('./test.jpg')
# 查看图片属性
print(image.format, image.size, image.mode)
# 显示加载的图像
image.show()
```
如果图片文件打开失败，会提示IOError错误。

如果读取图片操作成功，我们可以查看图片的属性。
- `format`这个属性代表图片文件的扩展名，如果图片文件打开失败，则其值为None。
- `size`属性代表图片的大小，以像素为单位，使用包含两个元素的元组来返回。
- `mode`属性代表图片的band属性，一般情况（黑白）下为“L”，当图片是彩色的时候是“RGB”，如果图片经过压缩，则是“CMYK”。

### 修改图片的格式
Python的图像处理库支持绝大多数的图片格式. 直接使用来自 Image 模块的 open() 方法就能从硬盘读取图片文件. 不需要你来区分不同的图片格式, 这个库会自动匹配对应的解码器来打开图片文件.

直接使用来自 Image 模块的 save() 方法来保存图片文件. 当你保存图片文件的时候, 文件名显得尤为重要. 除非你指定扩展名, 默认情况下是自动沿袭本地存储格式的.第二个参数支持 save() 方法来指定图片的扩展名. 如果使用了非标准的扩展名, 则必须加上第二个参数.
```python
image1 = Image.open('./test1.jpg').save('test1.png')
```

### 生成缩略图
```python
size = 128, 128
image.thumbnail(size)
image.show()
```

值得注意的是, 库默认情况下是不会解码光栅图片数据除非是必须的. 当你打开一个文件的时候, 文件的头部将被用来识别文件扩展名和大小等等属性, 但是剩下的数据不会马上被处理.

这也暗示了打开一个图片其实是一个很快的操作, 只关乎到文件大小和压缩方式。

### 裁剪、粘贴和合成图片
Image类包含了可以让你操作图片的方法，当你想从图片截取一部分的时候，直接使用`crop()`方法。
### 裁剪图像
```python
image = Image.open('./test.jpg')
rect = 80, 20, 310, 360
image.crop(rect).show()
```
这个图像区域由含有4个元素的元组组成, 这四个元素分别代表 (左, 上, 右, 下). Python Imaging Library 使用(0, 0)来表示在左上角的情况. 另外值得注意的是, 这些坐标的单位是像素(px), 所以上面的例子实际上表示了 230x340 像素.

这个图像区域现在可以在某些情况下进行处理.

### 黏贴图像
```python
image1 = Image.open('./test.jpg')
image2 = Image.open('./test1.jpg')
rect = 0, 00, 200, 200
image2_head = image2.crop(rect)
image1.paste(image2_head, (172, 40))
image1.show()
```
效果如下图：
{% qnimg python_series/21-2.png %}

## 几何变换
PIL.Image.Image类包含了resize()和rotate() 方法来操作图像。前者需要传入一个表达新大小的元组，后者则需要传入旋转的角度。
### 缩放
```python
image2 = Image.open('./test1.jpg')
width, height = image2.size
image2_head.resize((int(width / 1.5), int(height / 1.5))
```

### 旋转和翻转
```python
image = Image.open('./test.jpg')
image.rotate(180).show()
image.transpose(Image.FLIP_LEFT_RIGHT).show()
```
使用rotate()也能完成transpose(ROTATE)操作，把expand参数设置为True来同时修改图片的尺寸。

修改图片方向的一般方法是使用transform()方法。

## 色彩转换
Python Imaging Library 允许你使用 convert() 方法, 以像素为单位修改图像.

### 模式转换
```python
im = Image.open("lena.ppm").convert("L")
```
这个库支持 “L” 模式和 “RGB” 模式的互相转换. 要想转换到其它的模式, 可能需要使用一个中介模式, 比如 “RGB”.

## 滤镜效果
ImageFilter 模块内置一个预定义的图像效果增强的滤镜, 可用 filter() 方法来实现效果增强。

ImageFilter类中预定义了多种滤镜方法，如 BLUR（模糊滤镜）、CONTOUR（轮廓滤镜）、DETAIL（细节滤镜）、EDGE_ENHANCE（边界增强滤镜）、EDGE_ENHANCE_MORE（边界增强滤镜，程度更深）、EMBOSS（浮雕滤镜）等等。

```python
from PIL import Image, ImageFilter

image = Image.open('./test1.jpg')
image.filter(ImageFilter.CONTOUR).show()
```
使用轮廓滤镜效果如下图所示：
{% qnimg python_series/21-1.png %}

## 操作像素
```python
image = Image.open('./res/guido.jpg')
for x in range(80, 310):
    for y in range(20, 360):
        image.putpixel((x, y), (128, 128, 128))

image.show()
```

**完整的示例代码：** [python-learning](https://github.com/Hanpeng-Chen/python-learning/tree/master/21-%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86)

未完待续，持续更新中......