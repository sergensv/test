# Gaussian Filter #

Example demonstrates working of **`xf::GaussianBlur()`** and **`xf::resize()`** functions of xfOpenCV library. Example designed to process one image once. If you would like to process many images in loop you need to extract from kernel interface wrapper FPGA & kernel initialization and finalization operation and move them to host application before and after processing loop respectively.

## Code structure ##

![](./../Code_Structure.png)

| Component | Source files |
| :--- | :--- |
| *Kernel&nbsp;Configuration*          |**`xf_gaussian_filter_config.h`**<br/>**`xf_config_params.h`**|
| *Host&nbsp;Application*              |**`xf_gaussian_filter_tb.cpp`**|
| *Kernel&nbsp;Interface&nbsp;Wrapper* |**`xf_gaussian_filter_accel_aws.cpp`**|
| *Kernel&nbsp;Driver*                 |**`xcl2.cpp (in SDx library)`**|
| *Kernel*                             |**`xf_gaussian_filter_kernel_aws.cpp`**|

## Kernel Configuration #

Following constants in header files define kernel configuration

| Constant | Possible values | Default Value | Description |
|
| **`FILTER_SIZE_3`**<br/>**`FILTER_SIZE_5`**<br/>**`FILTER_SIZE_7`**|**`0, 1`**| **`1`**<br/>**`0`**<br/>**`0`**| Select window size of the Gaussian filter. One of them should be defined as 1. And only one can be defined as 1 - others should be defined as 0 |
| **`FILTER_WIDTH`**            |||The window size of the Gaussian filter. Value set automatically depending on which **`FILTER_SIZE_X`** set to 1. 
| **`SIGMA`**                   |||Standard deviation of of Gaussian Filter. Value set automatically depending on which **`FILTER_SIZE_X`** set to 1.|
| **`NPC1`**                    |**`XF_NPPC1`**<br/>**`XF_NPPC8`**|**`XF_NPPC1`**|Select level of parallelism in kernel (number of pixels which kernel process per clock cycle).|
| **`XF_RESIZE_INTERPOLATION`** |**`###########`**|**`XF_INTERPOLATION_NN`**||
| **`CV_RESIZE_INTERPOLATION`** |**`###########`**|**`cv::INTER_NEAREST`**||
| **`XF_GAUSSIAN_BORDER`**      |**`###########`**|**`XF_BORDER_CONSTANT`**||
| **`CV_GAUSSIAN_BORDER`**      |**`###########`**|**`cv::BORDER_CONSTANT`**||
| **`COLS_INP`**                |**`multiple of 8`**|**`1920`**|Maximum width  of input image|
| **`ROWS_INP`**                |**`multiple of 8`**|**`1080`**|Maximum height of input image|
| **`SCALE`**                   |**`> 0 and !=1`**|**`0.5`**|Define scale factor of image after Gaussian Filter.<br/>**Note: The **`xf::resize()`** doesn't support scale factor 1.**  |
| **`COLS_OUT`**                |**`multiple of 8`**|**`COLS_INP/2`**|Maximum width of output image. Please keep value to correspond to the scale factor (**`SCALE`**). Value should be **`>= ceil(COLS_INP * SCALE)`** and should be multiple of 8.|
| **`ROWS_OUT`**                |**`> 0`**|**`ROWS_INP/2`**|Maximum height of input image. Please keep value to correspond to the scale factor (**`SCALE`**). Value should be **`>= ceil(ROWS_INP * SCALE)`**|

## Host Application ##
Host application reads test image from file, process it with help of regular OpenCV library on host, perform same processing with help of FPGA kernel with function from xfOpenCV library and compare result.

Input image of example is ***im0.jpg*** place in root folder of example. First filter applied to the image is **`xf::GaussianBlur()`**, next is **`xf::resize()`**. Both has analog with same name in OpenCV library. Application calculate difference between result images - images assumed equal if difference for each pixel not exceed 1. Result images will be stored into run folder. 

The following images will be in run folder after execution:

- ***xf_img_out.jpg*** - result of FPGA kernel processing
- ***cv_img_out.jpg*** - result of OpenCV processing
- ***error.png*** - contains difference of values for each pixel of result images


## Kernel Interface Wrapper ##

In conjunction with xfOpenCV library on host application is convenient to use xf::Mat or cv::Mat class and image manipulation functions. Unfortunately the XOCC kernel compiler doesn't support classes/structures as kernel input/output parameters. To pass xf::Mat to a kernel a wrapper is needed. The kernel interface wrapper convert interface convenient to host application to kernel interface available in Amazon F1 instance.

For this example kernel interface wrapper also perform FPGA initialization, kernel downloading, initialization and finalization.


| Parameter&nbsp;Name |Direction|Type | Description |
|
| **`img_inp`** |Input  | **`xf::Mat<XF_8UC1, ROWS_INP, COLS_INP, NPC1> &`** | Input image |
| **`img_out`** |Output | **`xf::Mat<XF_8UC1, ROWS_OUT, COLS_OUT, NPC1> &`** | Output image |
| **`sigma`**   | Input | **`float`**                                      | Standard deviation of of Gaussian Filter |

To forward these parameters to kernel wrapper create 2 buffers in global memory for images data. Wrapper decompose **`img_inp`** and **`img_out`** classes and pass member separately.


## Kernel Driver ###

Example use modification of SDx xcl kernel driver v.2 for Amazon F1 instance. Source code of this driver and description could be found in Amazon aws-fpga framework.

## Kernel ##

During synthesis for FPGA kernel's parameters should be mapped to HW interfaces supported on Amazon F1 instance. To map kernel parameters **`HLS INTERFACE`** pragma should be used. Supported following interfaces: **`m_axi`** and **`s_axilite`**. For **`m_axi`** offset can be set through **`s_axilite`** port only.

Because functions from xfOpenCV library operate with **`xf::Mat`** class as image container kernel's parameters should be packed back to variables of this class. To do this you need following: 

- Declare **`xf::Mat`** variable <br/> ***Note: due to XOCC issues use default constructor only - do not try initialize class members with help of non-default constructors***
- Assign image size to **`rows`** and **`cols`** member
- Copy image from input buffer to **`data`** member of **`xf::Mat`** or from **`data`** to output buffer

dfgdfgdfgdfgdg

```cpp
xf::Mat<XF_8UC1, ROWS_INP, COLS_INP, NPC1> mi;
mi.rows = rows_inp;
  mi.cols = cols_inp;
```


```
project_data
 ├─ aws-fpga
 └─ xfopencv
```



has parameters of basic types: pointers to buffers with images, int

Kernel pipeline **`xf::GaussianBlur()`** and **`xf::resize()`**. Both function accept as input and output parameters **`xf::Mat<>`** class. Because kernel has parameters of basic type he should construct from them variables of **`xf::Mat<>`** class. 


Common makefile contains following variables and makefile's targets for host application and kernel. 





### Variables for host part ###
 

### Variables for kernel part ###

| Variable&nbsp;Name |Default value | Description |
| :------------- | :------------- |
| **`XOCC_OPTS`**         |-                              | Contains XOCC options. Please see default value in [`common_makefile`](../common_makefile) |
| **`XOCC_INCL`**         |-                              | Contains paths to search header files. Please see default value in [`common_makefile`](../common_makefile) |   
| **`KERNEL_BLD_DIR`**    |**`$(TARGET)/build/kernel`**   | Build folder for kernel build artifacts|
| **`KERNEL_RUN_DIR`**    |-                              | Folder to store kernel binary (`.xclbin`). Default value depends on target flow. Please see default value in [`common_makefile`](../common_makefile)|


### Makefile targets ###

| Target&nbsp;label | Description |
| :------------- | :------------- |
| **`all`**      | Build host application and kernel for target flow specified by **`$(TARGET)`** variable |
| **`host`**     | Build host application only for target flow specified by **`$(TARGET)`** variable |   
| **`krnl`**     | Build kernel only for target flow specified by **`$(TARGET)`** variable |   
| **`clean`**    | Clean build artifacts of target flow specified by **`$(TARGET)`** variable. <br/>***Note: afi folder of FPGA flow ($(TARGET) == hw) kept untouched. You should clean it manually if needed *** |   



## REVISION HISTORY

Date      | Readme Version | Release Notes
--------  |----------------|-------------------------
May 2018  | 1.0            | Initial version.
