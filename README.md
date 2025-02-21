# GMFSS ncnn ~~Vulkan~~

![CI](https://github.com/nihui/rife-ncnn-vulkan/workflows/CI/badge.svg)
![download](https://img.shields.io/github/downloads/nihui/rife-ncnn-vulkan/total.svg)

ncnn implementation of GMFSS.

gmfss-ncnn-vulkan uses [ncnn project](https://github.com/Tencent/ncnn) as the universal neural network inference framework.

## TODOs

Important Notes:

Currently, only 576p(960x576) is supported as input. For other input resolution, you need to export ncnn.param and ncnn.bin on your own, referering  https://github.com/Justin62628/gmfss-ncnn-vulkan/blob/main/GMF2NCNN.ipynb

- [ ] Vulkan Acceleration
- [ ] Dynamic input resolution
- [ ] TTA
- [ ] Modern Distribution

## (TODO) [Download](https://github.com/nihui/rife-ncnn-vulkan/releases) 

~~Download Windows/Linux/MacOS Executable for Intel/AMD/Nvidia GPU~~

~~**https://github.com/nihui/rife-ncnn-vulkan/releases**~~

~~This package includes all the binaries and models required. It is portable, so no CUDA or PyTorch runtime environment is needed :)~~

## About GMFSS

https://github.com/98mxr/GMFSS_Fortuna.

A VFI algorithm for Anime

## Usages

Input two frame images, output one interpolated frame image.

### Currently, only CPU inference is supported:

```bash
rife-ncnn-vulkan.exe -0 images\0022.png -1 images\0023.png -o images\out.png -m models\gmfss -g -1 -j 1:16:1
```



### (TODO) Example Commands

```shell
./rife-ncnn-vulkan -m models/gmfss -0 0.jpg -1 1.jpg -o 01.jpg
./rife-ncnn-vulkan -m models/gmfss -i input_frames/ -o output_frames/
```

Example below runs on CPU, Discrete GPU, and Integrated GPU all at the same time. Uses 2 threads for image decoding, 4 threads for one CPU worker, 4 threads for another CPU worker, 2 threads for discrete GPU, 1 thread for integrated GPU, and 4 threads for image encoding.
```shell
./rife-ncnn-vulkan -m models/gmfss -i input_frames/ -o output_frames/ -g -1,-1,0,1 -j 2:4,4,2,1:4
```

### Video Interpolation with FFmpeg

```shell
mkdir input_frames
mkdir output_frames

# find the source fps and format with ffprobe, for example 24fps, AAC
ffprobe input.mp4

# extract audio
ffmpeg -i input.mp4 -vn -acodec copy audio.m4a

# decode all frames
ffmpeg -i input.mp4 input_frames/frame_%08d.png

# interpolate 2x frame count
./rife-ncnn-vulkan -m models/gmfss -i input_frames -o output_frames

# encode interpolated frames in 48fps with audio
ffmpeg -framerate 48 -i output_frames/%08d.png -i audio.m4a -c:a copy -crf 20 -c:v libx264 -pix_fmt yuv420p output.mp4
```

### Full Usages

```console
Usage: rife-ncnn-vulkan -0 infile -1 infile1 -o outfile [options]...
       rife-ncnn-vulkan -i indir -o outdir [options]...

  -h                   show this help
  -v                   verbose output
  -0 input0-path       input image0 path (jpg/png/webp)
  -1 input1-path       input image1 path (jpg/png/webp)
  -i input-path        input image directory (jpg/png/webp)
  -o output-path       output image path (jpg/png/webp) or directory
  -n num-frame         target frame count (default=N*2)
  -s time-step         time step (0~1, default=0.5)
  -m model-path        rife model path (default=rife-v2.3)
  -g gpu-id            gpu device to use (-1=cpu, default=auto) can be 0,1,2 for multi-gpu
  -j load:proc:save    thread count for load/proc/save (default=1:2:2) can be 1:2,2,2:2 for multi-gpu
  -x                   enable spatial tta mode
  -z                   enable temporal tta mode
  -u                   enable UHD mode
  -f pattern-format    output image filename pattern format (%08d.jpg/png/webp, default=ext/%08d.png)
```

- `input0-path`, `input1-path` and `output-path` accept file path
- `input-path` and `output-path` accept file directory
- `num-frame` = target frame count
- `time-step` = interpolation time
- `load:proc:save` = thread count for the three stages (image decoding + rife interpolation + image encoding), using larger values may increase GPU usage and consume more GPU memory. You can tune this configuration with "4:4:4" for many small-size images, and "2:2:2" for large-size images. The default setting usually works fine for most situations. If you find that your GPU is hungry, try increasing thread count to achieve faster processing.
- `pattern-format` = the filename pattern and format of the image to be output, png is better supported, however webp generally yields smaller file sizes, both are losslessly encoded

If you encounter a crash or error, try upgrading your GPU driver:

- Intel: https://downloadcenter.intel.com/product/80939/Graphics-Drivers
- AMD: https://www.amd.com/en/support
- NVIDIA: https://www.nvidia.com/Download/index.aspx

## Build from Source

1. Download and setup the Vulkan SDK from https://vulkan.lunarg.com/
  - For Linux distributions, you can either get the essential build requirements from package manager
```shell
dnf install vulkan-headers vulkan-loader-devel
```
```shell
apt-get install libvulkan-dev
```
```shell
pacman -S vulkan-headers vulkan-icd-loader
```

2. Clone this project with all submodules

```shell
git clone https://github.com/Justin62628/gmfss-ncnn-vulkan.git
cd gmfss-ncnn-vulkan/src
git clone https://github.com/Tencent/ncnn.git
cd ..
git submodule update --init --recursive
```

3. Build with Visual Studio(CMake)

> Open the folder where gmfss-ncnn-vulkan is located
> 
> Select src/CMakeLists in Visual Studio
> 
> Click Generate - Generate All

### (TODO) Model

| model | upstream version |
|---|---|
| gmfss | ?                |

## Sample Images

### Original Image

![origin0](images/0022.png)
![origin1](images/0023.png)

### Interpolate with gmfss ? model

```shell
rife-ncnn-vulkan.exe -m models/gmfss -0 images/0022.png -1 images/0023.png -o images/out.png
```

![rife-4.6](images/out_rife4.6.png)
![gmfss](images/out_cpu.png)


### (TODO) Interpolate with gmfss ? model + TTA-s

```shell
rife-ncnn-vulkan.exe -m modelsgmfss -x -0 images/0022.png -1 images/0023.png -o images/out.png
```

## Other Open-Source Code Used

- https://github.com/Tencent/ncnn for fast neural network inference on ALL PLATFORMS
- https://github.com/webmproject/libwebp for encoding and decoding Webp images on ALL PLATFORMS
- https://github.com/nothings/stb for decoding and encoding image on Linux / MacOS
- https://github.com/tronkko/dirent for listing files in directory on Windows



# Install

```bash
# pnnx
cmake -DCMAKE_INSTALL_PREFIX=install -DTorch_INSTALL_DIR=D:/Program/Python37/Lib/site-packages/torch -DPY_VERSION=3.7 ..

```

