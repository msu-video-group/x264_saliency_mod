## Copyright (C) 2014 x264 project
##
## Authors: Steve Borho <steve at borho.org>
##          Dnyaneshwar G <dnyaneshwar at multicorewareinc.com>
##
## This cmake script will generate VisualStudio 2013 solution and project
## files for x264. Out-of-tree builds are suggested:
##   mkdir build && cd build
##   cmake -G "Visual Studio 12 Win64" ..
##       or
##   cmake -G "Visual Studio 12" ..
##   start x264.sln
##
## This script generates config.h and x264_config.h files which work
## on Windows with VisualStudio.

cmake_minimum_required(VERSION 2.8.11) # MSVC 2013 requires cmake 2.8.11
project (x264)

option(HIGH_BIT_DEPTH "Store pixels as 16bit values" OFF)
option(X264_BUILD_CLI "Build standalone application" ON)
option(X264_HAVE_GPL "Allow linkage with GPL libraries, disable if commercial licensee" ON)
option(X264_HAVE_AVS "Allow Avisynth" ON)
option(X264_HAVE_FFMS "Allow FFMS" OFF)
option(X264_HAVE_OPENCL "Allow OpenCL" OFF)

# Compiler verification
if(NOT MSVC12)
    message(FATAL_ERROR "MSVC2013 (VC12) is required to build x264 (C99)")
endif()

add_definitions(/MP) # multithreaded build
add_definitions(/Oi) # enable SIMD intrinsics 

add_definitions(-DHAVE_CONFIG_H=1)
file(WRITE config.h
    "#define SYS_WINDOWS 1\n"
    "#define HAVE_WIN32THREAD 1\n"
    "#define HAVE_THREAD 1\n"
    "#define USE_AVXSYNTH 0\n"
    "#define HAVE_VECTOREXT 1\n"
    "#define HAVE_INTERLACED 1\n"
    "#define HAVE_MALLOC_H 0\n"
    "#define HAVE_ALTIVEC 0\n"
    "#define HAVE_ALTIVEC_H 0\n"
    "#define HAVE_ARMV6 0\n"
    "#define HAVE_ARMV6T2 0\n"
    "#define HAVE_NEON 0\n"
    "#define HAVE_BEOSTHREAD 0\n"
    "#define HAVE_POSIXTHREAD 0\n"
    "#define HAVE_GPAC 0\n"
    "#define HAVE_CPU_COUNT 0\n"
    "#define HAVE_THP 0\n"
    "#define HAVE_LSMASH 0\n"
    "#define HAVE_STRING_H 1\n")

# VC headers cannot deal with fseek being redefined via macro to _fseeki64
# It causes two conflicting funcdefs from stdio.h. x264 should probably use X264_FSEEK here
#   "#define fseek _fseeki64\n"
#   "#define ftell _ftelli64\n"

file(WRITE x264_config.h
    "#define X264_INTERLACED 1\n"
    "#define X264_CHROMA_FORMAT 0\n"
    "#define X264_VERSION \"\"\n"
    "#define X264_POINTVER \"\"\n")

if("${CMAKE_SIZEOF_VOID_P}" MATCHES 8)
    set(X64 1)
    add_definitions(/wd4267) # 'function' : conversion from 'size_t' to 'int', possible loss of data
    file(APPEND config.h "#define ARCH_X86_64 1\n")
else()
    file(APPEND config.h "#define ARCH_X86 1\n")
endif()
if(HIGH_BIT_DEPTH)
    file(APPEND x264_config.h "#define X264_BIT_DEPTH 10\n")
    file(APPEND config.h      "#define HIGH_BIT_DEPTH 1\n")
else(HIGH_BIT_DEPTH)
    file(APPEND x264_config.h "#define X264_BIT_DEPTH 8\n")
    file(APPEND config.h      "#define HIGH_BIT_DEPTH 0\n")
endif(HIGH_BIT_DEPTH)
if(X264_HAVE_GPL)
    file(APPEND x264_config.h "#define X264_HAVE_GPL 1\n")
    file(APPEND config.h      "#define HAVE_GPL 1\n")
else(X264_HAVE_GPL)
    file(APPEND x264_config.h "#define X264_HAVE_GPL 0\n")
    file(APPEND config.h      "#define HAVE_GPL 0\n")
endif(X264_HAVE_GPL)
if(X264_HAVE_OPENCL)
    file(APPEND config.h      "#define HAVE_OPENCL 1\n")
else(X264_HAVE_OPENCL)
    file(APPEND config.h      "#define HAVE_OPENCL 0\n")
endif(X264_HAVE_OPENCL)

file(GLOB InputFiles
    input/input.c input/input.h 
    input/raw.c 
    input/thread.c 
    input/timecode.c 
    input/y4m.c)
    
file(GLOB OutputFiles
    output/matroska_ebml.c output/matroska_ebml.h
    output/flv.c 
    output/flv_bytestream.c
    output/matroska.c
    output/output.c
    output/raw.c)
    
file(GLOB EncoderFiles
    encoder/analyse.c encoder/analyse.h
    encoder/me.c encoder/me.h
    encoder/ratecontrol.c encoder/ratecontrol.h
    encoder/set.c encoder/set.h
    encoder/macroblock.c encoder/macroblock.h
    encoder/cabac.c
    encoder/cavlc.c
    encoder/encoder.c
    encoder/lookahead.c)
    
file(GLOB FilterFiles
    filters/filters.c filters/filters.h 
    filters/video/cache.c 
    filters/video/crop.c 
    filters/video/depth.c 
    filters/video/fix_vfr_pts.c 
    filters/video/internal.c 
    filters/video/resize.c 
    filters/video/select_every.c 
    filters/video/source.c 
    filters/video/video.c
    filters/video/internal.h 
    filters/video/video.h)

file(GLOB CommonFiles
    common/threadpool.c common/threadpool.h
    common/win32thread.c common/win32thread.h
    common/macroblock.c common/macroblock.h
    common/rectangle.c common/rectangle.h
    common/bitstream.c common/bitstream.h
    common/predict.c common/predict.h
    common/common.c common/common.h
    common/cabac.c common/cabac.h
    common/frame.c common/frame.h
    common/osdep.c common/osdep.h
    common/pixel.c common/pixel.h
    common/quant.c common/quant.h
    common/set.c common/set.h
    common/cpu.c common/cpu.h
    common/dct.c common/dct.h
    common/mc.c common/mc.h
    common/vlc.c
    common/deblock.c
    common/mvpred.c
    x264.h)

if (X264_HAVE_AVS)
    file(APPEND config.h    "#define HAVE_AVS 1\n")
    list(APPEND InputFiles input/avs.c)
else(X264_HAVE_AVS)
    file(APPEND config.h    "#define HAVE_AVS 0\n")
endif(X264_HAVE_AVS)

if (X264_HAVE_FFMS)
    file(APPEND config.h    "#define HAVE_FFMS 1\n")
    list(APPEND InputFiles  input/ffms.c)
    
    #As we link ffmpeg libraries, we link libav libs also
    file(APPEND config.h    "#define HAVE_SWSCALE 1\n")
    file(APPEND config.h    "#define HAVE_LAVF 1\n")
    list(APPEND InputFiles  input/lavf.c)
    
    set(FFMPEG_INCLUDE_DIRS "ffmpeg/include"    CACHE PATH "Path to directory with ffmpeg include files")
    set(FFMPEG_LIBS_DIRS    "ffmpeg/libs"       CACHE PATH "Path to directory with ffmpeg static .lib files")
    set(FFMS2_INCLUDE_DIRS  "ffms2/include"     CACHE PATH "Path to directory with FFmpegSource2 include files")
    set(FFMS2_LIBS_DIRS     "ffms2/x64/"        CACHE PATH "Path to directory with FFmpegSource2 static .lib files")
else(X264_HAVE_FFMS)
    file(APPEND config.h    "#define HAVE_FFMS 0\n")
    file(APPEND config.h    "#define HAVE_SWSCALE 0\n")
    file(APPEND config.h    "#define HAVE_LAVF 0\n")
endif(X264_HAVE_FFMS)

set(ASM_SRCS cabac-a.asm const-a.asm cpu-a.asm pixel-a.asm
    dct-a.asm deblock-a.asm mc-a.asm mc-a2.asm bitstream-a.asm
    predict-a.asm quant-a.asm)
if(HIGH_BIT_DEPTH)
    list(APPEND ASM_SRCS sad16-a.asm)
else()
    list(APPEND ASM_SRCS sad-a.asm)
endif()
if(X64)
    list(APPEND ASM_SRCS dct-64.asm trellis-64.asm)
else()
    list(APPEND ASM_SRCS dct-32.asm pixel-32.asm)
endif()
file(GLOB ASM_C common/x86/*.h common/x86/*.c)

# Custom YASM rules since cmake + MSVC + yasm is generally busted
# http://www.cmake.org/Bug/print_bug_page.php?bug_id=8170
find_program(YASM_EXECUTABLE 
    NAMES yasm yasm-1.2.0-win32 yasm-1.2.0-win64
    HINTS $ENV{YASM_ROOT} ${YASM_ROOT}
    PATH_SUFFIXES bin
)
if(YASM_EXECUTABLE)
    execute_process(COMMAND ${YASM_EXECUTABLE} --version
        OUTPUT_VARIABLE yasm_version
        ERROR_QUIET
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
    if(yasm_version MATCHES "^yasm ([0-9\\.]*)")
        set(YASM_VERSION_STRING "${CMAKE_MATCH_1}")
    endif()
    if (YASM_VERSION_STRING VERSION_LESS "1.2.0")
        message(STATUS "Yasm version ${YASM_VERSION_STRING} is too old. 1.2.0 or later required")
    else()
        message(STATUS "Found Yasm ${YASM_VERSION_STRING} to build assembly primitives")
        file(APPEND config.h "#define HAVE_MMX 1\n")

        if (X64)
            set(FLAGS -f win64 -m amd64 -DARCH_X86_64=1 -DHAVE_ALIGNED_STACK=0)
        else()
            set(FLAGS -f win32 -DARCH_X86_64=0 -DHAVE_ALIGNED_STACK=0 -DPREFIX)
        endif()
        if (HIGH_BIT_DEPTH)
            set(FLAGS ${FLAGS} -DHIGH_BIT_DEPTH=1 -DBIT_DEPTH=10)
        else()
            set(FLAGS ${FLAGS} -DHIGH_BIT_DEPTH=0 -DBIT_DEPTH=8)
        endif()
        foreach(ASM ${ASM_SRCS})
            set(YASM_SRC ${CMAKE_CURRENT_SOURCE_DIR}/common/x86/${ASM})
            set(YASM_SRCS ${YASM_SRCS} ${YASM_SRC})
            set(YASM_OBJ asm/${ASM}.obj)
            set(YASM_OBJS ${YASM_OBJS} ${YASM_OBJ})
            add_custom_command(
                OUTPUT ${YASM_OBJ}
                COMMAND ${YASM_EXECUTABLE} ARGS ${FLAGS} ${YASM_SRC} -o ${YASM_OBJ}
                DEPENDS ${YASM_SRC})
        endforeach()
        source_group(Assembly FILES ${ASM_C} ${YASM_SRCS})
    endif()
endif()

add_definitions("/wd4003") # common\pixel.c(497): warning C4003: not enough actual parameters for macro 'SATD_X_DECL7', 'SATD_X_DECL6', 'INIT_ADS'
add_definitions("/wd4018") # >= signed/unsigned mismatch
add_definitions("/wd4244") # 'initializing' : conversion from 'float' to 'int', possible loss of data
add_definitions("/wd4305") # 'function' : truncation from 'double' to 'float'
add_definitions("/wd4996") # disable suggestions for ISO C APIs, allow POSIX function names
add_definitions(-D_CRT_SECURE_NO_WARNINGS) # disable suggestions for proprietary secure APIs

include_directories(.)
add_library(common  OBJECT ${CommonFiles} ${ASM_C})
add_library(encoder OBJECT ${EncoderFiles})
add_library(libx264 STATIC $<TARGET_OBJECTS:encoder> $<TARGET_OBJECTS:common> ${YASM_OBJS} ${YASM_SRCS})

if(X264_BUILD_CLI)
    source_group(input   FILES ${InputFiles})
    source_group(output  FILES ${OutputFiles})
    source_group(filters FILES ${FilterFiles})
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /LARGEADDRESSAWARE")
    include_directories(extras/)
    add_executable(x264 ${InputFiles} ${OutputFiles} ${FilterFiles} x264.c extras/getopt.c extras/getopt.h)
    target_link_libraries(x264 libx264)
    
    if(X264_HAVE_FFMS)
        target_include_directories(x264 PRIVATE ${FFMS2_INCLUDE_DIRS})
        target_link_libraries(x264 ${FFMS2_LIBS_DIRS}/ffms2.lib)
        
        target_include_directories(x264 PRIVATE ${FFMPEG_INCLUDE_DIRS})
        file(GLOB FFMS_FOUND_LIBS ${FFMPEG_LIBS_DIRS}/*.lib)
        target_link_libraries(x264 ${FFMS_FOUND_LIBS})
        
        if (MSVC AND NOT X64)
            SET(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} /SAFESEH:NO") # to avoid errors on linking with ffmpeg libs
        endif()
    endif(X264_HAVE_FFMS)
    
endif()
# vim: syntax=cmake:expandtab
