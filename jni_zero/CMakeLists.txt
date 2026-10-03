# jni_zero's C++ runtime, following Chromium's BUILD.gn (the jni_zero
# component). Its generated headers (generate_jni/, system_jni/ and
# system_jni_unchecked_exceptions/) are checked in; see RECIPE.md.
# An OBJECT library: common_apis.cc defines JNI entry points that nothing
# references, so they would be dropped from a static archive.
add_library(jni_zero OBJECT
  common_apis.cc
  default_conversions.cc
  java_refs.cc
  jni_zero.cc
  logging.cc
)
target_link_libraries(jni_zero PUBLIC chromium_src_root)
if(ANDROID)
  target_link_libraries(jni_zero PUBLIC log)
endif()
