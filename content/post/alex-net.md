---
title: "机器学习图像分类"
date: 2021-04-02
draft: false
tags: ["ML", "AI"]
keywords: ["ML", "AI", "OpenCV", "numpy"]

categories:
  - "AI"

thumbnail: "/images/alex-net/image.png"
---

近期公司在组织“人工智能”培训，本周作业是对“充气拱门”数据进行分类。上网搜了一下经典分类算法分别有LeNet-5、AlexNet、ZFNet、VGG、GoogleNet、ResNet等等。经过对比挑选了一个比较简单的AlexNet作为本次训练的模型

## AlexNet

AlexNet是2012年Alex Krizhevsky提出的深度卷积神经网络模型，可以看作为LeNet-5的加强版，并且使用了ReLU、Dropout、LRN等比较新的技术点。

AlexNet包含了6亿3000万个链接，6000万个参数和65万个神经元。拥有5个卷积层，其中3个卷积层链接了最大池化，还有3个全链接层。

## 数据源

本次训练数据为标定好的充气拱门数据，经过裁剪缩放为227 * 227 * 3 的RGB数据

## paper 地址
[https://papers.nips.cc/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf](https://papers.nips.cc/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf)

## 代码

模型

```python
class AlexNet(M.Module):
    def __init__(self):
        super(AlexNet, self).__init__()
        # 输入 227 * 227 * 3 RGB 图像，每批次为64个训练数据，shape为(64, 3, 227, 227)，使用 11*11 卷积核，输出 55 * 55 * 96
        # 输出尺寸计算公式 输出宽 = (输入宽 - 卷积核 + 2 * 填充) / 步长 + 1
        # 代入 55 = (227 - 11 + 2 * 0) / 4 + 1
        self.conv1 = M.Conv2d(3, 96, 11, stride=4)
        # 使用ReLU激活
        self.relu1 = M.ReLU()
        # 池化，使用3 * 3 stride = 2进行池化， 输出 27 * 27 * 96
        self.pool1 = M.MaxPool2d(kernel_size=3, stride=2)
        # 归一化处理
        self.bn1 = M.BatchNorm2d(96)

        # 第二层
        # 输入为第一层的55 * 55 * 96，输出27 * 27 * 256
        self.conv2 = M.Conv2d(96, 256, 5, padding=2)
        self.relu2 = M.ReLU()
        self.pool2 = M.MaxPool2d(kernel_size=3, stride=2)
        self.bn2 = M.BatchNorm2d(256)

        # 第三层
        # 输出 13 * 13 * 384
        self.conv3 = M.Conv2d(256, 384, 3, padding=1)
        self.relu3 = M.ReLU()

        self.conv4 = M.Conv2d(384, 384, 3, padding=1)
        self.relu4 = M.ReLU()

        self.conv5 = M.Conv2d(384, 256, 3, padding=1)
        self.relu5 = M.ReLU()
        self.pool5 = M.MaxPool2d(kernel_size=3, stride=2)

        # 输入 256 * 6 * 6 数据，输出 1 * 4096
        self.fc1 = M.Linear(256 * 6 * 6, 4096)
        self.relu6 = M.ReLU()

        self.fc2 = M.Linear(4096, 4096)
        # 丢掉部分参数
        self.drop = M.Dropout(0.5)
        self.relu7 = M.ReLU()
        
        # 分类器，输出为softmax分类结果，1 * 1000
        self.classifier = M.Linear(4096, 1000)

    def forward(self, x):
        x = self.bn1(self.pool1(self.relu1(self.conv1(x))))
        x = self.bn2(self.pool2(self.relu2(self.conv2(x))))
        x = self.relu3(self.conv3(x))
        x = self.relu4(self.conv4(x))
        x = self.pool5(self.relu5(self.conv5(x)))

        # 改变数据形状(N, C, H, W)变为(N, C * H * W) 的一维数据
        x = F.flatten(x, 1)
        x = self.relu6(self.fc1(x))
        x = self.relu7(self.drop(self.fc2(x)))
        x = self.classifier(x)
        return x
```

数据集
```python
def resize_img(image, target_size=(224, 224), resize=False):
    if resize:
        return cv2.resize(image, target_size)

    original_height, original_width, _ = image.shape
    _h, _w = target_size
    scale = _h / original_height
    if scale * original_width > _w:
        scale = _w / original_width
    resized_height, resized_width = int(original_height * scale), int(original_width * scale)
    image = cv2.resize(image, (resized_width, resized_height))
    data = np.zeros((_h, _w, 3), dtype='uint8')
    data[:resized_height, :resized_width, :] = image
    return data

def dataset():
    dataloader = []
    i = 0
    batch_data = []
    batch_label = []
    for image in images:
        # 每张图rect不一样，这里写个假的。。
        rect = [[0.0, 0.0], [0.9, 0.0], [0.99, 0.99], [0.0, 0.9]]
        im = cv2.imread(image)
        if im is not None:
            w = im.shape[1]
            h = im.shape[0]

            x = rect[0][0]
            y = rect[0][1]
            x1 = rect[2][0]
            y2 = rect[2][1]

            # 计算裁剪位置
            h1 = int(h * y)
            h2 = int(h * y2)
            w1 = int(w * x)
            w2 = int(w * x1)

            crop = im[h1:h2, w1: w2]

            if crop.shape[0] > 20 and crop.shape[1] > 20:
                crop = resize_img(crop, (227, 227))
                # H W C 变换为 C H W 
                crop = crop.transpose((2, 0, 1))
                batch_data.append(crop)
                batch_label.append(0)
                i += 1

                if i % 64 == 0:
                    dataloader.append((np.array(batch_data), np.array(batch_label)))
                    batch_data = []
                    batch_label = []
    print("dataloader success")
    return dataloader
```

模型训练

```python
if __name__ == '__main__':
    dataloader = dataset()

    net = AlexNet()
    # 使用梯度下降法进行优化
    optimizer = optim.SGD(
        net.parameters(),
        lr=0.01
    )
    # 初始化求导器
    gm = GradManager().attach(net.parameters())

    total_epochs = 20
    for epoch in range(total_epochs):
        total_loss = 0
        batch_data = []
        batch_label = []
        for step, (batch_data, batch_label) in enumerate(dataloader):
            with gm:
                logits = net(mge.Tensor(batch_data))
                # 计算loss
                loss = F.loss.cross_entropy(logits, mge.Tensor(batch_label))
                # 梯度计算
                gm.backward(loss)
            optimizer.step()
            optimizer.clear_grad()
            total_loss += loss.numpy().item()
        print("epoch: {}, loss {}".format(epoch, total_loss))
    mge.save(net.state_dict(), "net.mge")
```
