.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

===========================================
|vpl_short_name| 适用于 |msdk_full_name| 的用户
===========================================

|vpl_short_name| 与 |msdk_full_name| 是源代码兼容的。应用程序可以使用
|msdk_full_name| 来定位较旧的硬件，使用 |vpl_short_name| 来定位其他所有硬件。
|vpl_short_name| 已省略 |msdk_full_name| 的一些过时功能。
在本规范中，术语“旧版”将用于描述 |msdk_full_name| 应用程序调用 |vpl_short_name| 时的行为。

-----------------------------------------
|vpl_short_name| 增强了易用性
-----------------------------------------

|vpl_short_name| 相比 |msdk_full_name| 更易于使用。易用性增强功能包括：

- 具有发现实现功能的智能调度程序。有关更多详细信息，请参阅 :ref:`sdk-session`。
- 简化解码器初始化。有关更多详细信息，请参阅 :ref:`decoding-proc`。
- 新的内存管理和组件（会话）互操作性。有关更多详细信息，请参阅 :ref:`internal-mem-manage` 和 :ref:`decoding-proc`。

.. _new-api:

----------------------------
|vpl_short_name|中新增加的API
----------------------------

|vpl_short_name| 引入了 |msdk_full_name| 中没有的一些新功能。

.. _dispatcher-api:

新的 |vpl_short_name| 调度函数:

- :cpp:func:`MFXLoad`
- :cpp:func:`MFXUnload`
- :cpp:func:`MFXCreateConfig`
- :cpp:func:`MFXSetConfigFilterProperty`
- :cpp:func:`MFXEnumImplementations`
- :cpp:func:`MFXCreateSession`
- :cpp:func:`MFXDispReleaseImplDescription`

新的 |vpl_short_name| 内存管理函数:

- :cpp:func:`MFXMemory_GetSurfaceForVPP`
- :cpp:func:`MFXMemory_GetSurfaceForVPPOut`
- :cpp:func:`MFXMemory_GetSurfaceForEncode`
- :cpp:func:`MFXMemory_GetSurfaceForDecode`

新的 |vpl_short_name| 实现功能的检索函数:

- :cpp:func:`MFXQueryImplsDescription`
- :cpp:func:`MFXReleaseImplDescription`

新的 |vpl_short_name| 会话初始化函数:

- :cpp:func:`MFXInitialize`

---------------------------------
|msdk_full_name| 移除的特性
---------------------------------


以下 |msdk_full_name| 功能已过时，不包含在 |vpl_short_name| 中：

- **音频支持。** |vpl_short_name| 适用于视频处理。与其他音频库（例如 `Sound Open Firmware <https://github.com/thesofproject>`__）功能重复的音频 API 已被删除。

- **ENC 和 PAK 接口。**灵活编码基础架构 (FEI) 的一部分和插件接口，它们为 AVC 和 HEVC 编码器提供对编码过程的额外控制。此功能已被删除，因为它未被客户广泛使用。

- **用户插件架构。** |vpl_short_name| 通过许多不同视频处理框架的 API 实现实现强大的视频加速。对 |msdk_full_name| 用户插件框架的支持已过时。|msdk_full_name|作为插件实现的 RAW 加速（相机 API）
也已过时，|vpl_short_name| 通过 |vpl_short_name| 运行时（如 |vpl_short_name| GPU 运行时）启用 RAW 加速（相机 API）。

- **外部缓冲内存管理。** 一组用于替换内部内存分配的回调函数已过时。

- **视频处理扩展运行时功能。** 视频处理函数MFXVideoVPP_RunFrameVPPAsyncEx 仅用于插件，已过时。

- **外部线程。** 新的线程模型使 MFXDoWork 函数已过时。

- **多帧编码。** 一组外部缓冲区，用于将多个帧组合成一个编码调用。此功能已被删除，因为它是设备特定的，并且不常用。

- **surface类型转码。** 不透明内存支持已被删除，并用内部内存分配概念替换。

- **原始加速。** |msdk_full_name| 作为插件实现的 RAW 加速（相机 API）已过时，由 |vpl_short_name| 和 |vpl_short_name| 运行时实现取代。
|vpl_short_name| 重用了 |msdk_full_name| 的大部分内容相机 API，但 |vpl_short_name| 相机 API 不向后兼容 |msdk_full_name| 相机 API，因为 |vpl_short_name| 中的插件机制已过时，并且 |vpl_short_name| 和 |msdk_full_name| 之间存在一些差异。 |vpl_short_name| 和 |msdk_full_name| 之间的主要差异如下：
1) 在 |vpl_short_name| 中删除了 mfxCamGammaParam 和 mfxExtCamGammaCorrection；2) 在 mfxExtCamHotPixelRemoval、mfxCamVignetteCorrectionParam 和 mfxCamVignetteCorrectionElement 中添加了保留字节，以供将来扩展；3) 在 mfxExtCamColorCorrection3x3 中将 CCM 从 mfxF64 更改为 mfxF32，并添加了更多保留字节。

.. _deprecated-api:

-----------------------------
|msdk_full_name| 移除的API
-----------------------------

下面 |msdk_full_name| 函数没有包含在 |vpl_short_name|中:

- **音频相关的函数**

  - MFXAudioCORE_SyncOperation()
  - MFXAudioDECODE_Close()
  - MFXAudioDECODE_DecodeFrameAsync()
  - MFXAudioDECODE_DecodeHeader()
  - MFXAudioDECODE_GetAudioParam()
  - MFXAudioDECODE_Init()
  - MFXAudioDECODE_Query()
  - MFXAudioDECODE_QueryIOSize()
  - MFXAudioDECODE_Reset()
  - MFXAudioENCODE_Close()
  - MFXAudioENCODE_EncodeFrameAsync()
  - MFXAudioENCODE_GetAudioParam()
  - MFXAudioENCODE_Init()
  - MFXAudioENCODE_Query()
  - MFXAudioENCODE_QueryIOSize()
  - MFXAudioENCODE_Reset()

- **灵活编码基础架构**

  - MFXVideoENC_Close()
  - MFXVideoENC_GetVideoParam()
  - MFXVideoENC_Init()
  - MFXVideoENC_ProcessFrameAsync()
  - MFXVideoENC_Query()
  - MFXVideoENC_QueryIOSurf()
  - MFXVideoENC_Reset()
  - MFXVideoPAK_Close()
  - MFXVideoPAK_GetVideoParam()
  - MFXVideoPAK_Init()
  - MFXVideoPAK_ProcessFrameAsync()
  - MFXVideoPAK_Query()
  - MFXVideoPAK_QueryIOSurf()
  - MFXVideoPAK_Reset()

- **用户插件函数**

  - MFXAudioUSER_ProcessFrameAsync()
  - MFXAudioUSER_Register()
  - MFXAudioUSER_Unregister()
  - MFXVideoUSER_GetPlugin()
  - MFXVideoUSER_ProcessFrameAsync()
  - MFXVideoUSER_Register()
  - MFXVideoUSER_Unregister()
  - MFXVideoUSER_Load()
  - MFXVideoUSER_LoadByPath()
  - MFXVideoUSER_UnLoad()
  - MFXDoWork()

- **内存函数**

  - MFXVideoCORE_SetBufferAllocator()

- **视频处理函数**

  - MFXVideoVPP_RunFrameVPPAsyncEx()

- **内存类型和IOPattern 枚举**
  
  - MFX_IOPATTERN_IN_OPAQUE_MEMORY
  - MFX_IOPATTERN_OUT_OPAQUE_MEMORY
  - MFX_MEMTYPE_OPAQUE_FRAME

.. 重要提示:: 相应的扩展缓冲区也将被删除。

尝试使用 |vpl_short_name| 不支持的 |msdk_full_name| API 时，会发生以下行为：

- 使用 |vpl_short_name| API 头文件编译的代码在尝试使用已删除的 API 时将生成编译和/或
链接错误。

- 之前使用 |msdk_full_name| 编译并使用 |vpl_short_name|
运行时执行的代码在调用已删除的函数时将生成 :cpp:enumerator:`MFX_ERR_NOT_IMPLEMENTED` 错误。

---------------------------
|msdk_full_name| 传统API
---------------------------

|vpl_short_name| 包含来自 |msdk_full_name| 的以下头文件，用于简化现有应用程序向 |vpl_short_name| 的迁移：

- mfxvideo++.h

.. 重要提示：|msdk_full_name| 已从这些头文件中删除过时的 API。
使用 |vpl_short_name| API 头文件编译的代码在尝试使用已删除的 API 时将生成编译和/或
链接错误。
