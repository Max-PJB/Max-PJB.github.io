---
title: NMS的python实现
categories:
  - NMS
tags:
  - NMS
  - ML
  - python
---

# NMS的python实现

### IOU交并比（Intersection-over-Union，IoU）

在目标检测领域，通常指在图片中两个方框面积之间的交并比。

给定两个方框C，G。

![image-20200917132025017](/public/img/image-20200917132025017.png)

则有： $  IOU(C,G) = \frac{area(C)\bigcap area(G)}{area(C)\bigcup area(G)} $ 

其中area(C)代表方框C的面积，其它同理。 

![image-20200917132143617](/public/img/image-20200917132143617.png)

### IOU 的计算

我们首先进行分析下交集和并集到底应该怎么计算：我们首先需要计算交集，然后并集通过两个边框的面积的和减去交集部分即为并集，因此 IoU 的计算的难点在于交集的计算。

为了计算交集，我们脑子里**首先想到的方法**应该是：考虑两个边框的相对位置，然后按照相对位置（左上，左下，右上，右下，包含，互不相交）分情况讨论，来计算交集。

![在这里插入图片描述](/public/img/20180923221316210)

上图就是直觉，这样想没有错。但计算一个交集，就要分多种情况讨论，要是程序真的按照这逻辑编写就太搞笑了。因此对这个问题进行进一步地研究显得十分有必要。

让我们**重新思考**一下两个框交集的计算。两个框交集的计算的实质是两个集合交集的计算，因此我们可以将两个框的交集的计算简化为：

![在这里插入图片描述](/public/img/20180924130149184)

想象一下，把这两个框框投影在 X 轴上~~

通过简化，我们可以清晰地看到，交集计算的关键是交集上下界点（图中蓝点）的计算。

我们假设集合 A 为 $ [x_{1}，x_{2}] $，集合 B 为  $ [y_{1}，y_{2}] $ 。然后我们来求AB交集的上下界限。

交集计算的逻辑

- 交集下界 $ z_{1}：{max}(x_{1}, y_{1}) $ 
- 交集上界 $ z_{2}：{min}(x_{2}, y_{2}) $
- 如果 $ z_{2}-z_{1} $  小于0，则说明集合 A 和集合 B 没有交集。

```python
'''
Input: 给除左上角的点坐标+长高
p_x=[x1,y1,w1,h1]
p_y=[x2,y2,w2,h2]

'''

def IoU(p_x,p_y):
    area_x = p_x[2]*p_x[3]
    area_y = p_y[2]*p_y[3]
    
    x_tl = max(p_x[0],p_y[0])
    y_tl = max(p_x[1],p_y[1])
    x_br = min(p_x[0]+p_x[2],p_y[0]+p_y[2])
    y_br = min(p_x[1]+p_x[3],p_y[1]+p_y[3])

    inter_w = max(x_br-x_tl,0)   # 与 0 比较是为了防止两个不相交的情况
    inter_h = max(y_br-y_tl,0)   # 与 0 比较是为了防止两个不相交的情况

    inter_area = inter_h*inter_w

    return inter_area*1.0 / (area_x+area_y - inter_area)
```

再看一个基于TensorFlow 的 IOU实现：

```python
import tensorflow as tf

def IoU_calculator(x, y, w, h, l_x, l_y, l_w, l_h):
    """calaulate IoU
    Args:
      x: net predicted x
      y: net predicted y
      w: net predicted width
      h: net predicted height
      l_x: label x
      l_y: label y
      l_w: label width
      l_h: label height
    
    Returns:
      IoU
    """
    
    # convert to coner
    x_max = x + w/2
    y_max = y + h/2
    x_min = x - w/2
    y_min = y - h/2
 
    l_x_max = l_x + l_w/2
    l_y_max = l_y + l_h/2
    l_x_min = l_x - l_w/2
    l_y_min = l_y - l_h/2
    # calculate the inter
    inter_x_max = tf.minimum(x_max, l_x_max)
    inter_x_min = tf.maximum(x_min, l_x_min)
 
    inter_y_max = tf.minimum(y_max, l_y_max)
    inter_y_min = tf.maximum(y_min, l_y_min)
 
    inter_w = inter_x_max - inter_x_min
    inter_h = inter_y_max - inter_y_min
    
    inter = tf.cond(tf.logical_or(tf.less_equal(inter_w,0), tf.less_equal(inter_h,0)), 
                    lambda:tf.cast(0,tf.float32), 
                    lambda:tf.multiply(inter_w,inter_h))
    # calculate the union
    union = w*h + l_w*l_h - inter
    
    IoU = inter / union
    return IoU
```

以上都是基于两个框框之间的 IOU 计算，可以用于 目标检测 Loss 计算。

这里我们会有一堆预测的 Anchor， 还有对应的一堆真实的框框 bbox_b,我们通过矩阵来一次求得：

```python
def bbox_iou(bbox_a, bbox_b):
    """
	计算两组方框之间的IOU值。
	
	bbox_a：一个numpy.ndarray型二维数组，shape为（N，4），N代表bbox_a包含的方框个数
	bbox_b: 一个numpy.ndarray型二维数组，shape为（k，4），K代表bbox_b包含的方框个数
	4表示每个方框的坐标，按顺序依次表示y_min,x_min,y_max,x_max
	min代表左上角，max代表右下角，在图片中，左上角代表坐标原点，正方向为下、右。
	
	此函数需要计算的就是bbox_a中的每个方框与bbox_b中的每个方框的IOU值，所以最后返回的数组shape为（N,K）
	
Calculate the Intersection of Unions (IoUs) between bounding boxes.

    IoU is calculated as a ratio of area of the intersection
    and area of the union.

    This function accepts both :obj:`numpy.ndarray` and :obj:`cupy.ndarray` as
    inputs. Please note that both :obj:`bbox_a` and :obj:`bbox_b` need to be
    same type.
    The output is same type as the type of the inputs.

    Args:
        bbox_a (array): An array whose shape is :math:`(N, 4)`.
            :math:`N` is the number of bounding boxes.
            The dtype should be :obj:`numpy.float32`.
        bbox_b (array): An array similar to :obj:`bbox_a`,
            whose shape is :math:`(K, 4)`.
            The dtype should be :obj:`numpy.float32`.

    Returns:
        array:
        An array whose shape is :math:`(N, K)`. \
        An element at index :math:`(n, k)` contains IoUs between \
        :math:`n` th bounding box in :obj:`bbox_a` and :math:`k` th bounding \
        box in :obj:`bbox_b`.

    """
    if bbox_a.shape[1] != 4 or bbox_b.shape[1] != 4:
        raise IndexError

    # top left
    tl = np.maximum(bbox_a[:, None, :2], bbox_b[:, :2])
    # bottom right
    br = np.minimum(bbox_a[:, None, 2:], bbox_b[:, 2:])

    area_i = np.prod(br - tl, axis=2) * (tl < br).all(axis=2)
    area_a = np.prod(bbox_a[:, 2:] - bbox_a[:, :2], axis=1)
    area_b = np.prod(bbox_b[:, 2:] - bbox_b[:, :2], axis=1)
    return area_i / (area_a[:, None] + area_b - area_i)
```



## NMS 非极大值抑制（non maximum suppression）

还有一个场景是把 IOU 用于 NMS 候选框的选取。[^ 1 ]

> 以Faster_Rcnn为例，对于一张图片，模型会预测生成多个Anchor（预测方框），这些预测的方框和图片中实际的真实方框肯定是不完全匹配的， 如何确定每个Anchor是大概属于哪个真实方框，通过计算两者之间IOU值即可，通常还规定一个iou阈值(iou_threshold)，若预测框和真实框之间的iou值大于iou_threshold，就表明此预测框具有和真实框的相同属性（如类别、是否前景）。

![在这里插入图片描述](/public/img/20200723143828794.png)

**NMS的算法思想就是**：若两个方框之间的IOU值大于设定的阈值，即可认同两个方框属于同一种类方框（识别的目标一样），显然需要去除得分较低的哪个方框。整个算法需要做的就是重复上述过程，直到没有可以去除的。

![在这里插入图片描述](/public/img/20200723144313462.png)

我们手动实现一下NMS：[^ 2]

```python
import numpy as np

'''
input:
dets=[[x1,y1,x2,y2,score],[x1,y1,x2,y2,score],...] 
thresh: float
'''

def nms(dets, thresh):
   # 以下 5 个都是多维的数组
    x1 = dets[:, 0] 
    y1 = dets[:, 1]
    x2 = dets[:, 2]
    y2 = dets[:, 3]
    scores = dets[:, 4]
   
   # 用数组存储所有 box 的面积
    areas = (x2 - x1 + 1) * (y2 - y1 + 1)
    
    # 将数组的类别分数从大到小排序，并得到它们排序后的索引顺序 
    order = scores.argsort()[::-1]

    keep = []
    while order.size > 0:
        i = order[0]
        keep.append(i)
        xx1 = np.maximum(x1[i], x1[order[1:]])
        yy1 = np.maximum(y1[i], y1[order[1:]])
        xx2 = np.minimum(x2[i], x2[order[1:]])
        yy2 = np.minimum(y2[i], y2[order[1:]])

        w = np.maximum(0.0, xx2 - xx1 + 1)
        h = np.maximum(0.0, yy2 - yy1 + 1)
        inter = w * h
        ovr = inter / (areas[i] + areas[order[1:]] - inter)

        inds = np.where(ovr <= thresh)[0]  # np.where() 直接将小于阈值的去除了，返回 tuple（[index1,index2,...]）
        order = order[inds + 1]

    return keep
```

FasterRcnn作者给出的[**python版本的nms代码**](https://github.com/rbgirshick/fast-rcnn/blob/master/lib/utils/nms.py)。除此之外，还可以直接调用pytorch(>=1.2.0)、torchvision(>= 0.3)中封装好了的nms函数。

```python
from torchvision.ops import nms
keep = nms(boxes,scores,iou_threshold)  

#返回的keep表示为经过nms后保留下来的boxes的索引值，并且keep的中索引值是按照scores得分降序排列的
#len(keep) <= len(boxes)

#通过下列操作即可获取具体nms后的boxes
boxes = boxes[keep]

‘‘‘
参数：
boxes (Tensor[N, 4])) – bounding boxes坐标. 格式：(x1, y1, x2, y2)

scores (Tensor[N]) – bounding boxes得分

iou_threshold (float) – IoU过滤阈值
’’’
```

[^ 1 ]: [目标检测-IOU和NMS计算](https://blog.csdn.net/qq_35268841/article/details/107526707?utm_medium=distribute.pc_relevant.none-task-blog-title-2&spm=1001.2101.3001.4242)
[^ 2]: [IOU计算及NMS计算](https://blog.csdn.net/chris_zhangrx/article/details/100736847)

