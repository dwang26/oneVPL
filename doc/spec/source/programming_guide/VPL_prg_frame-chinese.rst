.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

================
帧和场
================

在 |vpl_short_name| 术语中，帧（也称为帧表面）包含逐行帧或互补场对。如果帧是互补场对，则表面缓冲区的奇数行存储顶部场，表面缓冲区的偶数行存储底部场。

.. _frame-surface-manag:

------------------------
帧Surface管理
------------------------

在编码、解码或视频处理过程中，会出现需要保留
输入或输出帧以供将来使用的情况。例如，在解码时，准备好输出的帧必须作为参考帧保留，直到当前序列模式结束。管理此问题的常用方法是内部缓存帧。
此方法需要复制操作，这会大大降低性能。

|vpl_short_name| 有两种方法可以避免复制操作。传统方法使用帧锁定机制，其工作原理如下：

#. 应用程序分配一个足够大的帧表面池，以包含 |vpl_short_name| 函数 I/O 帧表面和内部缓存需求。每个帧表面    维护一个 ``Locked`` 计数器，它是 :cpp:struct:`mfxFrameData` 结构的一部分。``Locked`` 计数器最初设置为零。
#. 应用程序调用 |vpl_short_name|函数使用池中的框架表面，该池的 ``Locked`` 计数器设置为适合所需操作。对于解码或视频处理操作，|vpl_short_name| 使用表面进行写入，``Locked`` 计数器应等于零。如果 |vpl_short_name| 函数需要保留任何框架表面，则 |vpl_short_name| 函数会增加框架表面的 ``Locked`` 计数器。非零 ``Locked`` 计数器表示调用应用程序必须将框架表面视为“正在使用”。当框架表面正在使用时，应用程序可以读取但不能更改、移动、删除或释放框架表面。
#. 在后续 |vpl_short_name| 执行中，如果框架表面不再使用，则 |vpl_short_name| 会减少 ``Locked`` 计数器。当 ``Locked`` 计数器达到零时，应用程序可以自由地对框架表面执行其希望的操作。

一般而言，应用程序不应增加或减少 ``Locked`` 计数器，因为 |vpl_short_name| 管理此字段。如果出于某种原因，应用程序需要修改 ``Locked`` 计数器，则操作必须是原子的，以避免出现竞争条件。

|vpl_short_name| API 版本 2.0 引入了 :cpp:struct:`mfxFrameSurfaceInterface` 结构，该结构为 :cpp:struct:`mfxFrameSurface1` 结构提供了一组回调函数，以便与框架表面配合使用。此接口将 :cpp:struct:`mfxFrameSurface1` 定义为可由 |vpl_short_name| 或应用程序分配的引用计数对象。应用程序必须遵循引用计数对象的一般操作规则。例如，当表面由 |vpl_short_name| 分配时在
:cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 期间或借助
:cpp:func:`MFXMemory_GetSurfaceForVPP` 或 :cpp:func:`MFXMemory_GetSurfaceForVPPOut` 或
:cpp:func:`MFXMemory_GetSurfaceForEncode`，应用程序必须为不再使用的表面调用相应的
:cpp:member:`mfxFrameSurfaceInterface::Release` 函数。

.. attention:: 请注意，“锁定”计数器定义读/写访问策略，而参考计数器负责管理帧的生命周期。

避免复制操作的第二种方法基于
:cpp:struct:`mfxFrameSurfaceInterface`，其工作原理如下：

#. |vpl_short_name| 或应用程序分配框架表面，应用程序存储通过
:cpp:member:`mfxFrameSurfaceInterface::GetRefCounter` 获得的引用计数器值。

#. 应用程序使用框架表面调用 |vpl_short_name| 函数。如果 |vpl_short_name| 需要
保留框架表面，它会通过
:cpp:member:`mfxFrameSurfaceInterface::AddRef` 调用增加引用计数器。
当 |vpl_short_name| 不再使用框架表面时它通过 :cpp:member:`mfxFrameSurfaceInterface::Release` 调用减少引用计数器，并将引用计数器返回到原始值。

#. 应用程序检查框架表面的引用计数器，当它在分配后等于原始值时，它可以重新使用引用计数器进行后续操作。


.. note:: 从 mfxFrameSurface1::mfxStructVersion = {1,1} 开始的 ll :cpp:struct:`mfxFrameSurface1` 结构支持 :cpp:struct:`mfxFrameSurfaceInterface`。
