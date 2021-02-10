---
title: Python 环境管理
date: 2019-01-07 17:54:05
tags:
    Python
category: Python
---

## virtualenv

安装：

``` sh
pip install virtualenv
```

Mac 上要添加 `sudo`

每个 `virtualenv` 使用文件夹进行区分，新建一个 `evn`：

``` sh
virtualenv my_project
```

进入 `my_project` 文件夹，这时 `env` 还没有激活，激活 `evn`：

``` sh
source bin/activate
```

这时命令会变成：

``` sh
(my_project) ➜  my_project
```

`my_project` 使用的是默认的 `python` 版本，如果要指定 `env` 的版本

``` sh
virtualenv -p /usr/local/bin/python3 my_project2
```

激活后 `python` 的版本就会改变：

``` sh
(my_project2) ➜  my_project2 python --version
Python 3.7.1
```

创建 `env` 是需要注意的一个命令是 `–system-site-packages`，加上这个命令会使用系统已经按照的第三方包，不加就算一个纯净的 `env` 版本。如果要关闭 `env`，输入 `deactivate` 就可以了。

如果要删除 `env`，直接删除对应的文件夹就行。

## virtualenvwrapper

一个文件夹一个 `env`，看上去非常方便，但是时间一长，谁知道自己创建了几个 `env` 的文件夹，管理起来不方便，所有我们需要有一个工具帮助我们使用 `virtualenv`。`virtualenvwrapper` 是 `virtualenv`，可以帮助我们管理 `env` 环境。

安装：

``` sh
pip3 install virtualenvwrapper
```

添加环境变量：

```
if [ -f /usr/local/bin/virtualenvwrapper.sh ]; then
   export WORKON_HOME=$HOME/.virtualenvs 
   source /usr/local/bin/virtualenvwrapper.sh
fi
```

以后我们所有的 `env` 都会安装到 `~/.virtualenvs` 文件夹下。如果打开名命令行的时候显示：

```
/usr/bin/python: No module named virtualenvwrapper
virtualenvwrapper.sh: There was a problem running the initialization hooks.

If Python could not import the module virtualenvwrapper.hook_loader,
check that virtualenvwrapper has been installed for
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python and that PATH is
set properly.
```

那么把 `VIRTUALENVWRAPPER_PYTHON` 设置为 `Python3`。

创建一个 `virtualenv`：

``` sh
mkvirtualenv venv
```

创建的时候 `virtualenv` 参数同样适用：

``` sh
mkvirtualenv -p /usr/local/bin/python3 py3
```

其他命令：

``` sh
lsvirtualenv -b # 列出虚拟环境

workon [虚拟环境名称] # 切换虚拟环境

lssitepackages # 查看环境里安装了哪些包

cdvirtualenv [子目录名] # 进入当前环境的目录

cpvirtualenv [source] [dest] # 复制虚拟环境

deactivate # 退出虚拟环境

rmvirtualenv [虚拟环境名称] # 删除虚拟环境
```

## pyenv

使用 `virtualenv` 和 `virtualenvwrapper` 可以方便的管理 `python` 环境，但是现在我们系统中只有两个版本 `python2` 和 `python3`，如果一个项目要求我们的 `python` 版本是 `3.6`，但是系统的 `python` 版本是 `3.7`，我们该怎么做呢？我们可以下载 `python3.6`，安装到一个文件夹，然后使用 `-p` 新建一个 `virtualenv` 环境。但是这样的话首先下载安装就很麻烦，需要打开浏览器，下载，解压，安装，其次鬼知道我们安装了多少个 `python` 版本，时间一长，很难管理，所以我们需要一个工具 `pyenv`。

安装：

``` sh
brew install pyenv
```

添加环境变量：

```
eval "$(pyenv init -)"
```

查看 `python` 版本列表：

``` sh
pyenv install --list
```

安装对应的版本：

``` sh
pyenv install 3.6.7
```

结果出错了：

``` sh
BUILD FAILED (OS X 10.14.2 using python-build 20180424)

Inspect or clean up the working tree at /var/folders/1d/_h3bhfsd33z3yf4x566ncqkw0000gn/T/python-build.20190107184844.88127
Results logged to /var/folders/1d/_h3bhfsd33z3yf4x566ncqkw0000gn/T/python-build.20190107184844.88127.log

Last 10 log lines:
  File "/private/var/folders/1d/_h3bhfsd33z3yf4x566ncqkw0000gn/T/python-build.20190107184844.88127/Python-3.6.7/Lib/ensurepip/__main__.py", line 5, in <module>
    sys.exit(ensurepip._main())
  File "/private/var/folders/1d/_h3bhfsd33z3yf4x566ncqkw0000gn/T/python-build.20190107184844.88127/Python-3.6.7/Lib/ensurepip/__init__.py", line 204, in _main
    default_pip=args.default_pip,
  File "/private/var/folders/1d/_h3bhfsd33z3yf4x566ncqkw0000gn/T/python-build.20190107184844.88127/Python-3.6.7/Lib/ensurepip/__init__.py", line 117, in _bootstrap
    return _run_pip(args + [p[0] for p in _PROJECTS], additional_paths)
  File "/private/var/folders/1d/_h3bhfsd33z3yf4x566ncqkw0000gn/T/python-build.20190107184844.88127/Python-3.6.7/Lib/ensurepip/__init__.py", line 27, in _run_pip
    import pip._internal
zipimport.ZipImportError: can't decompress data; zlib not available
make: *** [install] Error 1
```

Google 的解决方法，首先：

``` sh
xcode-select --install
```

如果不行的话：

``` sh
CFLAGS="-I$(brew --prefix openssl)/include -I$(xcrun --show-sdk-path)/usr/include" \
LDFLAGS="-L$(brew --prefix openssl)/lib"
```

最终还是不行，尝试用 `brew` 安装 `zlib`

``` sh
brew install zlib
```

然后添加环境变量：

```
LDFLAGS="-L/usr/local/opt/zlib/lib"
CPPFLAGS="-I/usr/local/opt/zlib/include"
```

但是仍然没有用，最后：

``` sh
sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
```

成功！

``` sh
➜  ~ pyenv install 3.6.7
python-build: use openssl from homebrew
python-build: use readline from homebrew
Downloading Python-3.6.7.tar.xz...
-> https://www.python.org/ftp/python/3.6.7/Python-3.6.7.tar.xz
Installing Python-3.6.7...
python-build: use readline from homebrew
Installed Python-3.6.7 to /Users/sunwei/.pyenv/versions/3.6.7
```

然后查看 `python` 版本：

``` sh
➜  ~ pyenv versions
* system (set by /Users/sunwei/.pyenv/version)
  3.6.7
```

当前使用的是系统中的 `python`，我们切换成刚才安装的 `3.6.7`：

```
➜  ~ pyenv global 3.6.7
➜  ~ python --version
Python 3.6.7
```

设置全局版本：

``` sh
pyenv global 3.6.7
```

只对当前目录有效：

``` sh
pyenv local 3.6.7
```

对当前 `shell` 有效：
``` sh
pyenv shell 3.6.7
```

和 `virtualenv` 结合使用，需要安装插件：

``` sh
brew install pyenv-virtualenv
```

新建一个指定 `python` 版本的 `virtualenv`：

``` sh
# pyenv virtualenv <版本号> <文件夹>
pyenv virtualenv 3.6.7 my_env
```

新建的 `virtualenv` 需要指定目录，还是不好管理，那么终极解决方案，使用 ` pyenv-virtualenvwrapper`：

``` sh
brew install pyenv-virtualenvwrapper
```

安装完后需要激活一下：

``` sh
pyenv virtualenvwrapper 
```

然后是使用，我们想新建一个 `python3.6.7` 版本的 `virtualenvwrapper`，

``` sh
pyenv shell 3.6.7
pip install virtualenvwrapper  # 第一次使用新的 Python 环境需要安装此包，否则创建的虚拟环境 Python 版本仍为系统默认
mkvirtualenv python3
```

检查一下结果：

``` sh
➜  ~ workon py3.6.7
(py3.6.7) ➜  ~ python --version
Python 3.6.7
```

大公告成！


参考：

<http://codingpy.com/article/virtualenv-must-have-tool-for-python-development/>
<https://blog.csdn.net/jeikerxiao/article/details/53635267>
<https://lisupy.github.io/2018/10/01/2018-10-01-Mojave%E4%BD%BF%E7%94%A8pyenv%E5%AE%89%E8%A3%85python/#%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95>
<https://my.oschina.net/OHC1U9jZt/blog/2243919>
<https://zhuanlan.zhihu.com/p/30859003>


