# `vpl-infer` Sample
## Table of Contents
- [Intro](#intro)
- [Purpose](#purpose)
- [Key Implementation Details](#key-implementation-details)
- [Command Line Options](#command-line-options)
- [License](#license)
- [Building and Executing the vpl-infer Program](#building-and-executing-the-vpl-infer-Program)
  - [On a Linux* System](#on-a-linux-system)
  - [On a Windows* System](#on-a-windows-system)
  - [Example of Output](#example-of-output)


## Intro

This sample shows how to use the oneAPI Video Processing Library (oneVPL) to
perform a simple video decode and resize, and how to use OpenVINO for inferencing 

| Optimized for    | Description
|----------------- | ----------------------------------------
| OS               | Ubuntu* 20.04; Windows* 10
| Hardware runtime | Compatible with Intel® oneAPI Video Processing Library(oneVPL) GPU implementation, which can be found at https://github.com/oneapi-src/oneVPL-intel-gpu 
|                  | and Intel® Media SDK GPU implementation, which can be found at https://github.com/Intel-Media-SDK/MediaSDK
| What You Will Learn | How to use oneVPL to decode an H.265 encoded video file and resize and perform per-frame object detection inference
| Time to Complete | 5 minutes


## Purpose

This sample is a command line application that takes a file containing an H.265
video elementary stream and network model as an argument, decodes and resize it with oneVPL and perform 
object detection on each frame using OpenVINO.


## Key Implementation Details

| Configuration          | Default setting
| ---------------------- | ----------------------------------
| Target device          | GPU
| Input video format     | H.265 video elementary stream
| Input IR network model | Object detection 
| Output                 | Class ID, Bounding Box Location, and Confidence Score


## Command Line Options

| Option   | Description | Note
| -------  | -------------------------------| -----------------
| -hw      | oneVPL hardware implementation | Default
| -i       | H.265 video elementary stream  |
| -m       | Object detection network model |
| -legacy  | Run sample in legacy gen (ex: gen 9.x - SKL, KBL, etc) |
| -zerocopy| process data without copying between oneVPL and OpenVINO in hardware implemenation mode | Linux only
|          | | with `-hw` only
|          | | not with `-legacy`


## License

This code sample is licensed under MIT license.


## Building and Executing the `vpl-infer` Program

The first step is to set up a build environment with prerequisites installed.  
This can be set up in a bare metal Ubuntu 20.04 system or with Docker for Linux, and in Windows. 


### On a Linux* System

1. Install the prerequisite software:

   - Intel® oneAPI Base Toolkit for Linux*
   - Intel® OpenVINO for Linux*
   - [Python](http://python.org)
   - [CMake](https://cmake.org)
   - OpenCL headers: 'sudo apt-get install -y opencl-headers' 


2. Install Intel general purpose GPU (GPGPU) software packages:

   Follow steps in 
   https://dgpu-docs.intel.com/installation-guides/index.html#


3. Download the Mobilenet-ssd object detection model from OpenVINO model Zoo and covert it to IR model:

    Start Python virtual environments from Command Prompt
    ```
    apt-get install python3 python3.8-venv
    python3 -m venv openvino_env
    source openvino_env/bin/activate
    ```
    
    From the Python virtual environment `(openvino_env) .. $` 
    ```
    python -m pip install --upgrade pip
    pip install openvino-dev[caffe]==2022.3.0
    omz_downloader --name mobilenet-ssd
    omz_converter --name mobilenet-ssd --precision FP32 --download_dir . --output_dir .
    deactivate
    ```

    mobilenet-ssd IR model will be generated in `./public/mobilenet-ssd/FP32` 
    ```
    mobilenet-ssd.bin  mobilenet-ssd.xml  mobilenet-ssd.mapping
    ```


4. Set up oneAPI and OpenVINO environment:

   If the installation directories are `"/opt/intel/oneapi"`, and `"/opt/intel/openvino_2022"`
    ```
    source /opt/intel/oneapi/setvars.sh
    source /opt/intel/openvino_2022/setupvars.sh
    ```
 

5. Build the program:

    ```
    mkdir build && cd build
    cmake .. && cmake --build . --config release
    ```


6. Run the program:

   Please set the proper path for the content and OpenVINO IR model you prepared

    ```
    ./vpl-infer -hw -i cars_320x240.h265 -m mobilenet-ssd.xml
    ```


### Using a Docker Container (Linux)

1. Check groups of current user (Let's say your username is `user1`):

    ```
    groups
    user1 adm sudo render docker
    ```
    User1 should be in `render` group.
    If you don't see `render` from `groups` command, then you should add user1 to `render` group:
    ```
    usermod -a -G render user1
    newgrp render
    ```
    This sample is using Intel® DL Streamer docker image as base. (https://hub.docker.com/r/intel/dlstreamer)


2. Set the target base image by platform:

    #### For Intel Gen12/DG1 GPUs (ex: TGL)
    ```
    DLS_IMAGE=intel/dlstreamer:2022.2.0-ubuntu20-gpu815-devel
    ```
    #### For Intel Data Center GPU Flex Series (ex: DG2)
    ```
    DLS_IMAGE=intel/dlstreamer:2022.2.0-ubuntu20-gpu419.40-devel
    ```


3. Go to vpl-infer/docker and build docker image:

    ```
    docker build -t onevpl_openvino --build-arg BASE_IMAGE=$DLS_IMAGE .
    ```
    If there’re proxy servers, then you might need to pass that information with using “--build-arg”:
    ```
    docker build -t onevpl_openvino $(env | grep -E '(_proxy=|_PROXY)' | sed 's/^/--build-arg /') --build-arg BASE_IMAGE=$DLS_IMAGE .
    ```


4. Start the container, mounting the examples directory:

    Read render group id (it might be different by system configuration)

    ```
    stat -c "%g" /dev/dri/render*
    109
    ```
    ```
    docker run -u 0 -it -v `pwd`/../../../../examples:/oneVPL/examples --device /dev/dri --group-add 109 onevpl_openvino
    ```
    `"groups: cannot find name for group ID 109"` message is expected.


5. Build and run the program in the container:

    ```
    source /opt/intel/oneapi/setvars.sh

    cd /oneVPL/examples/interop/vpl-infer
    mkdir build && cd build
    cmake .. && cmake --build . --config release
    ./vpl-infer -hw -i /oneVPL/examples/content/cars_320x240.h265 -m /oneVPL/nm/mobilenet-ssd/FP32/mobilenet-ssd.xml
    ```


### On a Windows* System

1. Install the prerequisite software:

   - Intel® oneAPI Base Toolkit for Windows*
   - Intel® OpenVINO for Windows*
   - [Python](http://python.org)
   - [CMake](https://cmake.org)


2. Download the Mobilenet-ssd object detection model from OpenVINO model Zoo and covert it to IR model:

    Start Python virtual environments from Command Prompt
    ```
    python -m venv openvino_env
    openvino_env\Scripts\activate
    ```
    
    From the Python virtual environment `(openvino_env) .. >`
    ```
    python -m pip install --upgrade pip
    pip install openvino-dev[caffe]==2022.3.0
    omz_downloader --name mobilenet-ssd
    omz_converter --name mobilenet-ssd --precision FP32 --download_dir . --output_dir .
    deactivate
    ```

    mobilenet-ssd IR model will be generated in `.\public\mobilenet-ssd\FP32` 
    ```
    mobilenet-ssd.bin  mobilenet-ssd.xml  mobilenet-ssd.mapping
    ```


3. Set up oneAPI and OpenVINO environment:

   If the installation directories are `"c:\Program Files (x86)\intel\oneAPI"`, and `"c:\Program Files (x86)\intel\OpenVINO"`

    ```
    "c:\Program Files (x86)\intel\oneAPI\setvars.bat"
    "c:\Program Files (x86)\intel\OpenVINO\setupvars.bat"
    ```


4. Build the program:

    Go to `examples\interop\vpl-infer`
    ```
    mkdir build && cd build
    cmake .. && cmake --build . --config release
    ```


5. Run the program with defaults:

    Please set the proper path for the content and OpenVINO IR model you prepared
    ```
    cd release
    vpl-infer -hw -i cars_320x240.h265 -m mobilenet-ssd.xml
    ```


### Example of Output

This is the output from Linux, but the test result will be similar to Windows:

```
OpenVINO Runtime
    Version : 2022.3.0
    Build   : 2022.3.0-9052-9752fafe8eb-releases/2022/3

Loading network model files: /oneVPL/nm/mobilenet-ssd/FP32/mobilenet-ssd.xml
    Model name: MobileNet-SSD
    Inputs
        Input name: data
        Input type: f32
        Input shape: [1,3,300,300]
    Outputs
        Output name: detection_out
        Output type: f32
        Output shape: [1,1,100,7]

libva info: VA-API version 1.15.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_15
libva info: va_openDriver() returns 0
libva info: VA-API version 1.15.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_15
libva info: va_openDriver() returns 0

oneVPL Implementation details:
    ApiVersion:           2.7
    Implementation type:  HW
    AccelerationMode via: VAAPI
    Path: /usr/lib/x86_64-linux-gnu/libmfx-gen.so.1.2.7

libva info: VA-API version 1.15.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_15
libva info: va_openDriver() returns 0
Decoding VPP, and inferring /oneVPL/examples/content/cars_320x240.h265 with /oneVPL/nm/mobilenet-ssd/FP32/mobilenet-ssd.xml
Result:
    Class ID (7),  BBox (  92,  112,  201,  217),  Confidence (0.999)
    Class ID (7),  BBox ( 207,   50,  296,  144),  Confidence (0.997)
    Class ID (7),  BBox (  35,   43,  120,  134),  Confidence (0.995)
    Class ID (7),  BBox (  74,   81,  167,  171),  Confidence (0.933)
    Class ID (7),  BBox ( 168,  200,  274,  238),  Confidence (0.609)

  ...

Result:
    Class ID (7),  BBox (  64,   68,  161,  178),  Confidence (0.998)
    Class ID (7),  BBox ( 115,  131,  229,  239),  Confidence (0.919)
    Class ID (7),  BBox ( 266,   81,  319,  190),  Confidence (0.843)
    Class ID (7),  BBox (  17,   44,   71,   94),  Confidence (0.749)

Decoded 30 frames and detected objects

```
This execution is the object detection use case with `mobilenet-ssd` network model.

`Class ID` is predicted class ID (1..20 - PASCAL VOC defined class ids).

Mapping to class names provided by `<omz_dir>/data/dataset_classes/voc_20cl_bkgr.txt` file, which is downloaded when you install OpenVINO development version.

`7` is `car` from the list.