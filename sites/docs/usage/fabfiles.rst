============================
Fabfile 结构和使用
============================

这个文档包含了 fabfiles 的很多方面，包括怎样写好它们和怎样一次性使用怎么样写。

.. _fabfile-discovery:

探索 Fabfile
=================

Fabric 能载入 Python 模块(例如  ``fabfile.py``) 或者 包 (例如，一个 ``fabfile/`` 目录包含了 一个 ``__init__.py`` 文件)。
默认的，它会寻找一些类似 ``fabfile`` 的名字(根据Python的import机制) - 不管是 ``fabfile/`` 或者是 ``fabfile.py``。


fabfile 寻找算法是搜索调用用户的当前工作目录或任何父目录。
因此,它是面向“项目”使用的, 例如在源代码目录树的根目录放置一个  ``fabfile.py`` 文件。
这样用户无论在项目目录下的哪个目录下都可以找到 fabfile 并调用 ``fab``。


要搜寻一个具体的名字，可以使用命令行的  :选项:`-f` 选项来覆盖，或者添加一个  :引用:`fabricrc <fabricrc>`行，并设置 ``fabfile``的值为文件名。
例如,如果你想命名fabfile 为 ``fab_tasks.py``，你可以先创建这样一个文件 然后调用 ``fab -f fab_tasks.py <task name>``，或者添加
 ``fabfile = fab_tasks.py`` 到 ``~/.fabricrc`` 文件中。


如果给定fabfile名称除了包含文件名还有路径信息(例如 ``../fabfile.py`` 或者 ``/dir1/dir2/custom_fabfile``)，那么它将被当做文件路径直接查找是否存在，而不经过任何的查找。
在这种模式下，tilde-expansion 也会被应用，例如 ``~/personal_fabfile.py``。


.. 注意::

    Fabric 为了进入 fabfile ，对 fabfile 做了一个正常的 ``import`` (实际是一个``__import__``)，不会有任何 ``邪恶`` 或者类似的东西。
    为了这个工作，Fabric 临时的把包含 fabfile 的文件夹加入了 Python的系统路径(随后又立刻删掉了)

.. 版本变动:: 0.9.2
    支持加载 fabfiles 包

.. _importing-the-api:

导入 Fabric
================

因为 Fabric 就是 Python的，你 *可以* 用任何方式导入它。然而，为了封装和易用性
(和为了更容易封装 Fabric 的脚本) Fabric 公共的 API 都维护在 ``fabric.api``  模块中。

Fabric 所有的 :doc:`../api/core/operations`,
:doc:`../api/core/context_managers`, :doc:`../api/core/decorators` and
:doc:`../api/core/utils`  都包含在这个模块作为一个独立的，flat 的命名空间。这使得
你的 fabfiles 可以使用一个简单方面的 Fabric 接口 ::

    from fabric.api import *

    # call run(), sudo(), etc etc

这不是技术上的最佳实践( `有很多原因`_) ，如果你只是使用了几个 Fab API，
明确的导入可能是一个好的做法, 类似 ````from fabric.api  import env, run``。
实际上，在大多数的 fabfiles 中，你将会使用几乎所有的 API，使用 * 来导入::

    from fabric.api import *

将比下面容易读写::

    from fabric.api import abort, cd, env, get, hide, hosts, local, prompt, \
        put, require, roles, run, runs_once, settings, show, sudo, warn

所以在这种情况下实用主义要好过最佳实践。

.. _a number of reasons: http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html#importing


定义任务和导入调用
======================================

关于 Fabric 加载你的 fabfile 之后，处理任务的重要细节 还有怎样更好的导入到其他代码中的一些建议，
请查看  :doc:`/usage/tasks`  在  :doc:`execution` 文档中。
