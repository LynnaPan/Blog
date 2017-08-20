---
title:  实景图像的人脸识别方法
date:   2017-04-19 13:10:53 +0800
categories: Machine Learning
---

人脸识别系统一般需要在校准好的特定情景下捕捉并处理图像。然而，现实生活中情况千变万化，有不同的角度，不同表情以及光照条件。传统的人脸识别算法在处理这种图像时就显得捉襟见肘。这里展示一种用于在现实场景中的人脸识别方法，这种方法只需要很少的训练样本就可以高效地训练。我们的方法包含了一个创新的校准流程和稀疏表示法分类器。这个方法在一个复杂的实景人脸数据集中的识别率，远远超过了当前的最先进技术。

这是一种非常高效高精度的算法，可以在复杂的现实场景中识别人脸。最初的人脸识别算法只用于检测脸上的五官，比如捕捉眼睛嘴巴等。后来，人们提出了新的方法，比如用主分量分析（PCA），稀疏表示法分类器（SRC），以及后面提出的算法MSPCPC/PCPC。最近几年开始比较流行使用深度学习方法来进行人脸识别，这些算法识别率固然好，但都需要有相当量的数据和特定硬件来训练和使用。
作者提出的算法采用改良版的鲁棒稀疏编码(RSC)算法。由于RSC算法需要线型组合的训练样本，也就是每张图像都是相似大小，高校准，纯正面的，所以用RSC算法前需要进行自动脸部校准的预处理。

## 一．自动脸部校准

预处理的目的是对照参考图像，把输入的脸部进行网格变形。步骤是1. 采用Viola-Jones方法在图片中探测到脸；2. 用回归树方法探测脸部标记；3. 计算Delaunay三角剖分网格；4. 最后就可以利用仿射变换将网格中的点对应到参考图像的点，进行一系列列转换，缩放和变形，最终得到校准的图像。
![](/images/face/1.png)
<center>图1</center>
## 二．脸部识别
脸部识别算法采用修改版的RSC，是SRC方法的改良算法。SRC方法是创建一个包含所有训练图像的字典矩阵D，对于一张位置的图像y，目标就是需要找到一个向量的权重x使得y=Dx。用x的L1范数作为正则化项，x必须要是一个稀疏矩阵。
RSC与SRC的区别是，RSC采用一个对角线权重矩阵W来提高对于光照变化的适应性和健壮性，并且采用了最大似然估计来解决稀疏编码带来的问题。综上，RSC算法解决了加权套索问题：
![](/images/face/2.png)
这里ε是一个表示噪声等级的常量。如果我们用L2范数代替L1范数，我们会得到最小二乘问题:
![](/images/face/3.png)
当`λ > 0`，可以得到分析解
![](/images/face/4.png)
        虽然这个方法需要转换`#（训练样本数）× #（训练样本数）`大小的矩阵，但比L1范数版本的算法会高效得多。
## 三．实验结果
在高校准的AR数据集中，作者的算法达到了95%的识别率。为了跟现今流行的方法比较，在实验中作者采用了更有挑战性的包含13000张5749个人在不同环境下的图像数据库，用LFWa版本的数据集，选取了7张图像做训练，3张图像用于测试，设计了3个实验：
1.	在原始LFWa数据集上应用RSC算法
2.	在预先有脸部探测到的LFWa图像上应用RSC算法
3.	在进行了校准步骤的LFWa图像上应用改良版的RSC算法。

数据集中的一些图像如下图（a）中，没有很好的校准。
![](/images/face/8.PNG)
3个实验的结果如下
![](/images/face/5.png)
除此之外，作者还采用只有2个和5个训练样本，与当下最领先的方法进行了比较得到了如下结果：
![](/images/face/6.png)
最后，作者还做了每个人仅有一张训练图像的实验，得到了如下结果：
![](/images/face/7.png)
以上实验结果显示，文中的方法可以在近实时下比别的算法有更好的识别率。虽然深度学习方法可以在LFW数据集中有高达96%的识别率，但这些算法都需要大量（300000张）样本和强大的硬件来运算。文中的方法可以有相对精确的识别率，却只使用少于10张训练样本，因此非常高效。
## 结论
我们介绍了一种可以用于实景图像下非常高效的脸部识别技术，只需要少于10张训练样本和普通的硬件就可以得到近似实时的识别效果。相比于现今比较流行的方法，在计算运行时减半之际，我们的算法几乎可以达到两倍的识别率。在LFW数据集中的实验结果表明，我们的方法远远超过了现有的非深度学习算法。

## Github
### 安装
- 安装一些python安装包:
```
pip install numpy scipy Pillow l1ls sklearn matplotlib
```
- 你还需要python的OpenCV。这是一种安装方法：可以用以下 cmake 命令:
```
cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D BUILD_EXAMPLES=ON  -D WITH_OPENGL=ON -D WITH_VTK=ON -D WITH_GTK=ON -D WITH_CUDA=OFF ..
```
- 别忘了其他一些依赖包:
```
sudo apt-get install libopenblas-dev
```
- 从Boost官方网站下载Boost，然后进入boost目录，运行一下命令：
```
./bootstrap.sh --with-libraries=python
./b2
sudo ./b2 install
```
- 然后下载 DLIB:
```
git clone https://github.com/davisking/dlib.git
git checkout tags/v19.0 # current version did not work for me
sudo python setup.py install
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```
- 在 `config.py` 文件中修改 `haarcascade_frontalface_default.xml` 和 `the shape_predictor_68_face_landmarks.dat` 文件的路径。
### 数据集
为了能够比较与其他算法的结果，我采用LFW数据集的LFWa版本。可以用如下命令下载：
```wget http://www.openu.ac.il/home/hassner/data/lfwa/lfwa.tar.gz
tar xvfz lfwa.tar.gz
cp -R lfw2 <directory-to-store-dataset>/.
```
在`config.py`中修改`lfw2`文件夹的路径。
### 运行脸部识别器
运行 `python recognizer.py` 来运行论文中的代码。你可以在`config.py`中修改参数（比如训练样本的数量）。你只需要修改这个文件，其余可原封不动。
### 参考文献
[1] Yang, M., Zhang, L., Yang, J., & Zhang, D. (2011, June). Robust sparse coding for face recognition. In Computer Vision and Pattern Recognition (CVPR), 2011 IEEE Conference on (pp. 625-632). IEEE.

