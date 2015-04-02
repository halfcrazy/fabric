================
环境字典 ``env``
================

Fabric 的一个简单但完整的方面是 “environment”：一个 Python 字典子类，被用作设置组合记录以及共享任务内部的数据和命名空间。

环境字典当前通过一个全局单例来实现， ``fabric.state.env``，并且为了方便被包含在 ``fabric.api`` 中。``env`` 中的键有时也被认为是“环境变量”。

环境即配置
==========

大多数 Fabric 的行为通过修改 ``env`` 变量是可控的，例如 ``env.hosts``（就像在 :ref:`the tutorial <defining-connections>` 中能看到的）。其他通常会被修改的环境变量有：

* ``user``：当建立 SSH 连接时，Fabric 默认使用你本地的用户名，但是你可以使用 ``env.user`` 去覆盖它如果你需要的话。:doc:`execution` 文档中也有一些关于如何为每一台主机指定用户名的基础信息。
* ``password``：用于具体设置你默认连接的密码或者 sudo 密码，如果需要的话。Fabric 会提示你如果你还没有设置或者设置的内容不合法。
* ``warn_only``：一个布尔值，用于设定 Fabric 在探测到错误时是否退出。查看这个行为的更多信息，请看 :doc:`execution`

还有很多其他的环境变量；查看完整列表，请看文档底部的 :ref:`env-vars`

上下文管理器 `~fabric.context_managers.settings`
------------------------------------------------

在很多请看下，临时修改 ``env`` 变量是很有用的，因此一个给定的设置修改只作用于一个代码块。Fabric 提供了一个上下文管理器 `~fabric.context_managers.settings`，它接受任意数目的 键/值 对 并用它们来修改它所包裹的块中的 ``env`` 。

举例来说，有很多设定 ``warn_only`` 很有用的场景（看下面）。
将它用到代码中，使用 ``settings(warn_only=True)``，正如在这个 ``contrib``  简化版本中看到的 `~fabric.contrib.files.exists` 函数::

    from fabric.api import settings, run

    def exists(path):
        with settings(warn_only=True):
            return run('test -e %s' % path)

查看 :doc:`../api/core/context_managers` API 文档了解 `~fabric.context_managers.settings` 以及其他类似工具的详细信息。

环境作为共享状态
================

正如前面提到的，``env`` 对象只是一个字典的子类，所以你的 fabfile 文件中的代码也可以在里面存储信息。有时这对于保持一次执行中多个任务之间的状态是很有用的。

.. note::

    ``env`` 的这方面是由于历史原因：在过去，fabfile 文件并不是纯 Python 代码，因此环境变量是任务间沟通的唯一办法。现在，你可以直接调用其他任务或者子程序库，如果你愿意的话，甚至可以保存模块级别的共享状态。

    在未来的版本中，Fabric 会变得线程安全，在这种情况下 ``env`` 可能是保持全局状态的唯一简单并安全的方法。

其他考虑
========

因为它是 ``dict`` 的子类，Fabric 的 ``env`` 的值可以通过访问属性来读/写，正如在上面的材料中看到的。换句话说，``env.host_string`` 和 ``env['host_string']`` 在功能上是等价的。我们觉得访问属性通常可以简化一点输入并增加代码的可读性，所以，使用属性来读写值是推荐的与 ``env`` 交互的方式。

它是个字典的事实在其他方面也很有用，例如基于 Python  ``dict`` 的字符串插值，如果你需要在一个字符串中插入多个变量值的时候是很有用的。使用“常规”字符串插值看起来像这样::

    print("Executing on %s as %s" % (env.host, env.user))

使用字典风格的差值可读性更强，也更简短。

        print("Executing on %(host)s as %(user)s" % env)

.. _env-vars:

环境变量完整清单
================

下面的所有预定义的（或者是 Fabric在运行中定义的）环境变量清单。尽管它们中的许多可以被直接操作，但是通常最好使用 `~fabric.context_managers`，或者通过 `~fabric.context_managers.settings` 又或者是指定具体的上下文管理器例如 `~fabric.context_managers.cd`。

注意，它们中的许多可以通过 ``fab`` 的命令行参数来设置 -- 详细信息请看 :doc:`fab`。相应的地方也提供有交叉引用。

.. seealso:: :option:`--set`

.. _abort-exception:

``abort_exception``
-------------------

**默认值：** ``None``

Fabric 通过打印一个错误信息到 stderr 并调用 ``sys.exit(1)`` 来处理中止。这个设置允许你复写这个行为（即 ``env.abort_exception`` 发生时做什么。）

给它一个可调用的对象，它可以接受一个字符串（原来将被打印的错误信息），并返回一个异常实例。这个异常对象将被抛出以替代（原来 ``sys.exit`` 执行的） ``SystemExit`` 。

很多时候，你想你把它设置为一个简单的异常类，它完美的匹配上面的描述（可调用，接受一个字符串参数，返回一个异常实例。）例如 ``env.abort_exception = MyExceptionClass``。

.. _abort-on-prompts:

``abort_on_prompts``
--------------------

**默认值：** ``False``

当为``真``时，Fabric 会运行在非交互模式，这时任何要求用户输入（例如输入密码，输入你连接的主机，fabfile调用 `~fabric.operations.prompt`等。）都会调用 `~fabric.utils.abort` 。这允许用户确保在不可预见的情况发生时一个 Fabric 会话总是会干净的结束而不是被等待用户的输入阻塞。

.. versionadded:: 1.1
.. seealso:: :option:`--abort-on-prompts`


``all_hosts``
-------------

**默认值：** ``[]``

由 ``fab`` 设置的当前执行命令的完整主机列表。仅供显示信息。

.. seealso:: :doc:`execution`

.. _always-use-pty:

``always_use_pty``
------------------

**默认值：** ``True``

当设置为 ``false`` 时，会使 `~fabric.operations.run`/`~fabric.operations.sudo` 的行为像他们被用 ``pty=False`` 参数调用时一样。

.. seealso:: :option:`--no-pty`
.. versionadded:: 1.0

.. _colorize-errors:

``colorize_errors``
-------------------

**默认值** ``False``

当设置为 ``True`` 时，错误以红色输出到终端，警告以品后色输出到终端，以使它们更容易被看见。

.. versionadded:: 1.7

.. _combine-stderr:

``combine_stderr``
------------------

**默认值**: ``True``

使 SSH 层合并远端程序的 stdout 和 stderr 流以避免打印时混淆在一起。查看 :ref:`combine_streams` 了解更多细节，了解为什么需要这个功能以及它有什么效果。

.. versionadded:: 1.0

``command``
-----------

**默认值：** ``None``

由 ``fab`` 设置的当前执行的任务名（例如，当执行 ``$ fab task1 task2`` 时，当``task1`` 正在执行时 ``env.command`` 会被设置为 ``task1``，当``task2`` 正在执行时，会被设置为 ``"task2"``。）仅供显示信息。

.. seealso:: :doc:`execution`

``command_prefixes``
--------------------

**默认值：** ``[]``

由 `~fabric.context_managers.prefix` 修改，并追加在由 `~fabric.operations.run`/`~fabric.operations.sudo` 执行的命令之前。

.. versionadded:: 1.0

.. _command-timeout:

``command_timeout``
-------------------

**默认值：** ``None``

远程命令超时时间，以秒作单位。

.. versionadded:: 1.6
.. seealso:: :option:`--command-timeout`

.. _connection-attempts:

``connection_attempts``
-----------------------

**默认值：** ``1``

当 Fabric 连接一台新服务器时尝试连接的次数。处于向后兼容的原因，默认只尝试连接一次。

.. versionadded:: 1.4
.. seealso:: :option:`--connection-attempts`, :ref:`timeout`

``cwd``
-------

**默认值：** ``''``

当前工作目录。用于保持 `~fabric.context_managers.cd` 上下文管理器的状态。

.. _dedupe_hosts:

``dedupe_hosts``
----------------

**默认值：** ``True``

合并重复的主机列表，使给定的主机字串只出现一次（例如，当组合使用 ``@hosts`` + ``@roles``，或者 ``-H`` 和 ``-R`` 时。）

当设置为 ``False`` 时，允许重复，这将明确允许用户在一台相同的主机上运行一个任务多次（并行的，尽管串行也能正常工作）。

.. versionadded:: 1.5

.. _disable-known-hosts:

``disable_known_hosts``
-----------------------

**默认值：** ``False``

如果为 ``True``，SSH 层会跳过加载用户的已知主机文件。对于一台已知主机修改了它的主机 key 时会有效的避免异常（例如 EC2 等云服务器）。

.. seealso:: :option:`--disable-known-hosts <-D>`, :doc:`ssh`


.. _eagerly-disconnect:

``eagerly_disconnect``
----------------------

**默认值：** ``False``

如果为 ``True``，使 ``fab`` 在各个独立任务执行后关闭连接，而不是在整个运行结束之后。这可以帮助预防堆积大量未使用的网络会话或者由于单个进程打开文件数量限制或者网络硬件限制引起的问题。

.. note::
    当激活时，这个设置会导致断开连接的信息遍及你的输出，而不是在最后。在后续版本中或许会改进这点。
    
.. _effective_roles:

``effective_roles``
-------------------

**默认值：** ``[]``

由 ``fab`` 设置的当前正在执行的命令的角色列表。仅供显示信息。

.. versionadded:: 1.9
.. seealso:: :doc:`execution`

.. _exclude-hosts:

``exclude_hosts``
-----------------

**默认值：** ``[]``

指定一个主机字串列表以被在 ``fab`` 执行中跳过 :ref:`skipped over <exclude-hosts>`。通常通过 :option:`--exclude-hosts/-x <-x>` 设置。

.. versionadded:: 1.1


``fabfile``
-----------

**默认值：** ``fabfile.py``

当加载 fabfiles 文件时， ``fab`` 搜索的文件名。
为了表示一个具体的文件，使用文件的绝对路径。显然不可能在一个 fabfile 中设置这个，但是可以在一个 ``.fabricrc`` 文件或者命令行设置它。

.. seealso:: :option:`--fabfile <-f>`, :doc:`fab`


.. _gateway:

``gateway``
-----------

**默认值：** ``None``

有问题！！！
允许通过指定的主机 SSH 驱动的网关。它的值应该是一个常规 Fabric 主机字串，就像在例子 :ref:`env.host_string <host_string>` 中使用的。当它被设置后，新创建的连接会被设置将他们的 SSH 数据传输通过这个远程的 SSH 到达最终目的地。
Enables SSH-driven gatewaying through the indicated host. The value should be a
normal Fabric host string as used in e.g. :ref:`env.host_string <host_string>`.
When this is set, newly created connections will be set to route their SSH
traffic through the remote SSH daemon to the final destination.

.. versionadded:: 1.5

.. seealso:: :option:`--gateway <-g>`


.. _host_string:

``host_string``
---------------

**默认值：** ``None``

定义了 Fabric 执行 `~fabric.operations.run`，`~fabric.operations.put` 之类命令时使用的用户/主机/端口。它由 ``fab`` 在遍历一个先前的主机列表时设置，也可以在使用 Fabric 时作为一个库手动设置。

.. seealso:: :doc:`execution`


.. _forward-agent:

``forward_agent``
--------------------

**默认值：** ``False``

如果设置为 ``True``，允许映射你的本地 SSH 代理到远端。

.. versionadded:: 1.4

.. seealso:: :option:`--forward-agent <-A>`

.. _host:

``host``
--------

**默认值：** ``None``

由 ``fab`` 设置为 ``env.host_string`` 的主机名部分。仅供显示信息。

.. _hosts:

``hosts``
---------

**默认值：** ``[]``

全局主机列表用于构成每个任务的主机列表。

.. seealso:: :option:`--hosts <-H>`, :doc:`execution`

.. _keepalive:

``keepalive``
-------------

**默认值：** ``0`` (i.e. no keepalive)

一个整数值用于指定一个 SSH 保持活动间隔；基本来说，它映射到 SSH 设置选项 ``ServerAliveInterval``。如果你发现由于网络硬件或者其他原因导致连接超时时它是很有用的。

.. seealso:: :option:`--keepalive`
.. versionadded:: 1.1


.. _key:

``key``
----------------

**默认值：** ``None``

一个字符串，或者类文件对象，包含一个 SSH key；用于认证连接。

.. note::
    The most common method for using SSH keys is to set :ref:`key-filename`.

.. versionadded:: 1.7


.. _key-filename:

``key_filename``
----------------

**默认值：** ``None``
感觉有问题！！
可以是一个字符串或者是一个字符串列表，包含连接时使用的 SSH key 文件的路径。直接传到 SSH 层。可以被设置/追加，通过 :option:`-i`。
May be a string or list of strings, referencing file paths to SSH key files to
try when connecting. Passed through directly to the SSH layer. May be
set/appended to with :option:`-i`.

.. seealso:: `Paramiko's documentation for SSHClient.connect() <http://docs.paramiko.org/en/latest/api/client.html#paramiko.client.SSHClient.connect>`_

.. _env-linewise:

``linewise``
------------

**默认值：** ``False``

通常当运行在并行模式，强制按行缓冲而不是按字/字节缓冲。可以通过 :option:`--linewise` 来激活。这个选项由 :ref:`env.parallel <env-parallel>` 实现 -- 即使 ``linewise`` 为 False，如果 ``parallel`` 为 True，那么 linewise 也会按为 True 时的设置运行。

.. seealso:: :ref:`linewise-output`

.. versionadded:: 1.3


.. _local-user:

``local_user``
--------------

一个只读的值包含本地系统用户名。这个与 :ref:`user` 的初始值相同，但是 :ref:`user` 也可以被命令行参数，Python 代码或者指定的主机字串更改，:ref:`local-user` 总会包含相同的值。

.. _no_agent:

``no_agent``
------------

**默认值：** ``False``


如果设置为 ``True``，就会告诉 SSH 层使用基于秘钥的认证时不要寻找运行中的 SSH 代理。

.. versionadded:: 0.9.1
.. seealso:: :option:`--no_agent <-a>`

.. _no_keys:

``no_keys``
-----------

**默认值：** ``False``

如果设置为 ``True``，就会告诉 SSH 不要从 ``$HOME/.ssh/`` 文件夹中加载任何私钥。（当然通过 ``fab -i`` 明确加载的秘钥文件还是会被使用。)

.. versionadded:: 0.9.1
.. seealso:: :option:`-k`

.. _env-parallel:

``parallel``
------------

**默认值：** ``False``

当设置为 ``True``，强制所有任务并行运行。说明 :ref:`env.linewise
<env-linewise>`。

.. versionadded:: 1.3
.. seealso:: :option:`--parallel <-P>`, :doc:`parallel`

.. _password:

``password``
------------

**默认值：** ``None``

用于 SSH 层连接远程主机时使用的默认密码，**并且/或者**当响应 `~fabric.operations.sudo` 请求时。

.. seealso:: :option:`--initial-password-prompt <-I>`, :ref:`env.passwords <passwords>`, :ref:`password-management`

.. _passwords:

``passwords``
-------------

**默认值：** ``{}``

这个字典主要用于内部使用，并且被被缓存的主机密码键值对自动填充。键是完整的 :ref:`host strings<host-strings>`，值是密码（字符串）。

.. 警告::
    如果你手动修改或者生成这个字典的话，**你必须使用合适的主机字串**包含用户和端口。查看上面关于主机字串 API 的连接以了解详情。

.. seealso:: :ref:`password-management`

.. versionadded:: 1.0


.. _env-path:

``path``
--------

**默认值：** ``''``

用来设置执行 `~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local` 时的 ``$PATH`` shell 环境变量。
推荐使用 `~fabric.context_managers.path` 上下文管理器来管理这个值，而不是直接设置它。

.. versionadded:: 1.0


.. _pool-size:

``pool_size``
-------------

**默认值：** ``0``

用来设置设置并发执行任务时的并发进程数。

.. versionadded:: 1.3
.. seealso:: :option:`--pool-size <-z>`, :doc:`parallel`

.. _prompts:

``prompts``
-----------

**默认值：** ``{}``

``prompts`` 字典允许用户控制交互的提示符。如果一个字典中的键在一个命令的标准输出流中被找到了，Fabric 会自动以字典中对应的值响应。

.. versionadded:: 1.9

.. _port:

``port``
--------

**默认值：** ``None``

当遍历一个主机列表时，通过 ``fab`` 设置的 ``env.host_string`` 的端口部分。也可以用来指定一个具体的默认端口。

.. _real-fabfile:

``real_fabfile``
----------------

**默认值：** ``None``

由 ``fab`` 设置的启动时加载的 fabfile 文件的路径，如果设置了的话。仅供显示信息。

.. seealso:: :doc:`fab`


.. _remote-interrupt:

``remote_interrupt``
--------------------

**默认值：** ``None``

设置 Ctrl-C 触发器是用来中断远程主机还是被本地主机捕获，如下：

* ``None`` （默认值）：只有 `~fabric.operations.open_shell` 会展示远程中断行为，`~fabric.operations.run`/`~fabric.operations.sudo` 会捕捉本地中断。
* ``False``：即使是 `~fabric.operations.open_shell` 也捕捉本地的中断。
* ``True``：所有功能都会发送中断到远端。

.. versionadded:: 1.6


.. _rcfile:

``rcfile``
----------

**默认值：** ``$HOME/.fabricrc``

用于本地 Fabric 设置文件的路径。

.. seealso:: :option:`--config <-c>`, :doc:`fab`

.. _reject-unknown-hosts:

``reject_unknown_hosts``
------------------------

**默认值：** ``False``

如果设置为 ``True``，当连接捕捉用户已知主机列表文件中的主机时，SSH 层会抛出一个异常。

.. seealso:: :option:`--reject-unknown-hosts <-r>`, :doc:`ssh`

.. _system-known-hosts:

``system_known_hosts``
----------------------

**默认值：** ``None``

如果被设置，应该被设置为一个 :file:`known_hosts` 文件。 SSH 层在读取用户已知主机列表文件前会读取这个文件。

.. seealso:: :doc:`ssh`

.. _roledefs:

``roledefs``
------------

**默认值：** ``{}``

定义了主机权限的字典。

.. seealso:: :doc:`execution`

.. _roles:

``roles``
---------

**默认值：** ``[]``

全局权限列表用于构成每个任务主机列表。

.. seealso:: :option:`--roles <-R>`, :doc:`execution`

.. _shell:

``shell``
---------

**默认值：** ``/bin/bash -l -c``
有问题！！！
当执行命令时，值被用来当做 shell 封装器，例如 `~fabric.operations.run`。
Value used as shell wrapper when executing commands with e.g.
`~fabric.operations.run`. Must be able to exist in the form ``<env.shell>
"<command goes here>"`` -- e.g. the default uses Bash's ``-c`` option which
takes a command string as its value.

.. seealso:: :option:`--shell <-s>`,
             :ref:`FAQ on bash as default shell <faq-bash>`, :doc:`execution`

.. _skip-bad-hosts:

``skip_bad_hosts``
------------------

**默认值：** ``False``

如果设置为 ``True``，使得 ``fab`` （或者 `~fabric.tasks.execute`）跳过不能连接的主机。

.. versionadded:: 1.4
.. seealso::
    :option:`--skip-bad-hosts`, :ref:`excluding-hosts`, :doc:`execution`


.. _skip-unknown-tasks:

``skip_unknown_tasks``
----------------------

**默认值：** ``False``

如果设置为 ``True``，使得 ``fab`` （或者 `~fabric.tasks.execute`）跳过未知任务，而不是中止。

.. seealso::
    :option:`--skip-unknown-tasks`


.. _ssh-config-path:

``ssh_config_path``
-------------------

**默认值：** ``$HOME/.ssh/config``

允许指定一个替代的 SSH 配置文件路径。

.. versionadded:: 1.4
.. seealso:: :option:`--ssh-config-path`, :ref:`ssh-config`

``ok_ret_codes``
----------------

**默认值：** ``[0]``

这个列表中的返回码被用来决定对 `~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.sudo` 的调用，怎么才被认为成功执行。

.. versionadded:: 1.6

.. _sudo_prefix:

``sudo_prefix``
---------------

**默认值：** ``"sudo -S -p '%(sudo_prompt)s' " % env``

实际上 ``sudo`` 前缀被加在 `~fabric.operations.sudo` 调用的命令钱。默认远端 ``$PATH`` 中没有 ``sudo`` 的用户和那些想做其他改变的用户（例如当使用不需要密码的 sudo 时移除 ``-p`` 选项）可以使用这个选项。

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prompt
    <sudo_prompt>`

.. _sudo_prompt:

``sudo_prompt``
---------------

**默认值：** ``"sudo password:"``

传递给远端系统上的 ``sudo`` 程序，使得 Fabric 可以正确的认证它的密码提示。

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prefix
    <sudo_prefix>`

.. _sudo_user:

``sudo_user``
-------------

**默认值：** ``None``

被用作一个 `~fabric.operations.sudo` 的 ``user`` 参数的预备值，如果 ``user`` 为 none。在与 `~fabric.context_managers.settings` 组合使用时非常有用。

.. seealso:: `~fabric.operations.sudo`

.. _env-tasks:

``tasks``
---------

**默认值：** ``[]``

由 ``fab`` 设置的完整任务列表用来被当前任务执行。仅供显示信息。

.. seealso:: :doc:`execution`

.. _timeout:

``timeout``
-----------

**默认值：** ``10``

网络连接超时，以秒为单位。

.. versionadded:: 1.4
.. seealso:: :option:`--timeout`, :ref:`connection-attempts`

``use_shell``
-------------

**默认值：** ``True``

没想好怎么翻译。。有问题！！
全局设置
Global setting which acts like the ``shell`` argument to
`~fabric.operations.run`/`~fabric.operations.sudo`: if it is set to ``False``,
operations will not wrap executed commands in ``env.shell``.


.. _use-ssh-config:

``use_ssh_config``
------------------

**默认值：** ``False``

设置为 ``True`` 时，使得 Fabric 加载你本地的 SSH 配置文件。

.. versionadded:: 1.4
.. seealso:: :ref:`ssh-config`


.. _user:

``user``
--------

**默认值：** 用户本地用户名

SSH 层连接到远程主机时使用的用户名。可以被设置为全局的，那么它将被使用除非在主机字串中明确指定。然而，当明确指定时，这个变量会临时的被当前的值覆盖 -- 即它总是会显示当前用来连接的用户。

为了说明这点，来看一个 fabfile 文件::

    from fabric.api import env, run

    env.user = 'implicit_user'
    env.hosts = ['host1', 'explicit_user@host2', 'host3']

    def print_user():
        with hide('running'):
            run('echo "%(user)s"' % env)

然后使用::

    $ fab print_user

    [host1] out: implicit_user
    [explicit_user@host2] out: explicit_user
    [host3] out: implicit_user

    Done.
    Disconnecting from host1... done.
    Disconnecting from host2... done.
    Disconnecting from host3... done.

正如你所看到的，在 ``host2`` 主机执行过程中，``env.user`` 被设置为 ``"explicit_user"``，但是随后被恢复为它先前的值（``"implicit_user"``）。

.. note::
    ``env.user`` 目前有点令人困惑（它被用于配置**和**显示信息），因此期望会修改这点 -- 显示信息的部分很有可能被分离成一个单独的环境变量。

.. seealso:: :doc:`execution`, :option:`--user <-u>`

``version``
-----------

**默认值：** 当前 Fabric 版本字串

多用于显示信息目的。不推荐修改，但应该不会影响到任何部分。

.. seealso:: :option:`--version <-V>`

.. _warn_only:

``warn_only``
-------------

**默认值：** ``False``

当 `~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local` 遇到错误情况时，指定是否警告，而不是中止。

.. seealso:: :option:`--warn-only <-w>`, :doc:`execution`
