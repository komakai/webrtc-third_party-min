# PFFFT, following Chromium's BUILD.gn.
add_library(pffft STATIC src/pffft.c)
target_compile_options(pffft PRIVATE -Wno-shadow)
if(MSVC)
  target_compile_definitions(pffft PRIVATE _USE_MATH_DEFINES)
endif()
if(NOT CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|arm|x86_64|AMD64|i.86|x86)")
  target_compile_definitions(pffft PRIVATE PFFFT_SIMD_DISABLE)
endif()
