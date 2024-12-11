.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

.. _decoding-proc:

===================
解码流程
===================

有几种方法可以解码视频帧。第一种方法基于此处介绍的内部分配机制：

.. literalinclude:: ../snippets/prg_decoding.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

请注意以下有关示例的关键点：

- 应用程序调用 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 函数进行解码操作，使用比特流缓冲区（位），帧表面由库内部分配。

.. Note：如上例所示，从 API 版本 2.0 开始，应用程序可以提供 NULL作为导致内部内存分配的工作帧表面。

- 如果解码输出不可用，该函数将返回一个状态代码请求额外的比特流输入，如下所示：

  - :cpp:enumerator: `mfxStatus::MFX_ERR_MORE_DATA`: 该函数需要额外的比特流输入。现有缓冲区包含的比特流数据少于一帧。

- 成功解码后，:cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 函数将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE` 。但是，解码后的帧数据（由 surface_out 指针标识）尚不可用，因为 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 函数是异步的。应用程序必须使用 :cpp:func:`MFXVideoCORE_SyncOperation` 或 :cpp:member:`mfxFrameSurfaceInterface::Synchronize` 来同步解码操作，然后才能检索解码后的帧数据。

- 在比特流结束时，应用程序不断调用 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 函数，并使用 NULL 比特流指针来耗尽解码器中缓存的任何剩余帧，直到函数返回 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA` 。

- 当应用程序完成帧表面的工作时，它必须调用 release 以避免内存泄漏。

下一个示例演示了应用程序如何使用内部预分配的视频表面块：

.. literalinclude:: ../snippets/prg_decoding.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

Here the application should use the :cpp:func:`MFXVideoDECODE_QueryIOSurf`
function to obtain the number of working frame surfaces required to reorder
output frames. It is also required that
:cpp:func:`MFXMemory_GetSurfaceForDecode` call is done after decoder
initialization. In the :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` the |vpl_short_name|
library increments reference counter of incoming surface frame so it is required
that the application releases frame surface after the call.  

Another approach to decode frames is to allocate video frames on-fly with help
of :cpp:func:`MFXMemory_GetSurfaceForDecode` function, feed the library and
release working surface after :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` call.

  .. attention:: Please pay attention on two release calls for surfaces:
                 after :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` to decrease
                 reference counter of working surface returned by 
                 :cpp:func:`MFXMemory_GetSurfaceForDecode`.
                 After :cpp:func:`MFXVideoCORE_SyncOperation` to decrease
                 reference counter of output surface returned by 
                 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync`.

这里，应用程序应使用 :cpp:func:`MFXVideoDECODE_QueryIOSurf`
函数来获取重新排序输出帧所需的工作帧表面数量。还要求在解码器初始化后执行 :cpp:func:`MFXMemory_GetSurfaceForDecode` 调用。在 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 中，|vpl_short_name| 库会增加传入表面帧的引用计数器，因此要求应用程序在调用后释放帧表面。

解码帧的另一种方法是借助 :cpp:func:`MFXMemory_GetSurfaceForDecode` 函数动态分配视频帧，在 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 调用后向库提供数据并释放工作表面。

  .. attention:: 请注意两个表面释放调用：在 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 之后，减少由 :cpp:func:`MFXMemory_GetSurfaceForDecode` 返回的工作表面的引用计数器。 在 :cpp:func:`MFXVideoCORE_SyncOperation` 之后，减少 由 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 返回的输出表面的引用计数器。

.. literalinclude:: ../snippets/prg_decoding.c
   :language: c++
   :start-after: /*beg6*/
   :end-before: /*end6*/
   :lineno-start: 1

以下伪代码显示了根据传统模式使用外部视频帧分配的解码过程：

.. literalinclude:: ../snippets/prg_decoding.c
   :language: c++
   :start-after: /*beg3*/
   :end-before: /*end3*/
   :lineno-start: 1

请注意以下有关示例的关键点：

- 应用程序可以使用 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数从比特流中检索解码初始化参数。如果数据可从其他来源（例如音频/视频分离器）检索，则此步骤是可选的。

- 除了上述状态代码外，:cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 函数还可以返回以下状态代码：

  - :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_SURFACE` ：该函数需要一个以上的帧表面来产生任何输出。

  - :cpp:enumerator:`mfxStatus::MFX_ERR_REALLOC_SURFACE` ：动态分辨率更改情况 - 该函数需要更大的工作帧表面（工作）。

以下伪代码显示了简化的解码过程：

.. literalinclude:: ../snippets/prg_decoding.c
   :language: c++
   :start-after: /*beg4*/
   :end-before: /*end4*/
   :lineno-start: 1

.. _simplified-decoding-procedure:

|vpl_short_name| API 版本 2.0 引入了一种新的解码方法。对于简单的用例，当用户想要解码流并且不想设置其他参数时，已经提出了一种简化的解码器初始化程序。在这种情况下，可以跳过流的头解码和解码器初始化的显式阶段，而是在解码第一帧时隐式执行这些步骤。此更改还需要设置附加字段 :cpp:member:`mfxBitstream::CodecId` 以指示编解码器类型。在此模式下，解码器在内部分配 :cpp:struct:`mfxFrameSurface1` ，因此用户应将输入表面设置为零。

-----------------------
比特率重定位
-----------------------

应用程序可以在解码期间使用以下过程重新定位比特流：

#. 使用 :cpp:func:`MFXVideoDECODE_Reset` 函数重置 |vpl_short_name| 解码器。
#. 可选：如果应用程序维护一个正确解码新位置比特流的序列头，则应用程序可以将序列头插入比特流缓冲区。
#. 将新位置的比特流附加到比特流缓冲区。
#. 恢复解码过程。如果在前面的步骤中未插入序列头，则 |vpl_short_name| 解码器在开始解码之前搜索新的序列头。



-----------------------
破损比特率处理
-----------------------

稳健性和处理损坏的输入流的能力是解码器的重要组成部分。

首先，使用起始代码前缀（ITU-T\* H.264 3.148 和 ITU-T H.265 3.142）来分隔 NAL 单元。然后解析并验证比特流中的所有语法元素。如果任何元素违反规范，则输入比特流被视为无效，解码器将尝试重新同步（查找下一个起始代码）。后续解码器行为取决于哪个语法元素损坏：

* SPS 标头损坏：返回 :cpp:enumerator:`mfxStatus::MFX_ERR_INCOMPATIBLE_VIDEO_PARAM` （仅限 HEVC 解码器，AVC 解码器使用最后一个有效）。

* PPS 标头损坏：重新同步，使用最后一个有效的 PPS 进行解码。

* 切片标头损坏：跳过此切片，重新同步。

* 切片数据损坏：输出表面上设置了损坏标志。

许多流的 IDR 帧带有“frame_num != 0”，而规范规定“如果当前图片是 IDR 图片，frame_num 应等于
0”（ITU-T H.265 7.4.3）。

VUI 也需要经过验证，但错误不会使整个 SPS 无效。解码器要么不使用损坏的 VUI（AVC），要么将不正确的值重置为
默认值（HEVC）。

  .. attention::  有些要求被放宽，因为有许多流违反了严格的标准，但可以解码而不会出错。

参考帧的损坏会扩散到所有使用参考帧进行预测的帧间编码图片中。为了解决这个问题，你必须定期插入 I 帧（帧内编码）或使用帧内刷新技术。帧内刷新技术允许在预定义的时间间隔内从损坏中恢复。帧内刷新的主要目的是将循环帧内编码模式（通常是一行）宏块插入帧间编码图片中，从而相应地限制运动矢量。帧内刷新通常与恢复点 SEI 结合使用，其中“recovery_frame_cnt”来自帧内刷新间隔。恢复点 SEI 消息在 ITU-T H.264 D.2.7 和 ITU-T H.265 D.2.8 中有很好的描述。如果解码从与此 SEI 消息关联的 AU 开始，则解码器可以使用该消息来确定所有后续图片都没有错误的图片。与 IDR 相比，恢复点消息不会将参考图片标记为“未用于参考”。

除了验证语法元素及其约束之外，解码器还使用各种提示来处理损坏的流：


* 如果当前帧没有有效切片，则跳过整个帧。
* 跳过违反切片段头语义 (ITU-T H.265 7.4.7.1) 的切片。目前仅检查 ``slice_temporal_mvp_enabled_flag`` 。
* 由于 LTR（长期参考帧）停留在 DPB，直到被 IDR 或 MMCO 明确清除，因此错误的 LTR 可能会导致长期存在的视觉伪影。AVC 解码器使用以下方法来处理此问题：

  * 如果在错误的 MMCO 命令将参考图片标记为 LT 的情况下发生 DPB 溢出，则操作将回滚。
  * 具有 ``frame_num != 0`` 的 IDR 帧不能是 LTR。


* 如果解码器检测到帧间隙，它会插入“假”帧（标记为不存在），更新参考帧的 FrameNumWrap（ITU-T H.264 8.2.4.1），并应用滑动窗口（ITU-T H.264 8.2.5.3）标记过程。假帧被标记为参考，但由于它们被标记为不存在，因此不会用于帧间预测。


--------------------
VP8的具体细节
--------------------

与其他 |vpl_short_name| 支持的解码器不同，VP8 只能接受完整帧作为输入。应用程序应提供完整帧，并附带 :cpp:enumerator:`MFX_BITSTREAM_COMPLETE_FRAME` 标志。这是唯一具体的区别。

----
JPEG
----

应用程序可以使用相同的解码程序进行 JPEG/运动 JPEG 解码，如下面的伪代码所示：

.. code-block:: c++

   // optional; retrieve initialization parameters
   MFXVideoDECODE_DecodeHeader(...);
   // decoder initialization
   MFXVideoDECODE_Init(...);
   // single frame/picture decoding
   MFXVideoDECODE_DecodeFrameAsync(...);
   MFXVideoCORE_SyncOperation(...);
   // optional; retrieve meta-data
   MFXVideoDECODE_GetUserData(...);
   // close
   MFXVideoDECODE_Close(...);

如果输入比特流包含不支持的功能，:cpp:func:`MFXVideoDECODE_Query` 函数将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_UNSUPPORTED` 。

对于静态图像 JPEG 解码，输入可以是任何符合 ITU-T 建议 T.81 的 JPEG 比特流，并带有 EXIF 或 JFIF 标头。 对于运动 JPEG 解码，输入可以是任何符合 ITU-T 建议 T.81 的 JPEG 比特流。

与其他 |vpl_short_name| 解码器不同，JPEG 解码支持三种不同的输出颜色格式：:term:`NV12`、:term:`YUY2` 和 :term:`RGB32` 。 这种支持有时需要内部颜色转换和更复杂的初始化。输入比特流的颜色格式由
:cpp:member:`mfxInfoMFX::JPEGChromaFormat` 和 :cpp:member:`mfxInfoMFX::JPEGColorFormat` 字段描述。 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数通常会填充它们。如果 JPEG 比特流不包含颜色格式信息，则应用程序 应该提供它。输出颜色格式由通用 |vpl_short_name| 参数描述：:cpp:member:`mfxFrameInfo::FourCC` 和 :cpp:member:`mfxFrameInfo::ChromaFormat` 字段。

Motion JPEG 通过单独压缩每个字段（半高帧）来支持隔行内容。此行为与 |vpl_short_name| 转码管道的其余部分不兼容，其中 |vpl_short_name| 要求场位于同一帧表面的奇数行和偶数行。解码过程修改如下：

- 应用程序使用第一个字段 JPEG 比特流调用 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数来检索初始化参数。
- 应用程序使用以下设置初始化 |vpl_short_name| JPEG 解码器：
- 将 :cpp:struct:`mfxVideoParam` 结构的 ``PicStruct`` 字段设置为 正确的隔行扫描类型，:cpp:enumerator:`MFX_PICSTRUCT_FIELD_TFF` 或 :cpp:enumerator:`MFX_PICSTRUCT_FIELD_BFF` ，来自运动 JPEG 标头。
- 将 :cpp:struct:`mfxVideoParam` 结构中的 ``Height`` 字段翻倍，因为 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数返回的值仅描述第一个字段。实际的帧表面应包含两个字段。
- 在解码过程中，应用程序将两个字段发送到同一个 :cpp:struct:`mfxBitstream` 中进行解码。应用程序还应将 :cpp:member:`mfxBitstream::DataFlag` 设置为 :cpp:enumerator:`MFX_BITSTREAM_COMPLETE_FRAME` 。|vpl_short_name| 解码两个字段，并根据 |vpl_short_name| 约定将它们组合成奇数行和偶数行。

默认情况下，:cpp:func:`MFXVideoDECODE_DecodeHeader` 函数返回 ``Rotation`` 参数，因此旋转后，第一行和第一列的像素位于左上角。应用程序可以在调用 :cpp:func:`MFXVideoDECODE_Init` 之前覆盖默认旋转。

应用程序可以在解码器初始化期间通过将 :cpp:struct:`mfxExtJPEGQuantTables` 和 :cpp:struct:`mfxExtJPEGHuffmanTables` 缓冲区附加到 :cpp:struct:`mfxVideoParam` 结构来指定 Huffman 和量化表。在这种情况下，解码器会忽略来自比特流的表并使用应用程序指定的表。应用程序还可以通过将相同的缓冲区附加到 :cpp:struct:`mfxVideoParam` 并调用 :cpp:func:`MFXVideoDECODE_GetVideoParam` 或 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数来检索这些表。

-------------------------
多视角视频解码
-------------------------

|vpl_short_name| MVC 解码器在包含所有视图和时间配置的完整 MVC 流上运行。应用程序可以配置 |vpl_short_name|
解码器以在解码输出处生成子集。为此，应用程序 必须了解流结构并使用流信息为目标视图配置
解码器。

解码器初始化过程如下：

#. 应用程序调用 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数来
获取流结构信息。这分为两个步骤：

#. 应用程序调用 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数
并将 :cpp:struct:`mfxExtMVCSeqDesc` 结构附加到
:cpp:struct:`mfxVideoParam` 结构。此时，不要为 :cpp:struct:`mfxExtMVCSeqDesc` 结构中的数组分配内存。将 ``View``、``ViewId`` 和 ``OP`` 指针设置为 NULL，并将 ``NumViewAlloc``、``NumViewIdAlloc`` 和 ``NumOPAlloc`` 设置为零。该函数解析比特流并返回 :cpp:enumerator:`mfxStatus::MFX_ERR_NOT_ENOUGH_BUFFER`，其中包含 ``NumView``、``NumViewId`` 和 ``NumOP`` 的正确值。如果应用程序能够从其他来源获取 ``NumView``、``NumViewId`` 和 ``NumOP`` 值，则可以跳过此步骤。
#.应用程序为 ``View``、``ViewId`` 和 ``OP`` 数组分配内存，并再次调用 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数。该函数在分配的数组中返回 MVC 结构信息。

#. 应用程序填充 :cpp:struct:`mfxExtMVCTargetViews` 结构以
根据 :cpp:struct:`mfxExtMVCSeqDesc` 结构中描述的信息选择目标视图。

#. 应用程序使用 :cpp:func:`MFXVideoDECODE_Init` 函数初始化 |vpl_short_name| 解码器。应用程序必须将 :cpp:struct:`mfxExtMVCSeqDesc` 结构和 :cpp:struct:`mfxExtMVCTargetViews` 结构附加到 :cpp:struct:`mfxVideoParam` 结构。

在上述步骤中，请勿在 :cpp:func:`MFXVideoDECODE_DecodeHeader` 函数之后修改 :cpp:struct:`mfxExtMVCSeqDesc` 结构的值，因为 |vpl_short_name| 解码器使用结构中的值进行内部内存分配。一旦应用程序配置了 |vpl_short_name| 解码器，其余解码过程将保持不变。如下面的伪代码所示，应用程序多次调用 :cpp:func:`MFXVideoDECODE_DecodeFrameAsync` 函数以获取当前帧图片的所有目标视图，每次一个目标视图。目标视图由 :cpp:struct:`mfxFrameInfo` 结构的 ``FrameID`` 字段标识。

.. literalinclude:: ../snippets/prg_decoding.c
   :language: c++
   :start-after: /*beg5*/
   :end-before: /*end5*/
   :lineno-start: 1

---------------------------------------------------
结合解码和多通道视频处理
---------------------------------------------------

|vpl_short_name| 公开了用于在一次调用中执行解码和视频处理操作的接口。用户可以指定多个输出处理通道和每个通道的多个视频过滤器。此接口仅支持内部内存分配模型，并通过 :cpp:struct:`mfxSurfaceArray` 引用对象返回已处理帧的数组，如示例所示：

.. literalinclude:: ../snippets/prg_decoding_vpp.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

不同的视频处理通道可能有不同的延迟：

.. literalinclude:: ../snippets/prg_decoding_vpp.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

应用程序可以使用
:cpp:member:`mfxFrameData::TimeStamp`、:cpp:member:mfxFrameData::FrameOrder` 和
:cpp:member:`mfxFrameInfo::ChannelId` 将解码后的帧与特定 VPP 通道进行匹配。

应用程序可以借助 `skip_channels` 和 `num_skip_channels` 参数跳过部分或所有通道（包括解码输出），如下所示：应用程序使用 `ChannelId` 填充 `skip_channels` 数组以禁用对应通道的输出。在这种情况下，:cpp:member:`surf_array_out` 将仅包含剩余通道的表面。如果解码器的通道和/或受影响的 VPP 通道没有当前调用的输出帧（例如，输入比特流不包含完整帧或去隔行/FRC 滤波器有延迟），则此通道的 `skip_channels` 参数将被忽略。

如果应用程序禁用所有通道，则 SDK 将返回 NULL 作为
:cpp:struct:`mfxSurfaceArray` 。

如果应用程序不需要禁用任何通道，则将 `num_skip_channels`
设置为零，当 `num_skip_channels` 为零时，将忽略 `skip_channels` 。

如果应用程序不需要进行缩放或裁剪操作，则必须将 VPP 通道的以下字段
:cpp:member:`mfxFrameInfo::Width`、:cpp:member:`mfxFrameInfo::Height` 、
:cpp:member:`mfxFrameInfo::CropX`、:cpp:member:`mfxFrameInfo::CropY`
:cpp:member:`mfxFrameInfo::CropW`、:cpp:member:`mfxFrameInfo::CropH` 设置为零。
在这种情况下，输出表面具有原始解码分辨率和裁剪。此操作支持
比特流的分辨率变化，无需 :cpp:func:`MFXVideoDECODE_VPP_Reset` 调用。

 .. attention:: 即使使用了多个输入压缩帧，:cpp:func:`MFXVideoDECODE_VPP_DecodeFrameAsync` 也只会生成一个解码帧和来自 VPP 通道的对应帧。

