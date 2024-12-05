.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

======================
转码流程
======================

应用程序可以结合使用 |vpl_short_name| 编码、解码和视频处理
功能进行转码操作。本节介绍将两个或多个 |vpl_short_name| 功能连接在一起的关键
方面。

---------------------
异步管道
---------------------

应用程序将上游 |vpl_short_name| 函数的输出传递给下游 |vpl_short_name| 函数的输入，以构建异步管道。管道构建在运行时完成，并且可以动态更改，如以下示例所示：

.. literalinclude:: ../snippets/prg_transcoding.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

|vpl_short_name| 简化了异步管道同步的要求。

应用程序只需在最后一个 |vpl_short_name| 函数之后进行同步。不需要显式同步中间结果，这可能会降低性能。

|vpl_short_name| 跟踪动态管道构造并验证对输入和输出参数的依赖性，以确保管道函数的执行顺序。

在上例中，|vpl_short_name| 将确保 :cpp:func:`MFXVideoENCODE_EncodeFrameAsync`
在 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 或 :cpp:func:`MFXVideoVPP_RunFrameVPPAsync` 完成之前不会开始其操作。

在异步管道执行期间，应用程序必须将输入数据视为“正在使用”，并且在执行完成之前不得更改它。

应用程序还必须将输出数据视为不可用，直到执行完成。此外，对于编码器，应用程序必须在输入surface锁定时将扩展
和有效载荷缓冲区视为“正在使用”。

|vpl_short_name| 通过比较管道中每个
|vpl_short_name| 函数的输入和输出参数来检查依赖关系。在上一个异步操作完成之前，请勿修改输入和输出
参数的内容。这样做会破坏依赖关系检查，并可能导致未定义的行为。当输入和输出参数是结构时会发生异常，在这种情况下，允许覆盖结构中的字段。

.. 注意：依赖关系检查仅适用于指向结构的指针。

中间同步有两个例外：

- 如果输入来自任何异步操作，则应用程序必须在调用 |vpl_short_name| :cpp:func:`MFXVideoDECODE_DecodeFrameAsync`
函数之前同步任何输入。
- 当应用程序调用异步函数在视频内存中生成输出
surface并将该surface传递给非|vpl_short_name|组件时，它必须
在将surface传递给非|vpl_short_name|
组件之前明确同步该操作。

.. _surface_pool_alloc:

-----------------------
Surface池分配
-----------------------

当将 API 函数 **A** 连接到 API 函数 **B** 时，应用程序必须

考虑这两个函数的要求，以计算surface池中的

帧surface数量。通常，应用程序可以使用公式 **Na+Nb**，其中 **Na** 是 |vpl_short_name| 函数 **A** 输出的帧surface要求，而 **Nb** 是 |vpl_short_name| 函数 **B** 输入的帧surface要求。

出于性能考虑，应用程序必须提交多个操作

并尽可能延迟同步，这为 |vpl_short_name| 提供了组织内部流水线的灵活性。例如，比较以下两个操作

序列，其中第一个序列是推荐顺序：

.. graphviz::
   :caption: 推荐的操作推荐顺序

   digraph {
      rankdir=LR;
      labelloc="t";
      label="Operation sequence 1";
      f1 [shape=record label="ENCODE(F1)" ];
      f2 [shape=record label="ENCODE(F2)" ];
      f3 [shape=record label="SYNC(F1)" ];
      f4 [shape=record label="SYNC(F2)" ];
      f1->f2->f3->f4;
   }


.. graphviz::
   :caption: 不推荐的操作推荐顺序

   digraph {
      rankdir=LR;
      labelloc="t";
      label="Operation sequence 2";
      f1 [shape=record label="ENCODE(F1)" ];
      f2 [shape=record label="ENCODE(F2)" ];
      f3 [shape=record label="SYNC(F1)" ];
      f4 [shape=record label="SYNC(F2)" ];
      f1->f3->f2->f4;
   }


在此示例中，surface池需要额外的surface来在同步之前考虑多个异步操作。应用程序可以使用 :cpp:member:`mfxVideoParam::AsyncDepth` 字段来通知 |vpl_short_name| 函数应用程序计划在同步之前执行的异步操作的数量。相应的 |vpl_short_name| **QueryIOSurf** 函数将在 :cpp:member:`mfxFrameAllocRequest::NumFrameSuggested` 值中反映此数字。以下示例展示了一种基于 :cpp:member:`mfxFrameAllocRequest::NumFrameSuggested` 值计算surface需求的方法：

.. literalinclude:: ../snippets/prg_transcoding.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

------------------------
管道错误报告
------------------------

在异步管道构建过程中，每个管道阶段函数都会
返回一个同步点（sync point）。这些同步点
有助于跟踪异步管道操作过程中的错误。

例如，假设以下管道：

.. graphviz::

   digraph {
      rankdir=LR;
      A->B->C;
   }

应用程序在同步点 **C** 上同步。如果错误发生在函数 **C** 中，则同步将返回准确的错误代码。如果错误发生在函数 **C** 之前，则同步将返回
:cpp:enumerator:`mfxStatus::MFX_ERR_ABORTED`。然后，应用程序可以尝试在同步点 **B** 上同步。同样，如果错误发生在函数 **B** 中，则同步将返回准确的错误代码，否则
:cpp:enumerator:`mfxStatus:: MFX_ERR_ABORTED`。如果错误发生在函数 **A** 中，则适用相同的逻辑。
