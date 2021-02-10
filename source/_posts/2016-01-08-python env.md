---
title: python env
date: 2016-01-08 14:39:39
tags:
    Python
category: Python
---
## 安装 pyenv
使用 git 把 pyenv 下载到用户目录：
```sh
$ cd
$ git clone git://github.com/yyuu/pyenv.git .pyenv
```
然后需要修改环境变量，使用 Bash Shell 的输入
```sh
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile  
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
```
使用 Zsh Shell 的输入
```sh
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
```
最后添加 pyenv init
```sh
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile #Bash
```
或
```sh
$ echo 'eval "$(pyenv init -)"' >> ~/.zshrc #Zsh
```
输入以下命令重启 Shell
```sh
$ exec $SHELL
```
然后就可以使用 pyenv 了。

## 更新 pyenv

使用 git 更新 pyenv
```sh
$ cd ~/.pyenv
$ git pull
```

## 安装Python版本
列出所有的版本
```sh
$ pyenv install --list
```
或者
```sh
$ pyenv install miniconda
python-build: definition not found: miniconda

The following versions contain `miniconda' in the name:
  miniconda-latest
  miniconda-2.2.2
  miniconda-3.0.0
  miniconda-3.0.4
  miniconda-3.0.5
  ...
  miniconda3-3.16.0
  miniconda3-3.18.3
  miniconda3-3.19.0

See all available versions with `pyenv install --list'.

If the version you need is missing, try upgrading pyenv:

  cd /Users/sunwei/.pyenv/plugins/python-build/../.. && git pull && cd -
```
安装卸载
```sh
$ pyenv install ...
$ pyenv uninstall ...
```
查看已经安装的
```sh
$ pyenv versions
```
## 设置全局版本
```sh
$ pyenv global ...
```
## 设置文件夹版本
```sh
$ pyenv local ...
```
## 安装pyenv virtualenv
```sh
$ git clone https://github.com/yyuu/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
```
或者
```sh
$ brew install pyenv-virtualenv
```
## 新建一个virtual env
```sh
$ pyenv virtualenv 2.7.10 my-virtual-env-2.7.10
```
## list env
```sh
$ pyenv virtualenvs
```
## 使用evn
```sh
$ pyenv activate <name>
$ pyenv deactivate
```

## 参考
<https://github.com/yyuu/pyenv-virtualenv>
<http://huangziwei.com/tech/setting-up-scientific-python-environment-in-os-x-10-10-using-miniconda/>