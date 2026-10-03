# The parts of Chromium's third_party/ that WebRTC's Android library and iOS
# framework use, and the libraries checked out inside it as submodules. Code
# includes these as "third_party/<lib>/...", so the directory containing this
# one must be on the include path: the chromium_src_root target carries that.
#
# With WEBRTC_MIN (set by webrtc-min's top-level CMakeLists.txt), only what
# WebRTC uses is built.
cmake_minimum_required(VERSION 3.22)
project(webrtc_third_party C CXX ASM)

option(WEBRTC_MIN "Build only what WebRTC uses" OFF)

add_library(chromium_src_root INTERFACE)
target_include_directories(chromium_src_root INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/..)

add_subdirectory(abseil-cpp)
add_subdirectory(boringssl/src boringssl)
if(ANDROID)
  add_subdirectory(cpu_features/src cpu_features)
  add_subdirectory(jni_zero)
  add_subdirectory(libjpeg_turbo)
else()
  # As in gn for iOS: libyuv without MJPEG support (and so without libjpeg).
  set(LIBYUV_DISABLE_JPEG ON CACHE BOOL "" FORCE)
endif()
add_subdirectory(libsrtp)
add_subdirectory(libyuv)
add_subdirectory(opus)
add_subdirectory(pffft)
add_subdirectory(rnnoise)
add_subdirectory(sframe/src sframe)

