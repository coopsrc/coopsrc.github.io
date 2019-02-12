---
title: 使用Android Studio开发FFmpeg的正确姿势
date: 2019-02-12 16:38:37
tags:
    - Android
    - FFMpeg
---

2018-04-21: 更新至 ffmpeg-4.0
2018-11-16: 更新腳本

>  使用AndroidStudio 开发 FFmpeg
> Keywords: gradle, cmake
> 关键步骤，编译FFmpeg，Android Studio 集成。
> 
> abi support: `armeabi-v7a` `arm64-v8a` `x86` `x86_64`
> ndk version android-ndk-r14b
>
> export NDK_HOME=/opt/android/android-ndk-r14b
> export HOST_PLATFORM=linux-x86_64
>
<!-- more -->
# 第一步，编译ffmpeg
---
#### 首先下载并解压
```
wget https://ffmpeg.org/releases/ffmpeg-4.0.tar.bz2
tar xvf ffmpeg-4.0.tar.bz2
```
#### 然后编写编译脚本
``` shell
#!/bin/sh

PREFIX=android-build
HOST_PLATFORM=linux-x86_64

COMMON_OPTIONS="\
	--target-os=android \
	--disable-static \
	--enable-shared \
	--enable-small \
	--disable-programs \
	--disable-ffmpeg \
	--disable-ffplay \
	--disable-ffprobe \
	--disable-doc \
	--disable-symver \
	--disable-asm \
	--enable-decoder=vorbis \
	--enable-decoder=opus \
	--enable-decoder=flac 
	"

build_all(){
	for version in armeabi-v7a arm64-v8a x86 x86_64; do
		echo "======== > Start build $version"
		case ${version} in
		armeabi-v7a )
			ARCH="arm"
			CPU="armv7-a"
			CROSS_PREFIX="$NDK_HOME/toolchains/arm-linux-androideabi-4.9/prebuilt/$HOST_PLATFORM/bin/arm-linux-androideabi-"
			SYSROOT="$NDK_HOME/platforms/android-21/arch-arm/"
			EXTRA_CFLAGS="-march=armv7-a -mfpu=neon -mfloat-abi=softfp -mvectorize-with-neon-quad"
			EXTRA_LDFLAGS="-Wl,--fix-cortex-a8"
		;;
		arm64-v8a )
			ARCH="aarch64"
			CPU="armv8-a"
			CROSS_PREFIX="$NDK_HOME/toolchains/aarch64-linux-android-4.9/prebuilt/$HOST_PLATFORM/bin/aarch64-linux-android-"
			SYSROOT="$NDK_HOME/platforms/android-21/arch-arm64/"
			EXTRA_CFLAGS=""
			EXTRA_LDFLAGS=""
		;;
		x86 )
			ARCH="x86"
			CPU="i686"
			CROSS_PREFIX="$NDK_HOME/toolchains/x86-4.9/prebuilt/$HOST_PLATFORM/bin/i686-linux-android-"
			SYSROOT="$NDK_HOME/platforms/android-21/arch-x86/"
			EXTRA_CFLAGS=""
			EXTRA_LDFLAGS=""
		;;
		x86_64 )
			ARCH="x86_64"
			CPU="x86_64"
			CROSS_PREFIX="$NDK_HOME/toolchains/x86_64-4.9/prebuilt/$HOST_PLATFORM/bin/x86_64-linux-android-"
			SYSROOT="$NDK_HOME/platforms/android-21/arch-x86_64/"
			EXTRA_CFLAGS=""
			EXTRA_LDFLAGS=""
		;;
		esac

		echo "-------- > Start clean workspace"
		make clean

		echo "-------- > Start config makefile"
		configuration="\
            --prefix=${PREFIX} \
            --libdir=${PREFIX}/libs/${version}
            --incdir=${PREFIX}/includes/${version} \
            --pkgconfigdir=${PREFIX}/pkgconfig/${version} \
            --arch=${ARCH} \
            --cpu=${CPU} \
            --cross-prefix=${CROSS_PREFIX} \
            --sysroot=${SYSROOT} \
            --extra-ldexeflags=-pie \
            ${COMMON_OPTIONS}
		    "

		echo "-------- > Start config makefile with ${configuration}"
		./configure ${configuration}

		echo "-------- > Start make ${version} with -j8"
		make j8

		echo "-------- > Start install ${version}"
		make install
		echo "++++++++ > make and install ${version} complete."

	done
}

echo "-------- Start --------"
build_all
echo "-------- End --------"

```
此脚本实现了`armeabi-v7a`,`arm64-v8a`,`x86`,`x86_64` 4个平台的编译。
- 需要添加系统环境变量 `$NDK_PATH`
- `--target-os=android`指定`android`平台。
- `make install-libs` 表示只安装`so`文件

编译完成结果：
![ffmpeg](/assets/blogImg/How-to-develop-ffmpeg-in-android-studio/ffmpeg.png)

# 第二步，项目集成
---
- 新建项目，增加`C++`支持。手动创建`jniLibs`文件夹
- 然后将上一步生成的所有文件复制到`jniLibs`文件夹下面
最终目录结构：

![Project](/assets/blogImg/How-to-develop-ffmpeg-in-android-studio/project.png)
然后修改`CMakeLists.txt`文件，集成`so`。
``` cmake
cmake_minimum_required(VERSION 3.4.1)

find_library(log-lib log)

add_library(native-lib
            SHARED
            src/main/cpp/native-lib.cpp )

set(JNI_LIBS_DIR ${CMAKE_SOURCE_DIR}/src/main/jniLibs)

add_library(avutil
            SHARED
            IMPORTED )
set_target_properties(avutil
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libavutil.so )

add_library(swresample
            SHARED
            IMPORTED )
set_target_properties(swresample
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libswresample.so )

add_library(swscale
            SHARED
            IMPORTED )
set_target_properties(swscale
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libswscale.so )

add_library(avcodec
            SHARED
            IMPORTED )
set_target_properties(avcodec
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libavcodec.so )

add_library(avformat
            SHARED
            IMPORTED )
set_target_properties(avformat
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libavformat.so )

add_library(avfilter
            SHARED
            IMPORTED )
set_target_properties(avfilter
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libavfilter.so )

add_library(avdevice
            SHARED
            IMPORTED )
set_target_properties(avdevice
                      PROPERTIES IMPORTED_LOCATION
                      ${JNI_LIBS_DIR}/${ANDROID_ABI}/libavdevice.so )

include_directories(${JNI_LIBS_DIR}/includes)

target_link_libraries(native-lib
                      avutil swresample swscale avcodec avformat avfilter avdevice
                      ${log-lib} )
```

简要说明：
- `${ANDROID_ABI}`表示目标ABI，在官方文档中有说明：https://developer.android.com/ndk/guides/cmake.html

最后再放出效果图：
![Preview](/assets/blogImg/How-to-develop-ffmpeg-in-android-studio/preview.png)

示例代码：https://github.com/coopsrc/FFPlayerDemo