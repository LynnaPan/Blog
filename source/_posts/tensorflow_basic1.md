---
title: tensorflow 环境建立（Anaconda)
date:   2017-09-28 21:30:53 +0800
categories: Machine Learning
---

## Anaconda安装
刚接触数据分析的同学会经常看到大牛们提到用conda创建环境，安装一些包。其实这些是来自于Anaconda。Anaconda是一个包含数据科学常用包的Python发行版本。它基于conda——一个包和环境管理器——衍生而来。你可以使用conda创建环境，以便分隔使用不同Python版本和不同程序包的项目。你还将使用它在环境中安装、卸载和更新包。通过使用Anaconda，处理数据的过程将更加愉快。：）

下载链接：https://www.anaconda.com/download/ 。不得不吐槽一下，这个不翻墙下载下来真的很慢。有必要传一个百度网盘给大家用。

这里只针对mac或linux系统。在mac和linux系统上，下载安装后可以在命令窗口敲conda，有这个工具就说明安装成功了。conda有以下两个主要的功能。

1. **管理包**：其实conda和pip很相似，不同之处是可用的包以数据科学包为主，而pip适合一般用途。与此同时，conda并非像pip那样专门适用于Python，它也可以安装非Python的包。它是支持任何软件的包管理器。也就是说，虽然并非所有的Python库都能通过 Anaconda发行版和conda获得，但同时它也支持非Python库的获得。在使用conda的同时，你仍可以使用pip来安装包。

2. **虚拟环境**：conda除了可以管理包，它还可以管理虚拟环境。它跟[virthalenv](https://virtualenv.pypa.io/en/stable/)和[pyenv](https://github.com/pyenv/pyenv)很类似。虚拟环境在你要同时开发很多个项目时很有用，例如，你的代码可能使用了Numpy中的新功能，或者使用了已删除的旧功能。实际上，不可能同时安装两个Numpy版本。你要做的应该是，为每个Numpy版本创建一个环境，然后项目的对应环境中工作。
在应对Python 2和Python 3时，此问题也会常常发生。你可能会使用在 Python 3 中不能运行的旧代码，以及在Python 2中不能运行的新代码。同时安装两个版本可能会造成许多混乱和错误，而创建独立的环境会好很多。
你也可以将环境中的包列表导出为文件，然后将该文件与代码包括在一起。这能让其他人轻松加载代码的所有依赖项。pip 提供了类似的功能，即`pip freeze > requirements.txt`。

## 创建tensorflow环境
用conda创建虚拟环境有基本的几个步骤：
1. 创建或管理环境：要创建环境，请在终端中使用`conda create -n <env_name>  <list of packages>`。 <env_name>对应你要创建的环境名称，<list of packages>对应你要在环境中安装的包。比如我要创建一个环境my_env, 要在环境中安装numpy，对应的命令是`conda create -n my_env numpy`
创建环境时，可以指定要安装在环境中的 Python 版本。这在你同时使用 Python 2.x 和 Python 3.x 中的代码时很有用。要创建具有特定 Python 版本的环境，请键入类似于`conda create -n py3 python=3`或`conda create -n py2 python=2`的命令。

2. 进入环境：创建了环境后，在 OSX/Linux 上使用`source activate <env_name>`进入环境。

3. 进入环境中仍然可以继续安装一些包：用`conda install <package_list>`来安装


那么我们要创建一个tensorflow的环境，可以用下列的命令来创建。

```
conda create -n tensorflow python=3.5
source activate tensorflow
conda install pandas matplotlib jupyter notebook scipy scikit-learn
conda install -c conda-forge tensorflow
```

### 更多环境操作

1. 保存环境：保存环境能让其他人安装你的代码中使用的所有包，并确保这些包的版本正确。你可以使用`conda env export > environment.yaml`将包保存为 YAML。命令的第一部分`conda env export`用于输出环境中的所有包的名称（包括 Python 版本）。

2. 加载环境：要通过环境文件创建环境，这会创建一个新环境，而且它具有同样的在environment.yaml中列出的库。请使用
```
conda env create -f environment.yaml
```

3. 退出环境：
```
source deactivate <env_name>
```

4. 列出环境：
```
conda env list
```

### 加速下载

一个小tip，国内conda install来安装包也特别慢，翻墙了也效果不佳。可以用清华提供的anaconda镜像，使用以后真的很快！https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

运行以下命令:
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

即可添加 Anaconda Python 免费仓库。亲测有效～



