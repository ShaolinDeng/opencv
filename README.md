## OpenCV Cuda IO

### 简介

主要IO for CUDA VideoReader and VideoWriter

本文先针对VideoReader 解析文件与RTMP流  GpuMat数据 直接送往TensorRT 进行运算 推理。

### 依赖
* ffmpeg 4.4.3 增加cuda 编解码 与nvenc nvdec   nvheader
* cuda 11.7 对应的驱动--"NVIDIA-SMI 535.98"
* cudnn 8.6.0
* tensorrt 8.6.1
* Video_Codec_SDK_11.1.5
* opencv 4.5.5

### build

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/ffmpeg/lib/
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/ffmpeg/lib/pkgconfig/
export PKG_CONFIG_LIBDIR=$PKG_CONFIG_LIBDIR:/usr/local/ffmpeg/lib/


-DCMAKE_INSTALL_PREFIX=./build/

-DBUILD_LIST=calib3d,videoio,ts  一般默认

-DCMAKE_BUILD_TYPE=Debug

-DBUILD_SHARED_LIBS=OFF


```shell
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=./opencv117install -D WITH_CUDA=ON -D WITH_NVCUVID=ON -D WITH_NVCUVENC=ON -D WITH_CUDNN=ON -D ENABLE_FAST_MATH=1 -D WITH_CUBLAS=ON -D WITH_TBB=ON -D OPENCV_DNN_CUDA=ON  -D OPENCV_ENABLE_NONFREE=ON -D CUDA_ARCH_BIN="6.1 7.5 8.6"  -D CUDNN_LIBRARY=/usr/local/cuda/lib64/libcudnn.so  -D CUDNN_INCLUDE_DIR=/usr/local/cuda/include -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules  -D BUILD_opencv_python2=OFF  -D BUILD_opencv_python3=ON -D OPENCV_PYTHON3_INSTALL_PATH=/usr/local/lib/python3.8/site-packages ../opencv

```

### install

lib
安装在prefix中


python3
```
python3 -c "import cv2; print(cv2.__version__)"


```


### 测试验证


#### 测试文件

 CUDA_VISIBLE_DEVICES=4 LD_LIBRARY_PATH=lib/:boost171/lib:tensorrt/lib:log4cxx/lib:hiredis/lib:opencv455/lib:/usr/local/ffmpeg/lib:/usr/lib/x86_64-linux-gnu:/home/shaolin/Disk_C/dsl/shaolin/mygithub/opencvbuild/opencv117install/lib  ./testopencv_cuda ~/apvideo/ap111111-1.mp4  0


```
shaolin@mg6:~/lb-ai-integration/ai_handle_cpp$ CUDA_VISIBLE_DEVICES=4 LD_LIBRARY_PATH=lib/:boost171/lib:tensorrt/lib:log4cxx/lib:hiredis/lib:opencv455/lib:/usr/local/ffmpeg/lib:/usr/lib/x86_64-linux-gnu:/home/shaolin/Disk_C/dsl/shaolin/mygithub/opencvbuild/opencv117install/lib  ./testopencv_cuda ~/apvideo/ap111111-1.mp4  0
[ERROR:0@3.736] global /home/shaolin/Disk_C/dsl/shaolin/mygithub/opencv_contrib/modules/cudacodec/src/ffmpeg_video_source.cpp (137) FFmpegVideoSource   malong log::: file or Stream: (/home/shaolin/apvideo/ap111111-1.mp4)====ML
[ERROR:0@3.758] global /home/shaolin/Disk_C/dsl/shaolin/mygithub/opencv_contrib/modules/cudacodec/src/ffmpeg_video_source.cpp (146) FFmpegVideoSource   malong log:get  format: ====ML
[ERROR:0@3.758] global /home/shaolin/Disk_C/dsl/shaolin/mygithub/opencv_contrib/modules/cudacodec/src/ffmpeg_video_source.cpp (163) FFmpegVideoSource   malong log::: file or Stream  info code:4  1920 X 1080   fps:29.998911    ====ML
 VideoReaderProps  israw: 0  extraDataIdx: 1   rawIdxBase: -1   number:0
 VideoReader Format info (4为H264  8为HEVC )   Codec: 4  (1 为 YUV420)  ChromaFormat:  1   1920 X 1080
 测试  帧数:301
```



#### 测试rtmp流


推流

ffmpeg -stream_loop  -1 -re -i ap111111-1.mp4   -c:v copy  -f flv rtmp://127.0.0.1/live/top

```
shaolin@mg6:~/apvideo$ ffmpeg -stream_loop  -1 -re -i ap111111-1.mp4   -c:v copy  -f flv rtmp://127.0.0.1/live/top
```

```
shaolin@mg6:~/lb-ai-integration/ai_handle_cpp$ CUDA_VISIBLE_DEVICES=4 LD_LIBRARY_PATH=lib/:boost171/lib:tensorrt/lib:log4cxx/lib:hiredis/lib:opencv455/lib:/usr/local/ffmpeg/lib:/usr/lib/x86_64-linux-gnu:/home/shaolin/Disk_C/dsl/shaolin/mygithub/opencvbuild/opencv117install/lib  ./testopencv_cuda "rtmp://127.0.0.1/live/top"   0

[ERROR:0@3.682] global /home/shaolin/Disk_C/dsl/shaolin/mygithub/opencv_contrib/modules/cudacodec/src/ffmpeg_video_source.cpp (137) FFmpegVideoSource   malong log::: file or Stream: (rtmp://127.0.0.1/live/top)====ML
[ERROR:0@4.760] global /home/shaolin/Disk_C/dsl/shaolin/mygithub/opencv_contrib/modules/cudacodec/src/ffmpeg_video_source.cpp (146) FFmpegVideoSource   malong log:get  format: ====ML
[ERROR:0@4.760] global /home/shaolin/Disk_C/dsl/shaolin/mygithub/opencv_contrib/modules/cudacodec/src/ffmpeg_video_source.cpp (163) FFmpegVideoSource   malong log::: file or Stream  info code:4  1920 X 1080   fps:30.000000    ====ML
 VideoReaderProps  israw: 0  extraDataIdx: 1   rawIdxBase: -1   number:0
 VideoReader Format info (4为H264  8为HEVC )   Codec: 4  (1 为 YUV420)  ChromaFormat:  1   1920 X 1080
 测试  帧数:301

```