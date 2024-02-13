---
layout: article
title: Windows 下 NVM 的下载和安装
mode: immersive
header:
  theme: dark
article_header: 
  type: cover
  image:
    src: https://raw.githubusercontent.com/landfallbox/Pictures/master/20240213141513.png
tags: 
    - NVM 
    - Node
    - Vue 
    - 前端
---

如果希望使用 Vue，则需要使用 Node.js 。然而，手动管理 Node 的版本和下载安装是比较麻烦的，于是 NVM 应运而生。

# 下载和安装 NVM

NVM 是开源在 Github 上的项目，其下载地址为：https://github.com/coreybutler/nvm-windows/releases ，推荐下载 nvm-setup.exe 。

下载完成后，运行安装程序。其中，需要指明 NVM 的安装路径和 Node.js 的安装路径，注意不要出现空格或中文。

可以在命令行中输入 `nvm -v` 判断是否安装成功。成功时显示 nvm 版本号。

# 安装 Node.js

## 添加镜像

为更好地下载安装 Node 和 npm ，需要配置镜像地址。

在 nvm 的安装路径下找到 setting.txt ，打开后在其中添加

```txt
node_mirror: https://npm.taobao.org/mirrors/node/ 
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

## 下载安装

打开命令行，输入 `nvm install xx` 即可安装对应版本的 Node。

安装完成后，输入 `nvm use xx.xx.xx` 以使用对应版本的 Node。

可以使用 `node -v` 和 `npm -v` 检验 Node 安装成功与否。成功时显示 node 和 npm 的版本。

## 多版本

此时便可见到使用 NVM 管理 Node 的好处，无需配置环境变量即可在电脑上安装多个版本的 Node 。当需要使用某个版本时，只需要使用 `nvm use xx.xx.xx` 即可在版本间切换。

可以使用 `nvm list` 查看当前已安装的有哪些版本。

可以使用 `nvm current` 查看当前使用的是哪个版本。

# 可能存在的问题和对应的解决方案

如果已经使用过 Node ，即电脑上装有一个或多个版本的 Node ，转而希望借助 nvm 管理 Node ，需要先卸载已有的全部版本，再执行以上步骤。此外，还要删除先前为使用 Node 而配置的环境变量。

然而，即使完成上述步骤，还可能存在这样的情况：nvm 和 Node 安装一切顺利，指定使用使用某个版本时，也提示使用成功，但 `nvm current` 就是没有效果，node 和 npm 版本检验始终无效，项目也不能运行。

此时，建议手动检查原来 Node 的安装位置，如果发现 *node_global* 或 *node_cache* 之类未卸载干净的文件夹，需要把他们删除，然后使用 `nvm use` 命令。

# 参考资料

[ 使用nvm管理nodejs（把高版本降级为低版本）_nvm降低node版本_Yunlord的博客-CSDN博客](https://blog.csdn.net/kobepaul123/article/details/125360217)