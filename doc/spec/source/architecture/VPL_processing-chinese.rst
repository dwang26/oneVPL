.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

==================
视频处理(VPP)
==================

视频处理函数（:term:`VPP`）将原始帧作为输入并提供原始帧作为输出。

实际的转换过程是一个具有许多单功能过滤器的链式操作。

.. graphviz::
   :caption: Video processing operation pipeline

   digraph {
     rankdir=LR;
     F1 [shape=record label="Function 1" ];
     F2 [shape=record  label="Function 2"];
     F3 [shape=record  label="Additional filters"];
     F4 [shape=record label="Function N-1" ];
     F5 [shape=record  label="Function N"];
     F1->F2->F3->F4->F5;
   }

The application specifies the input and output format; |vpl_short_name| configures the
pipeline according to the specified input and output formats. The application
can also attach one or more hint structures
to configure individual filters or turn them on and off. Unless specifically
instructed, |vpl_short_name| builds the pipeline in a way that best utilizes hardware
acceleration or generates the best video processing quality.

The :ref:`Video Processing Features table <vid-processing-feat-table>` shows |vpl_short_name|
video processing features. The application can configure supported video
processing features through the video processing I/O parameters. The application
can also configure optional features through hints.
See :ref:`Video Processing Procedures <vid_process_procedure>` for more details
on how to configure optional filters.

应用程序指定输入和输出格式；|vpl_short_name| 根据指定的输入和输出格式配置管道。
应用程序还可以附加一个或多个结构体来配置单个过滤器或打开和关闭它们。除非特别设置，
否则 |vpl_short_name| 会以最佳利用硬件加速或生成最佳视频处理质量的方式构建管道。

:ref:`Video Processing Features table <vid-processing-feat-table>` 展示了 |vpl_short_name| 视频处理功能。
应用程序可以通过视频处理 I/O 参数来配置支持的视频处理功能。应用程序还可以通过配置标志位来配置可选功能。
有关如何配置可选过滤器的更多详细信息，请参阅 :ref:`Video Processing Procedure <vid_process_procedure>`。

.. _vid-processing-feat-table:

.. list-table:: 视频处理特性
   :header-rows: 1
   :widths: 70 30

   * - **视频处理特性**
     - **配置**
   * - 对输入帧颜色格式转换
     - I/O parameters
   * - 解交织并输出连续帧
     - I/O parameters
   * - 对输入帧进行裁剪和缩放
     - I/O parameters
   * - 对输入帧进行帧率转换来匹配输出帧率
     - I/O parameters
   * - 执行逆电视电影操作
     - I/O parameters
   * - 域交织
     - I/O parameters
   * - 域分割
     - I/O parameters
   * - 降噪
     - Hint (可选)
   * - 细节和边缘增强
     - Hint (可选)
   * - 调整亮度，对比度，包含度和色调设置
     - Hint (可选)
   * - 图像稳定
     - Hint (可选)
   * - 根据帧插值转换输入帧速率以匹配输出
     - Hint (可选)
   * - 执行图片结构检测
     - Hint (可选)
