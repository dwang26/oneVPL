.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0
..
  Intel(r) Video Processing Library (Intel(r) VPL)

============
架构
============

|vpl_short_name| 中的函数分为下面几类:

.. glossary::

   DECODE
      将压缩视频流解码为原始视频帧的函数

   ENCODE
      将原始视频帧编码为压缩比特流的函数

   VPP
      对原始视频帧执行视频处理的函数

   DECODE_VPP
      执行解码和视频处理组合操作的函数

   CORE
      用于同步的辅助函数

   Misc
      全局辅助函数

除全局辅助函数外，|vpl_short_name| 函数均以其功能域和类别命名。|vpl_short_name| 提供了视频相关的函数

.. figure:: ../images/sdk_function_naming_convention.png

   |vpl_short_name| 函数名称表示法

应用程序通过链接 |vpl_short_name| 调度程序库来使用 |vpl_short_name| 函数。

.. graphviz::
   :caption: |vpl_short_name| 调度机制

   digraph {
     rankdir=TB;
     Application [shape=record label="Application" ];
     Sdk [shape=record  label="Intel® VPL 调度库"];
     Lib1 [shape=record  label="Intel® VPL 库 1 (CPU)"];
     Lib2 [shape=record  label="Intel® VPL 库 2 (平台 1)"];
     Lib3 [shape=record  label="Intel® VPL 库 3 (平台 2)"];
     Application->Sdk;
     Sdk->Lib1;
     Sdk->Lib2;
     Sdk->Lib3;
   }


调度库识别运行平台上的硬件加速设备，为所识别的硬件加速确定最合适的平台库，然后相应地重定向函数调用。


.. toctree::
   :hidden:

   VPL_decoding-chinese
   VPL_encoding-chinese
   VPL_processing-chinese
   VPL_decoding_vpp-chinese
