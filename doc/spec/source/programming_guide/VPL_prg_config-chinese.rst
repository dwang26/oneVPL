.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

.. _config-interface:

=======================
参数配置
=======================

|vpl_short_name| API 2.10 引入了一个用于配置 |vpl_short_name| 进行编码、解码或视频处理的新接口。应用程序可以
选择性地使用新函数 :cpp:member:`mfxConfigInterface::SetParameter`
来填充用于初始化的数据结构，包括 :cpp:struct:`mfxVideoParam` 和类型为 :cpp:struct:`mfxExtBuffer` 的扩展缓冲区。

:cpp:member:`mfxConfigInterface::SetParameter` 接受 ``char *`` 字符串的键值对作为输入，将这些字符串转换为适当的 C 数据类型，并将
结果写入应用程序提供的初始化结构。这可以为接受字符串形式的用户输入或以 XML、YAML 或 JSON 等格式存储配置信息的应用程序提供更简单、更灵活的初始化方法。

应用程序可以自由地混合使用
:cpp:member:`mfxConfigInterface::SetParameter` 和相同结构的标准 C 样式初始化。此外，使用
:cpp:member:`mfxConfigInterface::SetParameter` 有助于支持将来可能添加到 |vpl_short_name| API 的新参数。当新的扩展缓冲区添加到 |vpl_short_name| API 时，
:cpp:member:`mfxConfigInterface::SetParameter` 将使应用程序能够分配适当大小的缓冲区，并使用所需的值初始化它们，而无需重新编译应用程序。

------------------------------------
设置mfxVideoParam中的参数
------------------------------------

下面的代码片段显示了在结构：cpp:struct:`mfxVideoParam` 中设置参数的示例。

.. literalinclude:: ../snippets/prg_config.cpp
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

------------------------------------------
设置扩展缓冲区中的参数
------------------------------------------

以下代码片段显示了在扩展缓冲区 :cpp:struct:`mfxExtHEVCParam` 中设置参数并将其附加到结构 :cpp:struct:`mfxVideoParam` 的示例。

在设置映射到扩展缓冲区的参数时，该函数首先检查所需的扩展缓冲区是否已附加到提供的 :cpp:struct:`mfxVideoParam`。 如果是，|vpl_short_name| 将更新扩展缓冲区中的相应字段并返回 MFX_ERR_NONE。

如果未附加所需的扩展缓冲区，|vpl_short_name| 将返回 MFX_ERR_MORE_EXTBUFFER。 如果发生这种情况，则应用程序需要分配一个扩展缓冲区，其大小和缓冲区 ID 由从调用 :cpp:member:`mfxConfigInterface::SetParameter` 返回的 ext_buffer 参数指示。然后必须将此扩展缓冲区附加到 :cpp:struct:`mfxVideoParam`，然后应使用相同的参数再次调用 :cpp:member:`mfxConfigInterface::SetParameter`。如果键和值字符串表示新附加的扩展缓冲区中的有效参数，则该函数现在将返回 MFX_ERR_NONE。

.. literalinclude:: ../snippets/prg_config.cpp
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

