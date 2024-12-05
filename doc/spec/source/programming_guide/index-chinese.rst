.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0
..
  Intel(r) Video Processing Library (Intel(r) VPL)

=================
编程指南
=================

本章介绍使用 |vpl_short_name| 编程所使用的概念。

C/C++ 应用程序必须使用包含文件 :file:`mfx.h` ，并链接 |vpl_short_name| 调度程序库 :file:`libvpl.so`。

包含以下文件：

.. code-block:: c++

   #include "mfx.h"    /* Intel® VPL include file */

链接下面的库：

.. code-block:: c++

   libvpl.so                /* Intel® VPL 动态调度库 (Linux\*) */

.. toctree::
   :hidden:

   VPL_prg_stc-chinese
   VPL_prg_session-chinese
   VPL_prg_frame-chinese
   VPL_prg_decoding-chinese
   VPL_prg_encoding-chinese
   VPL_prg_vpp-chinese
   VPL_prg_transcoding-chinese
   VPL_prg_config-chinese
   VPL_prg_hw-chinese
   VPL_prg_mem-chinese
   VPL_prg_surface_sharing-chinese
   VPL_prg_err-chinese
