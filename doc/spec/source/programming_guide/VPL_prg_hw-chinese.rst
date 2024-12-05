.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0
..
  Intel® Video Processing Library (Intel® VPL)
.. _hw-acceleration:

=====================
硬件加速
=====================

|vpl_short_name| 提供了一种使用硬件加速的新模型，同时
继续支持传统模式下的硬件加速。

--------------------------------------------
与硬件加速配合使用的新模型
--------------------------------------------

|vpl_short_name| API 版本 2.0 引入了一种新的内存模型：内部分配，其中
|vpl_short_name| 负责视频内存分配。在此模式下，
应用程序不依赖于低级视频框架 API（例如
DirectX\* 或 VA-API），也不需要创建和设置相应的
低级 |vpl_short_name| 原语（例如 `ID3D11Device` 或 `VADisplay`）。相反，
|vpl_short_name| 创建所有必需的对象以在内部使用硬件加速和视频
surface。应用程序可以使用
:cpp:func:`MFXVideoCORE_GetHandle` 或借助
:cpp:struct:`mfxFrameSurfaceInterface` 接口访问这些对象。

这种方法简化了 |vpl_short_name|初始化，可选调用
:cpp:func:`MFXVideoENCODE_QueryIOSurf`、:cpp:func:`MFXVideoDECODE_QueryIOSurf` 或 :cpp:func:`MFXVideoVPP_QueryIOSurf` 函数。请参阅
:ref:`内部内存管理 <internal-mem-manage>`。

.. note：应用程序可以在会话创建之前通过 :cpp:func:`MFXSetConfigFilterProperty` 设置设备句柄，如以下代码所示：

.. literalinclude:: ../snippets/prg_hw.cpp
   :language: c++
   :start-after: /*beg3*/
   :end-before: /*end3*/
   :lineno-start: 1


----------------------------------------------
在传统模式下使用硬件加速
----------------------------------------------

兼容多种视频设备
--------------------------------

如果您的系统有多个图形适配器，您可能需要提示哪个
适配器更适合处理特定工作负载。
|vpl_short_name| 的旧模式提供了一个辅助 API，可根据提供的工作负载描述为您的
工作负载选择最合适的适配器。

.. important:: 从 API 2.9 开始，:cpp:func:`MFXQueryAdapters`、:cpp:func:`MFXQueryAdaptersDecode` 和 :cpp:func:`MFXQueryAdaptersNumber` 已弃用。应用程序应使用 :cpp:func:`MFXEnumImplementations` 和 :cpp:func:`MFXSetConfigFilterProperty` 来查询适配器功能，并为输入工作负载选择合适的适配器。

以下示例显示了传统模式下离散适配器上的工作负载初始化：


.. literalinclude:: ../snippets/prg_hw.cpp
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

该示例显示，在使用
:cpp:func:`MFXQueryAdapters` 获取适配器列表后，将以常规方式进一步初始化 :cpp:type:`mfxSession`。使用 :cpp:enumerator:`MFX_IMPL_HARDWARE`、:cpp:enumerator:`MFX_IMPL_HARDWARE2`、
:cpp:enumerator:`MFX_IMPL_HARDWARE3` 或 :cpp:enumerator:`MFX_IMPL_HARDWARE4`
的 :cpp:type:`mfxIMPL` 值来选择特定适配器。

以下示例显示了如何使用 :cpp:func:`MFXQueryAdapters` 查询
最适合特定编码工作负载的适配器：

.. literalinclude:: ../snippets/prg_hw.cpp
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

请参阅 :cpp:func:`MFXQueryAdapters` 描述以了解适配器优先级规则。

使用视频内存
----------------------

要充分利用 |vpl_short_name| 加速功能，应用程序应支持特定于操作系统的基础架构。如果使用 Microsoft\* Windows\*，则应用程序应支持 Microsoft DirectX\*。如果使用 Linux\*，则应用程序应支持 Linux 的 VA-API。

应用程序中的硬件加速支持包括视频内存支持和加速设备支持。

根据使用模型，应用程序可以在管道的不同阶段使用视频内存。下图显示了三种主要场景：

.. graphviz::

  digraph {
    rankdir=LR;
    labelloc="t";
    label="Intel® VPL functions interconnection";
    F1 [shape=octagon label="Intel® VPL Function"];
    F2 [shape=octagon label="Intel® VPL Function"];
    F1->F2 [ label="Video Memory" ];
  }

|

.. graphviz::

  digraph {
    rankdir=LR;
    labelloc="t";
    label="Video memory as output";
    F3 [shape=octagon label="Intel® VPL Function"];
    F4 [shape=octagon label="Application" fillcolor=lightgrey];
    F3->F4 [ label="Video Memory" ];
  }

|

.. graphviz::

  digraph {
    rankdir=LR;
    labelloc="t";
    label="Video memory as input";
    F5 [shape=octagon label="Application"];
    F6 [shape=octagon label="Intel® VPL Function"];
    F5->F6 [ label="Video Memory" ];
  }

|

应用程序必须使用 :cpp:member:`mfxVideoParam::IOPattern` 字段来指示初始化期间的 I/O 访问模式。后续函数调用必须遵循此访问模式。例如，如果函数在输入和输出处都对视频内存surface进行操作，则应用程序必须在初始化时在 :cpp:enumerator:`MFX_IOPATTERN_IN_VIDEO_MEMORY` 中为输入指定访问模式 `IOPattern`，在 :cpp:enumerator:`MFX_IOPATTERN_OUT_VIDEO_MEMORY` 中为输出指定访问模式。此特定 I/O 访问模式不得在 **Init** **-** **Close** 序列内更改。

任何硬件加速 |vpl_short_name| 组件的初始化都需要加速设备句柄。|vpl_short_name| 组件也使用此句柄来查询硬件功能。应用程序可以通过 :cpp:func:`MFXVideoCORE_SetHandle`
函数传递设备句柄，与 |vpl_short_name|
共享其设备。建议在实际使用 |vpl_short_name| 之前共享句柄。

.. _work_ms_directx_app:

使用 Microsoft DirectX\* 应用​​程序
------------------------------------------

|vpl_short_name| 支持 Microsoft Windows 操作系统上的两种不同的硬件加速基础架构：Direct3D\* 9 DXVA2 和 Direct3D 11 Video API。如果 Direct3D 9
DXVA2 用于硬件加速，则应用程序应使用
`IDirect3DDeviceManager9` 接口作为加速设备句柄。如果
Direct3D 11 Video API 用于硬件加速，则应用程序应使用
`ID3D11Device` 接口作为加速设备句柄。

应用程序应通过
:cpp:func:`MFXVideoCORE_SetHandle` 函数与 |vpl_short_name| 共享其中一个接口。如果应用程序不提供
该接口，则 |vpl_short_name| 会创建自己的内部加速设备。因此，
|vpl_short_name| 输入和输出将仅限于外部分配​​模式的系统内存，这将降低 |vpl_short_name| 性能。如果 |vpl_short_name| 无法创建有效的加速设备，则 |vpl_short_name| 无法继续进行硬件加速，并向应用程序返回错误状态。

.. note：如果应用程序不提供 `IDirect3DDeviceManager9` 或 `ID3D11Device` 接口，建议在内部分配模式下工作。

应用程序必须使用标志 ``D3DCREATE_MULTITHREADED`` 创建 Direct3D 9 设备。还建议使用标志 ``D3DCREATE_FPU_PRESERVE``。这会影响浮点计算，包括 PTS 值。

应用程序还必须为 Direct3D 11 设备设置多线程模式。
以下示例显示如何为 Direct3D 11 设备设置多线程模式：：

.. code-block:: c++
   :lineno-start: 1

   ID3D11Device            *pD11Device;
   ID3D11DeviceContext     *pD11Context;
   ID3D10Multithread       *pD10Multithread;

   pD11Device->GetImmediateContext(&pD11Context);
   pD11Context->QueryInterface(IID_ID3D10Multithread, &pD10Multithread);
   pD10Multithread->SetMultithreadProtected(true);

在硬件加速期间，如果发生 Direct3D“设备丢失”事件，则 |vpl_short_name|
操作将以 :cpp:enumerator:`mfxStatus::MFX_ERR_DEVICE_LOST`
返回状态终止。如果应用程序提供了 Direct3D 设备句柄，则应用程序必须重置 Direct3D 设备。

当 |vpl_short_name| 解码器为硬件加速创建辅助设备时，它必须分配用于 I/O 访问的 Direct3D surface列表（也称为
surface链），并将surface链作为设备创建命令的一部分传递。
在大多数情况下，surface链是
:ref:`Frame Surface Locking <frame-surface-manag>` 部分中提到的框架surface池。

应用程序通过 |vpl_short_name| 外部分配器回调将surface链传递给 |vpl_short_name| 组件 **Init** 函数。有关详细信息，请参阅
:ref:`内存分配和外部分配器 <mem-alloc-ext-alloc>` 部分。

只有解码器 **Init** 函数从应用程序请求外部surface链并将其用于辅助设备创建。编码器和 VPP **Init** 函数只能请求内部surface。有关不同内存类型的更多详细信息，请参阅
:ref:`ExtMemFrameType 枚举器 <extmemframetype>`。

根据配置参数，|vpl_short_name| 需要不同的surface类型。
强烈建议调用 :cpp:func:`MFXVideoENCODE_QueryIOSurf` 函数、:cpp:func:`MFXVideoDECODE_QueryIOSurf` 函数或
:cpp:func:`MFXVideoVPP_QueryIOSurf` 函数来确定外部分配模式中的适当类型。

使用 VA-API 应用程序
-----------------------------

|vpl_short_name| 支持 Linux 上硬件加速的 VA-API 基础架构。

应用程序应使用 `VADisplay` 接口作为此基础架构的加速设备

句柄，并通过
:cpp:func:`MFXVideoCORE_SetHandle` 函数与 |vpl_short_name| 共享。

以下示例显示如何从 X Window 系统获取 VA 显示器：

.. code-block::
   :lineno-start: 1

   Display   *x11_display;
   VADisplay va_display;

   x11_display = XOpenDisplay(current_display);
   va_display  = vaGetDisplay(x11_display);

   MFXVideoCORE_SetHandle(session, MFX_HANDLE_VA_DISPLAY, (mfxHDL) va_display);

以下示例显示如何从直接渲染管理器获取 VA 显示：

.. code-block::
   :lineno-start: 1

   int card;
   VADisplay va_display;

   card = open("/dev/dri/card0", O_RDWR); /* primary card */
   va_display = vaGetDisplayDRM(card);
   vaInitialize(va_display, &major_version, &minor_version);

   MFXVideoCORE_SetHandle(session, MFX_HANDLE_VA_DISPLAY, (mfxHDL) va_display);

当 |vpl_short_name| 解码器创建硬件加速设备时，它必须分配
用于 I/O 访问的视频内存surface列表（也称为surface链），
并将surface链作为设备创建命令的一部分传递。
应用程序通过 |vpl_short_name| 外部分配器回调将surface链传递给 |vpl_short_name| 组件 **Init** 函数。有关详细信息，请参阅
:ref:`内存分配和外部分配器 <mem-alloc-ext-alloc>` 部分。
从 |vpl_short_name| API 版本 2.0 开始，如果未设置外部分配器，|vpl_short_name| 将创建自己的surface链。有关详细信息，请参阅:ref`与硬件加速配合使用的新模型 <hw-acceleration>` 部分。


.. note:: VA-API 未定义任何surface类型，应用程序可以使用 :cpp:enumerator:`MFX_MEMTYPE_VIDEO_MEMORY_DECODER_TARGET` 或 :cpp:enumerator:`MFX_MEMTYPE_VIDEO_MEMORY_PROCESSOR_TARGET` 来指示视频内存中的数据。

