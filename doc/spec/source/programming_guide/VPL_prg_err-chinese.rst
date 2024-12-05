.. SPDX-FileCopyrightText: 2019-2020 Intel Corporation
..
.. SPDX-License-Identifier: CC-BY-4.0

==============================
硬件设备错误处理
==============================

对于通过硬件设备加速​​解码、编码和视频处理的实现，如果硬件设备遇到错误，API 函数可能会返回错误或警告。有关错误和警告的详细信息，请参阅：ref：`硬件设备错误和警告表 <hw-device-errors-table>`。

.. _hw-device-errors-table:

.. list-table:: 硬件设备错误和警告
   :header-rows: 1
   :widths: 60 40

   * - **状态**
     - **描述**
   * - :cpp:enumerator:`mfxStatus::MFX_ERR_DEVICE_FAILED`
     - 硬件设备返回意外错误。|vpl_short_name| 无法恢复操作。
   * - :cpp:enumerator:`mfxStatus::MFX_ERR_DEVICE_LOST`
     - 由于系统锁定或关机，硬件设备丢失。
   * - :cpp:enumerator:`mfxStatus::MFX_WRN_PARTIAL_ACCELERATION`
     - 硬件不完全支持指定的配置。编码、解码或视频处理操作可能会部分加速。
   * - :cpp:enumerator:`mfxStatus::MFX_WRN_DEVICE_BUSY`
     - 硬件设备目前正忙。


|vpl_short_name| **Query**, **QueryIOSurf**, and **Init** functions return
:cpp:enumerator:`mfxStatus::MFX_WRN_PARTIAL_ACCELERATION` to indicate that the encoding,
decoding, or video processing operation can be partially hardware accelerated or
not hardware accelerated at all. The application can ignore this warning and
proceed with the operation. (Note that |vpl_short_name| functions may return
errors or other warnings overwriting
:cpp:enumerator:`mfxStatus::MFX_WRN_PARTIAL_ACCELERATION`, as it is a lower priority warning.)

|vpl_short_name| functions return :cpp:enumerator:`mfxStatus::MFX_WRN_DEVICE_BUSY` to indicate that the
hardware device is busy and unable to receive commands at this time. The recommended approach is:

   * If the asynchronous operation returns synchronization point along with :cpp:enumerator:`mfxStatus::MFX_WRN_DEVICE_BUSY` - call the :cpp:func:`MFXVideoCORE_SyncOperation` with it.
   * If application has buffered synchronization point(s) obtained from previous asynchronous operations - call :cpp:func:`MFXVideoCORE_SyncOperation` with the oldest one.
   * If no synchronization point(s) available - wait for a few milliseconds.
   * Resume the operation by resubmitting the request.


.. literalinclude:: ../snippets/prg_err.c
   :language: c++
   :start-after: /*beg1*/
   :end-before: /*end1*/
   :lineno-start: 1

The same procedure applies to encoding and video processing.

|vpl_short_name| functions return :cpp:enumerator:`mfxStatus::MFX_ERR_DEVICE_LOST` or
:cpp:enumerator:`mfxStatus::MFX_ERR_DEVICE_FAILED` to indicate that there is a complete
failure in hardware acceleration. The application must close and reinitialize
the |vpl_short_name| function class. If the application has provided a hardware acceleration
device handle to |vpl_short_name|, the application must reset the device.



