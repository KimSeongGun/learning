本文件用于记录学习和复现论文。

```python
# 训练的模板
transform = transforms.Compose([
    transforms.Resize(224),
    transforms.ToTensor(),
    transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5)),
])

train_dataset = CIFAR10(root='./data', train=True, download=True, transform=transform)
test_dataset = CIFAR10(root='./data', train=False, download=True, transform=transform)

train_loader = DataLoader(train_dataset, batch_size=128, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=128, shuffle=False)

# 初始化模型、损失函数和优化器
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model = AlexNet(num_classes=10).to(device)  # 注意这里num_classes应与数据集类别数匹配
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

# 训练模型
num_epochs = 10
for epoch in range(num_epochs):
    model.train()
    running_loss = 0.0
    for i, data in enumerate(train_loader, 0):
        inputs, labels = data[0].to(device), data[1].to(device)
        
        optimizer.zero_grad()
        
        outputs = model(inputs)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
        
        running_loss += loss.item()
        if i % 100 == 99:    # 每100个小批量打印一次损失
            print(f"[{epoch + 1}, {i + 1}] loss: {running_loss / 100:.3f}")
            running_loss = 0.0

print("Finished Training")

# 测试模型
model.eval()
correct = 0
total = 0
with torch.no_grad():
    for data in test_loader:
        images, labels = data[0].to(device), data[1].to(device)
        outputs = model(images)
        _, predicted = torch.max(outputs.data, 1)
        total += labels.size(0)
        correct += (predicted == labels).sum().item()

print(f"Accuracy on test set: {100 * correct / total}%")
```



# AlexNet

> period: 2024/07/28 -   

使用了非饱和神经元（non-saturating neurons），即不压缩输出范围的激活函数。Sigmoid和Tanh属于饱和神经元，ReLU和Leaky ReLU则属于非饱和神经元。

AlexNet由五个卷积层和三个全连接层组成。对于输入，首先将短的一边调成256，然后进行中心裁剪。数据预处理部分从每个像素中减去训练集上的平均值。

ReLU不需要输入归一化来防止他们饱和。如果至少一些训练样本对ReLU产生正输入，那么该神经元就会发生学习。即不是每一个输入样本都会导致ReLU神经元学习，从而体现了ReLU的非线性。



### 网络结构

前五层是卷积层，后三层是全连接层，最后一个全连接层连上一个1000维的softmax激活函数。

- 单一映射：卷积第二层，第四层，第五层的和只连接到前一层的核映射上
- 多映射：卷积第三层的核连接到第二层的所有核映射上
- 全连接：全连接层中的神经元与前一层的所有神经元相连
- 非线性激活：每个卷积层和全连接层的输出应用ReLU非线性激活

> AlexNet论文中提到的映射概念主要是针对多GPU并行计算的优化策略，目的是减少不同GPU间的通信开销，提高计算效率。在实际的复现中，如果使用单个GPU进行训练，可以忽略这些优化细节，直接在单个GPU上构建和训练模型即可。

![image-20240731100011395](assets\AlexNet.png)

输入为224 × 224 × 3的图像，第一层含96个11 × 11 × 3，stride取4的kernel。第二层含256个5 × 5 × 48的kernel。第三，四，五层的卷积核不进行pooling和normalization操作。第三层有384个3 × 3 × 256kernel。第四层含384个3 × 3 × 192kernel，第五层含256个3 × 3 × 192kernel。每个全连接层含4096个神经元。

### 复现

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torchvision.transforms as transforms
from torchvision.datasets import CIFAR10
from torch.utils.data import DataLoader

# LRN层
# 自定义局部响应归一化层
class LRN(nn.Module):
    def __init__(self, local_size=5, alpha=1e-4, beta=0.75, k=1.0):
        super(LRN, self).__init__()
        self.local_size = local_size
        self.alpha = alpha
        self.beta = beta
        self.k = k

    def forward(self, x):
        batch_size, channels, height, width = x.size()
        square = x.pow(2)
        pad = self.local_size // 2
        extra_channels = nn.functional.pad(square, (pad, pad, pad, pad))
        
        scale = self.k + self.alpha * extra_channels.unfold(1, channels, 1).mean(dim=4, keepdim=True)
        scale = scale.pow(self.beta)
        
        return x / scale
    
# 带LRN的AlexNet模型
class AlexNet(nn.Module):
    def __init__(self, num_classes=1000):
        super(AlexNet, self).__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 96, kernel_size=11, stride=4, padding=2),
            LRN(),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),
            nn.Conv2d(96, 256, kernel_size=5, padding=2),
            LRN(),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),
            nn.Conv2d(256, 384, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(384, 384, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(384, 256, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),
        )
        self.avgpool = nn.AdaptiveAvgPool2d((6, 6))
        self.classifier = nn.Sequential(
            nn.Dropout(),
            nn.Linear(256 * 6 * 6, 4096),
            nn.ReLU(inplace=True),
            nn.Dropout(),
            nn.Linear(4096, 4096),
            nn.ReLU(inplace=True),
            nn.Linear(4096, num_classes),
        )

    def forward(self, x):
        x = self.features(x)
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.classifier(x)
        return x
```

LRN层的主要目的是增强模型对局部响应模式的鲁棒性，但实验证明LRN对网络的帮助不大甚至起副作用。现在普遍用BN层代替LRN层。
