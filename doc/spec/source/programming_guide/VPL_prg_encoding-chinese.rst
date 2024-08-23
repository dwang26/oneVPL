.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

===================
解码流程
===================

|vpl_short_name| 中有两种共享内存分配和处理方法：外部和内部。

---------------
外部内存
---------------

以下伪代码显示了使用外部存储器的编码过程（传统模式）:

.. literalinclude:: ../snippets/prg_encoding.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

请注意有关示例的以下要点：

- 应用程序使用 :cpp:func:`MFXVideoENCODE_QueryIOSurf` 函数来获取重新排序输入帧所需的工作帧surface数量。
- 应用程序调用 :cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 函数进行编码操作。输入帧必须位于帧surface池中未锁定的帧surface中。如果编码输出不可用，
该函数将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA` 状态
代码以请求其他输入帧。
- 编码成功后，:cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 函数将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`。此时，
编码的比特流尚不可用，因为:cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 函数是异步的。
应用程序必须使用 :cpp:func:`MFXVideoCORE_SyncOperation` 函数来同步编码操作，然后才能检索编码的比特流。
- 在流的末尾，应用程序需要不断调用:cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 函数，并使用 NULL surface
指针来耗尽 |vpl_short_name| 编码器中缓存的任何剩余比特流，
直到函数返回 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA`。

.. 注意:: 当裁剪窗口小于要编码的帧时，应用程序有责任填充裁剪窗口之外的像素，尤其是在裁剪未与最小编码块大小对齐的情况下（AVC 为 16，HEVC 和 VP9 为 8）。

---------------
内部内存
---------------

下面的伪代码显示了使用内部存储器的编码过程：

.. literalinclude:: ../snippets/prg_encoding.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

与外部存储器（传统模式）相比，此示例有几个关键差异：

- 应用程序无需调用 :cpp:func:`MFXVideoENCODE_QueryIOSurf` 函数来获取工作帧surface的数量，因为分配由 |vpl_short_name| 完成。
- 应用程序调用 :cpp:func:`MFXMemory_GetSurfaceForEncode` 函数来获取用于后续编码操作的空闲surface。
- 在调用 :cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 函数后，应用程序必须调用
:cpp:member:`mfxFrameSurfaceInterface::Release` 函数来减少所获取surface的引用计数器。


.. _config-change:

--------------------
配置改变
--------------------


应用程序通过调用
:cpp:func:`MFXVideoENCODE_Reset` 函数在编码期间更改配置。根据更改前后配置参数的差异，|vpl_short_name| 编码器将继续当前序列或开始新序列。如果编码器开始新序列，它将完全重置内部状态并使用 IDR 帧开始新序列。

应用程序通过在重置期间将 :cpp:struct:`mfxExtEncoderResetOption` 结构附加到
:cpp:struct:`mfxVideoParam` 结构来控制参数更改期间的编码器行为。通过使用此结构，应用程序指示编码器在重置后启动或不启动新序列。在某些情况下，继续当前序列的请求无法满足，编码器将在重置期间失败。为避免这种情况，
应用程序可以在实际重置之前通过调用
:cpp:func:`MFXVideoENCODE_Query` 函数查询重置结果，该函数带有
:cpp:struct:`mfxExtEncoderResetOption`，附加到
:cpp:struct:`mfxVideoParam` 结构。

应用程序使用以下过程更改编码配置：

#. 应用程序通过调用
:cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 函数检索 |vpl_short_name| 编码器中的任何缓存帧，该函数带有 NULL 输入帧指针，直到函数返回
:cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA`。

#. 应用程序使用新配置调用:cpp:func:`MFXVideoENCODE_Reset` 函数：

   - 如果该函数成功设置配置，则应用程序可以继续照常编码。
   - 如果新配置需要新的内存分配，则该函数将返回：cpp:enumerator:`mfxStatus::MFX_ERR_INCOMPATIBLE_VIDEO_PARAM`。应用程序必须关闭 |vpl_short_name| 编码器并使用新配置重新初始化编码过程。

------------------------
外部比特率控制
------------------------


应用程序可以让编码器使用外部比特率控制 (BRC) 而不是原本的比特率控制。要让编码器使用外部 BRC，
应用程序应在编码器初始化期间将 :cpp:struct:`mfxExtCodingOption2` 结构与 ``ExtBRC = MFX_CODINGOPTION_ON`` 和 :cpp:struct:`mfxExtBRC` 回调结构附加到 :cpp:struct:`mfxVideoParam` 结构。**Init**、**Reset** 和 **Close** 回调将在其相应函数内调用：:cpp:func:`MFXVideoENCODE_Init`、
:cpp:func:`MFXVideoENCODE_Reset` 和 :cpp:func:`MFXVideoENCODE_Close`。下图显示了使用外部 BRC 的异步编码流程（使用 ``GetFrameCtrl``和 ``Update``）：

.. figure:: ../images/extbrc_async.png
   :alt: Asynchronous encoding flow with external BRC

   带外部BRC的异步编码流程

.. note:: ``IntAsyncDepth`` 是 |vpl_short_name| 最大内部异步编码
队列大小。它始终小于或等于:cpp:member:`mfxVideoParam::AsyncDepth`.

以下伪代码显示了外部 BRC 的使用：

.. literalinclude:: ../snippets/prg_encoding.c
   :language: c++
   :start-after: /*beg3*/
   :end-before: /*end3*/
   :lineno-start: 1

----
JPEG
----

应用程序可以使用相同的编码程序进行 JPEG/运动 JPEG 编码，如下面的伪代码所示：

.. code-block:: c++

   // encoder initialization
   MFXVideoENCODE_Init (...);
   // single frame/picture encoding
   MFXVideoENCODE_EncodeFrameAsync (...);
   MFXVideoCORE_SyncOperation(...);
   // close down
   MFXVideoENCODE_Close(...);


应用程序可以在编码器初始化期间通过将 :cpp:struct:`mfxExtJPEGQuantTables` 和 :cpp:struct:`mfxExtJPEGHuffmanTables` 缓冲区附加到 :cpp:struct:`mfxVideoParam` 结构来指定 Huffman 和量化表。如果应用程序未定义表，则 |vpl_short_name| 编码器将使用 ITU-T\* 建议 T.81 中推荐的表。如果应用程序未定义量化表，则必须指定 :cpp:member:`mfxInfoMFX::Quality` 参数。在这种情况下，|vpl_short_name| 编码器将根据指定的 :cpp:member:`mfxInfoMFX::Quality` 参数值缩放默认量化表。

应用程序应使用 :cpp:member:`mfxFrameInfo::FourCC` 和
:cpp:member:`mfxFrameInfo::ChromaFormat` 字段正确配置色度采样格式和颜色格式。例如，要对 4:2:2
垂直采样的 YCbCr 图片进行编码，应用程序应将:cpp:member:`mfxFrameInfo::FourCC` 设置为 :cpp:enumerator:`MFX_FOURCC_YUY2`，并将
:cpp:member:`mfxFrameInfo::ChromaFormat` 设置为 :cpp:enumerator:`MFX_CHROMAFORMAT_YUV422V`。要对 4:4:4 采样的 RGB
图片进行编码，应用程序应将 :cpp:member:`mfxFrameInfo::FourCC` 设置为 :cpp:enumerator:`MFX_FOURCC_RGB4`，并将 :cpp:member:`mfxFrameInfo::ChromaFormat`
设置为 :cpp:enumerator:`MFX_CHROMAFORMAT_YUV444`。

|vpl_short_name| 编码器支持不同平台上的不同色度采样和颜色格式集。应用程序必须调用 :cpp:func:`MFXVideoENCODE_Query` 函数来检查给定平台是否支持所需的颜色格式，然后使用 :cpp:member:`mfxFrameInfo::FourCC` 和 :cpp:member:`mfxFrameInfo::ChromaFormat` 的正确值初始化编码器。

应用程序不应定义扫描次数和组件数。这些数字由 |vpl_short_name| 编码器从 :cpp:member:`mfxInfoMFx::Interleaved` 标志和色度类型中得出。如果指定了交错
编码，则编码一个包含所有图像组件的扫描。否则，扫描次数等于组件数。
编码器使用以下组件 ID：“1”表示亮度 (Y)、“2”表示色度Cb (U)，以及“3”表示色度 Cr (V)。

应用程序应分配一个足够大的缓冲区来容纳编码的图片。可以使用以下公式计算粗略的上限
其中 **Width** 和 **Height** 是图片的宽度和高度（以像素为单位），
**BytesPerPx** 是一个像素的字节数：

.. code-block:: c++

   BufferSizeInKB = 4 + (Width * Height * BytesPerPx + 1023) / 1024;

对于单色图片，该等式等于 1；对于 NV12 和 YV12 颜色格式，该等式等于 1.5；对于 YUY2 颜色格式，该等式等于 2；对于 RGB32 颜色格式（未对 alpha 通道进行编码），该等式等于 3。

-------------------------
多视角视频编码
-------------------------

The following pseudo code shows the encoding procedure:
与解码和视频处理初始化过程类似，应用程序将 :cpp:struct:`mfxExtMVCSeqDesc` 结构附加到 :cpp:struct:`mfxVideoParam` 结构以进行编码初始化。
:cpp:struct:`mfxExtMVCSeqDesc` 结构将 |vpl_short_name| MVC 编码器配置为在三种模式下工作：

- **默认依赖模式**：应用程序将 :cpp:member:`mfxExtMVCSeqDesc::NumView` 和所有其他字段指定为零。
|vpl_short_name| 编码器创建一个单一操作点，其中所有视图（视图标识符 0：NumView-1）作为目标视图。第一个视图（视图标识符 0）是基本视图。其他视图依赖于基本视图。

- **显式依赖模式：**应用程序指定
:cpp:member:`mfxExtMVCSeqDesc::NumView` 和视图依赖数组，并将
其他所有字段设置为零。|vpl_short_name| 编码器创建一个单一操作
点，其中所有视图（视图标识符 View[0 : NumView-1].ViewId）作为目标
视图。第一个视图（视图标识符 View[0].ViewId）是基本视图。视图
依赖项定义为 :cpp:struct:`mfxMVCViewDependency` 结构。

- **完整模式：**应用程序完全指定视图及其
依赖项。|vpl_short_name| 编码器生成具有相应
流结构的比特流。

在编码期间，|vpl_short_name| 编码函数 :cpp:func:`MFXVideoENCODE_EncodeFrameAsync` 会累积输入帧，直到
可以对图片进行编码。如果输入有更多数据，则该函数返回 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA`；如果成功积累了足够的数据来编码图片，则返回
:cpp:enumerator:`mfxStatus::MFX_ERR_NONE`。生成的比特流包含完整的图片（多个视图）。应用程序可以更改此行为并指示编码器在单独的比特流缓冲区中输出每个视图。为此，应用程序必须打开 :cpp:member:`mfxExtCodingOption::ViewOutput` 标志。在这种情况下，如果编码器在输出时需要更多比特流缓冲区，则返回
:cpp:enumerator:`mfxStatus::MFX_ERR_MORE_BITSTREAM`；如果图片（多个视图）的处理已完成，则返回 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`。建议
每次 |vpl_short_name| 编码器请求新的比特流缓冲区时，应用程序都提供一个新的输入帧。应用程序必须按照 :cpp:struct:`mfxExtMVCSeqDesc` 结构中描述的顺序提交视图数据进行编码。
只有当特定视图数据所依赖的所有视图都已提交时，才能提交该数据进行编码。

以下伪代码显示了编码过程：

.. literalinclude:: ../snippets/prg_encoding.c
   :language: c++
   :start-after: /*beg4*/
   :end-before: /*end4*/
   :lineno-start: 1
