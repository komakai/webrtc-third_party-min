# RECIPE: webrtc-third_party-min

How the `main-min` branch of this repository was made from upstream, and
everything that differs from it. It is the `third_party/` submodule of [webrtc-min](https://github.com/komakai/webrtc-min): the parts of Chromium's `third_party/` that WebRTC's Android library uses, with the libraries Chromium checks out inside it as nested submodules.

## Upstream

- Repository: https://chromium.googlesource.com/chromium/src/third_party
- Revision: 6eabe54fa10922ba0d3f9a8e5a01055d371ebaf7 (2026-09-24T08:58:59-07:00)

The first commit on `main-min` is an unmodified snapshot of that revision, so
`git diff <first commit> main-min` shows every change made here.

## Stripped

Removed because nothing in the build needs them (run from the repository
root on the snapshot):

```sh
# abseil-cpp: other build systems, CI, Windows .def symbol lists, roll scripts,
# Chromium's patch records, tests, test helpers and benchmarks.
git rm -rq abseil-cpp/ci abseil-cpp/CMake abseil-cpp/.github abseil-cpp/patches
git rm -q abseil-cpp/BUILD.bazel abseil-cpp/MODULE.bazel abseil-cpp/CMakeLists.txt \
  abseil-cpp/conanfile.py abseil-cpp/convert_bazel_to_gn.py abseil-cpp/create_lts.py \
  abseil-cpp/generate_def_files.py abseil-cpp/roll_abseil.py abseil-cpp/absl.gni \
  abseil-cpp/absl_hardening_test.cc abseil-cpp/ABSEIL_ISSUE_TEMPLATE.md \
  abseil-cpp/CONTRIBUTING.md abseil-cpp/FAQ.md abseil-cpp/UPGRADES.md \
  abseil-cpp/symbols_*.def
git ls-files abseil-cpp | grep -E \
  '(_test|_test_common|_test_helper[s]?|_test_util|_testing|_benchmark|_benchmarks|_fuzz)\.(cc|h)$|/testdata/|(^|/)(BUILD\.bazel|CMakeLists\.txt|[^/]*\.bzl)$' \
  | xargs -r git rm -q
# opus: other build systems, docs, tests, the unused DNN (DRED/OSCE) code and
# its training scripts, and Chromium's test/assembler-conversion helpers.
git rm -rq opus/tests opus/src/dnn opus/src/training opus/src/scripts opus/src/doc \
  opus/src/tests opus/src/cmake opus/src/meson opus/src/m4
git rm -q opus/convert_rtcd_assembler.py opus/src/autogen.bat opus/src/autogen.sh \
  opus/src/CMakeLists.txt opus/src/configure.ac opus/src/Makefile.am \
  opus/src/Makefile.mips opus/src/Makefile.unix opus/src/meson.build \
  opus/src/meson_options.txt opus/src/opus.m4 opus/src/opus.pc.in \
  opus/src/opus-uninstalled.pc.in opus/src/releases.sha2 opus/src/tar_list.txt \
  opus/src/update_version opus/src/README.draft
git ls-files opus | grep -E '(^|/)(CMakeLists\.txt|meson\.build|Makefile[^/]*|[^/]*\.mk)$|/(test|tests)/' \
  | xargs -r git rm -q
# jni_zero: tests, samples, benchmarks, docs and agent skills. The Python
# generator is kept for regenerating the JNI headers.
git rm -rq jni_zero/test jni_zero/sample jni_zero/benchmarks jni_zero/docs jni_zero/skills
git rm -q jni_zero/parse_test.py jni_zero/proguard_for_test.flags \
  jni_zero/.style.yapf jni_zero/.style.mdformat
# pffft and rnnoise: tests, fuzzers, Chromium's patch records.
git rm -rq pffft/patches
git rm -q pffft/pffft_unittest.cc pffft/pffft_fuzzer.cc pffft/generate_seed_corpus.py
# boringssl wrapper: Chromium's test harness.
git rm -q boringssl/gtest_main_chromium.cc boringssl/test_data_chromium.cc
# gn build files and ownership/issue-tracker metadata everywhere.
git ls-files | grep -E '(^|/)(BUILD\.gn|[^/]*\.gni|DEPS|OWNERS|DIR_METADATA|PRESUBMIT\.py)$' \
  | xargs -r git rm -q
```

## Deviations from upstream

- **Partial snapshot**: upstream is Chromium's whole `third_party/` (~255,000
  files, Blink included), so the snapshot commit has only the directories
  WebRTC uses: `abseil-cpp`, `opus`, `jni_zero`, `rnnoise`, `pffft`, and the
  `boringssl`, `cpu_features` and `sframe` wrapper directories (their
  `README.chromium` and Chromium's own files; the code is in the nested
  submodules).
- **Nested submodules** (`.gitmodules`), at the paths Chromium's DEPS uses:
  `boringssl/src` (boringssl-min), `cpu_features/src` (cpu_features-min),
  `libjpeg_turbo` (libjpeg_turbo-min), `libsrtp` (libsrtp-min), `libyuv`
  (libyuv-min) and `sframe/src` (sframe-min).
- **jni_zero's pregenerated headers** (commit "Add jni_zero's pregenerated
  JNI headers"): `jni_zero/generate_jni/`, `jni_zero/system_jni/` and
  `jni_zero/system_jni_unchecked_exceptions/`, as gn generates them with
  `jni_zero.py` (the system classes from android-37.0's `android.jar`). See
  webrtc-src-min's RECIPE.md for how to regenerate them.
- **CMake instead of gn** (commit "Add CMake build"):
  - `CMakeLists.txt` adds every library, and defines `chromium_src_root`, an
    interface target that puts the directory above this one on the include
    path (code includes these libraries as `third_party/<lib>/...`).
    `WEBRTC_MIN` (default OFF here; webrtc-min's top level turns it on)
    builds only what WebRTC uses.
  - `abseil-cpp/CMakeLists.txt`: Chromium's `absl` component (182 gn targets)
    as one `absl` library. With `WEBRTC_MIN`, the 37 sources of the Abseil
    targets WebRTC doesn't depend on (flags, most of log, random, status,
    ...) are left out. Upstream's CMake files are removed.
  - `opus/CMakeLists.txt`: the `opus` target, with its NEON (Arm) and
    SSE4.1/AVX2 (x86) sources and defines.
  - `jni_zero/CMakeLists.txt`: the C++ runtime, as an OBJECT library so the
    JNI entry points in `common_apis.cc` are always linked (gn links it with
    `--whole-archive`).
  - `rnnoise/` and `pffft/CMakeLists.txt`: one library each.

## Updating to a new upstream revision

`main-min` isn't a git fork of upstream (no upstream history), so updates are
re-applied rather than merged:

1. Replace the tree with upstream at the new revision and commit it as
   "Snapshot of upstream at <rev>" (`git rm -rq . && git archive` of the new
   revision, or a copy of a fresh checkout).
2. Re-run the strip commands above and commit.
3. Re-apply the deviation commits (`git cherry-pick` them from the previous
   `main-min` history) and fix up any conflicts.
4. Update the source lists in the CMake files for files upstream added,
   removed or renamed (compare with the new BUILD.gn in the snapshot commit),
   then build webrtc-min and compare with a gn build.
