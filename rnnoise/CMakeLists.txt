# rnnoise, following Chromium's BUILD.gn: just the RNN VAD weights used by
# WebRTC's audio processing.
add_library(rnnoise STATIC src/rnn_vad_weights.cc)
target_link_libraries(rnnoise PUBLIC chromium_src_root)
