=====================
快速教程
=====================

欢迎来到 Fabric!

本篇文档将带领你快速体验 Fabric 的特性，并给出一个快速用法指南。额外的详细文档可以在 :ref:`用法文档 <usage-docs>` 中找到-- 请别忘了看。


什么是 Fabric?
===============

正如 ``README`` 所说:

    .. include:: ../../README.rst
        :end-before: It provides

更具体的来说，Fabric 是:

* 一个让你通过 **命令行** 执行 **任意 Python 函数** 的工具；
* 一个（建立于一个更低层次库的基础上的）程序库使得通过 SSH 执行 SHELL 指令更 **容易** 并且 **Pyhtonic**。

自然而然, 大多数用户将这两个特性结合起来，写函数并通过 Fabric 来执行函数，或者 **任务**，自动化与远程服务器的交互。让我们先睹为快。


你好， ``fab``
==============

没有下面这个“惯例”，这个教程在严格意义上就不能称作教程::

    def hello():
        print("Hello world!")

将代码放置在你当前工作目录下的 ``fabfile.py`` Python模块文件中，这样 ``hello`` 函数就可以被 ``fab`` 工具（作为 Fabric 的一部分安装的）执行，并且如你所料的工作::

    $ fab hello
    Hello world!

    Done.

这就是它所做的。这个功能允许 Fabric 作为一个非常基础的构建工具被使用，甚至不需要导入任何它的 API 。

.. note::

    ``fab`` 工具只是导入了你的 fabfile 并且执行了你指定的函数。这没什么神奇的 -- 你可以用一个普通的 Python 脚本做到的也同样可以在一个 fabfile 中做到！

.. seealso:: :ref:`execution-strategy`, :doc:`/usage/tasks`, :doc:`/usage/fab`


任务参数
==============

把运行时参数传递给你的任务通常是很有用的，就像常规 Python 编程中一样。 Fabric 支持一种 shell 兼容的用法: ``<task name>:<arg>,<kwarg>=<value>,...``。这有点不自然，不过让我们来拓展一下上面的例子，对你自己 say hello::

    def hello(name="world"):
        print("Hello %s!" % name)

默认的，调用 ``fab hello`` 将会表现的和上面一样；但是现在我们可以个性化它::

    $ fab hello:name=Jeff
    Hello Jeff!

    Done.

那些过去用 Python 写过程序的人也许已经猜到了，这个调用与下面的调用效果一样::

    $ fab hello:Jeff
    Hello Jeff!

    Done.

目前，你的参数值总是会做为字符串在 Python 中表现，对于复杂类型，例如列表也许需要一些字符串操作。后续的版本中可能会添加一个类型转换系统以简化此类操作。

.. seealso:: :ref:`task-arguments`

本地命令
==============

正如上面使用到的, ``fab`` 只是省略了几行 ``if __name__ == "__main__"`` 样板而已。 它被设计为通过函数（或 **操作**）使用 Fabric API ，包括执行 SHELL 命令，传输文件等。

让我们来为假想的 Web 应用构建一个 fabfile 文件。这个例子包含如下场景：Web 服务器通过远程主机 ``vcshost``上的 Git 来管理。在 ``localhost`` 上，我们有一个应用的本地克隆副本。当我们推送变更到 ``vcshost`` 时，我们希望能够自动的立刻应用这些变更到我们的远程主机 ``my_server`` 上。我们将通过本地即远程 Git 命令来自动化这个操作。

Fabfile文件通常最好放在项目的根目录下::
    .
    |-- __init__.py
    |-- app.wsgi
    |-- fabfile.py <-- our fabfile!
    |-- manage.py
    `-- my_app
        |-- __init__.py
        |-- models.py
        |-- templates
        |   `-- index.html
        |-- tests.py
        |-- urls.py
        `-- views.py

.. note::

    我们在这里使用一个Django应用，但仅作为一个例子 -- Fabric 不与任何外部代码库绑定，除了它的 SSH 库。

作为起步，也许你想运行我们的测试然后提交到我们的版本控制系统，为我们的部署做好准备::

    from fabric.api import local

    def prepare_deploy():
        local("./manage.py test my_app")
        local("git add -p && git commit")
        local("git push")

输出看起来应该像这样::

    $ fab prepare_deploy
    [localhost] run: ./manage.py test my_app
    Creating test database...
    Creating tables
    Creating indexes
    ..........................................
    ----------------------------------------------------------------------
    Ran 42 tests in 9.138s

    OK
    Destroying test database...

    [localhost] run: git add -p && git commit

    <interactive Git add / git commit edit message session>

    [localhost] run: git push

    <git push session, possibly merging conflicts interactively>

    Done.

这段代码本身很直观：导入一个 Fabric 的 API 函数，`~fabric.operations.local`，并用它来运行本地 SHELL 命令并与之交互。Fabric 其余的 API 也很类似 -- 它们都只是 Python 代码而已。

.. seealso:: :doc:`api/core/operations`, :ref:`fabfile-discovery`


用你的方式来组织
====================

由于 Fabric “只是 Python 代码” 你可以自由的组织你的 fabfile 通过任何你想的方式。例如，经常将任务分解为子任务::

    from fabric.api import local

    def test():
        local("./manage.py test my_app")

    def commit():
        local("git add -p && git commit")

    def push():
        local("git push")

    def prepare_deploy():
        test()
        commit()
        push()

``prepare_deploy`` 任务仍然可以像之前一样使用，但是现在只要你想，你就可以更细颗粒度的调用其他子任务。


故障
=======

我们的基本用例现在仍然工作正常，但如果我们的测试失败了会怎么样？我们希望在部署前，可以有机会消除阻碍，并修复失败测试。

Fabfile 会检验程序的返回值，如果它没正确退出那么就会中止操作。来看看如果我们的一个用例没通过会发生什么::

    $ fab prepare_deploy
    [localhost] run: ./manage.py test my_app
    Creating test database...
    Creating tables
    Creating indexes
    .............E............................
    ======================================================================
    ERROR: testSomething (my_project.my_app.tests.MainTests)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    [...]

    ----------------------------------------------------------------------
    Ran 42 tests in 9.138s

    FAILED (errors=1)
    Destroying test database...

    Fatal error: local() encountered an error (return code 2) while executing './manage.py test my_app'

    Aborting.

太棒了！我们什么都不用做：Fabfile 探测到了失败并中止操作，没有运行 ``commit`` 任务。

.. seealso:: :ref:`Failure handling (usage documentation) <failures>`

故障处理
----------------

但是如果我们希望能灵活的给用户一个选择，该怎么呢？一个叫做 :ref:`warn_only` 的设置
(或者叫 **envrionment variable**，通常缩写为 **env var**)让你将中止行为变成警告，允许灵活的处理出现的错误。

来将这个设置应用到我们的 ``test`` 函数中的，并审查 `~fabric.operations.local` 的调用结果::

    from __future__ import with_statement
    from fabric.api import local, settings, abort
    from fabric.contrib.console import confirm

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

    [...]

在添加这个新特性之前，我们引入了一些新的东西:

* 从 ``__future__`` 中你引入的东西，用于在 Python 2.5 版本中使用 ``with:`` 语句；
* Fabric的子模块 `contrib.console <fabric.contrib.console>` ，包含 `~fabric.contrib.console.confirm` 函数，用于简单的请求输入 yes/no；
* 上下文管理器 `~fabric.context_managers.settings`，用于应用设置到一个具体的代码块；
* 像 `~fabric.operations.local` 这样的运行命令会返回包含结果信息（如 ``.failed`` 或 ``.return_code``）的对象；
* 并且 `~fabric.utils.abort` 函数，用于手动的中止执行。

然而，除了增加了一点复杂度，仍然是相当容易使用的，并且它现在更灵活了。

.. seealso:: :doc:`api/core/context_managers`, :ref:`env-vars`


创建连接
==================

让我们回到重点， 将一个 ``deploy`` 任务放到我们的 fabfile 中，它会在一台或者多台机器上执行，确保你的代码是最新的::

    def deploy():
        code_dir = '/srv/django/myproject'
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

到目前为止，我们引入了几个新内容:

* Fabric 只是 Python 代码 -- 所以我们可以自由的使用常规 Python 代码，像构造变量，字符串插值等；
* 使用 `~fabric.context_managers.cd` 是一个简单的方式调用 ``cd
  /to/some/directory`` 命令。这跟本地运行的 `~fabric.context_managers.lcd` 命令很像。
* `~fabric.operations.run` 命令跟 `~fabric.operations.local` 命令很像，除了它是在**远程主机运行**而不是在本地。

我们还要确认在文件头部导入了新函数::

    from __future__ import with_statement
    from fabric.api import local, settings, abort, run, cd
    from fabric.contrib.console import confirm

改好之后，开始部署::

    $ fab deploy
    No hosts found. Please specify (single) host string for connection: my_server
    [my_server] run: git pull
    [my_server] out: Already up-to-date.
    [my_server] out:
    [my_server] run: touch app.wsgi

    Done.

我们从未在我们的 fabfile 中指定一个连接，因此 Fabric 不知道在哪个远程主机上执行这些命令。当出现这种情况时，Fabric
在运行时会提示我们。连接的定义使用 SSH 风格的“主机串”（例如 ``user@host:port``）并将默认使用你本地的用户名 -- 因此在这个例子中，我们只需要指定主机名 ``my_server``。


远程交互
--------------------

如果你已经部署过了一份你的源码，那么 ``git pull`` 命令会正常工作。-- 但如果这是第一次部署呢？那也很容易处理这种情况通过运行 ``git clone``::

    def deploy():
        code_dir = '/srv/django/myproject'
        with settings(warn_only=True):
            if run("test -d %s" % code_dir).failed:
                run("git clone user@vcshost:/path/to/repo/.git %s" % code_dir)
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

就如我们上面调用 `~fabric.operations.local` 一样，`~fabric.operations.run` 也允许我们基于运行 SHELL 命令来构造干净的 Python 级别逻辑。然而这里有意思的是调用 ``git clone``：由于我们使用 Git 的 SSH 方法来访问我们 Git 服务器上的仓库，这意味着我们的远程 `~fabric.operations.run` 调用会需要身份认证。

老版本的 Fabric （以及类似的高阶 SSH 库）在黑暗中运行远程程序，无法与本地交互。当你需要输入密码或者需要与远程程序交互时这个问题尤其明显。

Fabric 1.0 以及之后的版本突破了这个限制，并允许你总能与其他方面说话。来看看当我们在一台未部署过代码的新服务器运行更新后的 ``deploy`` 任务时会发生什么::

    $ fab deploy
    No hosts found. Please specify (single) host string for connection: my_server
    [my_server] run: test -d /srv/django/myproject

    Warning: run() encountered an error (return code 1) while executing 'test -d /srv/django/myproject'

    [my_server] run: git clone user@vcshost:/path/to/repo/.git /srv/django/myproject
    [my_server] out: Cloning into /srv/django/myproject...
    [my_server] out: Password: <enter password>
    [my_server] out: remote: Counting objects: 6698, done.
    [my_server] out: remote: Compressing objects: 100% (2237/2237), done.
    [my_server] out: remote: Total 6698 (delta 4633), reused 6414 (delta 4412)
    [my_server] out: Receiving objects: 100% (6698/6698), 1.28 MiB, done.
    [my_server] out: Resolving deltas: 100% (4633/4633), done.
    [my_server] out:
    [my_server] run: git pull
    [my_server] out: Already up-to-date.
    [my_server] out:
    [my_server] run: touch app.wsgi

    Done.

注意这里的 ``Password:`` 提示 -- 这是我们远程调用我们 Web服务器上的 ``git`` 命令，请求输入 Git 的密码。我们现在能够输入密码并正常的继续 clone 代码。

.. seealso:: :doc:`/usage/interactivity`


.. _defining-connections:

预定义连接
-------------------------------

在运行时输入连接信息太古老了也不够快，因此 Fabric 提供了其他几种方式在你的 fabfile 中或者其他命令行中指定。我们在这里不准备全讲到，不过我们会告诉你最常用的一个：设置全局主机列表，:ref:`env.hosts <hosts>`。

:doc:`env <usage/env>` 是一个全局类字典对象，驱动着 Fabric 的很多设置，并也可以被写入属性（实际上，
上面看到的 `~fabric.context_managers.settings`，就是它的一个简单包装)。
因此，我们可以在模块层级上修改它，我们的 fabfile 文件顶部看起来像这样::

    from __future__ import with_statement
    from fabric.api import *
    from fabric.contrib.console import confirm

    env.hosts = ['my_server']

    def test():
        do_test_stuff()

当 ``fab`` 加载我们的 fabfile 文件，我们对 ``env`` 的修改就会执行，存储我们修改的设置。最终结果就像上面那样：我们的 ``deploy`` 任务将在 ``my_server`` 服务器上执行。

在这里你也可以告诉 Fabric 在多台远程主机上同时执行：因为 ``env.hosts`` 是一个列表，``fab`` 迭代遍历它，在每一台主机上执行我们指定的任务。

.. seealso:: :doc:`usage/env`, :ref:`host-lists`


结论
==========

我们完成的 fabfile 仍然很简短。下面是它的完整内容::

    from __future__ import with_statement
    from fabric.api import *
    from fabric.contrib.console import confirm

    env.hosts = ['my_server']

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

    def commit():
        local("git add -p && git commit")

    def push():
        local("git push")

    def prepare_deploy():
        test()
        commit()
        push()

    def deploy():
        code_dir = '/srv/django/myproject'
        with settings(warn_only=True):
            if run("test -d %s" % code_dir).failed:
                run("git clone user@vcshost:/path/to/repo/.git %s" % code_dir)
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

这个 fabfile 文件用到了 Fabric 特性集中的很大一部分：

* 定义 fabfile 任务并通过 :doc:`fab <usage/fab>` 运行它们；
* 通过 `~fabric.operations.local` 调用本地 SHELL 命令；
* 修改 env 变量通过 `~fabric.context_managers.settings`；
* 处理命令故障，提示用户，并手动中止；
* 定义主机列表，并且调用 `~fabric.operations.run` 运行远程命令

然而，还有很多我们这里没有讲到的！请跟随 “see also”链接，并浏览文档索引 :doc:`the main index page <index>`。

感谢阅读！
