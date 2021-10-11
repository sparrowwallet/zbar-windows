cmake_minimum_required(VERSION 3.0)
project(zbar)

include(CMakeDependentOption)
include(CheckFunctionExists)
include(CheckIncludeFile)

set(CMAKE_C_STANDARD 90)

set(ZBAR_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR} CACHE INTERNAL "zbar root directory" FORCE)

if (MSVC)
# "-DCMAKE_TOOLCHAIN_FILE=c:/vcpkg/scripts/buildsystems/vcpkg.cmake"
find_package(iconv REQUIRED)
find_package(JPEG REQUIRED)
find_package(libpng CONFIG REQUIRED)
find_package(ZLIB REQUIRED)
else()
# sudo apt-get install libpng-dev libjpeg-dev zlib1g-dev -y
find_package(Threads REQUIRED )
find_package(Iconv REQUIRED)
find_package(PNG REQUIRED)
find_package(JPEG REQUIRED)
find_package(ZLIB REQUIRED)
endif()

# configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.in.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
add_definitions(-DHAVE_CONFIG_H)
include_directories(${ZBAR_ROOT_DIR}/include)
include_directories(${ZBAR_ROOT_DIR}/zbar)
include_directories(${ZBAR_ROOT_DIR}/zbar/decoder)
include_directories(${ZBAR_ROOT_DIR}/zbar/qrcode)
include_directories(${ZBAR_ROOT_DIR}/zbar/processor)
include_directories(${ZBAR_ROOT_DIR}/zbar/video)
include_directories(${ZBAR_ROOT_DIR}/zbar/window)
include_directories(${ZBAR_ROOT_DIR}/config-internal)
if (MSVC)
include_directories(${ZBAR_ROOT_DIR}/win32)
endif()

file(GLOB ZBAR_DECODER_SRCS RELATIVE ${ZBAR_ROOT_DIR} zbar/decoder/*.c)
file(GLOB ZBAR_QRCODE_SRCS RELATIVE ${ZBAR_ROOT_DIR} zbar/qrcode/*.c)

set(ZBAR_SRCS
    zbar/config.c
    zbar/convert.c
    zbar/decoder.c
    zbar/error.c
    zbar/image.c
    zbar/img_scanner.c
    zbar/jpeg.c
    zbar/misc.c
    zbar/processor.c
    zbar/refcnt.c
    zbar/scanner.c
    zbar/sqcode.c
    zbar/symbol.c
    zbar/video.c
    zbar/window.c
)


if (MSVC)

set(ZBAR_PROCESSOR_SRCS
    zbar/processor/lock.c
    zbar/processor/win.c
)

set(ZBAR_VIDEO_SRCS
    zbar/video/dshow.c
)

set(ZBAR_WINDOW_SRCS
    zbar/window/dib.c
    zbar/window/win.c
)

add_library( zbar SHARED
    zbar/libzbar.rc
    win32/zbar.def
    ${ZBAR_DECODER_SRCS}
    ${ZBAR_QRCODE_SRCS}
    ${ZBAR_PROCESSOR_SRCS}
    ${ZBAR_VIDEO_SRCS}
    ${ZBAR_WINDOW_SRCS}
    ${ZBAR_SRCS}
)

SET(CMAKE_LINK_DEF_FILE_FLAG "/DEF:win32/zbar.def")

target_link_libraries( zbar PRIVATE ${JPEG_LIBRARIES} Iconv::Charset Iconv::Iconv winmm )
target_include_directories( zbar PRIVATE ${JPEG_INCLUDE_DIR} )

add_executable( zbarcam zbarcam/zbarcam.c )
target_link_libraries( zbarcam zbar )

add_executable( scan_image examples/scan_image.c )
target_link_libraries( scan_image zbar png ZLIB::ZLIB )

else()

set(ZBAR_PROCESSOR_SRCS
    zbar/processor/lock.c
    zbar/processor/posix.c
    zbar/processor/null.c
)

set(ZBAR_VIDEO_SRCS
    zbar/video/v4l.c
    zbar/video/v4l2.c
)

set(ZBAR_WINDOW_SRCS
    zbar/window/null.c
)

add_library( zbar SHARED
    ${ZBAR_DECODER_SRCS}
    ${ZBAR_QRCODE_SRCS}
    ${ZBAR_PROCESSOR_SRCS}
    ${ZBAR_VIDEO_SRCS}
    ${ZBAR_WINDOW_SRCS}
    ${ZBAR_SRCS}
)
target_link_libraries( zbar jpeg )

add_definitions("-DLOCALEDIR=\"/tmp/\"")
add_executable( zbarcam zbarcam/zbarcam.c )
target_link_libraries( zbarcam zbar ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS} )

add_executable( scan_image examples/scan_image.c )
target_link_libraries( scan_image zbar png ZLIB::ZLIB ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS} )

endif()
