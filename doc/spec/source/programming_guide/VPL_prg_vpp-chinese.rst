.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

.. _vid_process_procedure:

===========================
视频处理流程
===========================

下面的伪代码显示了视频处理过程：

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

请注意以下有关示例的关键点：

- 应用程序使用 :cpp:func:`MFXVideoVPP_QueryIOSurf` 函数获取输入和输出所需的帧surface数量。应用程序必须分配两个帧surface池：一个用于输入，一个用于输出。
- 视频处理函数 :cpp:func:`MFXVideoVPP_RunFrameVPPAsync` 是异步的。应用程序必须使用 :cpp:func:`MFXVideoCORE_SyncOperation` 函数进行同步，以使输出结果准备就绪。
- 视频处理过程的主体涵盖以下三种情况：

- 如果输入时消耗的帧数等于输出时生成的帧数，则当输出准备就绪时，:term:`VPP` 返回 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`。应用程序必须在同步后处理输出帧，因为 :cpp:func:`MFXVideoVPP_RunFrameVPPAsync` 函数是异步的。应用程序必须在序列末尾提供 NULL 输入以耗尽所有剩余帧。
- 如果输入时消耗的帧数多于输出时生成的帧数，VPP 将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA` 以获取其他输入，直到输出准备就绪。输出准备就绪后，VPP 将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`。应用程序必须在同步后处理输出帧，并在序列末尾提供 NULL 输入以耗尽所有剩余帧。
- 如果输入时消耗的帧数小于输出时生成的帧数，VPP 将返回 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_SURFACE`
（当多个输出已准备就绪时）或 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`
（当一个输出已准备就绪且 VPP 需要新输入时）。在这两种情况下，应用程序都必须在同步后处理输出帧，并在序列末尾提供 NULL 输入以耗尽所有剩余帧。

-------------
配置
-------------

|vpl_short_name| 根据 :cpp:struct:`mfxVideoParam` 结构中指定的输入和输出格式之间的差异配置视频处理管道操作。以下列表显示了几个示例：

- 当输入颜色格式为 :term:`YUY2` 且输出颜色格式为:term:`NV12` 时，|vpl_short_name| 启用从 YUY2 到 NV12 的颜色转换。
- 当输入为隔行扫描且输出为逐行扫描时，|vpl_short_name| 启用去隔行扫描。
- 当输入为单场扫描且输出为隔行扫描或逐行扫描时， |vpl_short_name| 启用场编织，可选择去隔行扫描。
- 当输入为隔行扫描且输出为单场扫描时，|vpl_short_name| 启用场分割。

除了指定输入和输出格式之外，应用程序还可以提供提示以微调视频处理管道操作。应用程序可以使用
:cpp:struct:`mfxExtVPPDoNotUse` 结构禁用管道中的过滤器，使用:cpp:struct:`mfxExtVPPDoUse` 结构启用过滤器，并使用专用配置结构配置过滤器。请参阅：ref:`可配置 VPP 过滤器表 <vpp-filters-table>` 以获取可配置视频处理过滤器、其 ID 和配置结构的完整列表。请参阅：ref:`ExtendedBufferID 枚举器 <extendedbufferid>`
了解更多详细信息。

|vpl_short_name| 确保将输入格式转换为输出格式所需的所有过滤器都包含在管道中。|vpl_short_name|可能会跳过某些可选过滤器，即使应用程序明确请求了这些过滤器，例如由于底层硬件的限制。要通知应用程序有关跳过的可选过滤器，|vpl_short_name| 将返回 :cpp:enumerator:`mfxStatus::MFX_WRN_FILTER_SKIPPED`
警告。应用程序可以通过将 :cpp:struct:`mfxExtVPPDoUse` 结构附加到 :cpp:struct:`mfxVideoParam`
结构并调用 :cpp:func:`MFXVideoVPP_GetVideoParam` 函数来检索活动过滤器列表。应用程序必须为过滤器列表分配足够的内存。

有关可配置过滤器的完整列表，请参阅 :ref:`可配置 VPP 过滤器表 <vpp-filters-table>`。

.. _vpp-filters-table:

.. list-table:: Configurable VPP Filters
   :header-rows: 1
   :widths: 58 42

   * - **Filter ID**
     - **Configuration Structure**
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_DENOISE2`
     - :cpp:struct:`mfxExtVPPDenoise2`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_MCTF`
     - :cpp:struct:`mfxExtVppMctf`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_DETAIL`
     - :cpp:struct:`mfxExtVPPDetail`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_FRAME_RATE_CONVERSION`
     - :cpp:struct:`mfxExtVPPFrameRateConversion`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_IMAGE_STABILIZATION`
     - :cpp:struct:`mfxExtVPPImageStab`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_PROCAMP`
     - :cpp:struct:`mfxExtVPPProcAmp`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_FIELD_PROCESSING`
     - :cpp:struct:`mfxExtVPPFieldProcessing`
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_3DLUT`
     - :cpp:struct:`mfxExtVPP3DLut`

以下示例显示视频处理配置：

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

------------------
感兴趣区域
------------------

在视频处理操作期间，应用程序可以为每一帧指定一个感兴趣的区域，如下图所示：

.. figure:: ../images/vpp_region_of_interest_operation.png
   :alt: VPP region of interest operation

   	VPP 感兴趣区域操作

指定感兴趣的区域可指导调整大小函数实现特殊效果，例如从 16:9 调整为 4:3，同时保持宽高比不变。

在调用 :cpp:func:`MFXVideoVPP_RunFrameVPPAsync` 时，使用 :cpp:struct:`mfxVideoParam` 结构中的 ``CropX``、``CropY``、``CropW`` 和 ``CropH`` 参数为每​​个帧指定感兴趣的区域。注意：对于每帧动态更改，应用程序应在每帧调用 :cpp:func:`MFXVideoVPP_RunFrameVPPAsync` 时设置 ``CropX``、``CropY``、``CropW`` 和 ``CropH`` 参数。

:ref:`VPP 感兴趣区域操作表 <vpp-region-op-table>` 显示了应用于感兴趣区域的 VPP 操作的示例。

.. _vpp-region-op-table:

.. list-table:: VPP Region of Interest Operations
   :header-rows: 1
   :widths: 30 15 20 15 20

   * - | **Operation**
       |
       |
     - | **VPP 输入**
       | *宽 X 高*
       |
     - | **VPP 输入**
       | *CropX, CropY,*
       | *CropW, CropH*
     - | **VPP 输出**
       | *宽 X 高*
       |
     - | **VPP 输出**
       | *CropX, CropY,*
       | *CropW, CropH*
   * - 裁剪
     - 720 x 480
     - 16, 16, 688, 448
     - 720 x 480
     - 16, 16, 688, 448
   * - 调整大小
     - 720 x 480
     - 0, 0, 720, 480
     - 1440 x 960
     - 0, 0, 1440, 960
   * - 水平拉伸
     - 720 x 480
     - 0, 0, 720, 480
     - 640 x 480
     - 0, 0, 640, 480
   * - 16:9 4:3 顶部和底部带有边框
     - 1920 x 1088
     - 0, 0, 1920, 1088
     - 720 x 480
     - 0, 36, 720, 408
   * - 4:3 16:9 左右两侧均有柱状边框
     - 720 x 480
     - 0, 0, 720, 480
     - 1920 x 1088
     - 144, 0, 1632, 1088



---------------------------
多视角视频处理
---------------------------

|vpl_short_name| 视频处理支持处理多个视图。对于视频处理初始化，应用程序需要将 :cpp:struct:`mfxExtMVCSeqDesc` 结构附加到 :cpp:struct:`mfxVideoParam` 结构并调用 :cpp:func:`MFXVideoVPP_Init` 函数。该函数保存视图标识符。在视频处理期间，|vpl_short_name| 分别处理每个视图。|vpl_short_name| 引用 :cpp:struct:`mfxFrameInfo` 结构的 ``FrameID`` 字段，根据其处理管道配置每个视图。如果视频处理源帧不是 |vpl_short_name| MVC 解码器的输出，则应用程序需要在调用 :cpp:func:`MFXVideoVPP_RunFrameVPPAsync` 函数之前填充 ``FrameID`` 字段。如下列伪代码所示：

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg3*/
   :end-before: /*end3*/
   :lineno-start: 1

----------------------
视频处理 3DLUT
----------------------

|vpl_short_name| 视频处理支持具有 Intel HW 特定内存布局的 3DLUT。以下伪代码
显示了如何创建 :cpp:enumerator:`MFX_3DLUT_MEMORY_LAYOUT_INTEL_65LUT` 3DLUT surface。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg4*/
   :end-before: /*end4*/
   :lineno-start: 1
   
以下伪代码显示如何创建系统内存：cpp:struct:`mfx3DLutSystemBuffer` 3DLUT surface。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg5*/
   :end-before: /*end5*/
   :lineno-start: 1

----------------
HDR 色调映射
----------------

|vpl_short_name| 视频处理支持使用英特尔硬件的 HDR 色调映射。以下伪代码
显示了如何执行 HDR 色调映射。

以下伪代码显示了 HDR 到 SDR。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg6*/
   :end-before: /*end6*/
   :lineno-start: 1
   
以下伪代码显示了 SDR 到 HDR。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg7*/
   :end-before: /*end7*/
   :lineno-start: 1
   
以下伪代码显示了 HDR 到 HDR。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg8*/
   :end-before: /*end8*/
   :lineno-start: 1

-----------------------
摄像头源数据加速
-----------------------
|vpl_short_name| 支持使用 Intel 硬件处理相机原始数据格式。以下伪代码
显示了如何执行相机原始数据硬件加速。对于管道处理初始化，
应用程序需要将相机结构附加到 :cpp:struct:`mfxVideoParam` 结构
并调用 :cpp:func:`MFXVideoVPP_Init` 函数。

以下伪代码显示了相机原始数据处理。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg9*/
   :end-before: /*end9*/
   :lineno-start: 1

-------------------------------
任务提交同步
-------------------------------

|vpl_short_name| 可以返回同步对象 - syncpoint 来通知应用程序向 GPU 提交任务。以下示例演示了该方法。

.. literalinclude:: ../snippets/prg_vpp.c
   :language: c++
   :start-after: /*beg10*/
   :end-before: /*end10*/
   :lineno-start: 1
