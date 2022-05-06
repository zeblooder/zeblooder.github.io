---
title: 三篇图像动画论文的比较(FOMM,AA,SDEMT)
mathjax: true
date: 2021-12-14 10:45:19
tags: [图像动画]
---

FOMM和AA是很经典的论文，都来自作者Aliaksandr Siarohin，最近有一篇新论文在此基础上做了较大的改进，给出的效果看起来也很好，因此总结三个论文的内容。

* [FOMM](https://arxiv.org/abs/2003.00196)
* [AA](https://arxiv.org/abs/2104.11280)
* [Self-appearance-aided Differential Evolution for Motion Transfer](https://arxiv.org/abs/2110.04658)

## FOMM

### motion representation 

包含稀疏动作估计和稠密动作估计。

#### 稀疏动作

使用$\mathcal{T}_{S\leftarrow R}(p_k),\mathcal{T}_{D\leftarrow R}(p_k)$来计算D中每个关键点处到S的仿射变换，具体方法是用泰勒展开，然后将涉及到$\mathcal{T}_{S\leftarrow R}(p_k),\mathcal{T}_{D\leftarrow R}(p_k)$之外的变化都用$\mathcal{T}_{S\leftarrow R}(p_k),\mathcal{T}_{D\leftarrow R}(p_k)$来代替。
$$
\mathcal{T_{S\leftarrow D}}(z) \approx \mathcal{T_{S\leftarrow R}}(p_k) + J_k(z-\mathcal{T_{D\leftarrow R}}(p_k))
$$

#### 稠密动作

用$\mathcal{T}_{S\leftarrow R}(p_k),\mathcal{T}_{D\leftarrow R}(p_k)$的高斯分布之差计算**热力图**来找出变换发生的位置。对S进行稀疏动作的逆变换，得到K个结果(每个关键点一个变换)。然后将热力图和S变换结果一起喂给U-net，预测K+1个mask作为每个关键点上稀疏动作的权值，多出的一个mask用于处理背景。稠密动作就是各个关键点的稀疏动作的加权和。

### extraction 

使用u-net分别提取参考帧到两个输入图像的仿射变换$\mathcal{T}_{S\leftarrow R}(p_k),\mathcal{T}_{D\leftarrow R}(p_k)$，每个关键点对应一个变换。还额外输出一个遮挡图的通道。

### transfer

用稠密动作warp输入图像，然后用遮挡图计算阿达马乘积，最后通过一个解码器处理得到最终结果。

## Articulated Animation  

### motion representation 

包含稀疏动作估计和稠密动作估计。论文和代码里**热力图**的概念被重复用在两个地方，一个是指输入图像提取出来的热力图，相当于特征点，另一个是稠密动作里用高斯分布得到的热力图，用于找出发生变换的位置。

#### 稀疏动作

使用PCA处理热力图的协方差得到仿射变换的线性变换的部分，热力图的均值作为仿射变换的偏移量，总共5个自由度，加上背景的恒等变换就得到稀疏动作。

#### 稠密动作

和FOMM的做法完全一样（包括使用的神经网络结构），稠密动作就是各个关键点的稀疏动作的加权和。

### extraction 

使用U-net结构提取输入图像的特征图，经过一层卷积和softmax，得到热力图。

### transfer

和FOMM一样。

## Self-appearance-aided Differential Evolution for Motion Transfer  

### motion representation 

计算关键点的差值作为神经网络的输入，回归出系数来加权关键点的差值作为稀疏动作，同时作为微分方程
$$
\frac{d \mathscr{T}}{d t}=\mathscr{F}_{E}\left(\mathscr{T}^{(t)}, t\right), \quad \text { for } t \in[0,1]
$$
的初值$\mathscr{F}^{(0)}$：，两侧积分就得到稠密动作。

### extraction 

用编码器解码器网络获得S和D的关键点。

### transfer

1. 用特征提取网络提取输入图像的特征$\mathbb{F}_{\mathbb{S}}$。

2. 将$\mathbb{F}_{\mathbb{S}}$用稠密动作进行扭曲得到$\widetilde{\mathbb{F}}_{\mathbb{S D}}$。

3. 扭曲后用$\mathscr{F}_A$来预测self-appearance流变形场$\mathscr{T}_{App}$，$\mathscr{T}_{App}$是用来扭曲原区域到缺失区域的（流）特征的。

4. 计算$\mathbb{F}_{A p p}=\mathscr{T}_{A p p} \circ \widetilde{\mathbb{F}}_{\mathbb{S D}}$。

5. 用生成器$\mathscr{F}_G$为N个不同视角的view输出置信度掩码$C^{(j)}$，每个视角的置信度掩码各自进行归一化：$\widetilde{C}^{(j)}(\mathbf{x})=C^{(j)}(\mathbf{x}) / \sum_{j=0}^{N} C^{(j)}(\mathbf{x})$。

6. 用置信度掩码对$\mathbb{F}_{A p p} ,\widetilde{\mathbb{F}}_{\mathbb{S D}}$分别计算加权和：
   $$
   \overline{\mathbb{F}}_{A p p}=\sum_{j=1}^{N} \widetilde{C}^{(j)} \mathbb{F}_{A p p}^{(j)}, \quad \overline{\widetilde{\mathbb{F}}}_{\mathbb{D}}=\sum_{j=1}^{N} \widetilde{C}^{(j)} \widetilde{\mathbb{F}}_\mathbb{SD}^{(j)}
   $$

7. 只有一个视图时，用$\mathbb{F}_{A p p} ,\widetilde{\mathbb{F}}_{\mathbb{S D}}$连接起来喂给$\mathscr{F}_G$的编码器部分用于生成图像。有多个时，用加权后的连接起来喂给$\mathscr{F}_G$的编码器部分用于生成图像。

# 总结

## 差别

1. FOMM和Articulated Animation是同一个人的工作，只有稀疏动作表示是不一样的，提取特征里后者只是多了生成热力图的步骤，Articulated Animation简化了计算稀疏动作的过程，用PCA方法和热力图的均值直接获得仿射变换，非常简洁。
2. Self-appearance-aided Differential Evolution for Motion Transfer使用解微分方程的方法计算动作，原因是neural-ODEs已被证明能够捕捉复杂的变换，**可微动作演化**可以泛化稀疏动作的预测（我觉得意思是不会因为不同对象或者数据集效果差别很大），同时避免雅克比矩阵和SVD的大量计算。

## 优点

1. Articulated Animation认为PCA可以更好的描述动作，所以用PCA来获得仿射变换矩阵的线性部分的参数。
2. Self-appearance-aided Differential Evolution for Motion Transfer 里指出它的方法泛化了FOMM，AA，Monkey-Net方法。
3. Self-appearance-aided Differential Evolution for Motion Transfer 在CSIM上与FOMM,AA拉开很大的距离。即该方法相比FOMM,AA能更好的保留原图像的特征。
4. Self-appearance-aided Differential Evolution for Motion Transfer 泛化能力很好，在A训练集上训练后，在B训练集上依然效果很好。比如卡通动画生成中，该论文的方法能很好的保留个体特征，而FOMM和AA很差。

## 缺点

1. FOMM在提取稀疏动作的时候用了泰勒展开，非常繁琐。
2. Articulated Animation里指出FOMM这类用关键点的方法在处理物体边界内的动作会出现不真实的效果。
3. Articulated Animation认为该论文的方法的泛化能力较弱，生成非活物的动画有难度，也就是说生成素描动画效果不会很好。
3. Self-appearance-aided Differential Evolution for Motion Transfer的网络结构太大，训练时间久。