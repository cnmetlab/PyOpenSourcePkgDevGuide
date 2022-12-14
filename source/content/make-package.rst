创建你的 Python 包
=====================

当你要创建一个 Python 包的时候，你需要搞清楚到底什么是 Python 包。并不是说所有可以用过 ``import`` 语法导入的都是“包”，通常情况下我们创建的一个 ``.py`` 文件，可以叫脚本（script），但其实更官方的名称叫模块（module），模块是可以使用 ``import`` 关键字导入的，但它并不是包。包以目录（文件夹）的形式存在，可以用来组织大量层次化的代码结构，通常带有 ``__init__.py`` 初始化模块做引导。一般我们会默认把带有 ``__init__.py`` 文件的目录视为一个包。

组织你的包结构
---------------
严格意义上来说 Python 包项目的组织结构是自由的，不固定的。但是通常来说，开发者都会有一些约定俗成的组织规则。比如以我的习惯，拿 `pydingbot <https://github.com/Clarmy/pydingbot>`_ 这个项目为例，我会把我的项目结构组织成这样：

.. code::

    .
    ├── LICENSE
    ├── README.md
    ├── docs
    │   └── static
    │       └── config.png
    ├── pydingbot
    │   ├── __init__.py
    │   └── main.py
    ├── requirements.txt
    ├── setup.py
    └── tests
        ├── test_dingbot.py
        └── test_inform.py

项目中与安装、说明、许可证相关的文件放在顶级目录中。

在 ``tests`` 目录中存放单元测试的代码，并且不包含 ``__init__.py`` 文件，也就是说不把 ``tests`` 目录作为一个包来对待，这样后面提到的 ``find_packages`` 函数就不会把它作为包来安装。

由于我们这个包名就叫 ``pydingbot`` 因此在 ``pydingbot`` 目录下添加 ``__init__.py`` 文件，并把所有需要被安装的代码都放在 ``pydingbot`` 目录下，在这个例子里只有一个 ``main.py`` 文件，但即使这个包是一个代码量很大的多层级的包，我也都会把它放在 ``pydingbot`` 下面，而不会在创建一个与 ``pydingbot`` 同级的目录，否则可能会造成包的命名空间比较混乱。

``docs`` 目录存放文档相关的东西，可以在这里构建一个基于 sphinx 框架的文档源文件，用 readthedocs 去构建和发布， 例如 `MetPy <https://github.com/Unidata/MetPy>`_ 和 `cnmaps <https://github.com/cnmetlab/cnmaps>`_ 就是这么干的，关于如何构建 readthedocs 文档，可以参考 :doc:`构建项目文档/使用手册 <documentation>` 。


编写一个 ``setup.py`` 文件
--------------------------
``setup.py`` 文件是对代码进行打包和分发一个安装脚本，不管是你用源码安装，还是构建源码分布或轮子，都绕不开这个脚本。我们先来看一下 pydingbot 的 ``setup.py`` 例子：

.. code:: python

    import setuptools
    import os

    FILE_PATH = os.path.dirname(os.path.realpath(__file__))   # 获取当前文件路径

    with open(os.path.join(FILE_PATH, 'README.md'), 'r') as fh:  # 读取 README.md 的内容用于作为 long_description 参数传入
        long_description = fh.read()

    requirements_path = os.path.join(FILE_PATH, 'requirements.txt')  # 读取依赖列表用于传入下方 setup 函数
    with open(requirements_path) as f:
        required = f.read().splitlines()

    setuptools.setup(
        name='pydingbot',  # 库名称，发布到 Pypi 它就是整个项目的名称，可以与包名不一致。
        version='0.0.3',   # 版本号
        author='Wentao Li',   # 作者，发布到 Pypi 它会显示 
        author_email='clarmylee92510@gmail.com',  # 作者邮箱，发布到 Pypi 它会显示
        description='A package to make dingbot easily to use',  # 摘要，发布到 Pypi 它会作为摘要显示
        long_description=long_description,  # 详细说明，发布到 Pypi 它会作为项目页面的说明文档显示，这里是直接从 README.md 文件读取内容传过来的
        long_description_content_type='text/markdown',  # 详细说明的渲染方式，由于是 Markdown 格式，因此设置为 markdown
        url='https://github.com/Clarmy/pydingbot',   # 项目的主页链接
        include_package_data=True,   # 项目是否包含静态数据文件
        package_data={'': ['*.csv', '*.config', '*.nl', '*.json']},   # 所包含的数据文件声明，这里的意思是包目录中所有 以.csv, .config, .nl, .json 结尾的文件在安装时都要包含，否则安装时会被忽略
        packages=setuptools.find_packages(),   # 包列表，这里使用 find_packages 函数自动扫描和识别包名，其实它是把所有包含 __init__.py 的目录作为一个包来返回的
        install_requires=required,   # 依赖包列表，这里是直接从 requirements.txt 文件中读取后传递进来的，在安装本包的时候依赖包会先置安装
        classifiers=[   # 分类，它会显示在 Pypi 项目主页左侧边栏上，可选列表：https://Pypi.org/classifiers/
            'Programming Language :: Python :: 3', 
        ],
        python_requires='>=3.6'   # Python 版本限制，不满足版本限制的环境下将无法安装本包
    )

这是一个相对简单的项目下的相对简单的安装脚本，有时候除了这个脚本以外，你还会见到与之相关的 ``setup.cfg`` 或者 ``MANIFEST.in`` 文件，其实这些都是辅助的，没有也没关系。

对于上面的 ``setup.py`` 文件，我们也看到了，它的大部分篇幅其实都是在给 ``setup`` 函数塞一大堆参数，很多时候这种参数其实是可以与代码分离，做成一种配置的， ``setup.cfg`` 就是在这种目的下的产物。这种静态配置的方式在我看来有些鸡肋，在这里就不详细讨论了，如果有兴趣的可以自己查一下。

``MANIFEST.in`` 文件也是一个配置文件，你可以在这个文件中声明你的包在安装的时候必须包含那些目录或文件，或者必须排除那些目录或文件。一般来说，我们可以用 ``MANIFEST.in`` 来排除掉单元测试的目录。

.. note:: 
    
    如果从 `Python官方文档 <https://docs.python.org/3.9/distutils/setupscript.html>`_ 里你可能会看到它是从 ``distutils.core`` 中导入的 ``setup`` 函数， ``distutils.core`` 是 Python 标准库中的包分发工具，无需额外安装，但它只能应对小而简单的项目。而 ``setuptools`` 是一个需要额外安装的第三方库，是对 ``distutils`` 的增强，而目前来说 ``setuptools`` 方案已经成为了 Python 包安装的一个实质上的标准方案。


管理项目的版本
---------------
版本管理是一个很重要的事情，因为有代码的地方就会有更新迭代。有的更新是增加功能，有的更新是修复bug，有的更新意味着不再兼容旧代码。所以通常情况下，你的包一定要有版本信息，这样使用者才能根据自己的实际情况来选择适合自己的版本来安装。

一般开发者都会在包的顶级命名空间里加入 ``__version__`` 变量来存储版本号，这样使用者就可以在调用的时候检查包的版本号，例如 numpy 可以通过下面这种方式来获取版本号。

.. code:: python

    import numpy as np
    print(np.__version__)

如果想要实现这种检查版本号的功能，只需要在包顶层目录的 ``__init__.py`` 中给 ``__version__`` 变量赋值即可，但实际上我们在 ``setup.py`` 脚本中也会定义一个包版本号，而且当你发布到 Pypi 时， Pypi 会根据你 ``setup.py`` 脚本中定义的版本号来显示。那么就有可能导致你每次更新版本的时候都需要手动去两个地方修改版本号，如果一旦忘记了在 ``__init__.py`` 中修改版本号，那么就会导致 ``__version__`` 显示的版本号与安装时显示的版本号不一致，为了解决这个问题， Pypi 的指导手册给了 `一个例子 <https://packaging.python.org/en/latest/guides/single-sourcing-package-version/>`_ 让版本号只从一个源头产生。我们可以参照这个例子，让 ``setup.py`` 脚本从 ``__init__.py`` 中获取版本号，我们就只需要去修改 ``__version__`` 的值就行了。

例如 ``cnmaps`` 的 `setup.py <https://github.com/cnmetlab/cnmaps/blob/69d19cdc0fabef9e287d8c7147ee15c858075ebf/setup.py>`_ 就是参考那个例子编写的：

.. code:: python

    import setuptools
    import os
    import codecs


    def read(rel_path):
        here = os.path.abspath(os.path.dirname(__file__))
        with codecs.open(os.path.join(here, rel_path), "r", encoding="utf-8") as fp:
            return fp.read()


    def get_version(rel_path):
        for line in read(rel_path).splitlines():
            if line.startswith("__version__"):
                delim = '"' if '"' in line else "'"
                return line.split(delim)[1]
        else:
            raise RuntimeError("Unable to find version string.")


    FILE_PATH = os.path.dirname(os.path.realpath(__file__))

    with open(os.path.join(FILE_PATH, "README.md"), "r", encoding="utf-8") as fh:
        try:
            long_description = fh.read()
        except UnicodeDecodeError:
            pass

    requirements_path = os.path.join(FILE_PATH, "requirements.txt")
    with open(requirements_path, "r", encoding="utf-8") as f:
        required = f.read().splitlines()

    setuptools.setup(
        name="cnmaps",
        version=get_version("cnmaps/__init__.py"),
        author="Wentao Li",
        author_email="clarmylee92510@gmail.com",
        description="A python package to draw china maps more easily",
        long_description=long_description,
        long_description_content_type="text/markdown",
        url="https://github.com/Clarmy/cnmaps",
        include_package_data=True,
        package_data={"": ["*.geojson", "*.nc", "*.db"]},
        packages=setuptools.find_packages(),
        install_requires=required,
        classifiers=[
            "Programming Language :: Python :: 3",
        ],
        python_requires=">=3.6",
    )

而在 `cnmaps/__init__.py <https://github.com/cnmetlab/cnmaps/blob/69d19cdc0fabef9e287d8c7147ee15c858075ebf/cnmaps/__init__.py>`_ 中将版本号以字符串的形式赋值给 ``__version__`` 变量即可：

.. code:: python

    #...
    __version__ = "1.1.0"
    #...


.. note::

    在对包进行版本管理的时候，版本号的规则建议使用 `语义化版本号 <https://semver.org/lang/zh-CN/>`_ 


使用develop模式进行开发
------------------------
我们都知道，本地运行的脚本和通过 ``setup.py`` 安装的脚本的执行环境是不一样的，安装的包会被放置在一个叫 ``site-packages`` 的目录里，这个目录是在环境变量 ``PATH`` 列表里的，当我们在 ``import`` 某个第三方库的时候，Python 解释器会从这个目录里找到对应的包并导入。

当我们在本地开发一个包的时候，我们肯定希望让自己写的包能够像被安装的包一样运行，所以我们可以在写好 ``setup.py`` 脚本文件以后执行 ``python setup.py install`` 将自己的包安装到 ``site-packages`` 里去。但这样的问题是，这种方式是拷贝式安装，也就是说它会把你的项目拷贝到 ``site-packages`` 里去，甚至可能在里面构建一个轮子。这个时候，你在本地开发环境下再做任何修改，对于你安装的目录来说是无法感知的，想要更新你的包就必须将已安装的包卸载重装，这样会很麻烦。

当我们处在一种需要频繁修改源码的情况下，可以采用 develop 开发模式对包进行“安装”， 也就是执行 ``python setup.py develop`` ，这种模式可以让你对代码的每一次编辑修改都即时地反馈出来而无需卸载重装，因为这种模式并没有把你的代码内容复制到 ``site-packages`` 中，也不会构建轮子，它是把项目的路径加到 ``PATH`` 列表中，并且每次的执行都是从源码运行，你的修改它都能感知到。


构建你自己的命令行
----------------------
有时候对于一些功能非常明确且较为闭合的功能，我们可以把它写成一个命令行工具，这样就不需要每次都通过代码来调用了。对于这种需求， 我们也可以在 ``setup.py`` 中进行指定。

我们以 `mplfonts <https://github.com/Clarmy/mplfonts>`_ 项目为例，它的 ``setup.py`` 是这样写的：

.. code:: python

    import setuptools
    import os

    FILE_PATH = os.path.dirname(os.path.realpath(__file__))

    with open(os.path.join(FILE_PATH, 'README.md'), 'r', encoding='utf-8') as fh:
        long_description = fh.read()

    requirements_path = os.path.join(FILE_PATH, 'requirements.txt')
    with open(requirements_path, encoding='utf-8') as f:
        required = f.read().splitlines()

    setuptools.setup(
        name='mplfonts',
        version='0.0.7',
        author='Wentao Li',
        author_email='clarmylee92510@gmail.com',
        description='Fonts manager for matplotlib',
        long_description=long_description,
        long_description_content_type='text/markdown',
        url='https://github.com/Clarmy/mplfonts',
        include_package_data=True,
        package_data={'': ['rc/matplotlibrc', 'fonts/*']},
        packages=setuptools.find_packages(),
        install_requires=required,
        classifiers=[
            'Programming Language :: Python :: 3',
        ],
        python_requires='>=3.6',
        entry_points={
            'console_scripts': [
                'mplfonts = mplfonts.bin.cli:cli'
            ]
        }
    )

在这个安装文件里，对命令行工具的定义是这里：


.. code:: python

    # ...
    setuptools.setup(
        # ...
        entry_points={
            'console_scripts': [
                'mplfonts = mplfonts.bin.cli:cli'
            ]
        }
    )


它的意思是，定义 ``mplfonts`` 作为一个命令，它的执行路径是 ``mplfonts.bin.cli:cli`` 。这个路径其实是指的 ``mplfonts/bin/cli.py`` 文件里的 ``cli`` 函数。

我们再来看这个函数是怎么定义的：

.. code:: python

    import os

    import fire
    from mplfonts.util.manage import (
        install_fonts, install_font, update_custom_rc, list_font)
    from mplfonts.conf import FONT_DIR


    def init():
        """To set default cjk fonts and put into use"""
        install_fonts()
        update_custom_rc()


    def install(path=None, update=True):
        """
        To install font

        Args:
            path (str): The font file path or directory path
        """
        if not path:
            path = FONT_DIR
        if os.path.isdir(path):
            install_fonts(path)
        elif os.path.isfile(path):
            install_font(path)

        if update:
            updaterc()


    def updaterc(rcfp=None):
        """
        To update matplotlibrc by custom file

        Args:
            rcfp (str): The custom matplotlibrc
        """
        update_custom_rc(rcfp)


    def cli():
        fire.Fire({
            'init': init,
            'install': install,
            'updaterc': updaterc,
            'list': list_font})


可以看到 ``cli`` 函数其实是实例化一个 ``fire.Fire`` 对象， `fire <https://github.com/google/python-fire>`_ 是一个由 Google 团队开发的可以很方便构建命令行功能的包，fire 可以构建复杂的多级命令行结构，具体的使用方法可以参考其文档。本例为命令入口添加了 4 个子命令，当一切安装妥当以后，可以通过执行 ``mplfonts -h`` 来查看命令行的说明文档：

.. code:: bash

    NAME
    mplfonts

    SYNOPSIS
        mplfonts COMMAND

    COMMANDS
        COMMAND is one of the following:

        init
        To set default cjk fonts and put into use

        install
        To install font

        updaterc
        To update matplotlibrc by custom file

        list
        To list font names for choosing

也可以通过在子命令中查看 help: ``mplfonts init -h`` ，这些说明都是在每个子命令所定义的函数的 Docstring 中编写的。

.. code:: bash

    NAME
    mplfonts init - To set default cjk fonts and put into use

    SYNOPSIS
        mplfonts init -

    DESCRIPTION
        To set default cjk fonts and put into use

当然如果 fire 并非唯一的选择，比如另一个比较流行的用于封装 Python 命令行的包 `click <https://github.com/pallets/click/>`_ ，这个可以自己去探索如何使用，这里就不给出示例了。
