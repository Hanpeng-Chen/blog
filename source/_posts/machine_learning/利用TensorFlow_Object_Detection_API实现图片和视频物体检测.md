---
title: 利用TensorFlow Object Detection API实现图片和视频物体检测
urlname: Tensorflow-Object-Detection-image-video
tags:
  - TensorFlow
  - 物体检测
categories:
  - 机器学习
abbrlink: 42886
date: 2019-06-21 23:50:14
---
# TensorFlow Object Detection API介绍

物体检测是检测图片或视频中所出现的全部物体，并用矩形进行标注，物体的类别可以包括多种，比如：人、车、动物等等，即正确的答案可以是多个。

TensorFlow提供了用于检测图片或视频中所包含物体的接口(Object Detection API)，具体详情可参考下面链接：
https://github.com/tensorflow/models/tree/master/research/object_detection


这个API是用COCO数据集 (http://cocodataset.org/#home) 训练出来的，是一个大型的、丰富的物体检测，分割和字幕数据集,大约有30万张图像、90种最常见物体。


这个API提供了多种不同的，使用者可以通过设置不同检测边界范围来平衡运行速度和准确率。

<!-- ![Alt](/images/articles/2019/TensorFlow/tensorflow-object-detection-1.png) -->
{% qnimg TensorFlow/tensorflow-object-detection-1.png %}

图中的mAP（平均精度）是检测边界框的准确率和召回率的乘积。这是一个很好的混合测度，在评价模型对目标物体的敏锐度和它是否能很好避免虚假目标中非常好用。mAP值越高，模型的准确度越高，但运行速度会相应下降。


# 实现物体检测

## 环境

本文代码运行环境：Python3.6、jupyter notebook

首先安装相关依赖包
```
pip install jupyter
pip install tensorflow
pip install pillow  
pip install lxml
pip install matplotlib
pip install numpy
pip install opencv-python
```

## 图片物体检测:

### 1、加载库
```
import numpy as np
import tensorflow as tf
import matplotlib.pyplot as plt
from PIL import Image
```

### 2、从utils模块引入label_map_util和visualization_utils
label_map_util用于后面获取图像标签和类别，visualization_utils用于可视化
```
from utils import label_map_util
from utils import visualization_utils as vis_util
```

### 3、加载预训练好的模型
模型下载地址：
https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md
这里我们选用最轻量级的模型(ssd_mobilenet_v1_coco)。
```
PATH_TO_CKPT = 'ssd_mobilenet_v1_coco_2018_01_28/frozen_inference_graph.pb'
PATH_TO_LABELS = 'data/mscoco_label_map.pbtxt'
NUM_CLASSES = 90

detection_graph = tf.Graph()

with detection_graph.as_default():
    od_graph_def = tf.GraphDef()
    with tf.gfile.GFile(PATH_TO_CKPT, 'rb') as fid:
        od_graph_def.ParseFromString(fid.read())
        tf.import_graph_def(od_graph_def, name='')
```

### 4、加载分类标签数据
```
label_map = label_map_util.load_labelmap(PATH_TO_LABELS)
categories = label_map_util.convert_label_map_to_categories(label_map, max_num_classes=NUM_CLASSES, use_display_name=True)
category_index = label_map_util.create_category_index(categories)
```

### 5、核心代码：一个将图片转为数组的辅助函数，以及测试图片路径，使用模型进行物体检测：
```
def load_image_into_numpy_array(image):
    (im_width, im_height) = image.size
    return np.array(image.getdata()).reshape((im_height, im_width, 3)).astype(np.uint8)

TEST_IMAGE_PATHS = ['test_data/image1.jpg']

with detection_graph.as_default():
    with tf.Session(graph=detection_graph) as sess:
        image_tensor = detection_graph.get_tensor_by_name('image_tensor:0')
        detection_boxes = detection_graph.get_tensor_by_name('detection_boxes:0')
        detection_scores = detection_graph.get_tensor_by_name('detection_scores:0')
        detection_classes = detection_graph.get_tensor_by_name('detection_classes:0')
        num_detections = detection_graph.get_tensor_by_name('num_detections:0')
        for image_path in TEST_IMAGE_PATHS:
            image = Image.open(image_path)
            image_np = load_image_into_numpy_array(image)
            image_np_expanded = np.expand_dims(image_np, axis=0)
            (boxes, scores, classes, num) = sess.run(
                [detection_boxes, detection_scores, detection_classes, num_detections], 
                feed_dict={image_tensor: image_np_expanded})
            vis_util.visualize_boxes_and_labels_on_image_array(image_np, np.squeeze(boxes), np.squeeze(classes).astype(np.int32), np.squeeze(scores), category_index, use_normalized_coordinates=True, line_thickness=8)
            plt.figure(figsize=[12, 8])
            plt.imshow(image_np)
            plt.show()
```
### 检测结果如下：

<!-- ![Alt](/images/articles/2019/TensorFlow/output_image1.png) -->
{% qnimg TensorFlow/output_image1.png %}



## 视频物体检测

使用cv2读取视频并获取每一帧图片，然后将检测后的每一帧写入新的视频文件。

### 实现代码
```
import numpy as np
import tensorflow as tf
import cv2

from utils import label_map_util
from utils import visualization_utils as vis_util

cap = cv2.VideoCapture('test_data/test_video.mp4')
ret, image_np = cap.read()
out = cv2.VideoWriter('output_video.mp4', -1, cap.get(cv2.CAP_PROP_FPS), (image_np.shape[1], image_np.shape[0]))

PATH_TO_CKPT = 'ssd_mobilenet_v1_coco_2018_01_28/frozen_inference_graph.pb'
PATH_TO_LABELS = 'data/mscoco_label_map.pbtxt'
NUM_CLASSES = 90

detection_graph = tf.Graph()
with detection_graph.as_default():
    od_graph_def = tf.GraphDef()
    with tf.gfile.GFile(PATH_TO_CKPT, 'rb') as fid:
        od_graph_def.ParseFromString(fid.read())
        tf.import_graph_def(od_graph_def, name='')
        
label_map = label_map_util.load_labelmap(PATH_TO_LABELS)
categories = label_map_util.convert_label_map_to_categories(label_map, max_num_classes=NUM_CLASSES, use_display_name=True)
category_index = label_map_util.create_category_index(categories)

with detection_graph.as_default():
    with tf.Session(graph=detection_graph) as sess:
        image_tensor = detection_graph.get_tensor_by_name('image_tensor:0')
        detection_boxes = detection_graph.get_tensor_by_name('detection_boxes:0')
        detection_scores = detection_graph.get_tensor_by_name('detection_scores:0')
        detection_classes = detection_graph.get_tensor_by_name('detection_classes:0')
        num_detections = detection_graph.get_tensor_by_name('num_detections:0')
        while cap.isOpened():
            ret, image_np = cap.read()
            if len((np.array(image_np)).shape) == 0:
                break
            
            image_np = cv2.cvtColor(image_np, cv2.COLOR_BGR2RGB)
            image_np_expanded = np.expand_dims(image_np, axis=0)
            
            (boxes, scores, classes, num) = sess.run([detection_boxes, detection_scores, detection_classes, num_detections], feed_dict={image_tensor: image_np_expanded})
            
            vis_util.visualize_boxes_and_labels_on_image_array(image_np, np.squeeze(boxes), np.squeeze(classes).astype(np.int32), np.squeeze(scores), category_index, use_normalized_coordinates=True, line_thickness=8)
            out.write(cv2.cvtColor(image_np, cv2.COLOR_RGB2BGR))
            
cap.release()
out.release()
cv2.destroyAllWindows()
```

### 检测效果

<!-- ![Alt](/images/articles/2019/TensorFlow/video_output_image.png) -->
{% qnimg TensorFlow/video_output_image.png %}


至此我们利用tensorflow提供的物体检测API，实现了图片和视频的物体检测。

完整的代码和检测后的视频，请至github查看：
https://github.com/Hanpeng-Chen/tensorflow-learning
