---
title: GMM高斯混合模型
abbrlink: 6d95358d
date: 2023-10-20 20:53:10
tags:
cover: 
---

<!-- more -->
## 1. GMM高斯混合模型原理
### 1.1 混合密度模型
复杂的概率密度函数可以由简单密度函数线性组合构成
$$p(x|\theta) = {\sum\limits_{k=1}^{M} a_{k}p_{k}(x|\theta_k)},\quad a_{k}>0,\quad \sum\limits_{k=1}^{M} a_{k}=1$$
GMM 是混合密度模型的一个特例，由多个高斯（正态分布）函数的组合构成
$$p(x) = \sum\limits_{k=1}^{M} a_{k}N(x;\mu_k,\Sigma_k)$$
#### ==以下3点对于理解GMM模型的预测很重要 ！！！==
- 这里的$p(x)$是似然函数，完整表达应该是$p(x|\omega_i)$，这里的$\omega_i$表示是第$i$类数据，即训练GMM其实对每一类都单独学习一个GMM模型。
- 比如：在分类问题中，如10分类问题。
    - 首先学习到10个不同的GMM模型（即似然函数公式）；
    - 通过预测样本带入似然函数公式，得到概率密度值；
    - 再乘以不同类别的先验概率$p(\omega_i)$，得到10个对应预测样本的后验概率$p(\omega_i|x)$；
    - 取这10个值里最大的所对应的类别作为这个样本的类别。
- 这一切预测都是基于贝叶斯公式：
$$P(\omega_i|x)={\dfrac{p(x|\omega_i)P(\omega_i)}{p(x)}}$$
### 1.2 GMM的极大似然估计（参数估计）
GMM 需要估计的参数:
$$\theta=(\alpha_1,\dots,\alpha_M,\mu_1,\dots,\mu_M,\Sigma_1,\dots,\Sigma_M)$$
对数似然函数:
$$l({\theta})=\sum\limits_{i=1}^nln\left[\sum\limits_{k=1}^M a_{k}N(x_i;\mu_k,\Sigma_k)\right]$$
- 极值点方程是复杂的超越方程组，很难直接求解
- 常用的 GMM 参数估计方法是 EM 算法

训练集和待学习参数
- 训练数据集:  $D_X = \{X_1,\dots,X_n\}$
- 学习参数: $\theta=\{\alpha_k,\mu_k,\Sigma_k\}_{k=1,\dots,M}$

存在的问题
- 只有样本 $x_i$ ，但不知道是由哪一个成份高斯产生的
- 令 $y_i=k$表示 $x_i$是由第$k$个成份高斯产生，构造集合：
$$D_Y=\{y_1,\dots,y_n\}$$
- 完整的数据集 $D=D_X\cup D_Y$，其中 $D_Y$为缺失的数据

## 2. EM算法
- ==重要总结！！！==
对于$D_Y=\{y_1,\dots,y_n\}$,相对于武断的将每个训练样本确定为某个高斯产生的，更好的做法是计算每个样本由每个高斯产生的概率，后文的代码实现也是基于这一点的，其中$pdfs$为$m*k$维矩阵，每行为该样本由不同高斯产生的概率，共有m个样本。
### 2.1 E步（Expectation）
- 隐变量的概率估计

$$P(y_i=k|x_i)=\dfrac{P(y_i=k)P(x_i|y_i=k)}{P(x_i)}=\dfrac{a_{k}N(x_i;\mu_k,\Sigma_k)}{\sum\limits_{j=1}^M a_{j}N(x_i;\mu_j,\Sigma_j)} \tag{1}$$

### 2.2 M步（Maximize）
- 每个高斯分布参数也需要由所有样本参与估计，同时需要
- 考虑样本由不同高斯产生的概率
$$\hat{a}_k=\dfrac{1}{n}\sum\limits_{i=1}^nP(y_i=k) \tag{2}$$$$\hat{\mu}_k=\dfrac{\sum\limits_{i=1}^nP(y_i=k)x_i}{\sum\limits_{i=1}^nP(y_i=k)} \tag{3}$$$$\hat{\Sigma}_k=\dfrac{\sum\limits_{i=1}^nP(y_i=k)(x_i-\hat{\mu}_k)(x_i-\hat{\mu}_k)^t}{\sum\limits_{i=1}^nP(y_i=k)} \tag{4}$$

## 3. 算法
---
Input: 训练集$D_X = \{X_1,\dots,X_n\}$，高斯数$M$
Output: GMM的模型参数 $\theta$
1. 随机初始化$\theta$;
2. repeat
    E步：公式(1)估计样本由不同高斯产生的概率；
    M步：公式(2-4)重新估计模型的参数$\theta$；
3. until 达到收敛精度
---
类比KMeans算法：
- E步是在已知质心位置的条件下，把各点聚类到最近的质心；
- M步是根据类内各点，调整质心位置。

## 4. 实验及代码
### 4.1 实验内容
1.	使用Python编程实现GMM算法：要求独立完成算法编程，禁止调用已有函数库或工具箱中的函数；
2.	使用仿真数据测试程序的正确性：
    1. 两类2维各1000个训练样本；
    2. 	每个类别的样本分别采样自包含2个分量的高斯混合模型，分别使用两个类别的训练样本学习模型化每个类别条件概率密度的GMM模型参数，并与模型的真实值比较；
    3.	假设两个类别的先验概率相等，使用学习到的模型参数分类测试样本数据，统计分类的正确率
3.	MNIST数据集测试：
    1.	MNIST-Train-Samples.csv中包含30000个17维特征手写数字样本训练，MNIST-Train-Labels.csv中包含训练样本的标签；
    2.	分别使用每个数字的样本集学习该类别的GMM模型参数；
    3.	使用10个类别GMM模型构成的分类器，分类MNIST-Test-Samples.csv中的10000个样本，与MNIST-Test-Labels.csv的类别标记比对，计算识别的正确率；
    4.	尝试设置不同的GMM模型超参数—高斯数，测试不同高斯数的GMM分类器的识别正确率；

### 4.2 程序代码
#### 4.2.1 导入库并制作工具函数
```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Ellipse
import pandas as pd
import random
import warnings
import math
import copy
warnings.filterwarnings("ignore")

# 绘制椭圆参考代码
def plot_cov_ellipse(cov, pos, nstd=2, ax=None):
    def eigsorted(cov):
        vals, vecs = np.linalg.eigh(cov)
        order = vals.argsort()[::-1]
        return vals[order], vecs[:, order]
 
    if ax is None:
        ax = plt.gca()
 
    vals, vecs = eigsorted(cov)
    theta = np.degrees(np.arctan2(*vecs[:, 0][::-1]))
 
    # Width and height are "full" widths, not radius
    width, height = 2 * nstd * np.sqrt(vals)
    ellip = Ellipse(xy=pos, width=width, height=height, facecolor='#c8e0e4', angle=theta)
 
    ax.add_artist(ellip)
    return ellip
# 绘图函数 在二维特征时可用
def plot(data, mu, covariance, class_label):
    plt.scatter(data[:, 0], data[:, 1], c=class_label)
    n_components = len(mu)
    for j in range(n_components):
        plot_cov_ellipse(covariance[j], mu[j])
    plt.show()
# 定义高斯函数
def Gaussian(x, mu, covariance):
    ret = []
    for i in x:
        X, mu, covariance = np.array(i), np.array(mu), np.array(covariance)
        ret.append( np.exp(-0.5 * (np.dot(np.dot((X-mu).T, np.linalg.inv(covariance)), X-mu))) /
                    math.sqrt(np.linalg.det(2.0 * math.pi * covariance)))
    return np.array(ret)

```
#### 4.2.2 GMM面向对象实现
``` python
# GMM主体
class GMM:
    def __init__(self, X, y, n_components, iteration=40, eps=1e-9):
        self.X = X
        self.y = y
        self.m, self.n = X.shape
        # 组数
        self.n_components = n_components
        self.iteration = iteration
        self.eps = eps
        # 高斯混合模型参数
        self.alpha = np.ones(n_components) * 1 / n_components
        self.mu = np.random.random((self.n_components, self.n))
        self.covariance = np.zeros((self.n_components, self.n, self.n))
        # 1. 随机初始化 mu 为样本随机点
        # mu n_components * 2
        row_rand_array = np.arange(X.shape[0])
        np.random.shuffle(row_rand_array)
        self.mu = X[row_rand_array[0:n_components]]
        
        
        # 2. 初始化 cov alpha
        # 计算每个样本到 初始化中心点 的距离 
        distance = np.tile(np.sum(X * X, axis=1).reshape((self.m, 1)), (1, self.n_components)) + np.tile(np.sum(self.mu * self.mu, axis=1).T,(self.m, 1)) - 2 * np.dot(X, self.mu.T)
        # 初始标签 按照距离分类
        orginial_labels = np.argmin(distance, axis=1)
        for i in range(self.n_components):
            temp = X[orginial_labels == i, :]
            self.alpha[i] = temp.shape[0] / self.m
            self.covariance[i, :, :] = np.cov(temp.T)
            
        # 3. 概率密度 推断样本 𝐱𝑖 由每一个高斯产生的概率
        # m * n_components 
        # 第i行 为 样本 𝐱𝑖 由每一个高斯产生的概率
        self.pdfs = np.zeros((self.m, self.n_components))
        
    def Train(self):
        nowIter = 0
        preLogLikelihood = self.LogLikelihood()
        while nowIter < self.iteration:
            if nowIter % 20 == 0:
                print("Iter:", nowIter)
            self.Expectation()
            self.Maximize()
#             plot(self.X, self.mu, self.covariance, self.y)
            nowIter += 1
            logLikelihood = self.LogLikelihood()
            if abs(logLikelihood - preLogLikelihood) < self.eps:
                break
            preLogLikelihood = logLikelihood
            
            
            
    # 根据当前的各个组分 alpha、mu 和 covariance 计算对数似然函数值
    def LogLikelihood(self):
        for j in range(self.n_components):
            self.pdfs[:, j] = self.alpha[j] * Gaussian(self.X, self.mu[j], self.covariance[j])
        return np.mean(np.log(np.sum(self.pdfs, axis=1)))
        
    def Expectation(self):
        # E步
        for k in range(self.n_components):
            self.pdfs[:,k] = self.alpha[k] * Gaussian(self.X, self.mu[k], self.covariance[k])
            self.W = self.pdfs / np.sum(self.pdfs,axis=1).reshape(-1,1)
        
    def Maximize(self):
        # M步
        self.alpha = np.sum(self.W, axis=0) / np.sum(self.W)
        for k in range(self.n_components):
            self.mu[k] = np.average(self.X, axis=0, weights=self.W[:, k])
            cov = 0
            for i in range(self.m):
                temp = (self.X[i, :] - self.mu[k, :]).reshape(-1, 1)
                cov += self.W[i, k] * np.dot(temp, temp.T)
            self.covariance[k, :, :] = cov / np.sum(self.W[:, k]) 
```

#### 4.2.3 Emu数据测试
**参数估计**
```python
train_X = pd.read_csv(r'Emu-Train-Samples.csv', header=None)
train_Y = pd.read_csv(r'Emu-Train-Labels.csv', header=None)
train = copy.deepcopy(train_X)
train["label"] = train_Y
train = train.to_numpy()
train_1_X = np.array([[i[0], i[1]] for i in train if i[2] == 0])
train_1_Y = np.array([i[2] for i in train if i[2] == 0])
train_2_X = np.array([[i[0], i[1]] for i in train if i[2] == 1])
train_2_Y = np.array([i[2] for i in train if i[2] == 1])
plt.scatter(train_1_X[:, 0], train_1_X[:, 1], c='blue')
plt.scatter(train_2_X[:, 0], train_2_X[:, 1], c='green')

myGMM1 = GMM(train_1_X, train_1_Y, 2)
myGMM1.Train()
myGMM2 = GMM(train_2_X, train_2_Y, 2)
myGMM2.Train()

print("Class_1:")
print("mu:")
print(myGMM1.mu)
print("alpha:")
print(myGMM1.alpha)
print("covariance:")
print(myGMM1.covariance)

print("Class_2:")
print("mu:")
print(myGMM2.mu)
print("alpha:")
print(myGMM2.alpha)
print("covariance:")
print(myGMM2.covariance)
```

**预测**
```python
test_X = pd.read_csv(r'Emu-Test-Samples.csv', header=None).to_numpy()
test_Y = pd.read_csv(r'Emu-Test-Labels.csv', header=None).to_numpy().reshape(-1)

def Likelihood(X, alpha, mu, covariance, n_components):
    pdfs = np.zeros((len(X), n_components))
    for k in range(n_components):
        pdfs[:, k] = alpha[k] * Gaussian(X, mu[k], covariance[k])
    return np.sum(pdfs, axis=1)

pdfs_1 = Likelihood(test_X, myGMM1.alpha, myGMM1.mu, myGMM1.covariance, len(myGMM1.alpha))
pdfs_2 = Likelihood(test_X, myGMM2.alpha, myGMM2.mu, myGMM2.covariance, len(myGMM2.alpha))

predict = []
for i in range(len(test_X)):
    if(pdfs_1[i] > pdfs_2[i]):
        predict.append(0)
    else:
        predict.append(1)

accuracy = sum(predict == test_Y) / len(predict)
print(accuracy)
```


#### 4.2.4 MNIST数据集测试
```python
train_mnist_X = pd.read_csv(r'MNIST-Train-Samples.csv', header=None)
train_mnist_Y = pd.read_csv(r'MNIST-Train-Labels.csv', header=None)
test_mnist_X = pd.read_csv(r'MNIST-Test-Samples.csv', header=None)
test_mnist_Y = pd.read_csv(r'MNIST-Test-Labels.csv', header=None)
train_mnist = train_mnist_X
train_mnist["label"] = train_mnist_Y
train_mnist = train_mnist.to_numpy()
test_mnist_X = test_mnist_X.to_numpy()
test_mnist_Y = test_mnist_Y.to_numpy().reshape(-1)

def GmmOfMnist(number):
    alphas = []
    mus = []
    covariances = []
    pdfs = []

#     print("Train Begin!")
    for index in range(0, 10):
        print("GMM", number," ", "class", index)
        nowTrain_X = np.array([i[:-1] for i in train_mnist if i[-1] == index])
#         print(np.shape(nowTrain_X))
        nowTrain_Y = [index] * len(nowTrain_X)
        nowGMM = GMM(nowTrain_X, nowTrain_Y, number)
        nowGMM.Train()
        alphas.append(nowGMM.alpha)
        mus.append(nowGMM.mu)
        covariances.append(nowGMM.covariance)
        
#     print("Test Begin!")
    for i in range(0, 10):  
        pdfs.append(Likelihood(test_mnist_X, alphas[i], mus[i], covariances[i], len(alphas[i])))
    result = np.argmax(pdfs, axis = 0)
    accuracy_num = sum(result == test_mnist_Y) 
    accuracy = accuracy_num / len(result)
    return accuracy_num, accuracy

for i in range(2, 6):
    accuracy_num, accuracy = GmmOfMnist(i)
    print("GMM number:", i)
    print("正确识别数: ", accuracy_num)
    print("正确识别率: ", accuracy)
```

## 5. 遇到的问题及解决方法
- 高斯均值初始化问题：
    在前几轮训练中有几次收敛到局部最优解，即有个别高斯模型重叠覆盖了同一个训练样本，或者一个高斯完全覆盖了另一个高斯的样本范围。这是由于初始化不够好导致的，起初选用的是固定选点，后续使用的是随机样本选点，更好的做法应该是使用Kmeans聚类选择初始化点。
