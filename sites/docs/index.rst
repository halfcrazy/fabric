==================================
欢迎来到 Fabric 文档!
==================================

本网站包含 Fabric 的基本用法与 API 文档。想了解基本信息， 包含它的公开修改日志以及这个项目是如何维护的，
请访问 `项目主页 <http://fabfile.org>`_。


教程
--------

对于新用户或者想总览一下 Fabric 的基本功能的用户而言，请看 :doc:`tutorial`。文档剩下的部分将假设你已经对上面的材料有了基本的了解。

.. toctree::
    :hidden:

    tutorial


.. _usage-docs:

用法文档
-------------------

下面的列表中包含所有 Fabric 目录中 :doc:`tutorial` 的主要章节（non-API） 的详细文档，以及进阶话题。

.. toctree::
    :maxdepth: 2
    :glob:

    usage/*


.. _api_docs:

API 文档
-----------------

Fabric 维护有两个由代码中的 docstrings 自动生成的 API 文档集合 （严格来说非常详细。）

.. _core-api:

核心 API
~~~~~~~~

**核心** API 是指构成 Fabric 基础构建块的函数、类和方法（例如 ~fabric.operations.run 和 ~fabric.operations.sudo）。而其他部分（下文的“扩展 API”和用户的 fabfile）都是在这些核心 API 的基础之上构建的。

.. toctree::
    :maxdepth: 1
    :glob:

    api/core/*

.. _contrib-api:

拓展 API
~~~~~~~~~~~

Fabric 的**拓展**包包含一些非常有用的工具（通常是从用户的 fabfiles 中合并进来的）用于一些例如用户I/O，编辑远程文件等任务。核心 API 倾向于保持小巧、不随意变更，扩展包则会随着更多的用户案例被解决并添加进来，而不断成长进化（同时尽量保持向后兼容）。

.. toctree::
    :maxdepth: 1
    :glob:

    api/contrib/*
