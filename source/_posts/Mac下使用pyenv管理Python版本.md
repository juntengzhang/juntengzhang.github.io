---
title: Mac下使用pyenv管理Python版本
date: 2018-10-25 23:48:18
tags: 
---
Mac OS自带的Python版本是2.x，pip安装tensorflow到系统自带Python会失败，又不想使用anaconda，所以希望自己安装一个非系统的Python，同时有多个版本，方便使用，pyenv就是这样一个Python版本管理器。
<!--more-->
## 安装pyenv

```
brew install pyenv
```

然后在.bash_profile中添加一行命令执行初始化
```
eval "$(pyenv init -)"
```
## pyenv常用命令

1. 查看所有pyenv命令
```
pyenv commands
```
2. 查看已安装Python版本
```
pyenv versions
```
3. 查看可安装的Python版本
```
pyenv install -l
```
4. 安装Python
```
pyenv install <version> # version为版本号
```
5. Python版本管理
```
pyenv global <version> 
pyenv local <version>
pyenv shell <version>
```
6. 更新python环境(增删切换版本；pip安装依赖后执行)
```
pyenv rehash
```
7. Python卸载
```
pyenv uninstall <version>
```

