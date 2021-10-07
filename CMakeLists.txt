project(zbar_qrcode)
cmake_minimum_required(VERSION 3.10)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

set(CMAKE_THREAD_PREFER_PTHREAD ON)
find_package(Threads QUIET)
find_package(Iconv QUIET)

find_file(HAVE_INTTYPES_H "inttypes.h" DOC "inttypes.h exists")
find_file(HAVE_FEATURES_H "features.h" DOC "features.h exists")
find_file(HAVE_POLL_H "poll.h" DOC "poll.h exists")
find_file(HAVE_SYS_TIME_H "sys/time.h" DOC "sys/time.h exists")

add_library(zbar SHARED
    zbar/decoder/qr_finder.c
    zbar/qrcode/qrdec.c
    zbar/qrcode/qrdectxt.c
    zbar/qrcode/rs.c
    zbar/qrcode/isaac.c
    zbar/qrcode/bch15_5.c
    zbar/qrcode/binarize.c
    zbar/qrcode/util.c
    zbar/processor/posix.c
    zbar/symbol.c
    zbar/image.c
    zbar/video.c
    zbar/img_scanner.c
    zbar/scanner.c
    zbar/window.c
    zbar/decoder.c
    zbar/refcnt.c
    zbar/processor/lock.c
    zbar/processor.c
    zbar/convert.c
    zbar/error.c
    zbar/video/null.c
    zbar/processor/null.c
    zbar/window/null.c

    include/zbar.h
    zbar/decoder/qr_finder.h
    zbar/qrcode.h
    zbar/qrcode/qrdec.h
    zbar/qrcode/rs.h
    zbar/qrcode/isaac.h
    zbar/qrcode/bch15_5.h
    zbar/qrcode/binarize.h
    zbar/qrcode/util.h
    zbar/error.h
    zbar/symbol.h
    zbar/image.h
    zbar/processor.h
    zbar/refcnt.h
    zbar/timer.h
    zbar/mutex.h
    zbar/event.h
    zbar/thread.h
    zbar/window.h
    zbar/video.h
    zbar/img_scanner.h
    zbar/decoder.h
    zbar/processor/posix.h
)
target_compile_definitions(zbar
    PRIVATE
        ENABLE_QRCODE
        ZBAR_VERSION_MAJOR=0
        ZBAR_VERSION_MINOR=23
)

if (HAVE_INTTYPES_H)
    target_compile_definitions(zbar PRIVATE HAVE_INTTYPES_H=1)
endif()
if (HAVE_FEATURES_H)
    target_compile_definitions(zbar PRIVATE HAVE_FEATURES_H=1)
endif()
if (HAVE_POLL_H)
    target_compile_definitions(zbar PRIVATE HAVE_POLL_H=1)
endif()
if (HAVE_SYS_TIME_H)
    target_compile_definitions(zbar PRIVATE HAVE_SYS_TIME_H=1)
endif()
if(THREADS_FOUND)
    target_compile_definitions(zbar PRIVATE HAVE_LIBPTHREAD=1)
    target_link_libraries(zbar
        PRIVATE
            Threads::Threads
    )
endif()
if (ICONV_FOUND)
    target_link_libraries(zbar
        PRIVATE
            Iconv::Iconv
    )
endif()
target_include_directories(zbar
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        zbar
        zbar/qrcode
        zbar/decoder
)

install(TARGETS zbar EXPORT zbarTargets
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
install(
    FILES include/zbar.h
    DESTINATION include/zbar
)
# Package configuration
if(THREADS_FOUND)
    set(find_pkg_dependencies "find_dependency(Threads)")
else()
    set(find_pkg_dependencies "")
endif()
set(configPackageLocation ${CMAKE_INSTALL_LIBDIR}/cmake/zbar)
set(inputConfigFile "${CMAKE_CURRENT_SOURCE_DIR}/cmake/zbarConfig.cmake.in")
set(outputConfigFile "${CMAKE_CURRENT_BINARY_DIR}/zbarConfig.cmake")
    configure_file(${inputConfigFile} ${outputConfigFile} @ONLY)
install(FILES ${outputConfigFile}
        DESTINATION ${configPackageLocation}
)
export(EXPORT zbarTargets
    FILE "${CMAKE_CURRENT_BINARY_DIR}/zbarTargets.cmake"
    NAMESPACE zbar::
)

install(EXPORT zbarTargets
    NAMESPACE zbar::
    DESTINATION ${configPackageLocation}
)
