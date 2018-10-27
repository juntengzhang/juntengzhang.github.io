---
title: 优化MAC系统设置
date: 2018-10-24 01:07:58
tags:
---


MAC系统的设置提供了丰富的自定义功能，拿到mac后第一件事就是优化系统设置
<!--more-->
## 触摸板设置优化

触摸板与辅助功能设置入口如下图所示
{% asset_img 触摸板与辅助功能设置入口.png 触摸板与辅助功能设置入口 image %}
1. 左键单击 
轻点任意区域:打开设置-触摸板，参考下图勾选“轻拍来点按”，调节力度为弱
{% asset_img 左键右键单击设置.png 左键右键单击设置 image %}
2. 右键单击
按下右下角:打开设置-触摸板，参考上图修改“辅助点按”为“点按右下角”
3. 向上滚动
双指向下滑动，方向与体验与触摸屏一致:默认，不习惯反方向滚动可以参考下图取消勾选“滚动方向自然”
{% asset_img 滚动方向设置.png 滚动方向设置 image %}
4. 向下滚动
双指向上滑动，方向与体验与触摸屏一致:默认，不习惯反方向滚动可以参考上图取消勾选“滚动方向自然”
5. 拖动/选择内容
三指同时滑动，单手操作，中途可以暂时中断，追加选择内容:打开设置-辅助功能，参考下图勾选“启动拖移：三指拖移”
{% asset_img 三指拖动设置.png 三指拖动设置 image %}
6. 放大/缩小/旋转
双指展开/捏合/旋转，与触摸屏体验一致:默认，不需要修改

## 安装并设置iterm2 sz rz

1. 下载并安装[iterm2](http://www.iterm2.cn/download)
2. 安装brew
打开iterm2，在命令行中输入如下命令:
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
3. 安装lz rz
```
brew install lrzsz
```
4. 安装wget
```
brew install wget
```
5. 下载iterm2-zmodem
在iterm2中使用Zmodem传输文件
```
cd /usr/local/bin
wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-send-zmodem.sh
wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-recv-zmodem.sh
chmod 777 /usr/local/bin/iterm2-*
```
6. 配置iterm2
给终端 iterm2 添加触发trigger
打开 iterm2  -->  Profiles  -->  Default --> Advanced --> Triggers的Edit按钮
```
\*\*B0100               Run Silent Coprocess    /usr/local/bin/iterm2-send-zmodem.sh
\*\*B00000000000000     Run Silent Coprocess    /usr/local/bin/iterm2-recv-zmodem.sh
```
## iterm2共享ssh会话
为避免反复输入Token，需要配置相同host复用ssh链接
```
vim ~/.ssh/config

Host *
    ControlMaster auto
    ControlPath ~/.ssh/master-%r@%h:%p
ControlPersist yes
```
