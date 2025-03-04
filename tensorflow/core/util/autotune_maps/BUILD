# Defines data structures that store autotuning results.
# The autotuning results can be serialized as a string, allowing you to save
# and later restore them.

# TODO(ruochengw): Currently only supports contrib's fused_conv2d_bias_activation_op.
# We plan to add more ops and move fused_conv2d_bias_activation_op back into core library.
load(
    "//tensorflow/core/platform:build_config.bzl",
    "tf_proto_library",
)
load(
    "//tensorflow:tensorflow.bzl",
    "tf_cuda_library",
    "tf_cuda_only_cc_test",
)

package(
    # copybara:uncomment default_applicable_licenses = ["//tensorflow:license"],
    default_visibility = ["//tensorflow:__subpackages__"],
    features = ["-layering_check"],
    licenses = ["notice"],
)

cc_library(
    name = "conv_autotune_maps",
    hdrs = [
        "conv_autotune_maps.h",
    ],
    deps = [
        "//tensorflow/core/kernels:gpu_util_hdrs",
    ],
)

tf_proto_library(
    name = "conv_parameters_proto",
    srcs = [
        "conv_parameters.proto",
    ],
    cc_api_version = 2,
    protodeps = [
        "//tensorflow/core/framework:types_proto",
        "//tensorflow/compiler/xla/stream_executor:dnn_proto",
    ],
)

# This ensures the compilation of custom op fused_conv2d_bias_activation_op.
# TODO(ruochengw): Remove this target once we move fused_conv2d_bias_activation_op back in core library.
cc_library(
    name = "conv_parameters_hdrs",
    hdrs = ["conv_parameters.h"],
    deps = [
        ":conv_parameters_proto_cc",
    ],
)

# For a more maintainable build this target should not exist and the headers
# should  be split into the existing cc_library targets, but this change was
# automatically  done so that we can remove long standing issues and complexity
# in the build system. It's up to the OWNERS of this package to get rid of it or
# not. The use of the textual_hdrs attribute is discouraged, use hdrs instead.
# Here it is used to avoid header parsing errors in packages where the feature
# parse_headers was enabled since loose headers were not being parsed. See
# go/loose-lsc-one-target-approach for more details.
cc_library(
    name = "loose_headers",
    tags = ["avoid_dep"],
    textual_hdrs = [
        "conv_autotune_maps.h",
        "conv_parameters.h",
    ],
    visibility = ["//tensorflow/core/kernels:__pkg__"],
)

tf_cuda_library(
    name = "conv_parameters",
    srcs = ["conv_parameters.cc"],
    hdrs = [
        "conv_parameters.h",
    ],
    deps = [
        ":conv_parameters_proto_cc",
        "//tensorflow/core/platform:hash",
        "//tensorflow/core/platform:protobuf",
        "//tensorflow/core/platform:stream_executor",
        "@com_google_absl//absl/strings:str_format",
        "@com_google_absl//absl/types:optional",
    ],
)

tf_proto_library(
    name = "autotune_map_proto",
    srcs = [
        "autotune_map.proto",
    ],
    cc_api_version = 2,
    protodeps = [
        "//tensorflow/core/util/autotune_maps:conv_parameters_proto",
        "//tensorflow/compiler/xla/stream_executor:dnn_proto",
    ],
    visibility = ["//waymo/ml/deploy/system/autotuning:__subpackages__"],
)

# copybara:uncomment_begin(google-only)
# py_proto_library(
#     name = "autotune_map_py_pb2",
#     has_services = 0,
#     api_version = 2,
#     visibility = ["//waymo/ml/deploy/system/autotuning:__subpackages__"],
#     deps = [":autotune_map_proto"],
# )
# copybara:uncomment_end

tf_cuda_library(
    name = "autotune_serialize",
    srcs = [
        "autotune_serialize.cc",
    ],
    hdrs = [
        "autotune_serialize.h",
    ],
    visibility = ["//visibility:public"],
    deps = [
        ":autotune_map_proto_cc",
        ":conv_autotune_maps",
        ":conv_parameters",
        ":conv_parameters_proto_cc",
        "//tensorflow/compiler/xla:status_macros",
        "//tensorflow/compiler/xla/stream_executor:dnn_proto_cc",
        "//tensorflow/compiler/xla/stream_executor:lazy_op_runner",
        "//tensorflow/compiler/xla/stream_executor:stream_executor_headers",
        "//tensorflow/compiler/xla/stream_executor/gpu:gpu_init",
        "//tensorflow/core:framework",
        "//tensorflow/core/platform:status",
        "//tensorflow/core/platform:stream_executor",
    ],
)

tf_cuda_only_cc_test(
    name = "autotune_serialize_test",
    size = "small",
    srcs = ["autotune_serialize_test.cc"],
    tags = ["no_rocm"],
    deps = [
        ":autotune_serialize",
        ":conv_autotune_maps",
        ":conv_parameters",
        ":conv_parameters_proto_cc",
        "//tensorflow/compiler/xla/stream_executor/gpu:gpu_driver_header",
        "//tensorflow/compiler/xla/stream_executor/gpu:gpu_init",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core/platform:status_matchers",
    ],
)
