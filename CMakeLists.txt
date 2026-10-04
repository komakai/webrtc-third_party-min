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
endif()
# libyuv without MJPEG support, and so without libjpeg: WebRTC never converts
# MJPEG (its only MJPGToI420 call is under HAVE_LIBYUV_JPEG, which nothing
# defines), as the linker confirms: it loads no libjpeg code.
set(LIBYUV_DISABLE_JPEG ON CACHE BOOL "" FORCE)
add_subdirectory(libsrtp)
add_subdirectory(libyuv)
add_subdirectory(opus)
add_subdirectory(pffft)
add_subdirectory(rnnoise)
add_subdirectory(sframe/src sframe)

# With WEBRTC_MIN, leave out the files WebRTC never links: those the linker
# pulls out of none of the libraries' archives when building WebRTC for
# Android arm64 and iOS arm64 (lld --why-extract, ld64 -map). Files with CPU-
# or OS-specific code (cpu_*, rand/*, per-CPU SIMD, ...) are kept even if
# unused there, as another ABI may need them. See RECIPE.md for regenerating
# these lists.
if(WEBRTC_MIN)
  # Removes <files> (relative to <target>'s source directory) from <target>'s
  # sources, warning about any that it doesn't have (e.g. after an update).
  function(webrtc_min_exclude target)
    get_target_property(dir ${target} SOURCE_DIR)
    get_target_property(sources ${target} SOURCES)
    set(result)
    foreach(source IN LISTS sources)
      if(NOT source MATCHES "^\\$<" AND NOT IS_ABSOLUTE "${source}")
        set(source ${dir}/${source})
      endif()
      list(APPEND result ${source})
    endforeach()
    foreach(file IN LISTS ARGN)
      list(FIND result ${dir}/${file} index)
      if(index EQUAL -1)
        message(WARNING "WEBRTC_MIN: ${target} has no source ${file}")
      else()
        list(REMOVE_AT result ${index})
      endif()
    endforeach()
    set_property(TARGET ${target} PROPERTY SOURCES ${result})
  endfunction()

  webrtc_min_exclude(absl
    absl/base/casts.cc
    absl/base/internal/tracing.cc
    absl/base/log_severity.cc
    absl/crc/internal/crc_cord_state.cc
    absl/debugging/internal/decode_rust_punycode.cc
    absl/debugging/internal/demangle.cc
    absl/debugging/internal/demangle_rust.cc
    absl/debugging/internal/utf8_for_code_point.cc
    absl/debugging/symbolize.cc
    absl/profiling/internal/exponential_biased.cc
    absl/strings/cord.cc
    absl/strings/cord_analysis.cc
    absl/strings/internal/cord_internal.cc
    absl/strings/internal/cord_rep_btree.cc
    absl/strings/internal/cord_rep_btree_navigator.cc
    absl/strings/internal/cord_rep_btree_reader.cc
    absl/strings/internal/cord_rep_consume.cc
    absl/strings/internal/cord_rep_crc.cc
    absl/strings/internal/cordz_functions.cc
    absl/strings/internal/cordz_handle.cc
    absl/strings/internal/cordz_info.cc
    absl/strings/internal/damerau_levenshtein_distance.cc
    absl/strings/internal/ostringstream.cc
    absl/strings/str_replace.cc
    absl/strings/substitute.cc
    absl/synchronization/barrier.cc
    absl/synchronization/blocking_counter.cc
    absl/synchronization/internal/graphcycles.cc
    absl/synchronization/notification.cc
    absl/time/civil_time.cc
    absl/time/format.cc
    absl/time/internal/cctz/src/civil_time_detail.cc
    absl/time/internal/cctz/src/time_zone_format.cc
    absl/types/source_location.cc
  )
  webrtc_min_exclude(boringssl
    crypto/asn1/a_bool.cc
    crypto/asn1/asn_pack.cc
    crypto/asn1/f_int.cc
    crypto/asn1/f_string.cc
    crypto/bio/connect.cc
    crypto/bio/pair.cc
    crypto/bn/div.cc
    crypto/bn/sqrt.cc
    crypto/cipher/e_aesctrhmac.cc
    crypto/cipher/e_aeseax.cc
    crypto/cipher/e_aesgcmsiv.cc
    crypto/cipher/e_null.cc
    crypto/cipher/get_cipher.cc
    crypto/cms/cms.cc
    crypto/curve25519/spake25519.cc
    crypto/ec/ec_derive.cc
    crypto/ec/hash_to_curve.cc
    crypto/ecdsa/ecdsa_p1363.cc
    crypto/evp/sign.cc
    crypto/fipsmodule/fips_shared_support.cc
    crypto/fuzzer_mode.cc
    crypto/hrss/hrss.cc
    crypto/pem/pem_x509.cc
    crypto/rsa/rsa_extra.cc
    crypto/rsa/rsa_print.cc
    crypto/slhdsa/slhdsa.cc
    crypto/trust_token/pmbtoken.cc
    crypto/trust_token/trust_token.cc
    crypto/trust_token/voprf.cc
    crypto/x509/t_crl.cc
    crypto/x509/t_req.cc
    crypto/x509/x509_txt.cc
    crypto/x509/x509rset.cc
    crypto/x509/x509spki.cc
    pki/cert_error_id.cc
    pki/cert_error_params.cc
    pki/cert_errors.cc
    pki/cert_issuer_source_static.cc
    pki/certificate.cc
    pki/certificate_policies.cc
    pki/common_cert_errors.cc
    pki/crl.cc
    pki/encode_values.cc
    pki/extended_key_usage.cc
    pki/general_names.cc
    pki/input.cc
    pki/ip_util.cc
    pki/merkle_tree.cc
    pki/name_constraints.cc
    pki/ocsp.cc
    pki/parse_certificate.cc
    pki/parse_name.cc
    pki/parse_values.cc
    pki/parsed_certificate.cc
    pki/parser.cc
    pki/path_builder.cc
    pki/pem.cc
    pki/revocation_util.cc
    pki/signature_algorithm.cc
    pki/simple_path_builder_delegate.cc
    pki/string_util.cc
    pki/trust_store.cc
    pki/trust_store_collection.cc
    pki/trust_store_in_memory.cc
    pki/verify.cc
    pki/verify_certificate_chain.cc
    pki/verify_error.cc
    pki/verify_name_match.cc
    pki/verify_signed_data.cc
    ssl/bio_ssl.cc
    ssl/handoff.cc
    ssl/ssl_file.cc
  )
  # BoringSSL lists every platform's assembly; only this OS's can be non-empty.
  get_target_property(sources boringssl SOURCES)
  list(FILTER sources EXCLUDE REGEX "-win\\.S$")
  if(APPLE)
    list(FILTER sources EXCLUDE REGEX "-linux\\.S$")
  else()
    list(FILTER sources EXCLUDE REGEX "-apple\\.S$")
  endif()
  set_property(TARGET boringssl PROPERTY SOURCES ${sources})
  webrtc_min_exclude(libyuv
    source/convert_to_argb.cc
    source/convert_to_i420.cc
    source/rotate_argb.cc
    source/scale_rgb.cc
  )
  webrtc_min_exclude(opus
    src/silk/debug.c
    src/silk/float/LPC_inv_pred_gain_FLP.c
    src/silk/float/regularize_correlations_FLP.c
    src/src/mapping_matrix.c
    src/src/opus_projection_decoder.c
    src/src/opus_projection_encoder.c
  )
  webrtc_min_exclude(sframe
    src/result.cpp
  )
endif()
