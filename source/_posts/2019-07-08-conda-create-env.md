---
title: 在服务器用conda创建python环境
date: 2019-07-08 23:53:20
tags: 
- conda
- python
categories: 
- Workshop
---
2019年11月27日更新：

```
conda env export > environment.yml
conda env create -f environment.yml
```

可以导出environment配置环境：

[https://datascience.stackexchange.com/questions/24093/how-to-clone-python-working-environment-on-another-machine](https://datascience.stackexchange.com/questions/24093/how-to-clone-python-working-environment-on-another-machine)

另外，home/xxx/.conda/pkgs会保存很多安装包，可以通过用`conda clean -p`删除未使用的的包([https://segmentfault.com/q/1010000017438362](https://segmentfault.com/q/1010000017438362))

删除环境：conda remove -n your_env_name --all


-------------------------------------

[conda cheatsheet](http://know.continuum.io/rs/387-XNW-688/images/conda-cheatsheet.pdf)

首先，尝试conda list，提示not command，解决方法：

在.bashrc 的末尾添加如下一行：

`export PATH=/opt/anaconda3/bin/:$PATH`

之后conda list，正常。

创建：

conda create -n vso python=2.7

激活：

source activate vso

然后就可以开始安装需要的package

pip install imutils

pip install numpy

pip install opencv-python：遇到报错：libSM.so.6: cannot open shared object file: No such file or directory，解决方案：
（[https://blog.csdn.net/liuyingying0418/article/details/84580254](https://blog.csdn.net/liuyingying0418/article/details/84580254)）

(pip install opencv-python -i https://pypi.tuna.tsinghua.edu.cn/simple)

pip install dlib：安装dlib的时候又出现了cmake报错，用pip install CMake，搞定

pip install scipy

pip install matplotlib

pip install sklearn

pip install ffmpeg-normalize

conda install ffmpeg

conda install -c conda-forge librosa=0.6.3

conda install x264 ffmpeg -c conda-forge（为了用h.264编码，参考：[https://teratail.com/questions/156621](https://teratail.com/questions/156621)）
