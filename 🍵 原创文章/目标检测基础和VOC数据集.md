![](https://tva1.sinaimg.cn/large/0081Kckwly1glxkzcji5ij319y0gojtj.jpg)

## 一、目标检测基本概念

### 1. 什么是目标检测
目标检测是计算机视觉中的一个重要任务，近年来传统目标检测方法已经难以满足人们对目标检测效果的要求，随着深度学习在计算机视觉任务上取得的巨大进展，目前基于深度学习的目标检测算法已经成为主流。

相比较于基于深度学习的图像分类任务，目标检测任务更具难度，具体区别如下图所示。

**图像分类**：只需要判断输入的图像中是否包含感兴趣物体。

**目标检测**：需要在识别出图片中目标类别的基础上，还要精确定位到目标的具体位置，并用外接矩形框标出。
![分类和目标检测任务示意图](https://img-blog.csdnimg.cn/20201216195033640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)

### 2. 目标检测的思路
自2012年Alex Krizhevsky凭借Alex在ImageNet图像分类挑战赛中拿下冠军之后，深度学习在图像识别尤其是图像分类领域开始大放异彩，大众的视野也重新回到深度神经网络中。紧接着，不断有更深更复杂的网络出现，一再刷新ImageNet图像分类比赛的记录。

大家发现，通过合理的构造，神经网络可以用来预测各种各样的实际问题。于是人们开始了基于CNN的目标检测研究, 但是随着进一步的探索大家发现，似乎CNN并不善于直接预测坐标信息。并且一幅图像中可能出现的物体个数也是不定的，模型如何构建也比较棘手。

因此，人们就想，如果知道了图中某个位置存在物体，再将对应的局部区域送入到分类网络中去进行判别，那我不就可以知道图像中每个物体的位置和类别了吗？

但是，怎么样才能知道每个物体的位置呢？显然我们是没办法知道的，但是我们可以去猜啊！所谓猜，其实就是通过滑窗的方式，罗列图中各种可能的区域，一个个去试，分别送入到分类网络进行分类得到其类别，同时我们会对当前的边界框进行微调，这样对于图像中每个区域都能得到（class,x1,y1,x2,y2）五个属性，汇总后最终就得到了图中物体的类别和坐标信息。

总结一下我们的这种方案思路：先确立众多候选框，再对候选框进行分类和微调。

观察下图，更形象的理解下这种思想：

![从分类角度去看目标检测](https://img-blog.csdnimg.cn/20201216195316151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)


### 3. 目标框定义方式
任何图像任务的训练数据都要包括两项，图片和真实标签信息，通常叫做GT。

图像分类中，标签信息是类别。目标检测的标签信息除了类别label以外，需要同时包含目标的位置信息，也就是目标的外接矩形框bounding box。

用来表达bbox的格式通常有两种，(x1, y1, x2, y2) 和 (c_x, c_y, w, h) ，如图所示：

![目标框定义方式](https://img-blog.csdnimg.cn/20201216195446670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)


之所以使用两种不同的目标框信息表达格式，是因为两种格式会分别在后续不同场景下更加便于计算。

两种格式互相转换的实现在utils.py中，代码也非常简单：

```python
def xy_to_cxcy(xy):
    """
    Convert bounding boxes from boundary coordinates (x_min, y_min, x_max, y_max) to center-size coordinates (c_x, c_y, w, h).

    :param xy: bounding boxes in boundary coordinates, a tensor of size (n_boxes, 4)
    :return: bounding boxes in center-size coordinates, a tensor of size (n_boxes, 4)
    """
    return torch.cat([(xy[:, 2:] + xy[:, :2]) / 2,  # c_x, c_y
                      xy[:, 2:] - xy[:, :2]], 1)  # w, h


def cxcy_to_xy(cxcy):
    """
    Convert bounding boxes from center-size coordinates (c_x, c_y, w, h) to boundary coordinates (x_min, y_min, x_max, y_max).

    :param cxcy: bounding boxes in center-size coordinates, a tensor of size (n_boxes, 4)
    :return: bounding boxes in boundary coordinates, a tensor of size (n_boxes, 4)
    """
    return torch.cat([cxcy[:, :2] - (cxcy[:, 2:] / 2),  # x_min, y_min
                      cxcy[:, :2] + (cxcy[:, 2:] / 2)], 1)  # x_max, y_max
```
用torch.cat()将两个形状为(n,2)的tensor在第一维度拼接成(n,4)。


### 4. 交并比（IoU）
在目标检测任务中，关于IOU的计算贯穿整个模型的训练测试和评价过程，是非常非常重要的一个概念，其目的是用来衡量两个目标框的重叠程度。

IoU的全称是交并比（Intersection over Union），表示两个目标框的交集占其并集的比例。下图为IOU计算示意图:

![IOU计算示意图](https://img-blog.csdnimg.cn/20201216195805421.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)


图中可以看到，分子中黄色区域为红bbox和绿bbox的交集，分母中黄+红+绿区域为红bbox和绿bbox的并集，两者之比即为iou。

那么具体怎么去计算呢？这里给出计算流程的简述:

- 首先获取两个框的坐标，红框坐标: 左上(red_x1, red_y1), 右下(red_x2, red_y2)，绿框坐标: 左上(green_x1, green_y1)，右下(green_x2, green_y2)
- 计算两个框左上点的坐标最大值:(max(red_x1, green_x1), max(red_y1, green_y1)), 和右下点坐标最小值:(min(red_x2, green_x2), min(red_y2, green_y2))
- 利用2算出的信息计算黄框面积：yellow_area
- 计算红绿框的面积：red_area 和 green_area
- iou = yellow_area / (red_area + green_area - yellow_area)

如果文字表述的不够清晰，就再看下代码：

```python
def find_intersection(set_1, set_2):
    """ 
    Find the intersection of every box combination between two sets of boxes that are in boundary coordinates.

    :param set_1: set 1, a tensor of dimensions (n1, 4)                                                                                                           
    :param set_2: set 2, a tensor of dimensions (n2, 4)
    :return: intersection of each of the boxes in set 1 with respect to each of the boxes in set 2, a tensor of dimensions (n1, n2)
    """

    # PyTorch auto-broadcasts singleton dimensions
    lower_bounds = torch.max(set_1[:, :2].unsqueeze(1), set_2[:, :2].unsqueeze(0))  # (n1, n2, 2)
    upper_bounds = torch.min(set_1[:, 2:].unsqueeze(1), set_2[:, 2:].unsqueeze(0))  # (n1, n2, 2)
    intersection_dims = torch.clamp(upper_bounds - lower_bounds, min=0)  # (n1, n2, 2)
    return intersection_dims[:, :, 0] * intersection_dims[:, :, 1]  # (n1, n2)


def find_jaccard_overlap(set_1, set_2):
    """ 
    Find the Jaccard Overlap (IoU) of every box combination between two sets of boxes that are in boundary coordinates.

    :param set_1: set 1, a tensor of dimensions (n1, 4)
    :param set_2: set 2, a tensor of dimensions (n2, 4)
    :return: Jaccard Overlap of each of the boxes in set 1 with respect to each of the boxes in set 2, a tensor of dimensions (n1, n2)
    """

    # Find intersections
    intersection = find_intersection(set_1, set_2)  # (n1, n2)

    # Find areas of each box in both sets
    areas_set_1 = (set_1[:, 2] - set_1[:, 0]) * (set_1[:, 3] - set_1[:, 1])  # (n1)
    areas_set_2 = (set_2[:, 2] - set_2[:, 0]) * (set_2[:, 3] - set_2[:, 1])  # (n2)

    # Find the union
    # PyTorch auto-broadcasts singleton dimensions
    union = areas_set_1.unsqueeze(1) + areas_set_2.unsqueeze(0) - intersection  # (n1, n2)

    return intersection / union  # (n1, n2)
```
以上代码位于utils.py脚本的find_intersection和find_jaccard_overlap。

- **函数 `find_intersection`**
    `find_intersection(set_1, set_2) `是求形状为 (n1,4) 和 (n2,4) 的boxes的交集的面积。``set_1[:, :2]``的形状为(n1,2)，后面加上`.unsqueeze(1)`，形状变为(n1,1,2)。同理`set_2[:, :2].unsqueeze(0) `,形状为(1,n2,2)。

    (n1,1,2)和(1,n2,2)，作了torch.max,有广播存在，(n1,1,2)变成(n1,n2,2) ,（1,n2,2）也变成(n1,n2,2)。因此得到了形状为(n1,n2,2)的框的左上角坐标 那个2 就是储存了x1,y1。

    `torch.clamp()`是将函数限制在最大值和最小值范围内，如果超过就变成那个最大值或者最小值。
    这里min=0，意思是如果面积小于0,那么面积取0（排除异常）。

- **函数`find_jaccard_overlap`**
    计算iou，交集/并集，最后union计算,  升维 (n1)->(n1,1)    、  (n2)->(1,n2)   、 接下去相加，广播成(n1,n2),减去一个（n1,n2）的交集面积，得到并集面积。

### 5. 小结
以上便是本小节的全部内容了。

本小节我们首先介绍了目标检测的问题背景，随后分析了一个实现目标检测的解决思路，这也是众多经典检测网络和本章要介绍的模型所采用的思路(即先确立众多候选框，再对候选框进行分类和微调)。最后介绍了bbox和IoU这两个目标检测相关的基本概念。

下一小节，我们将会从数据入手，介绍下目标检测领域最常见的一个数据集VOC，以及数据读取相关的代码。

## 二、 目标检测数据集VOC
VOC数据集是目标检测领域最常用的标准数据集之一，几乎所有检测方向的论文，如faster_rcnn、yolo、SSD等都会给出其在VOC数据集上训练并评测的效果。因此我们我们的教程也基于VOC来开展实验，具体地，我们使用VOC2007和VOC2012这两个最流行的版本作为训练和测试的数据。

### **1. 数据集类别**

VOC数据集在类别上可以分为4大类，20小类，其类别信息下图所示。

![VOC数据集目标类别划分](https://img-blog.csdnimg.cn/20201216200236151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)

### **2. 数据集量级**

VOC数量集图像和目标数量的基本信息如下图所示:

![VOC数据集数据量级对比](https://img-blog.csdnimg.cn/20201216200308944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)

其中，Images表示图片数量，Objects表示目标数量

### **3. 数据集下载**

VOC官网经常上不去，为确保后续实验准确且顺利的进行，大家可以点击这里的百度云链接进行下载：

链接地址：https://pan.baidu.com/s/1SM2M6S366igDriiCWGyhDg   解压码：7aek

下载后放到dataset目录下解压即可。

下面是通过官网下载的步骤：

- 进入VOC官网链接：[http://host.robots.ox.ac.uk/pascal/VOC/](http://host.robots.ox.ac.uk/pascal/VOC/).

- 在下图所示区域找到历年VOC挑战赛链接，比如选择VOC2012.

    ![VOC官网页面](https://img-blog.csdnimg.cn/20201216200609929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)



- 在VOC2012页面，找到下图所示区域，点击下载即可。

    ![VOC2012数据集下载页面](https://img-blog.csdnimg.cn/20201216200640903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)

- VOC2007同理进行下载即可。

### **4. 数据集说明**

将下载得到的压缩包解压，可以得到如图3-9所示的一系列文件夹，由于VOC数据集不仅被拿来做目标检测，也可以拿来做分割等任务，因此除了目标检测所需的文件之外，还包含分割任务所需的文件，比如SegmentationClass,SegmentationObject，这里，我们主要对目标检测任务涉及到的文件进行介绍。

![VOC压缩包解压所得文件夹示例](https://img-blog.csdnimg.cn/20201216200720938.png#pic_center)

- JPEGImages：这个文件夹中存放所有的图片，包括训练验证测试用到的所有图片。
- ImageSets：这个文件夹中包含三个子文件夹，Layout、Main、Segmentation；Layout文件夹中存放的是train，valid，test和train+valid数据集的文件名
- Segmentation：文件夹中存放的是分割所用train，valid，test和train+valid数据集的文件名
- Main：文件夹中存放的是各个类别所在图片的文件名，比如cow_val，表示valid数据集中，包含有cow类别目标的图片名称。
- Annotations：Annotation文件夹中存放着每张图片相关的标注信息，以xml格式的文件存储，可以通过记事本或者浏览器打开，我们以000001.jpg这张图片为例说明标注文件中各个属性的含义。
    ![VOC数据集000001.jpg图片（左）和标注信息（右）](https://img-blog.csdnimg.cn/2020121620080890.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)


猛一看去，内容又多又复杂，其实仔细研究一下，只有红框区域内的内容是我们真正需要关注的。

- filename：图片名称

- size：图片宽高

- depth表示图片通道数

- object：表示目标，包含下面两部分内容。

    首先是目标类别name为dog，pose表示目标姿势为left，truncated表示是否是一个被截断的目标，1表示是，0表示不是，在这个例子中，只露出狗头部分，所以truncated为1。difficult为0表示此目标不是一个难以识别的目标。

    然后就是目标的bbox信息，可以看到，这里是以[xmin,ymin,xmax,ymax]格式进行标注的，分别表示dog目标的左上角和右下角坐标。

- 一张图片中有多少需要识别的目标，其xml文件中就有多少个object。上面的例子中有两个object，分别对应人和狗。

## 三、VOC数据集的dataloader的构建
### **1. 数据集准备**

根据上面的介绍可以看出，VOC数据集的存储格式还是比较复杂的，为了后面训练中的读取代码更加简洁，这里我们准备了一个预处理脚本create_data_lists.py。

该脚本的作用是进行一系列的数据准备工作，主要是提前将记录标注信息的xml文件(Annotations)进行解析，并将信息整理到json文件之中，这样在运行训练脚本时，只需简单的从json文件中读取已经按想要的格式存储好的标签信息即可。

注: 这样的预处理并不是必须的，和算法或数据集本身均无关系，只是取决于开发者的代码习惯，不同检测框架的处理方法也是不一致的。

可以看到，create_data_lists.py脚本仅有几行代码，其内部调用了utils.py中的create_data_lists方法：

```python
"""python
    create_data_lists
"""
from utils import create_data_lists

if __name__ == '__main__':
    # voc07_path，voc12_path为我们训练测试所需要用到的数据集，output_folder为我们生成构建dataloader所需文件的路径
    # 参数中涉及的路径以个人实际路径为准，建议将数据集放到dataset目录下，和教程保持一致
    create_data_lists(voc07_path='../../../dataset/VOCdevkit/VOC2007',
                      voc12_path='../../../dataset/VOCdevkit/VOC2012',
                      output_folder='../../../dataset/VOCdevkit')
```
设置好对应路径后，我们运行数据集准备脚本:

`tiny_detector_demo$ python create_data_lists.py`

很快啊！dataset/VOCdevkit目录下就生成了若干json文件，这些文件会在后面训练中真正被用到。

不妨手动打开这些json文件，看下都记录了哪些信息。

下面来介绍一下parse_annotation函数内部都做了什么，json中又记录了哪些信息。这部分作为选学，不感兴趣可以跳过，只要你已经明确了json中记录的信息的含义。

代码阅读可以参照注释，建议配下图一起食用：

```python
"""python
    xml文件解析
"""

import json
import os
import torch
import random
import xml.etree.ElementTree as ET    #解析xml文件所用工具
import torchvision.transforms.functional as FT

#GPU设置
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# Label map
#voc_labels为VOC数据集中20类目标的类别名称
voc_labels = ('aeroplane', 'bicycle', 'bird', 'boat', 'bottle', 'bus', 'car', 'cat', 'chair', 'cow', 'diningtable',
              'dog', 'horse', 'motorbike', 'person', 'pottedplant', 'sheep', 'sofa', 'train', 'tvmonitor')

#创建label_map字典，用于存储类别和类别索引之间的映射关系。比如：{1：'aeroplane'， 2：'bicycle'，......}
label_map = {k: v + 1 for v, k in enumerate(voc_labels)}
#VOC数据集默认不含有20类目标中的其中一类的图片的类别为background，类别索引设置为0
label_map['background'] = 0

#将映射关系倒过来，{类别名称：类别索引}
rev_label_map = {v: k for k, v in label_map.items()}  # Inverse mapping

#解析xml文件，最终返回这张图片中所有目标的标注框及其类别信息，以及这个目标是否是一个difficult目标
def parse_annotation(annotation_path):
    #解析xml
    tree = ET.parse(annotation_path)
    root = tree.getroot()

    boxes = list()    #存储bbox
    labels = list()    #存储bbox对应的label
    difficulties = list()    #存储bbox对应的difficult信息

    #遍历xml文件中所有的object，前面说了，有多少个object就有多少个目标
    for object in root.iter('object'):
        #提取每个object的difficult、label、bbox信息
        difficult = int(object.find('difficult').text == '1')
        label = object.find('name').text.lower().strip()
        if label not in label_map:
            continue
        bbox = object.find('bndbox')
        xmin = int(bbox.find('xmin').text) - 1
        ymin = int(bbox.find('ymin').text) - 1
        xmax = int(bbox.find('xmax').text) - 1
        ymax = int(bbox.find('ymax').text) - 1
        #存储
        boxes.append([xmin, ymin, xmax, ymax])
        labels.append(label_map[label])
        difficulties.append(difficult)

    #返回包含图片标注信息的字典
    return {'boxes': boxes, 'labels': labels, 'difficulties': difficulties}
```
- **为什么得到的新坐标减1？**
    VOC的矩形标注坐标是以1为基准的(1-based),而我们在处理图像坐标都是0起始的(0-based)。

    所以在这里才要对从xml文件中读取的xmin,ymin,xmax,ymax 统统减1将坐标变为我们做数据处理时所需要的0-based坐标。

- **返回值的形状**
    boxes (n,4) 的list，label (n) 的list，返回的都是标签对应的数字。difficulties （n）的list，返回的只有0或1。

  看了上面的代码如果还不太明白，试试结合这张图理解下：

![xml解析流程图](https://img-blog.csdnimg.cn/20201216201326596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)

```python
"""python
    分别读取train和valid的图片和xml信息，创建用于训练和测试的json文件
"""
def create_data_lists(voc07_path, voc12_path, output_folder):
    """
    Create lists of images, the bounding boxes and labels of the objects in these images, and save these to file.
    :param voc07_path: path to the 'VOC2007' folder
    :param voc12_path: path to the 'VOC2012' folder
    :param output_folder: folder where the JSONs must be saved
    """

    #获取voc2007和voc2012数据集的绝对路径
    voc07_path = os.path.abspath(voc07_path)
    voc12_path = os.path.abspath(voc12_path)

    train_images = list()
    train_objects = list()
    n_objects = 0

    # Training data
    for path in [voc07_path, voc12_path]:

        # Find IDs of images in training data
        #获取训练所用的train和val数据的图片id
        with open(os.path.join(path, 'ImageSets/Main/trainval.txt')) as f:
            ids = f.read().splitlines()

        #根据图片id，解析图片的xml文件，获取标注信息
        for id in ids:
            # Parse annotation's XML file
            objects = parse_annotation(os.path.join(path, 'Annotations', id + '.xml'))
            if len(objects['boxes']) == 0:    #如果没有目标则跳过
                continue
            n_objects += len(objects)        #统计目标总数
            train_objects.append(objects)    #存储每张图片的标注信息到列表train_objects
            train_images.append(os.path.join(path, 'JPEGImages', id + '.jpg'))    #存储每张图片的路径到列表train_images，用于读取图片

    assert len(train_objects) == len(train_images)        #检查图片数量和标注信息量是否相等，相等才继续执行程序

    # Save to file
    #将训练数据的图片路径，标注信息，类别映射信息，分别保存为json文件
    with open(os.path.join(output_folder, 'TRAIN_images.json'), 'w') as j:
        json.dump(train_images, j)
    with open(os.path.join(output_folder, 'TRAIN_objects.json'), 'w') as j:
        json.dump(train_objects, j)
    with open(os.path.join(output_folder, 'label_map.json'), 'w') as j:
        json.dump(label_map, j)  # save label map too

    print('\nThere are %d training images containing a total of %d objects. Files have been saved to %s.' % (
        len(train_images), n_objects, os.path.abspath(output_folder)))


    #与Train data一样，目的是将测试数据的图片路径，标注信息，类别映射信息，分别保存为json文件，参考上面的注释理解
    # Test data
    test_images = list()
    test_objects = list()
    n_objects = 0

    # Find IDs of images in the test data
    with open(os.path.join(voc07_path, 'ImageSets/Main/test.txt')) as f:
        ids = f.read().splitlines()

    for id in ids:
        # Parse annotation's XML file
        objects = parse_annotation(os.path.join(voc07_path, 'Annotations', id + '.xml'))
        if len(objects) == 0:
            continue
        test_objects.append(objects)
        n_objects += len(objects)
        test_images.append(os.path.join(voc07_path, 'JPEGImages', id + '.jpg'))

    assert len(test_objects) == len(test_images)

    # Save to file
    with open(os.path.join(output_folder, 'TEST_images.json'), 'w') as j:
        json.dump(test_images, j)
    with open(os.path.join(output_folder, 'TEST_objects.json'), 'w') as j:
        json.dump(test_objects, j)

    print('\nThere are %d test images containing a total of %d objects. Files have been saved to %s.' % (
        len(test_images), n_objects, os.path.abspath(output_folder)))
```
同时加载voc07，voc12两个数据集，`ids = f.read().splitlines()`是把文件名以列表形式存储。设图片数量为n,每张图片中的object数为m(非固定)。

- TRAIN_images.json 是列表，长度为n,装着是图片的绝对路径
- TRAIN_objects.json 是列表，长度为n,装着n个字典，字典里有键
- boxes (m,4) , label (m) , difficulties (m)    #括号里都是形状


同样，建议配图食用：

![数据准备流程图（以train_dataset为例）](https://img-blog.csdnimg.cn/20201216201429735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNTM5Mzc1,size_16,color_FFFFFF,t_70#pic_center)

到这里，我们的训练数据就准备好了，接下来开始一步步构建训练所需的dataloader吧！

### **2. 构建dataloader**

在这里，我们假设你对Pytorch的 Dataset 和 DataLoader 两个概念有最基本的了解。

下面开始介绍构建dataloader的相关代码:

首先了解一下训练的时候在哪里定义了dataloader以及是如何定义的。

以下是train.py中的部分代码段：

```python
 #train_dataset和train_loader的实例化
  train_dataset = PascalVOCDataset(data_folder,
                                   split='train',
                                   keep_difficult=keep_difficult)
  train_loader = torch.utils.data.DataLoader(train_dataset, batch_size=batch_size, shuffle=True,collate_fn=train_dataset.collate_fn, num_workers=workers,pin_memory=True) 
  # note that we're passing the collate function here
```
可以看到，首先需要实例化PascalVOCDataset类得到train_dataset，然后将train_dataset传入`torch.utils.data.DataLoader`，进而得到train_loader。

pin_memory就是锁页内存,创建DataLoader时，设置pin_memory=True，则意味着生成的Tensor数据最开始是属于内存中的锁页内存，这样将内存的Tensor转义到GPU的显存就会更快一些。显卡不好就不要开了。

collate_fn是如何将(C,H,W)组合成(N,C,H,W)的方式。

接下来看一下PascalVOCDataset是如何定义的。

代码位于 datasets.py 脚本中，可以看到，PascalVOCDataset继承了torch.utils.data.Dataset，然后重写了__init__ , __getitem__, __len__ 和 **collate_fn** 四个方法，这也是我们在构建自己的dataset的时候需要经常做的工作，配合下面注释理解代码：

```python
"""python
    PascalVOCDataset具体实现过程
"""
import torch
from torch.utils.data import Dataset
import json
import os
from PIL import Image
from utils import transform


class PascalVOCDataset(Dataset):
    """
    A PyTorch Dataset class to be used in a PyTorch DataLoader to create batches.
    """

    #初始化相关变量
    #读取images和objects标注信息
    def __init__(self, data_folder, split, keep_difficult=False):
        """
        :param data_folder: folder where data files are stored
        :param split: split, one of 'TRAIN' or 'TEST'
        :param keep_difficult: keep or discard objects that are considered difficult to detect?
        """
        self.split = split.upper()    #保证输入为纯大写字母，便于匹配{'TRAIN', 'TEST'}

        assert self.split in {'TRAIN', 'TEST'}

        self.data_folder = data_folder
        self.keep_difficult = keep_difficult

        # Read data files
        with open(os.path.join(data_folder, self.split + '_images.json'), 'r') as j:
            self.images = json.load(j)
        with open(os.path.join(data_folder, self.split + '_objects.json'), 'r') as j:
            self.objects = json.load(j)

        assert len(self.images) == len(self.objects)

    #循环读取image及对应objects
    #对读取的image及objects进行tranform操作（数据增广）
    #返回PIL格式图像，标注框，标注框对应的类别索引，对应的difficult标志(True or False)
    def __getitem__(self, i):
        # Read image
        #*需要注意，在pytorch中，图像的读取要使用Image.open()读取成PIL格式，不能使用opencv
        #*由于Image.open()读取的图片是四通道的(RGBA)，因此需要.convert('RGB')转换为RGB通道
        image = Image.open(self.images[i], mode='r')
        image = image.convert('RGB')

        # Read objects in this image (bounding boxes, labels, difficulties)
        objects = self.objects[i]
        boxes = torch.FloatTensor(objects['boxes'])  # (n_objects, 4)
        labels = torch.LongTensor(objects['labels'])  # (n_objects)
        difficulties = torch.ByteTensor(objects['difficulties'])  # (n_objects)

        # Discard difficult objects, if desired
        #如果self.keep_difficult为False,即不保留difficult标志为True的目标
        #那么这里将对应的目标删去
        if not self.keep_difficult:
            boxes = boxes[(1 - difficulties).bool()]  #uint8可以作为索引,但是转成bool去索引更好
            labels = labels[(1 - difficulties).bool()]
            difficulties = difficulties[(1 - difficulties).bool()]

        # Apply transformations
        #对读取的图片应用transform
        image, boxes, labels, difficulties = transform(image, boxes, labels, difficulties, split=self.split)

        return image, boxes, labels, difficulties

    #获取图片的总数，用于计算batch数
    def __len__(self):
        return len(self.images)

    #我们知道，我们输入到网络中训练的数据通常是一个batch一起输入，而通过__getitem__我们只读取了一张图片及其objects信息
    #如何将读取的一张张图片及其object信息整合成batch的形式呢？
    #collate_fn就是做这个事情，
    #对于一个batch的images，collate_fn通过torch.stack()将其整合成4维tensor，对应的objects信息分别用一个list存储
    def collate_fn(self, batch):
        """
        Since each image may have a different number of objects, we need a collate function (to be passed to the DataLoader).
        This describes how to combine these tensors of different sizes. We use lists.
        Note: this need not be defined in this Class, can be standalone.
        :param batch: an iterable of N sets from __getitem__()
        :return: a tensor of images, lists of varying-size tensors of bounding boxes, labels, and difficulties
        """

        images = list()
        boxes = list()
        labels = list()
        difficulties = list()

        for b in batch:
            images.append(b[0])
            boxes.append(b[1])
            labels.append(b[2])
            difficulties.append(b[3])

        #(3,224,224) -> (N,3,224,224)
        images = torch.stack(images, dim=0)

        return images, boxes, labels, difficulties  # tensor (N, 3, 224, 224), 3 lists of N tensors each
```
### **3. 关于数据增强**

到这里为止，我们的dataset就算是构建好了，已经可以传给torch.utils.data.DataLoader来获得用于输入网络训练的数据了。但是不急，构建dataset中有个很重要的一步我们上面只是提及了一下，那就是transform操作(数据增强)，也就是这一行代码

```python
image, boxes, labels, difficulties = transform(image, boxes, labels, difficulties, split=self.split)
```
这部分比较重要，但是涉及代码稍多，对于基础较薄弱的伙伴可以作为选学内容，后面再认真读代码。你只需知道，同分类网络一样，训练目标检测网络同样需要进行数据增强，这对提升网络精度和泛化能力很有帮助。

需要注意的是，涉及位置变化的数据增强方法，同样需要对目标框进行一致的处理，因此目标检测框架的数据处理这部分的代码量通常都不小，且比较容易出bug。这里为了降低代码的难度，我们只是使用了几种比较简单的数据增强。

transform 函数的具体代码实现位于 utils.py 中，下面简单进行讲解：

```python
"""python
    transform操作是训练模型中一项非常重要的工作，其中不仅包含数据增强以提升模型性能的相关操作，也包含如数据类型转换(PIL to Tensor)、归一化(Normalize)这些必要操作。
"""
import json
import os
import torch
import random
import xml.etree.ElementTree as ET
import torchvision.transforms.functional as FT

"""
可以看到，transform分为TRAIN和TEST两种模式，以本实验为例：

在TRAIN时进行的transform有：
1.以随机顺序改变图片亮度，对比度，饱和度和色相，每种都有50％的概率被执行。photometric_distort
2.扩大目标，expand
3.随机裁剪图片，random_crop
4.0.5的概率进行图片翻转，flip
*注意：a. 第一种transform属于像素级别的图像增强，目标相对于图片的位置没有改变，因此bbox坐标不需要变化。
         但是2，3，4，5都属于图片的几何变化，目标相对于图片的位置被改变，因此bbox坐标要进行相应变化。

在TRAIN和TEST时都要进行的transform有：
1.统一图像大小到(224,224)，resize
2.PIL to Tensor
3.归一化，FT.normalize()

注1: resize也是一种几何变化，要知道应用数据增强策略时，哪些属于几何变化，哪些属于像素变化
注2: PIL to Tensor操作，normalize操作必须执行
"""

def transform(image, boxes, labels, difficulties, split):
    """
    Apply the transformations above.
    :param image: image, a PIL Image
    :param boxes: bounding boxes in boundary coordinates, a tensor of dimensions (n_objects, 4)
    :param labels: labels of objects, a tensor of dimensions (n_objects)
    :param difficulties: difficulties of detection of these objects, a tensor of dimensions (n_objects)
    :param split: one of 'TRAIN' or 'TEST', since different sets of transformations are applied
    :return: transformed image, transformed bounding box coordinates, transformed labels, transformed difficulties
    """

    #在训练和测试时使用的transform策略往往不完全相同，所以需要split变量指明是TRAIN还是TEST时的transform方法
    assert split in {'TRAIN', 'TEST'}

    # Mean and standard deviation of ImageNet data that our base VGG from torchvision was trained on
    # see: https://pytorch.org/docs/stable/torchvision/models.html
    #为了防止由于图片之间像素差异过大而导致的训练不稳定问题，图片在送入网络训练之间需要进行归一化
    #对所有图片各通道求mean和std来获得
    mean = [0.485, 0.456, 0.406]
    std = [0.229, 0.224, 0.225]

    new_image = image
    new_boxes = boxes
    new_labels = labels
    new_difficulties = difficulties

    # Skip the following operations for evaluation/testing
    if split == 'TRAIN':
        # A series of photometric distortions in random order, each with 50% chance of occurrence, as in Caffe repo
        new_image = photometric_distort(new_image)

        # Convert PIL image to Torch tensor
        new_image = FT.to_tensor(new_image)

        # Expand image (zoom out) with a 50% chance - helpful for training detection of small objects
        # Fill surrounding space with the mean of ImageNet data that our base VGG was trained on
        if random.random() < 0.5:
            new_image, new_boxes = expand(new_image, boxes, filler=mean)

        # Randomly crop image (zoom in)
        new_image, new_boxes, new_labels, new_difficulties = random_crop(new_image, new_boxes, new_labels,new_difficulties)

        # Convert Torch tensor to PIL image
        new_image = FT.to_pil_image(new_image)

        # Flip image with a 50% chance
        if random.random() < 0.5:
            new_image, new_boxes = flip(new_image, new_boxes)

    # Resize image to (224, 224) - this also converts absolute boundary coordinates to their fractional form
    new_image, new_boxes = resize(new_image, new_boxes, dims=(224, 224))

    # Convert PIL image to Torch tensor
    new_image = FT.to_tensor(new_image)

    # Normalize by mean and standard deviation of ImageNet data that our base VGG was trained on
    new_image = FT.normalize(new_image, mean=mean, std=std)

    return new_image, new_boxes, new_labels, new_difficulties
```
**TRAIN transform的步骤：**

- 颜色变化、to_tensor(变形(CHW),归一化，pil变tensor)
- 创建一个背景并把图放上去(等效缩小图片)
- 随机裁剪图片(丢失了部分框)
- 转为pil、随机左右翻转、resize(这里面对boxes做了归一化处理)
- 再变tensor、标准化处理

### **4. 构建DataLoade**

至此，我们已经将VOC数据转换成了dataset，接下来可以用来创建dataloader，这部分pytorch已经帮我们实现好了，我们只需将创建好的dataset送入即可，注意理解相关参数。

```python
"""python
    DataLoader
"""
#参数说明：
#在train时一般设置shufle=True打乱数据顺序，增强模型的鲁棒性
#num_worker表示读取数据时的线程数，一般根据自己设备配置确定（如果是windows系统，建议设默认值0，防止出错）
#pin_memory，在计算机内存充足的时候设置为True可以加快内存中的tensor转换到GPU的速度，具体原因可以百度哈~
train_loader = torch.utils.data.DataLoader(train_dataset, batch_size=batch_size, shuffle=True,collate_fn=train_dataset.collate_fn, num_workers=workers, pin_memory=True)  # note that we're passing the collate function here
```
## 小结

到这里，这一小节的内容就介绍完了，回顾一下，我们首先介绍了VOC数据集的基本信息以及如何下载，随后我们介绍了和读取VOC数据集的相关代码。万事俱备，只欠模型～