.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

.. _mem-alloc-ext-alloc:

=========================================
内存分配和外部分配器
=========================================

|vpl_short_name| 中有两种内存管理模型：内部和外部。

--------------------------
外部内存分配管理
--------------------------

在外部内存模型中，应用程序必须为输入和输出参数以及缓冲区分配足够的内存，并在 |vpl_short_name| 函数完成其操作时释放内存。在执行期间，|vpl_short_name| 函数使用回调函数通过外部分配器接口 :cpp:struct:`mfxFrameAllocator` 管理视频帧的内存。

如果应用程序需要控制视频帧的分配，则可以通过 :cpp:struct:`mfxFrameAllocator` 接口使用回调函数。如果应用程序未指定分配器，则使用内部分配器。但是，如果应用程序使用视频内存表面进行输入和输出，则必须使用 :cpp:struct:`mfxFrameAllocator` 指定硬件加速设备和外部帧分配器。

外部帧分配器可以分配不同的帧类型：

- 系统内存。
- 视频内存，作为“解码器渲染目标”或“处理器渲染目标”。

有关更多详细信息，请参阅：ref:`使用硬件加速 <hw-acceleration>`。

外部帧分配器仅响应所请求内存类型的帧分配请求，并为所有其他类型返回：cpp:enumerator:`mfxStatus::MFX_ERR_UNSUPPORTED`。分配请求使用标志（内存类型字段的一部分）来指示哪个 |vpl_short_name| 类发起了请求，以便外部帧分配器可以做出相应的响应。

以下示例显示了一个简单的外部帧分配器：

.. literalinclude:: ../snippets/prg_mem.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

对于系统内存，强烈建议将同一帧的所有平面的内存分配为单个缓冲区（使用一次 malloc 调用）。

.. _internal-mem-manage:

--------------------------
内部内存管理
--------------------------

在内部内存管理模型中，|vpl_short_name| 提供了帧分配的接口函数：

- :cpp:func:`MFXMemory_GetSurfaceForVPP`

- :cpp:func:`MFXMemory_GetSurfaceForVPPOut`

- :cpp:func:`MFXMemory_GetSurfaceForEncode`

- :cpp:func:`MFXMemory_GetSurfaceForDecode`

这些函数与 :cpp:struct:`mfxFrameSurfaceInterface` 一起使用，用于表面管理。这些函数返回的表面是一个引用计数对象，应用程序必须在完成表面的所有操作后调用 :cpp:member:`mfxFrameSurfaceInterface::Release`。在此模型中，应用程序无需创建并将外部分配器设置为 |vpl_short_name|。

获取内部分配的表面的另一种方法是使用等于 NULL 的工作表面调用 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync`（请参阅 :ref:`简化解码过程 <simplified-decoding-procedure>`）。在这种情况下，解码器将分配一个新的可引用计数的 :cpp:struct:`mfxFrameSurface1` 并将其返回给用户。与用户的所有假定合同
都类似于**MFXMemory_GetSurfaceForXXX**函数。

------------------------
mfxFrameSurfaceInterface
------------------------

|vpl_short_name| API 版本 2.0 引入了 :cpp:struct:`mfxFrameSurfaceInterface`。此
接口是一组回调函数，用于管理已分配
表面的生命周期、访问像素数据以及获取本机句柄和设备
抽象（如果合适）。建议不要直接访问
:cpp:struct:`mfxFrameSurface1` 结构成员，而是使用
:cpp:struct:`mfxFrameSurfaceInterface`（如果存在）或调用外部
分配器回调函数（如果已设置）。

以下伪代码显示了 :cpp:struct:`mfxFrameSurfaceInterface` 的内存共享用法：

.. literalinclude:: ../snippets/prg_mem.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1
