cmake_minimum_required(VERSION 3.16)
project(QtAVPlayer VERSION 1.0 LANGUAGES C CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Set up AUTOMOC and some sensible defaults for runtime execution
# When using Qt 6.3, you can replace the code block below with
# qt_standard_project_setup()
set(CMAKE_AUTOMOC ON)
include(GNUInstallDirs)

find_package(QT NAMES Qt5 Qt6 REQUIRED COMPONENTS Core)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Concurrent Gui)
find_package(Qt${QT_VERSION_MAJOR} OPTIONAL_COMPONENTS Multimedia)
find_package(EGL)
find_package(X11)
find_package(WrapOpenGL)

set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")
# Define the list of source files
set(SOURCES
    ${SOURCE_DIR}/qavaudiocodec.cpp
    ${SOURCE_DIR}/qavaudiofilter.cpp
    ${SOURCE_DIR}/qavaudioframe.cpp
    ${SOURCE_DIR}/qavaudioinputfilter.cpp
    ${SOURCE_DIR}/qavaudiooutputfilter.cpp
    ${SOURCE_DIR}/qavcodec.cpp
    ${SOURCE_DIR}/qavdemuxer.cpp
    ${SOURCE_DIR}/qavfilter.cpp
    ${SOURCE_DIR}/qavfiltergraph.cpp
    ${SOURCE_DIR}/qavfilters.cpp
    ${SOURCE_DIR}/qavframe.cpp
    ${SOURCE_DIR}/qavframecodec.cpp
    ${SOURCE_DIR}/qavinoutfilter.cpp
    ${SOURCE_DIR}/qaviodevice.cpp
    ${SOURCE_DIR}/qavpacket.cpp
    ${SOURCE_DIR}/qavplayer.cpp
    ${SOURCE_DIR}/qavstream.cpp
    ${SOURCE_DIR}/qavstreamframe.cpp
    ${SOURCE_DIR}/qavsubtitlecodec.cpp
    ${SOURCE_DIR}/qavsubtitleframe.cpp
    ${SOURCE_DIR}/qavvideobuffer_cpu.cpp
    ${SOURCE_DIR}/qavvideobuffer_gpu.cpp
    ${SOURCE_DIR}/qavvideocodec.cpp
    ${SOURCE_DIR}/qavvideofilter.cpp
    ${SOURCE_DIR}/qavvideoframe.cpp
    ${SOURCE_DIR}/qavvideoinputfilter.cpp
    ${SOURCE_DIR}/qavvideooutputfilter.cpp
    ${SOURCE_DIR}/qtavplayerglobal.h
)

# Define the list of header files
set(HEADERS
    ${SOURCE_DIR}/qavaudiocodec_p.h
    ${SOURCE_DIR}/qavaudiofilter_p.h
    ${SOURCE_DIR}/qavaudioformat.h
    ${SOURCE_DIR}/qavaudioframe.h
    ${SOURCE_DIR}/qavaudioinputfilter_p.h
    ${SOURCE_DIR}/qavaudiooutputfilter_p.h
    ${SOURCE_DIR}/qavcodec_p.h
    ${SOURCE_DIR}/qavcodec_p_p.h
    ${SOURCE_DIR}/qavdemuxer_p.h
    ${SOURCE_DIR}/qavfilter_p.h
    ${SOURCE_DIR}/qavfilter_p_p.h
    ${SOURCE_DIR}/qavfiltergraph_p.h
    ${SOURCE_DIR}/qavfilters_p.h
    ${SOURCE_DIR}/qavframe.h
    ${SOURCE_DIR}/qavframe_p.h
    ${SOURCE_DIR}/qavframecodec_p.h
    ${SOURCE_DIR}/qavhwdevice_p.h
    ${SOURCE_DIR}/qavhwdevice_cuda_p.h
    ${SOURCE_DIR}/qavinoutfilter_p.h
    ${SOURCE_DIR}/qaviodevice.h
    ${SOURCE_DIR}/qavpacket_p.h
    ${SOURCE_DIR}/qavpacketqueue_p.h
    ${SOURCE_DIR}/qavplayer.h
    ${SOURCE_DIR}/qavstream.h
    ${SOURCE_DIR}/qavstreamframe.h
    ${SOURCE_DIR}/qavsubtitlecodec_p.h
    ${SOURCE_DIR}/qavsubtitleframe.h
    ${SOURCE_DIR}/qavvideobuffer_p.h
    ${SOURCE_DIR}/qavvideobuffer_cpu_p.h
    ${SOURCE_DIR}/qavvideobuffer_gpu_p.h
    ${SOURCE_DIR}/qavvideocodec_p.h
    ${SOURCE_DIR}/qavvideofilter_p.h
    ${SOURCE_DIR}/qavvideoframe.h
    ${SOURCE_DIR}/qavvideoinputfilter_p.h
    ${SOURCE_DIR}/qavvideooutputfilter_p.h
)

add_library(${PROJECT_NAME} SHARED
    ${SOURCES} ${HEADERS}
)

# Copy header files to a specific directory
foreach(HEADER ${HEADER_FILES})
    configure_file(${HEADER} ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
endforeach()

# Include the include directory
target_include_directories(${PROJECT_NAME}
    PUBLIC
        ${CMAKE_CURRENT_BINARY_DIR}/../
    PRIVATE
        ${CMAKE_CURRENT_BINARY_DIR}/../
)


target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt::Concurrent
    Qt::Core
    Qt::Gui
    avcodec
    avdevice
    avfilter
    avformat
    avutil
    swresample
    swscale
)

if(WIN32)
    target_sources(${PROJECT_NAME} PUBLIC
        ${SOURCE_DIR}/qavhwdevice_d3d11.cpp
    )

    target_include_directories(${PROJECT_NAME} PRIVATE
        ../ffmpeg/include
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/../ffmpeg/bin
        # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/../ffmpeg/lib
    )
endif()

if(DEFINES___contains___QT_AVPLAYER_MULTIMEDIA)
    target_sources(${PROJECT_NAME} PUBLIC
        ${SOURCE_DIR}/qavaudiooutput.cpp ${SOURCE_DIR}/qavaudiooutput.h
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt::Multimedia
    )
endif()

if((DEFINES___contains___QT_AVPLAYER_MULTIMEDIA) AND ((QT_VERSION_MAJOR EQUAL 6)))
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt::MultimediaPrivate
    )
endif()

if(DEFINES___contains___QT_AVPLAYER_VA_X11 AND QT_FEATURE_opengl)
    target_sources(${PROJECT_NAME} PUBLIC
        ${SOURCE_DIR}/qavhwdevice_vaapi_x11_glx.cpp
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        WrapOpenGL::WrapOpenGL
        X11::X11
        va
        va-x11
    )
endif()

if(DEFINES___contains___QT_AVPLAYER_VA_DRM AND QT_FEATURE_egl)
    target_sources(${PROJECT_NAME} PUBLIC
        qavhwdevice_vaapi_drm_egl.cpp
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        EGL::EGL
        WrapOpenGL::WrapOpenGL
        va
        va-drm
    )
endif()

if(DEFINES___contains___QT_AVPLAYER_VDPAU)
    target_sources(${PROJECT_NAME} PUBLIC
        ${SOURCE_DIR}/qavhwdevice_vdpau.cpp
    )
endif()

if(APPLE)
    target_sources(${PROJECT_NAME} PUBLIC
        ${SOURCE_DIR}/qavhwdevice_videotoolbox.mm
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        "-framework CoreMedia"
        "-framework CoreVideo"
        "-framework IOSurface"
        "-framework Metal"
        "-framework QuartzCore"
    )
endif()

if(ANDROID)
    target_sources(${PROJECT_NAME} PUBLIC
        ${SOURCE_DIR}/qavandroidsurfacetexture.cpp
        ${SOURCE_DIR}/qavhwdevice_mediacodec.cpp
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt::CorePrivate
    )
endif()

if(ANDROID AND ANDROID_TARGET_ARCH___equals___armeabi-v7a)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        # Remove: L$ENV{AVPLAYER_ANDROID_LIB_ARMEABI_V7A}
    )
endif()

if(ANDROID AND ANDROID_TARGET_ARCH___equals___arm64-v8a)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        # Remove: L$ENV{AVPLAYER_ANDROID_LIB_ARMEABI_V8A}
    )
endif()

if(ANDROID AND ANDROID_TARGET_ARCH___equals___x86)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        # Remove: L$ENV{AVPLAYER_ANDROID_LIB_X86}
    )
endif()

if(ANDROID AND ANDROID_TARGET_ARCH___equals___x86_64)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        # Remove: L$ENV{AVPLAYER_ANDROID_LIB_X86_64}
    )
endif()

install(TARGETS ${PROJECT_NAME}
    BUNDLE DESTINATION .
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Consider using qt_generate_deploy_app_script() for app deployment if
# the project can use Qt 6.3. In that case rerun qmake2cmake with
# --min-qt-version=6.3.
