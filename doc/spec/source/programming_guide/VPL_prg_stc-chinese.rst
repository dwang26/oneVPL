.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

============
状态代码
============

|vpl_short_name| 函数被组织成各种类别，以便于参考。这些类别包括 :term:`ENCODE`（编码函数）、:term:`DECODE`（解码函数）和 :term:`VPP`（视频处理函数）。

**Init**、**Reset** 和 **Close** 是 ENCODE、
DECODE 和 VPP 类中的成员函数，用于该类的初始化、重新启动和取消初始化的特定操作。应用程序按照 **Init** **-** **Reset** **-** **Close** 的顺序调用给定类的所有成员函数，除了**Query** 和 **QueryIOSurf** 。**Reset** 函数在这个调用顺序中是可选的。

**Init** 和 **Reset** 成员函数设置视频处理所需的内部
结构。**Init** 函数分配内存，而 **Reset** 函数仅重用分配的内部内存。如果 |vpl_short_name|需要分配
额外的内存，**Reset** 可能会失败。**Reset** 函数还可以
在这些过程中微调 ENCODE 和 VPP 参数，或在 DECODE 期间重新定位
比特流。

所有 |vpl_short_name| 函数都会返回状态代码以指示操作是成功还是失败。:cpp:enumerator:`mfxStatus::MFX_ERR_NONE` 状态代码表示
该函数已成功完成其操作。错误状态代码的值小于 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`，警告状态代码的值大于 :cpp:enumerator:`mfxStatus::MFX_ERR_NONE`。有关所有的状态代码的定义，请参阅 :cpp:enum:`mfxStatus` 枚举器。

如果 |vpl_short_name| 函数返回警告，则表示它已充分完成其操作。请注意，函数的输出可能不是严格可靠的。应用程序必须检查函数生成的输出的有效性。

如果 |vpl_short_name| 函数返回错误（除了 :cpp:enumerator:`mfxStatus::MFX_ERR_MORE_DATA`、
:cpp:enumerator:`mfxStatus::MFX_ERR_MORE_SURFACE` 或
:cpp:enumerator:`mfxStatus::MFX_ERR_MORE_BITSTREAM`），该函数将中止操作。应用程序必须调用 **Reset** 函数将类重置为干净状态，或调用 **Close** 函数终止操作。

如果应用程序继续调用任何类成员函数而没有 **Reset** 或 **Close**，则行为未定义。为避免内存泄漏，请始终在 **Init** 之后调用 **Close** 函数。
