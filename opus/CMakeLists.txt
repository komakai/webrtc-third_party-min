# Opus, following Chromium's third_party/opus/BUILD.gn: the floating-point
# build, with NEON intrinsics on Arm and SSE/AVX2 on x86.
cmake_minimum_required(VERSION 3.22)
project(opus C)

if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64)")
  set(OPUS_CPU arm64)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^arm")
  set(OPUS_CPU arm)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|AMD64|i.86|x86)$")
  set(OPUS_CPU x86)
else()
  set(OPUS_CPU other)
endif()

# opus_private_config, shared with the x86 intrinsics targets.
add_library(opus_private_config INTERFACE)
target_include_directories(opus_private_config INTERFACE
  src/include src src/celt src/silk src/silk/fixed src/silk/float)
target_compile_definitions(opus_private_config INTERFACE
  OPUS_BUILD OPUS_EXPORT= ENABLE_HARDENING USE_ALLOCA HAVE_ALLOCA_H)
if(MSVC)
  target_compile_definitions(opus_private_config INTERFACE inline=__inline)
else()
  target_compile_definitions(opus_private_config INTERFACE HAVE_LRINT HAVE_LRINTF)
endif()
if(OPUS_CPU STREQUAL "x86")
  target_compile_definitions(opus_private_config INTERFACE
    OPUS_HAVE_RTCD CPU_INFO_BY_ASM OPUS_X86_MAY_HAVE_SSE OPUS_X86_MAY_HAVE_SSE2
    OPUS_X86_PRESUME_SSE OPUS_X86_PRESUME_SSE2 OPUS_X86_MAY_HAVE_SSE4_1
    OPUS_X86_MAY_HAVE_AVX2)
elseif(OPUS_CPU MATCHES "^arm")
  if(OPUS_CPU STREQUAL "arm")
    target_compile_definitions(opus_private_config INTERFACE
      OPUS_ARM_ASM OPUS_ARM_INLINE_ASM OPUS_ARM_INLINE_EDSP OPUS_ARM_MAY_HAVE_EDSP
      OPUS_ARM_MAY_HAVE_MEDIA OPUS_ARM_PRESUME_EDSP OPUS_ARM_PRESUME_MEDIA)
  endif()
  target_compile_definitions(opus_private_config INTERFACE
    OPUS_ARM_MAY_HAVE_NEON OPUS_ARM_MAY_HAVE_NEON_INTR OPUS_ARM_PRESUME_NEON
    OPUS_ARM_PRESUME_NEON_INTR)
  if(OPUS_CPU STREQUAL "arm64")
    target_compile_definitions(opus_private_config INTERFACE
      OPUS_ARM_PRESUME_AARCH64_NEON_INTR)
  endif()
endif()

add_library(opus STATIC
  src/celt/bands.c
  src/celt/celt.c
  src/celt/celt_decoder.c
  src/celt/celt_encoder.c
  src/celt/celt_lpc.c
  src/celt/cwrs.c
  src/celt/entcode.c
  src/celt/entdec.c
  src/celt/entenc.c
  src/celt/kiss_fft.c
  src/celt/laplace.c
  src/celt/mathops.c
  src/celt/mdct.c
  src/celt/modes.c
  src/celt/pitch.c
  src/celt/quant_bands.c
  src/celt/rate.c
  src/celt/vq.c
  src/silk/A2NLSF.c
  src/silk/CNG.c
  src/silk/HP_variable_cutoff.c
  src/silk/LPC_analysis_filter.c
  src/silk/LPC_fit.c
  src/silk/LPC_inv_pred_gain.c
  src/silk/LP_variable_cutoff.c
  src/silk/NLSF2A.c
  src/silk/NLSF_VQ.c
  src/silk/NLSF_VQ_weights_laroia.c
  src/silk/NLSF_decode.c
  src/silk/NLSF_del_dec_quant.c
  src/silk/NLSF_encode.c
  src/silk/NLSF_stabilize.c
  src/silk/NLSF_unpack.c
  src/silk/NSQ.c
  src/silk/NSQ_del_dec.c
  src/silk/PLC.c
  src/silk/VAD.c
  src/silk/VQ_WMat_EC.c
  src/silk/ana_filt_bank_1.c
  src/silk/biquad_alt.c
  src/silk/bwexpander.c
  src/silk/bwexpander_32.c
  src/silk/check_control_input.c
  src/silk/code_signs.c
  src/silk/control_SNR.c
  src/silk/control_audio_bandwidth.c
  src/silk/control_codec.c
  src/silk/debug.c
  src/silk/dec_API.c
  src/silk/decode_core.c
  src/silk/decode_frame.c
  src/silk/decode_indices.c
  src/silk/decode_parameters.c
  src/silk/decode_pitch.c
  src/silk/decode_pulses.c
  src/silk/decoder_set_fs.c
  src/silk/enc_API.c
  src/silk/encode_indices.c
  src/silk/encode_pulses.c
  src/silk/gain_quant.c
  src/silk/init_decoder.c
  src/silk/init_encoder.c
  src/silk/inner_prod_aligned.c
  src/silk/interpolate.c
  src/silk/lin2log.c
  src/silk/log2lin.c
  src/silk/pitch_est_tables.c
  src/silk/process_NLSFs.c
  src/silk/quant_LTP_gains.c
  src/silk/resampler.c
  src/silk/resampler_down2.c
  src/silk/resampler_down2_3.c
  src/silk/resampler_private_AR2.c
  src/silk/resampler_private_IIR_FIR.c
  src/silk/resampler_private_down_FIR.c
  src/silk/resampler_private_up2_HQ.c
  src/silk/resampler_rom.c
  src/silk/shell_coder.c
  src/silk/sigm_Q15.c
  src/silk/sort.c
  src/silk/stereo_LR_to_MS.c
  src/silk/stereo_MS_to_LR.c
  src/silk/stereo_decode_pred.c
  src/silk/stereo_encode_pred.c
  src/silk/stereo_find_predictor.c
  src/silk/stereo_quant_pred.c
  src/silk/sum_sqr_shift.c
  src/silk/table_LSF_cos.c
  src/silk/tables_LTP.c
  src/silk/tables_NLSF_CB_NB_MB.c
  src/silk/tables_NLSF_CB_WB.c
  src/silk/tables_gain.c
  src/silk/tables_other.c
  src/silk/tables_pitch_lag.c
  src/silk/tables_pulses_per_block.c
  src/src/analysis.c
  src/src/extensions.c
  src/src/mapping_matrix.c
  src/src/mlp.c
  src/src/mlp_data.c
  src/src/opus.c
  src/src/opus_decoder.c
  src/src/opus_encoder.c
  src/src/opus_multistream.c
  src/src/opus_multistream_decoder.c
  src/src/opus_multistream_encoder.c
  src/src/opus_projection_decoder.c
  src/src/opus_projection_encoder.c
  src/src/repacketizer.c
  src/silk/float/LPC_analysis_filter_FLP.c
  src/silk/float/LPC_inv_pred_gain_FLP.c
  src/silk/float/LTP_analysis_filter_FLP.c
  src/silk/float/LTP_scale_ctrl_FLP.c
  src/silk/float/apply_sine_window_FLP.c
  src/silk/float/autocorrelation_FLP.c
  src/silk/float/burg_modified_FLP.c
  src/silk/float/bwexpander_FLP.c
  src/silk/float/corrMatrix_FLP.c
  src/silk/float/encode_frame_FLP.c
  src/silk/float/energy_FLP.c
  src/silk/float/find_LPC_FLP.c
  src/silk/float/find_LTP_FLP.c
  src/silk/float/find_pitch_lags_FLP.c
  src/silk/float/find_pred_coefs_FLP.c
  src/silk/float/inner_product_FLP.c
  src/silk/float/k2a_FLP.c
  src/silk/float/noise_shape_analysis_FLP.c
  src/silk/float/pitch_analysis_core_FLP.c
  src/silk/float/process_gains_FLP.c
  src/silk/float/regularize_correlations_FLP.c
  src/silk/float/residual_energy_FLP.c
  src/silk/float/scale_copy_vector_FLP.c
  src/silk/float/scale_vector_FLP.c
  src/silk/float/schur_FLP.c
  src/silk/float/sort_FLP.c
  src/silk/float/warped_autocorrelation_FLP.c
  src/silk/float/wrappers_FLP.c
)
target_include_directories(opus PUBLIC src/include)
target_link_libraries(opus PRIVATE opus_private_config)
# optimize_speed.
target_compile_options(opus PRIVATE -O3)

if(OPUS_CPU STREQUAL "x86")
  target_sources(opus PRIVATE
    src/celt/x86/celt_lpc_sse4_1.c
    src/celt/x86/pitch_sse.c
    src/celt/x86/pitch_sse2.c
    src/celt/x86/vq_sse2.c
    src/celt/x86/x86_celt_map.c
    src/celt/x86/x86cpu.c
    src/silk/x86/x86_silk_map.c
  )
  add_library(opus_sse41 OBJECT
    src/celt/x86/pitch_sse4_1.c
    src/silk/x86/NSQ_del_dec_sse4_1.c
    src/silk/x86/NSQ_sse4_1.c
    src/silk/x86/VAD_sse4_1.c
    src/silk/x86/VQ_WMat_EC_sse4_1.c
  )
  target_compile_options(opus_sse41 PRIVATE -msse4.1)
  add_library(opus_avx2 OBJECT
    src/celt/x86/pitch_avx.c
    src/silk/float/x86/inner_product_FLP_avx2.c
    src/silk/x86/NSQ_del_dec_avx2.c
  )
  target_compile_options(opus_avx2 PRIVATE -mavx -mfma -mavx2)
  foreach(t opus_sse41 opus_avx2)
    target_link_libraries(${t} PRIVATE opus_private_config)
    target_sources(opus PRIVATE $<TARGET_OBJECTS:${t}>)
  endforeach()
elseif(OPUS_CPU MATCHES "^arm")
  target_sources(opus PRIVATE
    src/celt/arm/celt_neon_intr.c
    src/celt/arm/pitch_neon_intr.c
    src/silk/arm/LPC_inv_pred_gain_neon_intr.c
    src/silk/arm/NSQ_del_dec_neon_intr.c
    src/silk/arm/NSQ_neon.c
    src/silk/arm/biquad_alt_neon_intr.c
  )
endif()
