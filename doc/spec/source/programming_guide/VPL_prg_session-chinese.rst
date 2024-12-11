.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0
..
  Intel® Video Processing Library (Intel® VPL)
.. _sdk-session:

==========================
|vpl_short_name| 会话
==========================

在调用任何 |vpl_short_name| 函数之前，应用程序必须初始化库并创建 |vpl_short_name| 会话。|vpl_short_name| 会话维护使用 :term:`DECODE`、:term:`ENCODE`、:term:`VPP`、:term:`DECODE_VPP` 函数的上下文。

--------------------------------------
|msdk_full_name| 调度器 (遗留问题)
--------------------------------------

:cpp:func:`MFXInit` 或 :cpp:func:`MFXInitEx` 函数启动（初始化）一个会话。:cpp:func:`MFXClose` 函数关闭（取消初始化）
会话。为避免内存泄漏，请始终在 :cpp:func:`MFXInit` 之后调用 :cpp:func:`MFXClose`。

.. 重要提示:: 从 API 2.0 开始，:cpp:func:`MFXInit` 和 :cpp:func:`MFXInitEx` 已弃用。应用程序必须使用 :cpp:func:`MFXLoad`
和 :cpp:func:`MFXCreateSession` 来初始化实现。

.. 重要提示:: 为了向后兼容现有的 |msdk_full_name|
应用程序 |vpl_short_name|会话可以由旧调度程序通过 :cpp:func:`MFXInit` 或 :cpp:func:`MFXInitEx` 调用创建和初始化。在这种情况下，在具有 X\ :sup:`e` 架构的 |intel_r| 平台上，报告的 API 版本将为 1.255。

应用程序可以将会话初始化为基于软件的会话(:cpp:enumerator:`MFX_IMPL_SOFTWARE`) 或基于硬件的会话
(:cpp:enumerator:`MFX_IMPL_HARDWARE`)。在基于软件的会话中，SDK 函数在 CPU 上执行。在基于硬件的会话中，SDK 函数使用平台硬件加速功能。对于有多个
图形设备（GPU）的平台，应用程序可以使用 :cpp:enumerator:`MFX_IMPL_HARDWARE`、
:cpp:enumerator:`MFX_IMPL_HARDWARE2`、
:cpp:enumerator:`MFX_IMPL_HARDWARE3` 或 :cpp:enumerator:`MFX_IMPL_HARDWARE4` 的 :cpp:type:`mfxIMPL` 值在任何备用图形设备上初始化会话。


应用程序还可以将会话初始化为自动（:cpp:enumerator:`MFX_IMPL_AUTO` 或 :cpp:enumerator:`MFX_IMPL_AUTO_ANY`），指示调度程序库检测平台功能并选择最佳的 SDK 库。初始化后，SDK 通过 :cpp:func:`MFXQueryIMPL` 函数返回实际实现。调度程序在内部的工作方式如下：

#. 调度程序搜索具有特定名称的共享库：

   ========= =============== ====================================
   **OS**    **名字**        **描述**
   ========= =============== ====================================
   Linux\*   libmfxsw64.so.1 64-bit 基于软件的实现
   Linux     libmfxsw32.so.1 32-bit 基于软件的实现
   Linux     libmfxhw64.so.1 64-bit 基于硬件的实现
   Linux     libmfxhw64.so.1 32-bit 基于硬件的实现
   Windows\* libmfxsw32.dll  64-bit 基于软件的实现
   Windows   libmfxsw32.dll  32-bit 基于软件的实现
   Windows   libmfxhw64.dll  64-bit 基于硬件的实现
   Windows   libmfxhw64.dll  32-bit 基于硬件的实现
   ========= =============== ====================================

#. 加载库后，调度程序将获取每个 SDK 函数的地址。请参阅
:ref:`导出函数/API 版本表 <export-func-version-table-2x>` 以获取要公开的函数列表。

.. _legacy_search_order:

使用实现搜索策略识别共享库的方式将因操作系统而异。

* 在 Windows 上，调度程序按指定顺序搜索以下位置以查找正确的实现库：

  #. 当前适配器的 :file:`Driver Store` 目录。
     所有类型的图形驱动程序都可以在此目录中安装库。`了解有关驱动程序存储的更多信息 <https://docs.microsoft.com/en-us/windows-hardware/drivers/install/driver-store>`__。
  #. 注册表项 ``HKEY_CURRENT_USER\Software\Intel\MediaSDK\Dispatch`` 下为当前硬件指定的目录。
  #. 注册表项 ``HKEY_LOCAL_MACHINE\Software\Intel\MediaSDK\Dispatch`` 下为当前硬件指定的目录。
  #. 存储在这些注册表项中的目录：:file:`C:\Program Files\Intel\Media SDK`。
        此目录是旧版图形驱动程序安装库的位置。
  #. 当前模块（链接调度程序的模块）所在的目录（仅当当前模块是 dll 时）。

   调度程序完成主要搜索后，还会检查：
  #.当前进程的 exe 文件的目录，它只在此查找软件实现，而不管应用程序请求了哪个实现。
  #. 默认 dll 搜索。这提供从应用程序的 exe 文件的目录以及 :file:`System32` 和 :file:`SysWOW64`
     目录加载。`了解有关默认 dll 搜索顺序的更多信息 <https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order?redirectedfrom=MSDN#search-order-for-desktop-applications>`__。
  #. :file:`System32` 和 :file:`SysWOW64` 目录，这是 DCH图形驱动程序安装库的位置。


* 在 Linux 上，调度程序会按指定的顺序搜索以下位置以查找正确的实现库：

  #. 环境变量“LD_LIBRARY_PATH”提供的目录。
  #. :file:`/etc/ld.so.cache` 缓存文件的内容。
  #. 默认路径为 :file:`/lib`，然后是 :file:`/usr/lib` 或 :file:`/lib64`，然后是在某些 64 位操作系统上为 :file:`/usr/lib64`。在 Debian 上：:file:`/usr/lib/x86_64-linux-gnu`。
  #. SDK 安装文件夹。


.. _vpl-dispatcher:

-----------------------------
|vpl_short_name|调度程序
-----------------------------

|vpl_short_name| 调度程序通过提供额外的功能来扩展旧调度程序，以便根据实现功能选择适当的实现。实现功能包括有关支持的解码器、编码器和 VPP 过滤器的信息。对于每个支持的编码器、解码器和过滤器，功能包括有关支持的内存类型、颜色格式和图像（帧）大小（以像素为单位）的信息。

配置调度程序的功能搜索过滤器，并基于合适的实现创建会话的推荐方法如下：

#. 使用 :cpp:func:`MFXLoad` 创建加载器。
#. 使用 :cpp:func:`MFXCreateConfig` 创建加载器的配置。
#. 使用 :cpp:func:`MFXSetConfigFilterProperty` 添加配置属性。
#. 使用 :cpp:func:`MFXEnumImplementations` 探索可用的实现。
#. 使用 :cpp:func:`MFXCreateSession` 创建合适的会话。

终止应用程序的过程如下：

#. 使用 :cpp:func:`MFXClose` 销毁会话。
#. 使用 :cpp:func:`MFXUnload` 销毁加载器。


.. note:: 可以创建多个加载器实例。

.. note:: 每个加载器可能有多个与之关联的配置对象。当通过 :cpp:func:`MFXSetConfigFilterProperty` 修改配置对象时，它会隐式影响关联加载器的状态和配置。

.. important:: 一个配置对象只能处理一个过滤器属性。

.. note:: 可以使用一个加载器对象创建多个会话。

当调度程序搜索实现时，它使用以下优先级规则：

#. 硬件实现优先于软件实现。

#. 通用硬件实现优先于 VSI 硬件实现。

#. 最高 API 版本优先于较低 API 版本。

.. note:: 实现优先于 API 版本。换句话说，调度程序必须返回具有最高 API 优先级（大于或等于请求的实现）的实现。

使用实现搜索策略识别共享库的方式将因操作系统而异。

* 在 Windows 上，调度程序会按指定顺序搜索以下位置以查找正确的实现库：

#. 所有可用适配器的 :file:`Driver Store` 目录。
所有类型的图形驱动程序都可以在此目录中安装库。`了解有关驱动程序存储的更多信息 <https://docs.microsoft.com/en-us/windows-hardware/drivers/install/driver-store>`__。
仅适用于英特尔实现。
#. 当前进程的 exe 文件的目录。
#. `PATH` 环境变量。
#. 为了向后兼容旧规范版本，调度程序还会检查由 `ONEVPL_SEARCH_PATH` 环境变量提供的用户定义搜索文件夹。


* 在 Linux 上，调度程序会按指定顺序搜索以下位置以查找正确的实现库：

   #. 环境变量 ``LD_LIBRARY_PATH`` 提供的目录。
   #. 默认路径 :file:`/lib`，然后是 :file:`/usr/lib` 或 :file:`/lib64`，然后是在某些 64 位操作系统上是 :file:`/usr/lib64`。在 Debian 上：:file:`/usr/lib/x86_64-linux-gnu`。
   #. 为了与旧规范版本向后兼容，调度程序还会检查由 `ONEVPL_SEARCH_PATH` 环境变量提供的用户定义搜索文件夹。

.. 重要提示：要优先加载自定义 |vpl_short_name| 库，用户可以使用用户定义文件夹的路径设置环境变量 `ONEVPL_PRIORITY_PATH`。ONEVPL_PRIORITY_PATH 中找到的所有库都具有相同的优先级（高于其他任何库，并且不应用 HW/SW 或 API 版本规则），并且
应根据 :cpp:func:`MFXSetConfigFilterProperty` 进行加载/过滤。

当 |vpl_short_name| 调度程序搜索旧版 |msdk_full_name| 实现时，它使用 :ref:`旧版调度程序搜索顺序 <legacy_search_order>`， 不包括当前工作目录和 :file:`/etc/ld.so.cache`。

调度程序支持不同的软件实现。用户可以使用
:cpp:member:`mfxImplDescription::VendorID` 字段、
:cpp:member:`mfxImplDescription::VendorImplID` 字段或
:cpp:member:`mfxImplDescription::ImplName` 字段来搜索特定实现。

在内部，调度程序的工作方式如下：

#.  调度程序会加载给定搜索文件夹中的所有共享库，这些共享库的名称与下表中的任何模式匹配：
 
   ================== ================== ================= =========================================
   Windows 64-bit     Windows 32-bit     Linux 64-bit      Description
   ================== ================== ================= =========================================
   libvpl\*.dll       libvpl\*.dll       libvpl\*.so\*     任何平台的运行库
   libmfx64-gen.dll   libmfx32-gen.dll   libmfx-gen.so.1.2 |vpl_short_name| |intel_r| 平台 X\ :sup:`e` 架构的运行库
   libmfxhw64.dll     libmfxhw32.dll     libmfxhw64.so.1   |msdk_full_name| 的运行库
   ================== ================== ================= =========================================


#.  对于每个已加载的库，调度程序都会尝试解析 :cpp:func:`MFXQueryImplsDescription` 函数的地址，以收集实现的功能。
#.  一旦用户请求基于此实现创建会话，调度程序就会获取每个 |vpl_short_name| 函数的地址。请参阅 :ref:`导出的函数/API 版本表 <export-func-version-table-2x>` 以获取要导出的函数列表。

-----------------------------------------------------------
|vpl_short_name| 调度程序配置属性
-----------------------------------------------------------
 :ref:`调度程序配置属性表 <dsp-conf-prop-table>` 显示调度程序支持的属性字符串。表以层次结构方式组织，要创建字符串，请从左到右逐列进行，并使用`.`（点）作为分隔符连接字符串。

.. _dsp-conf-prop-table:

.. container:: stripe-table

   .. table:: Dispatcher Configuration Properties
      :widths: 25 25 30 20

      +---------------------------------------+----------------------------+----------------------+---------------------------+
      | 结构名称                                     | 属性                            | 数据类型                     | 注释                   |
      +=======================================+============================+======================+===========================+
      | :cpp:struct:`mfxImplDescription`      | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .Impl                    |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 | 这个模式将被用作                     |
      |                                       | | .AccelerationMode        |                      | 会话初始化                             |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .ApiVersion              |                      |                           |
      |                                       | | .Version                 |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .ApiVersion              |                      |                           |
      |                                       | | .Major                   |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .ApiVersion              |                      |                           |
      |                                       | | .Minor                   |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向null结尾的string       |
      |                                       | | .ImplName                |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向null结尾的string       |
      |                                       | | .License                 |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向null结尾的string       |
      |                                       | | .Keywords                |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .VendorID                |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .VendorImplID            |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxSurfacePoolMode      |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向null结尾的string       |
      |                                       | | .mfxDeviceDescription    |                      |                           |
      |                                       | | .device                  |                      |                           |
      |                                       | | .DeviceID                |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .mfxDeviceDescription    |                      |                           |
      |                                       | | .device                  |                      |                           |
      |                                       | | .MediaAdapterType        |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxDecoderDescription   |                      |                           |
      |                                       | | .decoder                 |                      |                           |
      |                                       | | .CodecID                 |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .mfxDecoderDescription   |                      |                           |
      |                                       | | .decoder                 |                      |                           |
      |                                       | | .MaxcodecLevel           |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxDecoderDescription   |                      |                           |
      |                                       | | .decoder                 |                      |                           |
      |                                       | | .decprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxDecoderDescription   |                      |                           |
      |                                       | | .decoder                 |                      |                           |
      |                                       | | .decprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .decmemdesc              |                      |                           |
      |                                       | | .MemHandleType           |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向                                       |
      |                                       | | .mfxDecoderDescription   |                      | :cpp:struct:`mfxRange32U` |
      |                                       | | .decoder                 |                      | 对象                        |
      |                                       | | .decprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .decmemdesc              |                      |                           |
      |                                       | | .Width                   |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向                      |
      |                                       | | .mfxDecoderDescription   |                      | :cpp:struct:`mfxRange32U` |
      |                                       | | .decoder                 |                      | 对象                      |
      |                                       | | .decprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .decmemdesc              |                      |                           |
      |                                       | | .Height                  |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxDecoderDescription   |                      |                           |
      |                                       | | .decoder                 |                      |                           |
      |                                       | | .decprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .decmemdesc              |                      |                           |
      |                                       | | .ColorFormats            |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .CodecID                 |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .MaxcodecLevel           |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .BiDirectionalPrediction |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .ReportedStats           |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .encprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .encprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .encmemdesc              |                      |                           |
      |                                       | | .MemHandleType           |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向                                |
      |                                       | | .mfxEncoderDescription   |                      | :cpp:struct:`mfxRange32U` |
      |                                       | | .encoder                 |                      | 对象                        |
      |                                       | | .encprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .encmemdesc              |                      |                           |
      |                                       | | .Width                   |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向                                |
      |                                       | | .mfxEncoderDescription   |                      | :cpp:struct:`mfxRange32U` |
      |                                       | | .encoder                 |                      | 对象                        |
      |                                       | | .encprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .encmemdesc              |                      |                           |
      |                                       | | .Height                  |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxEncoderDescription   |                      |                           |
      |                                       | | .encoder                 |                      |                           |
      |                                       | | .encprofile              |                      |                           |
      |                                       | | .Profile                 |                      |                           |
      |                                       | | .encmemdesc              |                      |                           |
      |                                       | | .ColorFormats            |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxVPPDescription       |                      |                           |
      |                                       | | .filter                  |                      |                           |
      |                                       | | .FilterFourCC            |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U16 |                           |
      |                                       | | .mfxVPPDescription       |                      |                           |
      |                                       | | .filter                  |                      |                           |
      |                                       | | .MaxDelayInFrames        |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxVPPDescription       |                      |                           |
      |                                       | | .filter                  |                      |                           |
      |                                       | | .memdesc                 |                      |                           |
      |                                       | | .MemHandleType           |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向                      |
      |                                       | | .mfxVPPDescription       |                      | :cpp:struct:`mfxRange32U` |
      |                                       | | .filter                  |                      | 对象                        |
      |                                       | | .memdesc                 |                      |                           |
      |                                       | | .Width                   |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_PTR | 指向                      |
      |                                       | | .mfxVPPDescription       |                      | :cpp:struct:`mfxRange32U` |
      |                                       | | .filter                  |                      | 对象                      |
      |                                       | | .memdesc                 |                      |                           |
      |                                       | | .Height                  |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxVPPDescription       |                      |                           |
      |                                       | | .filter                  |                      |                           |
      |                                       | | .memdesc                 |                      |                           |
      |                                       | | .format                  |                      |                           |
      |                                       | | .InFormat                |                      |                           |
      |                                       +----------------------------+----------------------+---------------------------+
      |                                       | | mfxImplDescription       | MFX_VARIANT_TYPE_U32 |                           |
      |                                       | | .mfxVPPDescription       |                      |                           |
      |                                       | | .filter                  |                      |                           |
      |                                       | | .memdesc                 |                      |                           |
      |                                       | | .format                  |                      |                           |
      |                                       | | .OutFormats              |                      |                           |
      +---------------------------------------+----------------------------+----------------------+---------------------------+
      | :cpp:struct:`mfxImplementedFunctions` | | mfxImplementedFunctions  | MFX_VARIANT_TYPE_PTR | 指向string的缓冲区         |
      |                                       | | .FunctionsName           |                      |                           |
      +---------------------------------------+----------------------------+----------------------+---------------------------+
      | N/A                                   | | DXGIAdapterIndex         | MFX_VARIANT_TYPE_U32 | 根据                      |
      |                                       | |                          |                      | IDXGIFactory::EnumAdapters|
      |                                       | |                          |                      | 适配器的索引值             |
      +---------------------------------------+----------------------------+----------------------+---------------------------+
      | N/A                                   | | AutoSelectImpl           | MFX_VARIANT_TYPE_PTR | 指向一个结构，该结构用来    |
      |                                       | |                          |                      | 表示自动实现               |
      |                                       | |                          |                      | :ref:`selection<auto_sel>`|
      +---------------------------------------+----------------------------+----------------------+---------------------------+



.. 重要提示：DXGIAdapterIndex 属性仅适用于 Windows，并且仅过滤硬件实现。

属性名称字符串示例：

- mfxImplDescription.mfxDecoderDescription.decoder.decprofile.Profile
- mfxImplDescription.mfxDecoderDescription.decoder.decprofile.decmemdesc.MemHandleType
- mfxImplementedFunctions.FunctionsName

以下属性以特殊方式受支持：它们用于通过调度程序向实现发送
其他数据。应用程序需要使用 :cpp:func:`MFXSetConfigFilterProperty` 来设置它们，但它们不会影响实现的选择。它们在 :cpp:func:`MFXCreateSession` 函数调用期间使用，以微调实现。


.. list-table:: Dispatcher 的特殊属性
   :header-rows: 1
   :widths: auto

   * - **属性名称**
     - **属性值**
     - **值数据类型**
   * - mfxHandleType
     - :cpp:enum:`mfxHandleType`
     - :cpp:enumerator:`mfxVariantType::MFX_VARIANT_TYPE_U32`
   * - mfxHDL
     - :cpp:type:`mfxHDL`
     - :cpp:enumerator:`mfxVariantType::MFX_VARIANT_TYPE_PTR`
   * - NumThread
     - 无符号定点整数值
     - :cpp:enumerator:`mfxVariantType::MFX_VARIANT_TYPE_U32`
   * - DeviceCopy
     - :ref:`Device copy <gpu_copy>`
     - :cpp:enumerator:`mfxVariantType::MFX_VARIANT_TYPE_U16`
   * - ExtBuffer
     - 指向扩展缓冲区的指针
     - :cpp:enumerator:`mfxVariantType::MFX_VARIANT_TYPE_PTR`

.. _vpl-dispatcher-interactions:

----------------------------------------
|vpl_short_name| 调度程序交互
----------------------------------------

此序列图直观地显示了应用程序如何通过调度程序与实现进行通信。

调度程序 API
    此 API 在调度程序中实现。

实现  API
    此 API 由任何实现提供。


.. uml::

   @startuml
   actor Application as A
   participant "Intel® VPL Dispatcher" as D
   participant "Intel® VPL Implementation 1" as I1
   participant "Intel® VPL Implementation 2" as I2
   participant "Intel® VPL Implementation 3" as I3

   ref over A, D : Dispatcher API
   ref over D, I1, I2, I3 : Implementation API

   activate A
   == Initialization ==
   group Dispatcher API [Enumerate and load implementations]
      A -> D: mfxLoad()
      activate D

      D -> D: Search for the available runtimes

      A -> D: MFXCreateConfig()
      A -> D: MFXSetConfigProperty()
      A -> D: MFXCreateConfig()
      A -> D: MFXSetConfigProperty()
      A -> D: MFXEnumImplementations()

      note right of A
      MFXEnumImplementations() may also be called
      after MFXCreateSession().
      end note

      Activate I1

      D -> I1: MFXQueryImplsDescription()
      I1 --> D: mfxImplDescription

      Activate I2
      D -> I2: MFXQueryImplsDescription()
      I2 --> D: mfxImplDescription

      Activate I3
      D -> I3: MFXQueryImplsDescription()
      I3 --> D: mfxImplDescription

      D --> A: list of mfxImplDescription structures es for all implementations

      A -> D: MFXCreateSession(i=0)
      D -> I1: MFXInitilize()
      Activate I1 #DarkSalmon
      I1 --> D: mfxSession1
      D --> A: mfxSession1

      A -> D: MFXCreateSession(i=1)
      D -> I2: MFXInitilize()
      Activate I2 #DarkSalmon
      I2 --> D: mfxSession2
      D --> A: mfxSession2

      A -> D: MFXCreateSession(i=2)
      D -> I3: MFXInitilize()
      I3 --> D: mfxSession3
      Activate I3 #DarkSalmon
      D --> A: mfxSession3

      A -> D: MFXDispReleaseImplDescription(hdl=0)
      D -> I1: MFXReleaseImplDescription()

      A -> D: MFXDispReleaseImplDescription(hdl=1)
      D -> I2: MFXReleaseImplDescription()

      A -> D: MFXDispReleaseImplDescription(hdl=2)
      D -> I3: MFXReleaseImplDescription()
   end
   == Processing ==
   group Implementation API [Process the data]
      A -> I1: MFXVideoDECODE_Init()
      A -> I1: MFXVideoDECODE_Query()
      A -> I1: MFXVideoDECODE_DecodeFrameAsync()
      ...
      A -> I1: MFXVideoDECODE_Close()
      |||
      deactivate I1

      A -> I2: MFXVideoENCODE_Init()
      A -> I2: MFXVideoENCODE_Query()
      A -> I2: MFXVideoENCODE_EncodeFrameAsync()
      ...
      A -> I2: MFXVideoENCODE_Close()
      |||
      deactivate I2

      A -> I3: MFXVideoENCODE_Init()
      A -> I3: MFXVideoENCODE_Query()
      A -> I3: MFXVideoENCODE_EncodeFrameAsync()
      ...
      A -> I3: MFXVideoENCODE_Close()
      |||
      deactivate I3
   end
   == Finalization ==
   group Implementation API [Release the implementations]
      A -> I1: MFXClose()
      deactivate I1
      A -> I2: MFXClose()
      deactivate I2
      A -> I3: MFXClose()
      deactivate I3
   group Dispatcher API [Release the dispatcher's instance]
      A -> D: MFXUnload()
   end

   deactivate D
   @enduml

|vpl_short_name| 调度程序能够加载和初始化 |msdk_full_name| 旧版库。下面的序列图演示了该方法。

.. uml::

   @startuml
   actor Application as A
   participant "Intel® VPL Dispatcher" as D
   participant "Intel® MediaSDK (legacy)" as M
   A -> D: MFXLoad
   activate D
   D -> D: Search for the available runtimes
   A -> D: MFXCreateConfig
   
   note left of D
   Setting properties to filter implementation.
   MediaSDK supports only general parameters,
   no filtering for Decode/VPP/Encoder details.
   end note

   group MediaSDK LegacyAPI
      D -> M: MFXInitEx
      activate M

      M --> D: mfxSession
      D -> M: MFXQueryIMPL
      M --> D: Implementation speciefic

      D -> M: MFXQueryVersion
      M --> D: mfxVersion
   end

   D -> D: Fill mfxImplDescription for MediaSDK impl

   A -> D: mfxEnumImplementations
   D --> A: MediaSDK caps description

   A -> D: MFXCreateSession
   D --> A: mfxSession

   A -> M: MFXVideoDECODE_Init()
   A -> M: MFXVideoDECODE_Query()
   A -> M: MFXVideoDECODE_DecodeFrameAsync()
   ...
   A -> M: MFXVideoDECODE_Close()
   A -> M: MFXClose()

   deactivate M

   A -> D: MFXUnload()
   deactivate D
   @enduml

.. important:: 调度程序在枚举或创建 |msdk_full_name| 实现时不会过滤和报告
               :cpp:struct:`mfxDeviceDescription`,
               :cpp:struct:`mfxDecoderDescription`,
               :cpp:struct:`mfxEncoderDescription`,
               :cpp:struct:`mfxVPPDescription`。 一旦加载 |msdk_full_name|，应用程序就必须使用传统方法来查询功能。

------------------------------------------
|vpl_short_name| 调度程序调试日志
------------------------------------------

调度程序的调试输出由 `ONEVPL_DISPATCHER_LOG` 环境变量控制。要启用日志输出，请将 `ONEVPL_DISPATCHER_LOG`
环境变量值设置为“ON”。

默认情况下，|vpl_short_name| 调度程序会将所有日志消息打印到控制台。 要将日志输出重定向到所需文件，请将 `ONEVPL_DISPATCHER_LOG_FILE`
环境变量设置为日志文件的文件名。

------------------------------
Dispatcher用法示例
------------------------------

此代码说明了调度程序加载第一个可用库的简单用法:

.. literalinclude:: ../snippets/prg_disp.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

此代码说明了如何使用调度程序加载第一个可用的硬件加速库:

.. literalinclude:: ../snippets/prg_disp.c
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

此代码说明了如何创建来自多个加载器的多个会话:

.. literalinclude:: ../snippets/prg_disp.c
   :language: c++
   :start-after: /*beg3*/
   :end-before: /*end3*/
   :lineno-start: 1

此代码说明了如何从单个加载器创建多个解码器:

.. literalinclude:: ../snippets/prg_disp.c
   :language: c++
   :start-after: /*beg4*/
   :end-before: /*end4*/
   :lineno-start: 1

---------------------------------------
如何检查函数是否已实现
---------------------------------------

有两种方法可以检查特定函数是否由实现实现。

此代码说明了应用程序如何遍历整个已实现函数列表：

.. literalinclude:: ../snippets/prg_session.cpp
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

此代码说明了应用程序如何检查特定功能是否已实现：

.. literalinclude:: ../snippets/prg_session.cpp
   :language: c++
   :start-after: /*beg2*/
   :end-before: /*end2*/
   :lineno-start: 1

--------------------------------------------------------------
如何搜索可用的编码器/解码器实现
--------------------------------------------------------------

:ref:`CodecFormatFourCC <codec-format-fourcc>` 枚举指定编解码器的 FourCC
值。应用程序需要将此值分配给
:cpp:member:`mfxDecoderDescription::decoder::CodecID` 字段以搜索解码器
或 :cpp:member:`mfxEncoderDescription::encoder::CodecID` 字段以搜索
编码器。

此代码说明了解码器的实现搜索过程：

.. literalinclude:: ../snippets/prg_session.cpp
   :language: c++
   :start-after: /*beg3*/
   :end-before: /*end3*/
   :lineno-start: 1

---------------------------------------------------------
如何搜索可用的 VPP 过滤器实现
---------------------------------------------------------

每个 VPP 过滤器由过滤器 ID 标识。过滤器 ID 由对应于过滤器扩展缓冲区 ID 值的定义，该值以 FourCC 值的形式定义。过滤器 ID 值是通用 :ref:`ExtendedBufferID <extendedbufferid>`
枚举的子集。:ref:`table <vpp-filters-ids>` 引用了可供搜索的 VPP 过滤器 ID。应用程序需要将此值分配给 :cpp:member:`mfxVPPDescription::filter::FilterFourCC` 字段，以搜索所需的 VPP 过滤器。

.. _vpp-filters-ids:

.. list-table:: VPP Filters ID
   :header-rows: 1
   :widths: 58 42

   * - **过滤器 ID**
     - **描述**
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_DENOISE2`
     - 去噪过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_MCTF`
     - 运动补偿时间滤波器 (MCTF).
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_DETAIL`
     - 细节/边缘增强滤波器。
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_FRAME_RATE_CONVERSION`
     - 帧率转换滤镜
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_IMAGE_STABILIZATION`
     - 图像稳定滤镜
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_PROCAMP`
     - 调节图像亮度色度饱和度过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_FIELD_PROCESSING`
     - 场处理过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_COLOR_CONVERSION`
     - 颜色转换过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_SCALING`
     - 图像大小调节过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_COMPOSITE`
     - Surfaces拼接过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_DEINTERLACING`
     - 解交织过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_ROTATION`
     - 旋转过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_MIRRORING`
     - 镜像过滤器
   * - :cpp:enumerator:`MFX_EXTBUFF_VPP_COLORFILL`
     - 图像填充过滤器

此代码说明了VPP镜像过滤器实现搜索过程:

.. literalinclude:: ../snippets/prg_session.cpp
   :language: c++
   :start-after: /*beg4*/
   :end-before: /*end4*/
   :lineno-start: 1

.. _auto_sel:

-------------------------------------------------------------
如何从设备句柄自动选择实现
-------------------------------------------------------------

从 API 2.9 开始，应用程序可以请求调度程序加载与应用程序已初始化的硬件加速设备相对应的实现。应用程序必须使用适当的加速模式、句柄类型和硬件设备句柄初始化类型为
:cpp:struct:`mfxAutoSelectImplDeviceHandle` 的结构。然后必须通过调用 :cpp:func:`MFXSetConfigFilterProperty` 将此结构传递给调度程序，其中属性名称为
'AutoSelectImpl'，属性值为类型为 MFX_VARIANT_TYPE_PTR，指向
:cpp:struct:`mfxAutoSelectImplDeviceHandle` 结构。

这目前是一个实验性的 API。不保证向后兼容性和未来存在性。应用程序应检查从
:cpp:func:`MFXSetConfigFilterProperty` 和 :cpp:func:`MFXCreateSession` 返回的错误代码，以检查是否支持该功能并找到合适的实现。

此代码说明了使用应用程序提供的硬件设备句柄进行自动实现选择：

.. literalinclude:: ../snippets/prg_disp.c
   :language: c++
   :start-after: /*beg5*/
   :end-before: /*end5*/
   :lineno-start: 1

-------------------------------------------------------------
如何通过实现获取共享库的路径
-------------------------------------------------------------

会话可以从不同的实现中创建，每个实现可以位于不同的共享库中。要获取可以或曾经创建会话的实现的共享库的路径，应用程序可以使用：cpp：func：`MFXEnumImplementations`并传递：cpp：enumerator：`MFX_IMPLCAPS_IMPLPATH`
值作为输出数据请求。

此代码说明了如何收集和打印实现的共享库路径：

.. literalinclude:: ../snippets/prg_session.cpp
   :language: c++
   :start-after: /*beg5*/
   :end-before: /*end5*/
   :lineno-start: 1

.. _vpl_coexistense:

--------------------------------------------------------------------------------------------------------------------- 
|vpl_short_name| 在具有 X\ :sup:`e` 架构 的|intel_r| 平台上实现 和 |msdk_full_name| 共存
---------------------------------------------------------------------------------------------------------------------

|vpl_short_name| 取代 |msdk_full_name| 并与 |msdk_full_name| 部分二进制兼容。|vpl_short_name| 和 |msdk_full_name| 都包含自己的调度程序和
实现。在 |msdk_full_name| 尚未 EOL 之前，允许 |vpl_short_name| 和 |msdk_full_name| 调度程序和实现在单个系统上共存。

单个应用程序中的以下调度程序和实现组合仅允许用于旧版目的。在这种情况下，使用 |msdk_full_name| 开发的旧版应用程序将继续在由 |msdk_full_name| 或 |vpl_short_name| 支持的任何硬件上运行。

.. 注意：任何与 |vpl_short_name| 配合使用的应用程序从版本 2.0 开始的 API 必须仅使用 |vpl_short_name| 调度程序。

|msdk_full_name| API
    |msdk_full_name| 1.x 版本的API.

Removed API
    |msdk_full_name| :ref:`API <deprecated-api>` 从 |vpl_short_name|中移除 .

Core API
    |msdk_full_name| 不包含移除的API.

|vpl_short_name| API
    |vpl_short_name| 中引入的新 :ref:`API <new-api>` 仅从 API 2.0 版本开始支持这些API。

|vpl_short_name| 调度器 API
    调度程序：ref:`API <dispatcher-api>` 在 2.0 API 版本 |vpl_short_name| 中引入。这是 |vpl_short_name| API 的子集。

.. list-table:: 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台以及 |msdk_full_name|
   :header-rows: 1
   :widths: 25 25 25 25

   * - **调度程序**
     - **安装在设备上**
     - **加载**
     - **允许的 API**
   * - |vpl_short_name|
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台
     - 允许使用除已移除的 API 之外的任何 API.
   * - |vpl_short_name|
     - |msdk_full_name|
     - |msdk_full_name|
     - 仅允许使用核心 API 和调度程序 API。
   * - |vpl_short_name|
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台和 |msdk_full_name|
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台
     - 允许使用除已移除的 API 之外的任何 API.
   * - |msdk_full_name|
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台和 |msdk_full_name|
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台和 |msdk_full_name|
     - 仅允许使用核心 API.
   * - |msdk_full_name|
     - |vpl_short_name| 适用于具有 X\ :sup:`e` 架构的 |intel_r| 平台和 |msdk_full_name|
     - |msdk_full_name|
     - 允许使用 |msdk_full_name| API。
   * - |msdk_full_name|
     - |msdk_full_name|
     - |msdk_full_name|
     - 允许使用 |msdk_full_name| API。

.. note:: 如果系统有多个设备，则选择和加载实现的逻辑将根据系统枚举应用于每个设备。

-----------------
多个会话
-----------------

每个 |vpl_short_name| 会话可以运行 DECODE、ENCODE 和 VPP 函数的一个实例。这对于简单的转码操作来说已经足够了。如果应用程序在复杂的转码设置中需要多个 DECODE、ENCODE 或 VPP 实例，或者需要同时执行更多转码操作，则应用程序可以初始化从一个或多个 |vpl_short_name| 实现创建的多个|vpl_short_name| 会话。

应用程序可以独立使用多个 |vpl_short_name| 会话或运行 “joined” (已连接)会话。要将两个会话连接在一起，应用程序可以使用函数 :cpp:func:`MFXJoinSession`。或者，应用程序可以使用 :cpp:func:`MFXCloneSession` 函数复制现有会话。已连接 |vpl_short_name|会话作为一个会话一起工作，共享所有会话资源、线程控制和优先级操作（硬件加速设备和外部分配器除外）。加入后，第一个会话（第一个加入）将作为父会话，并将与所有其他子会话一起安排执行资源。子会话依赖父会话进行资源管理。

.. 重要提示：应用程序只能加入从同一 |vpl_short_name| 实现创建的会话。

对于已加入的会话，应用程序可以通过 :cpp:func:`MFXSetPriority` 函数设置会话操作的优先级。优先级较低的会话获得的 CPU 周期较少。会话优先级不会影响硬件加速处理。

完成所有会话操作后，应用程序可以使用 :cpp:func:`MFXDisjoinSession` 函数删除会话的加入状态。

在所有子会话都已解除或关闭之前，请勿关闭父会话。
