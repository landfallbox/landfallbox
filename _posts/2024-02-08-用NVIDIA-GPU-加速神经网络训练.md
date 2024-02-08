---
title: 用 NVIDIA GPU 加速神经网络训练
tags: 
    - 机器学习 
    - 深度学习
    - CUDA 
    - PyTorch
    - Python
header: 
    theme: dark
mode: immersive
key: accelerating-neural-network-training-with-GPU
lang: zh
article_header: 
  type: overlay 
  theme: dark
  background_color: '#203028'
  background_image: 
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0, .4), rgba(0, 0, 0, .6))'
    src: https://raw.githubusercontent.com/landfallbox/Pictures/master/mmexport1687019814894.jpg
---

对于使用英伟达显卡的电脑或服务器来说，如果希望使用显卡加速神经网络的训练，必须借助 CUDA 框架。

CUDA 框架用于加速神经网络需要安装两个内容：CUDA toolkit 和 cuDNN。其中，CUDA toolkit 的版本必须与本机显卡支持的 CUDA 框架版本相同；cuDNN 本质是一个 SDK，专用于神经网络加速。cuDNN 和 CUDA 版本间没有对应关系，一般一个版本的 CUDA toolkit 会支持多个版本的 cuDNN。

> 本文只涉及 Windows 系统下的下载、安装、配置等相关内容，其他情况请查询其他教程。
>
> 本文下载、安装、配置的版本为 CUDA 11.0，PyTorch 1.7.1，不同版本的上述过程可能存在一定差异。

# 确定 CUDA 版本

首先当然需要查询本机显卡支持的 CUDA 版本。方法有很多，本文只提一种：可以直接使用 Windows 的命令行工具查询。

在 cmd 中输入 `nvidia-smi` 便可以查到所需信息：

![image-20240202183037413](https://raw.githubusercontent.com/landfallbox/Pictures/master/image-20240202183037413.png)

此处的 `CUDA Version:11.0` 的意思是最高支持 11.0 版本的 CUDA，小于等于 11.0 的版本都是可以正常使用的。

# 下载 CUDA 

CUDA 的下载地址是 https://developer.nvidia.com/cuda-toolkit-archive。网页上有全部存档的不同版本的 CUDA，选择对应版本的即可。此外，还需要依次选择操作系统、架构、操作系统版本、安装类型。

网站上没有把 Windows 11 作为单独的操作系统版本列出来，但 Windows 10 的安装包是兼容 Windows 11 的，故 Windows 11 的用户选择 *10* 即可。

安装类型分为 local 和 network，前者下载的 exe 比较大但后续安装过程中不会再有联网下载的内容，后者下载的 exe 比较小但后续安装过程中还会再有联网下载的内容，按需选择即可。

![image-20240202183611188](https://raw.githubusercontent.com/landfallbox/Pictures/master/image-20240202183611188.png)

> CUDA 的代码编译是依赖 Microsoft Visual Studio 的，故如果希望使用 CUDA 的代码编译在下载 CUDA 之前还要下载安装 Microsoft Visual Studio。这也是一些配置深度学习的教程中会首先包含 Microsoft Visual Studio 相关内容的原因。
>
> 就加速神经网络而言，不安装 Microsoft Visual Studio 是不影响使用的。

# 安装 CUDA

运行下载好的 exe 会得到如下界面：

<img src="https://raw.githubusercontent.com/landfallbox/Pictures/master/image-20240202185054894.png" alt="image-20240202185054894" style="zoom:50%;" />

这个路径只是 CUDA Toolkit installer 的临时解压路径，修改与否影响不大。可以随便找个空间较大磁盘中的文件或者临时新建一个。

​	解压完成后安装程序会自动运行：

<img src="https://raw.githubusercontent.com/landfallbox/Pictures/master/image-20240202185610992.png" alt="image-20240202185610992" style="zoom:40%;" />

安装选项建议选择自定义而不是默认的精简。因为精简会跳过选择安装位置的步骤，而默认的安装位置在 C 盘的 Program Files 下。CUDA 的大小在数个 G 左右，对于 C 盘空间不那么宽裕的用户来说可能不是那么友好。当然，如果没有类似困扰，大可选择精简以省去麻烦。

其余可选内容保持默认即可，关于 Microsoft Visual Studio 的警告也是忽略即可，前文已经解释过了。

# 验证安装是否成功

在 cmd 中输入 `nvcc -V` 如果出现与下图相似的内容则说明安装成功：

<img src="https://raw.githubusercontent.com/landfallbox/Pictures/master/image-20240202191315269.png" alt="image-20240202191315269" style="zoom: 50%;" />

> 部分教程在安装完成后、验证安装是否成功之前还会有配置环境变量相关的内容，但在实际操作中，笔者的电脑上省去相关步骤后直接验证是成功的，故本文没有写与配置环境变量相关的内容。

# 下载 cuDNN 

cuDNN 的下载地址是 https://developer.nvidia.com/rdp/cudnn-archive。

下载页面给出了不同版本的 cuDNN 支持哪些版本的 CUDA，可以根据 CUDA 版本选择。需要说明的是，如果写的是 *for CUDA 11.x* 意思是支持所有 11 开头版本的 CUDA，如 11.4，11.5 等。如果写的是 *for CUDA 11.1* 意思是只支持 11.1 版本的 CUDA。

下载压缩包是需要 NVIDIA 账号的，没有的话需要自行注册。

解压后除了 LICENSE 还有三个文件夹：bin、include、lib，把三个文件夹包含的内容分别添加到 CUDA 安装目录下的 bin、include、lib 中即可。

> 笔者使用的是 Windows 11。直接选中 bin、include、lib 这三个文件夹复制粘贴到 CUDA 的安装路径下就会自动添加其中的内容到对应文件夹下，其他操作系统或其他版本的 Windows 系统不确定是否支持这样操作，仅供参考。

# 下载 PyTorch

PyTorch 的官网地址是 https://pytorch.org/。

官网往下翻可以看到最新版本的 PyTorch 的安装方法：

![image-20240202194257740](https://raw.githubusercontent.com/landfallbox/Pictures/master/image-20240202194257740.png)

可以看到 PyTorch 的版本受限于 Python 的版本。此外，安装命令会因为操作系统、安装工具、语言、计算平台存在不同，选择需要的版本就可以。

如果最新版本的 PyTorch 不能满足需要，可以点击 *Previous versions of PyTorch* 查找存档的过往版本中符合要求的。

例如支持 CUDA 11.0 的最新版本的 PyTorch 的版本是 1.7.1。其安装命令是：

```
conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cudatoolkit=11.0 -c pytorch
```

其中，`-c` 用于指定下载源地址。如果下载失败或下载速度较慢，可以更换为国内的镜像源。

# 验证 Pytorch

```python
import torch

# 判断pytorch是否支持GPU加速
print(torch.cuda.is_available())
```

运行后结果是 `True` 说明安装成功。

# 参考资料

https://blog.csdn.net/Castlehe/article/details/112329058

https://blog.csdn.net/Yao_Wan/article/details/128669932

