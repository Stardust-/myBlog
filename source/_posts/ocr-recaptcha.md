---
title: 基于Lstm、CTC的验证码识别
date: 2016-08-17 22:16:49
layout: 
subtitle: 使用mxnet与deep learning实现验证码识别 
author: Stardust
header-img: header-img.jpg 
tags:
	- Python 
	- Lstm 
	- CTC 
	- mxnet 
	- 深度学习 
	- 人工智能
	- OCR

---

> 本文为原创文章，转载请注明地址： http://ranjun.me/2016/08/17/ocr-recaptcha/

## 背景

当Google的AlphaGo出世以来，围棋界和学术界都不平静了。围棋是之前人类所能战胜计算机的唯一的智力游戏，当AlphaGo战胜李世乭的那一刻，我们见证了一个时代的结束，也见证着一个时代的开始。人工智能已仅仅不在是科幻小说的产物，已是触手可及。

AlphaGo所使用的深度神经网络在硬件水平大大提高的今天再一次在学术界和工业界火热起来，自然语言处理，图像识别，搜索引擎等等都在大量使用深度神经网络。

OCR一直是一个古老的研究话题，以下是Wikipedia对OCR的描述：

> **光学字符识别**（英语：Optical Character Recognition, OCR）是指对文本资料的图像文件进行分析识别处理，获取文字及版面信息的过程

验证码识别也属于OCR的一个方向，同样是从图片上识别字符信息。但是通常来说验证码有较强的干扰，识别率一直是一个头疼的问题。

最近在一个机缘巧合之下，入了Deep Learning的坑，接触到了OCR方面的东西，在[xlvector](https://github.com/xlvector)大神的4位纯数字验证码上经过自己的爬坑扩展到了包括数字和大小写英文字母的6位验证码识别，识别率最高能达到96% 

## 正文

### 理论基础

#### RNN

> **递归神经网络（RNN）**是两种人工神经网络的总称。一种是时间递归神经网络（recurrent neural network），另一种是结构递归神经网络（recursive neural network）。时间递归神经网络的神经元间连接构成有向图，而结构递归神经网络利用相似的神经网络结构递归构造更为复杂的深度网络。RNN一般指代时间递归神经网络。单纯递归神经网络因为无法处理随着递归，权重指数级爆炸或消失的问题（Vanishing gradient problem），难以捕捉长期时间关联；而结合不同的LSTM可以很好解决这个问题

时间递归神经网络可以描述动态时间行为，因为和前馈神经网络（feedforward neural network）接受较特定结构的输入不同，RNN将状态在自身网络中循环传递，因此可以接受更广泛的时间序列结构输入。手写识别是最早成功利用RNN的研究结果。

rnn有三个部分组成：

 * Input Layer
 * Hidden Layer
 * Output Layer 

![rnn](rnn_img1.png)

RNN与前馈神经网络不同，前馈神经网络在时间上不会对神经网络的神经元的权重和阈值的更新产生效益，而rnn在 t-1 时间点会对 t 时间点的影响，既 t 时间点的输入包含 t-1 时间点神经网络的输出。

![rnn](rnn_img2.png)

#### LSTM

> **长短期记忆人工神经网络（Long-Short Term Memory, LSTM）**是一种时间递归神经网络(RNN)，论文首次发表于1997年。由于独特的设计结构，LSTM适合于处理和预测时间序列中间隔和延迟非常长的重要事件。

LSTM属于RNN（递归神经网络）的一种，用于解决前后有关联的序列相关的问题。如：语音识别，文字识别等等。

传统的RNN对于时间间隔较长的前后关系序列的效果很差，因此处理和预测时间序列中间隔和延迟非常长序列事，LSTM又非常好的效果。

LSTM中存在三个重要的gate：

 * Input gate
 * Forget gate
 * Output gate

LSTM有很多个版本，其中一个重要的版本是GRU（Gated Recurrent Unit），根据谷歌的测试表明，LSTM中最重要的是Forget gate，其次是Input gate，最次是Output gate。

![lstm](lstm_img.png)

 >  LSTM memory block with one cell. The cell has a recurrent connection with fixed weight 1:0. The three gates collect input from the rest of the network, and control the cell via multiplicative units (small circles). The input and output gates scale the input and output of the cell, while the forget gate scales the recurrent connection of the cell. The cell squashing functions (g and h) are applied at the indicated places. The internal  connections from the cell to the gates are known as peephole weights.

#### CTC

> **Connectionist Temporal Classification(CTC)**: Labelling UnsegmentedSequence Data with Recurrent Neural Networks

CTC的作用大致是帮助lable更好的匹配，在处理字符连接空白处添加black lable，从而解决传统的rnn无法很好解决的lable匹配问题。

### 获取数据

生成验证码的代码是一位师兄提供的，我自己修改了一下可以生成6位的验证码图片，当然你也可以根据自己的需求修改代码生成自己的数据。

代码地址: https://github.com/Stardust-/DL-practices/blob/master/recaptcha/generator.py

图片名格式为 *编号+图片内容*，如`222_xYn65K`代表第222张图片，图片内容为xYn65K，既lable。如下图：
![captchar](captchar.png)

将训练和测试的图片放在两个不同的文件下
![folder](folder.png)

生成的用于训练集1000,000张，测试集100,000张。

### 载入数据

[xlvector]()的训练数据是在代码中生成直接加载进iterater的，为了加载自己的数据，改了iterater的部分代码，可以成功加载自己的数据进行训练。

```python

class SimpleBatch(object):
    def __init__(self, data_names, data, label_names, label):
        self.data = data
        self.label = label
        self.data_names = data_names
        self.label_names = label_names

        self.pad = 0
        self.index = None # TODO: what is index?

    @property
    def provide_data(self):
        return [(n, x.shape) for n, x in zip(self.data_names, self.data)]

    @property
    def provide_label(self):
        return [(n, x.shape) for n, x in zip(self.label_names, self.label)]


maps = {}
maps_value = 11

for char in 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ':
    maps[char] = maps_value
    maps_value += 1


def get_label(buf):
    global maps
    ret = np.zeros(len(buf))
    for i in range(len(buf)):
        if is_num(buf[i]):
            ret[i] = int(buf[i]) + 1
        else:
            ret[i] = maps[buf[i]]
    return ret


class OCRIter(mx.io.DataIter):
    def __init__(self, batch_size, num_label, init_states, path, check):
        super(OCRIter, self).__init__()
        self.batch_size = batch_size
        self.num_label = num_label
        self.init_states = init_states
        self.path = path
        self.init_state_arrays = [mx.nd.zeros(x[1]) for x in init_states]
        self.provide_data = [('data', (batch_size, 100 * 30))] + init_states
        self.provide_label = [('label', (self.batch_size, num_label))]
        self.check = check

    def __iter__(self):
        print 'iter'
        init_state_names = [x[0] for x in self.init_states]
        dir_list = os.listdir(self.path)
        pic_num = len(dir_list)
        num = 0
        for k in range(pic_num / self.batch_size):
            data = []
            label = []
            for i in range(self.batch_size):
                if num > pic_num-1:
                    break
                img = cv2.imread(self.path + '/' + dir_list[num], 0)
                lable_value = dir_list[num].split('.')[0]
                lable_value = lable_value.split('_')[1]
                num += 1
                img = cv2.resize(img, (100, 30))
                img = img.transpose(1, 0) 
                img = img.reshape((100 * 30))
                img = np.multiply(img, 1/255.0)

                data.append(img)
                label.append(get_label(lable_value))

            data_all = [mx.nd.array(data)] + self.init_state_arrays
            label_all = [mx.nd.array(label)]
            data_names = ['data'] + init_state_names
            label_names = ['label']            
            
            data_batch = SimpleBatch(data_names, data_all, label_names, label_all)
            yield data_batch

    def reset(self):
        pass

```

### 训练模型

#### 配置环境

 * 配置mxnet环境 [官方配置教程](https://github.com/dmlc/mxnet/blob/master/example/warpctc/README.md)
 * 配置百度wrap-ctc环境 [官方配置教程](http://phunter.farbox.com/post/mxnet-tutorial1)

#### 调整参数

根据自己的需要调整learning rate：

`learning_rate = 0.01`

然后指定训练和测试的目录：

```python
train_path  = '/home/img_data/train_data'
test_path  = '/home/img_data/test_data'

```

指定使用的gpu或者cpu：

```python
contexts = [mx.context.gpu(3)]

```

```python
contexts = [mx.context.cpu(0)]

```

ubuntu下安装cuda之后使用nvidia-smi命令可以查看gpu的使用情况，我使用4块tesla k40m，使用如下代码多gpu训练：

![gpu](gpu_img.png)

```python
contexts = [mx.context.gpu(i) for i in range(0,4)]

```

最后就是要把训练的模型保存：

```python
prefix = 'mymodel/mymodel'
iteration = 5

model.save(prefix, iteration)

```

之后训练或者测试加载保存的模型：

```python
model = mx.model.FeedForward.load(prefix,iteration,
                                  learning_rate = learning_rate,
                                  ctx = contexts, 
                                  numpy_batch_size = BATCH_SIZE,
                                  num_epoch=num_epoch)

```

#### 训练模型

直接运行python代码就可以开始运行

`python lstm_ocr_myiter.py`

ctc的收敛比较慢,小憩一会或者刷刷知乎，如果很长时间不收敛试着调整自己的学习率。

所有代码地址: https://github.com/Stardust-/DL-practices

### 模型效果
![effec](effect.png)

差不多可以到达96%的效果。

## 后记

因为刚接触深度学习方面的东西，希望将学习的东西记录下来，多有疏忽之处。如果发现问题或是存在疑惑，请留言提出，希望与前辈们多多交流。

## 参考资料

*A. Graves,  M. Liwicki, S. Fern´andez, R. Bertolami, H. Bunke and J. Schmidhuber,"A Novel Connectionist System for Unconstrained Handwriting Recognition"  2008 *

*Wikipedia *

> 本文为原创文章，转载请注明地址： http://ranjun.me/2016/08/17/ocr-recaptcha/
