# Bazel (https://bazel.build/) BUILD file for Protobuf.

load("@bazel_skylib//rules:common_settings.bzl", "string_flag")
load("@rules_cc//cc:defs.bzl", "cc_binary", "cc_library", "cc_test", "objc_library", native_cc_proto_library = "cc_proto_library")
load("@rules_pkg//:mappings.bzl", "pkg_files", "strip_prefix")
load("@rules_proto//proto:defs.bzl", "proto_lang_toolchain", "proto_library")
load("@rules_python//python:defs.bzl", "py_library")
load("@rules_java//java:defs.bzl", "java_binary", "java_lite_proto_library", "java_proto_library")
load(":cc_proto_blacklist_test.bzl", "cc_proto_blacklist_test")
load(":protobuf_release.bzl", "package_naming")

licenses(["notice"])

exports_files(["LICENSE"])

################################################################################
# build configuration
################################################################################

################################################################################
# ZLIB configuration
################################################################################

ZLIB_DEPS = ["@zlib//:zlib"]

################################################################################
# Protobuf Runtime Library
################################################################################

MSVC_COPTS = [
    "/wd4065",  # switch statement contains 'default' but no 'case' labels
    "/wd4244",  # 'conversion' conversion from 'type1' to 'type2', possible loss of data
    "/wd4251",  # 'identifier' : class 'type' needs to have dll-interface to be used by clients of class 'type2'
    "/wd4267",  # 'var' : conversion from 'size_t' to 'type', possible loss of data
    "/wd4305",  # 'identifier' : truncation from 'type1' to 'type2'
    "/wd4307",  # 'operator' : integral constant overflow
    "/wd4309",  # 'conversion' : truncation of constant value
    "/wd4334",  # 'operator' : result of 32-bit shift implicitly converted to 64 bits (was 64-bit shift intended?)
    "/wd4355",  # 'this' : used in base member initializer list
    "/wd4506",  # no definition for inline function 'function'
    "/wd4800",  # 'type' : forcing value to bool 'true' or 'false' (performance warning)
    "/wd4996",  # The compiler encountered a deprecated declaration.
]

COPTS = select({
    ":msvc": MSVC_COPTS,
    "//conditions:default": [
        "-DHAVE_ZLIB",
        "-Woverloaded-virtual",
        "-Wno-sign-compare",
    ],
})

load(":compiler_config_setting.bzl", "create_compiler_config_setting")

create_compiler_config_setting(
    name = "msvc",
    value = "msvc-cl",
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

# Android NDK builds can specify different crosstool_top flags to choose which
# STL they use for C++. We need these multiple variants to catch all of those
# versions of crosstool_top and reliably detect Android.
#
# For more info on the various crosstool_tops used by NDK Bazel builds, see:
# https://docs.bazel.build/versions/master/android-ndk.html#configuring-the-stl

config_setting(
    name = "android",
    values = {
        "crosstool_top": "//external:android/crosstool",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

config_setting(
    name = "android-stlport",
    values = {
        "crosstool_top": "@androidndk//:toolchain-stlport",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

config_setting(
    name = "android-libcpp",
    values = {
        "crosstool_top": "@androidndk//:toolchain-libcpp",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

config_setting(
    name = "android-gnu-libstdcpp",
    values = {
        "crosstool_top": "@androidndk//:toolchain-gnu-libstdcpp",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

config_setting(
    name = "android-default",
    values = {
        "crosstool_top": "@androidndk//:default_crosstool",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

# Android and MSVC builds do not need to link in a separate pthread library.
LINK_OPTS = select({
    ":android": [],
    ":android-stlport": [],
    ":android-libcpp": [],
    ":android-gnu-libstdcpp": [],
    ":android-default": [],
    ":msvc": [
        # Suppress linker warnings about files with no symbols defined.
        "-ignore:4221",
    ],
    "//conditions:default": [
        "-lpthread",
        "-lm",
    ],
})

load(
    ":protobuf.bzl",
    "adapt_proto_library",
    "cc_proto_library",
    "internal_copied_filegroup",
    "internal_gen_kt_protos",
    "internal_gen_well_known_protos_java",
    "internal_protobuf_py_tests",
    "py_proto_library",
)

cc_library(
    name = "protobuf_lite",
    srcs = [
        # AUTOGEN(protobuf_lite_srcs)
        "src/google/protobuf/any_lite.cc",
        "src/google/protobuf/arena.cc",
        "src/google/protobuf/arenastring.cc",
        "src/google/protobuf/arenaz_sampler.cc",
        "src/google/protobuf/extension_set.cc",
        "src/google/protobuf/generated_enum_util.cc",
        "src/google/protobuf/generated_message_tctable_lite.cc",
        "src/google/protobuf/generated_message_util.cc",
        "src/google/protobuf/implicit_weak_message.cc",
        "src/google/protobuf/inlined_string_field.cc",
        "src/google/protobuf/io/coded_stream.cc",
        "src/google/protobuf/io/io_win32.cc",
        "src/google/protobuf/io/strtod.cc",
        "src/google/protobuf/io/zero_copy_stream.cc",
        "src/google/protobuf/io/zero_copy_stream_impl.cc",
        "src/google/protobuf/io/zero_copy_stream_impl_lite.cc",
        "src/google/protobuf/map.cc",
        "src/google/protobuf/message_lite.cc",
        "src/google/protobuf/parse_context.cc",
        "src/google/protobuf/repeated_field.cc",
        "src/google/protobuf/repeated_ptr_field.cc",
        "src/google/protobuf/stubs/bytestream.cc",
        "src/google/protobuf/stubs/common.cc",
        "src/google/protobuf/stubs/int128.cc",
        "src/google/protobuf/stubs/status.cc",
        "src/google/protobuf/stubs/statusor.cc",
        "src/google/protobuf/stubs/stringpiece.cc",
        "src/google/protobuf/stubs/stringprintf.cc",
        "src/google/protobuf/stubs/structurally_valid.cc",
        "src/google/protobuf/stubs/strutil.cc",
        "src/google/protobuf/stubs/time.cc",
        "src/google/protobuf/wire_format_lite.cc",
    ],
    hdrs = glob([
        "src/google/protobuf/**/*.h",
        "src/google/protobuf/**/*.inc",
    ]),
    copts = COPTS,
    includes = ["src/"],
    linkopts = LINK_OPTS,
    visibility = ["//visibility:public"],
)

PROTOBUF_DEPS = select({
    ":msvc": [],
    "//conditions:default": ZLIB_DEPS,
})

cc_library(
    name = "protobuf",
    srcs = [
        # AUTOGEN(protobuf_srcs)
        "src/google/protobuf/any.cc",
        "src/google/protobuf/any.pb.cc",
        "src/google/protobuf/api.pb.cc",
        "src/google/protobuf/compiler/importer.cc",
        "src/google/protobuf/compiler/parser.cc",
        "src/google/protobuf/descriptor.cc",
        "src/google/protobuf/descriptor.pb.cc",
        "src/google/protobuf/descriptor_database.cc",
        "src/google/protobuf/duration.pb.cc",
        "src/google/protobuf/dynamic_message.cc",
        "src/google/protobuf/empty.pb.cc",
        "src/google/protobuf/extension_set_heavy.cc",
        "src/google/protobuf/field_mask.pb.cc",
        "src/google/protobuf/generated_message_bases.cc",
        "src/google/protobuf/generated_message_reflection.cc",
        "src/google/protobuf/generated_message_tctable_full.cc",
        "src/google/protobuf/io/gzip_stream.cc",
        "src/google/protobuf/io/printer.cc",
        "src/google/protobuf/io/tokenizer.cc",
        "src/google/protobuf/map_field.cc",
        "src/google/protobuf/message.cc",
        "src/google/protobuf/reflection_ops.cc",
        "src/google/protobuf/service.cc",
        "src/google/protobuf/source_context.pb.cc",
        "src/google/protobuf/struct.pb.cc",
        "src/google/protobuf/stubs/substitute.cc",
        "src/google/protobuf/text_format.cc",
        "src/google/protobuf/timestamp.pb.cc",
        "src/google/protobuf/type.pb.cc",
        "src/google/protobuf/unknown_field_set.cc",
        "src/google/protobuf/util/delimited_message_util.cc",
        "src/google/protobuf/util/field_comparator.cc",
        "src/google/protobuf/util/field_mask_util.cc",
        "src/google/protobuf/util/internal/datapiece.cc",
        "src/google/protobuf/util/internal/default_value_objectwriter.cc",
        "src/google/protobuf/util/internal/error_listener.cc",
        "src/google/protobuf/util/internal/field_mask_utility.cc",
        "src/google/protobuf/util/internal/json_escaping.cc",
        "src/google/protobuf/util/internal/json_objectwriter.cc",
        "src/google/protobuf/util/internal/json_stream_parser.cc",
        "src/google/protobuf/util/internal/object_writer.cc",
        "src/google/protobuf/util/internal/proto_writer.cc",
        "src/google/protobuf/util/internal/protostream_objectsource.cc",
        "src/google/protobuf/util/internal/protostream_objectwriter.cc",
        "src/google/protobuf/util/internal/type_info.cc",
        "src/google/protobuf/util/internal/utility.cc",
        "src/google/protobuf/util/json_util.cc",
        "src/google/protobuf/util/message_differencer.cc",
        "src/google/protobuf/util/time_util.cc",
        "src/google/protobuf/util/type_resolver_util.cc",
        "src/google/protobuf/wire_format.cc",
        "src/google/protobuf/wrappers.pb.cc",
    ],
    hdrs = glob([
        "src/**/*.h",
        "src/**/*.inc",
    ]),
    copts = COPTS,
    includes = ["src/"],
    linkopts = LINK_OPTS,
    visibility = ["//visibility:public"],
    deps = [":protobuf_lite"] + PROTOBUF_DEPS,
)

# This provides just the header files for use in projects that need to build
# shared libraries for dynamic loading. This target is available until Bazel
# adds native support for such use cases.
# TODO(keveman): Remove this target once the support gets added to Bazel.
cc_library(
    name = "protobuf_headers",
    hdrs = glob([
        "src/**/*.h",
        "src/**/*.inc",
    ]),
    includes = ["src/"],
    visibility = ["//visibility:public"],
)

# Map of all well known protos.
# name => (include path, imports)
WELL_KNOWN_PROTO_MAP = {
    "any": ("src/google/protobuf/any.proto", []),
    "api": (
        "src/google/protobuf/api.proto",
        [
            "source_context",
            "type",
        ],
    ),
    "compiler_plugin": (
        "src/google/protobuf/compiler/plugin.proto",
        ["descriptor"],
    ),
    "descriptor": ("src/google/protobuf/descriptor.proto", []),
    "duration": ("src/google/protobuf/duration.proto", []),
    "empty": ("src/google/protobuf/empty.proto", []),
    "field_mask": ("src/google/protobuf/field_mask.proto", []),
    "source_context": ("src/google/protobuf/source_context.proto", []),
    "struct": ("src/google/protobuf/struct.proto", []),
    "timestamp": ("src/google/protobuf/timestamp.proto", []),
    "type": (
        "src/google/protobuf/type.proto",
        [
            "any",
            "source_context",
        ],
    ),
    "wrappers": ("src/google/protobuf/wrappers.proto", []),
}

WELL_KNOWN_PROTOS = [value[0] for value in WELL_KNOWN_PROTO_MAP.values()]

LITE_WELL_KNOWN_PROTO_MAP = {
    "any": ("src/google/protobuf/any.proto", []),
    "api": (
        "src/google/protobuf/api.proto",
        [
            "source_context",
            "type",
        ],
    ),
    "duration": ("src/google/protobuf/duration.proto", []),
    "empty": ("src/google/protobuf/empty.proto", []),
    "field_mask": ("src/google/protobuf/field_mask.proto", []),
    "source_context": ("src/google/protobuf/source_context.proto", []),
    "struct": ("src/google/protobuf/struct.proto", []),
    "timestamp": ("src/google/protobuf/timestamp.proto", []),
    "type": (
        "src/google/protobuf/type.proto",
        [
            "any",
            "source_context",
        ],
    ),
    "wrappers": ("src/google/protobuf/wrappers.proto", []),
}

LITE_WELL_KNOWN_PROTOS = [value[0] for value in LITE_WELL_KNOWN_PROTO_MAP.values()]

filegroup(
    name = "well_known_protos",
    srcs = WELL_KNOWN_PROTOS,
    visibility = ["//visibility:public"],
)

exports_files(
    srcs = WELL_KNOWN_PROTOS,
    visibility = ["//pkg:__pkg__"],
)

filegroup(
    name = "lite_well_known_protos",
    srcs = LITE_WELL_KNOWN_PROTOS,
    visibility = ["//visibility:public"],
)

adapt_proto_library(
    name = "cc_wkt_protos_genproto",
    visibility = ["//visibility:public"],
    deps = [proto + "_proto" for proto in WELL_KNOWN_PROTO_MAP.keys()],
)

cc_library(
    name = "cc_wkt_protos",
    deprecation = "Only for backward compatibility. Do not use.",
    visibility = ["//visibility:public"],
)

################################################################################
# Well Known Types Proto Library Rules
#
# These proto_library rules can be used with one of the language specific proto
# library rules i.e. java_proto_library:
#
# java_proto_library(
#   name = "any_java_proto",
#   deps = ["@com_google_protobuf//:any_proto],
# )
################################################################################

[proto_library(
    name = proto[0] + "_proto",
    srcs = [proto[1][0]],
    strip_import_prefix = "src",
    visibility = ["//visibility:public"],
    deps = [dep + "_proto" for dep in proto[1][1]],
) for proto in WELL_KNOWN_PROTO_MAP.items()]

[native_cc_proto_library(
    name = proto + "_cc_proto",
    visibility = ["//visibility:private"],
    deps = [proto + "_proto"],
) for proto in WELL_KNOWN_PROTO_MAP.keys()]

cc_proto_blacklist_test(
    name = "cc_proto_blacklist_test",
    tags = [
        # Exclude this target from wildcard expansion (//...). Due to
        # https://github.com/bazelbuild/bazel/issues/10590, this test has to
        # be nominated using the `@com_google_protobuf//` prefix. We do that,
        # e.g., in kokoro/linux/bazel/build.sh.
        # See also https://github.com/protocolbuffers/protobuf/pull/7096.
        "manual",
    ],
    deps = [proto + "_cc_proto" for proto in WELL_KNOWN_PROTO_MAP.keys()],
)

################################################################################
# Protocol Buffers Compiler
################################################################################

cc_library(
    name = "protoc_lib",
    srcs = [
        # AUTOGEN(protoc_lib_srcs)
        "src/google/protobuf/compiler/code_generator.cc",
        "src/google/protobuf/compiler/command_line_interface.cc",
        "src/google/protobuf/compiler/cpp/enum.cc",
        "src/google/protobuf/compiler/cpp/enum_field.cc",
        "src/google/protobuf/compiler/cpp/extension.cc",
        "src/google/protobuf/compiler/cpp/field.cc",
        "src/google/protobuf/compiler/cpp/file.cc",
        "src/google/protobuf/compiler/cpp/generator.cc",
        "src/google/protobuf/compiler/cpp/helpers.cc",
        "src/google/protobuf/compiler/cpp/map_field.cc",
        "src/google/protobuf/compiler/cpp/message.cc",
        "src/google/protobuf/compiler/cpp/message_field.cc",
        "src/google/protobuf/compiler/cpp/padding_optimizer.cc",
        "src/google/protobuf/compiler/cpp/parse_function_generator.cc",
        "src/google/protobuf/compiler/cpp/primitive_field.cc",
        "src/google/protobuf/compiler/cpp/service.cc",
        "src/google/protobuf/compiler/cpp/string_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_doc_comment.cc",
        "src/google/protobuf/compiler/csharp/csharp_enum.cc",
        "src/google/protobuf/compiler/csharp/csharp_enum_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_field_base.cc",
        "src/google/protobuf/compiler/csharp/csharp_generator.cc",
        "src/google/protobuf/compiler/csharp/csharp_helpers.cc",
        "src/google/protobuf/compiler/csharp/csharp_map_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_message.cc",
        "src/google/protobuf/compiler/csharp/csharp_message_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_primitive_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_reflection_class.cc",
        "src/google/protobuf/compiler/csharp/csharp_repeated_enum_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_repeated_message_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_repeated_primitive_field.cc",
        "src/google/protobuf/compiler/csharp/csharp_source_generator_base.cc",
        "src/google/protobuf/compiler/csharp/csharp_wrapper_field.cc",
        "src/google/protobuf/compiler/java/context.cc",
        "src/google/protobuf/compiler/java/doc_comment.cc",
        "src/google/protobuf/compiler/java/enum.cc",
        "src/google/protobuf/compiler/java/enum_field.cc",
        "src/google/protobuf/compiler/java/enum_field_lite.cc",
        "src/google/protobuf/compiler/java/enum_lite.cc",
        "src/google/protobuf/compiler/java/extension.cc",
        "src/google/protobuf/compiler/java/extension_lite.cc",
        "src/google/protobuf/compiler/java/field.cc",
        "src/google/protobuf/compiler/java/file.cc",
        "src/google/protobuf/compiler/java/generator.cc",
        "src/google/protobuf/compiler/java/generator_factory.cc",
        "src/google/protobuf/compiler/java/helpers.cc",
        "src/google/protobuf/compiler/java/kotlin_generator.cc",
        "src/google/protobuf/compiler/java/map_field.cc",
        "src/google/protobuf/compiler/java/map_field_lite.cc",
        "src/google/protobuf/compiler/java/message.cc",
        "src/google/protobuf/compiler/java/message_builder.cc",
        "src/google/protobuf/compiler/java/message_builder_lite.cc",
        "src/google/protobuf/compiler/java/message_field.cc",
        "src/google/protobuf/compiler/java/message_field_lite.cc",
        "src/google/protobuf/compiler/java/message_lite.cc",
        "src/google/protobuf/compiler/java/name_resolver.cc",
        "src/google/protobuf/compiler/java/primitive_field.cc",
        "src/google/protobuf/compiler/java/primitive_field_lite.cc",
        "src/google/protobuf/compiler/java/service.cc",
        "src/google/protobuf/compiler/java/shared_code_generator.cc",
        "src/google/protobuf/compiler/java/string_field.cc",
        "src/google/protobuf/compiler/java/string_field_lite.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_enum.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_enum_field.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_extension.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_field.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_file.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_generator.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_helpers.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_map_field.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_message.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_message_field.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_oneof.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_primitive_field.cc",
        "src/google/protobuf/compiler/php/php_generator.cc",
        "src/google/protobuf/compiler/plugin.cc",
        "src/google/protobuf/compiler/plugin.pb.cc",
        "src/google/protobuf/compiler/python/generator.cc",
        "src/google/protobuf/compiler/python/helpers.cc",
        "src/google/protobuf/compiler/python/pyi_generator.cc",
        "src/google/protobuf/compiler/ruby/ruby_generator.cc",
        "src/google/protobuf/compiler/subprocess.cc",
        "src/google/protobuf/compiler/zip_writer.cc",
    ],
    copts = COPTS,
    includes = ["src/"],
    linkopts = LINK_OPTS,
    visibility = ["//visibility:public"],
    deps = [":protobuf"],
)

cc_binary(
    name = "protoc",
    srcs = ["src/google/protobuf/compiler/main.cc"],
    linkopts = LINK_OPTS,
    visibility = ["//visibility:public"],
    deps = [":protoc_lib"],
)

################################################################################
# Tests
################################################################################

filegroup(
    name = "testdata",
    srcs = glob(["src/google/protobuf/testdata/**/*"]),
    visibility = [
        "//:__subpackages__",
        "@upb//:__subpackages__",
    ],
)

RELATIVE_LITE_TEST_PROTOS = [
    # AUTOGEN(lite_test_protos)
    "google/protobuf/map_lite_unittest.proto",
    "google/protobuf/unittest_import_lite.proto",
    "google/protobuf/unittest_import_public_lite.proto",
    "google/protobuf/unittest_lite.proto",
]

LITE_TEST_PROTOS = ["src/" + s for s in RELATIVE_LITE_TEST_PROTOS]

RELATIVE_TEST_PROTOS = [
    # AUTOGEN(test_protos)
    "google/protobuf/any_test.proto",
    "google/protobuf/compiler/cpp/test_bad_identifiers.proto",
    "google/protobuf/compiler/cpp/test_large_enum_value.proto",
    "google/protobuf/map_proto2_unittest.proto",
    "google/protobuf/map_unittest.proto",
    "google/protobuf/unittest.proto",
    "google/protobuf/unittest_arena.proto",
    "google/protobuf/unittest_custom_options.proto",
    "google/protobuf/unittest_drop_unknown_fields.proto",
    "google/protobuf/unittest_embed_optimize_for.proto",
    "google/protobuf/unittest_empty.proto",
    "google/protobuf/unittest_enormous_descriptor.proto",
    "google/protobuf/unittest_import.proto",
    "google/protobuf/unittest_import_public.proto",
    "google/protobuf/unittest_lazy_dependencies.proto",
    "google/protobuf/unittest_lazy_dependencies_custom_option.proto",
    "google/protobuf/unittest_lazy_dependencies_enum.proto",
    "google/protobuf/unittest_lite_imports_nonlite.proto",
    "google/protobuf/unittest_mset.proto",
    "google/protobuf/unittest_mset_wire_format.proto",
    "google/protobuf/unittest_no_field_presence.proto",
    "google/protobuf/unittest_no_generic_services.proto",
    "google/protobuf/unittest_optimize_for.proto",
    "google/protobuf/unittest_preserve_unknown_enum.proto",
    "google/protobuf/unittest_preserve_unknown_enum2.proto",
    "google/protobuf/unittest_proto3.proto",
    "google/protobuf/unittest_proto3_arena.proto",
    "google/protobuf/unittest_proto3_arena_lite.proto",
    "google/protobuf/unittest_proto3_lite.proto",
    "google/protobuf/unittest_proto3_optional.proto",
    "google/protobuf/unittest_well_known_types.proto",
    "google/protobuf/util/internal/testdata/anys.proto",
    "google/protobuf/util/internal/testdata/books.proto",
    "google/protobuf/util/internal/testdata/default_value.proto",
    "google/protobuf/util/internal/testdata/default_value_test.proto",
    "google/protobuf/util/internal/testdata/field_mask.proto",
    "google/protobuf/util/internal/testdata/maps.proto",
    "google/protobuf/util/internal/testdata/oneofs.proto",
    "google/protobuf/util/internal/testdata/proto3.proto",
    "google/protobuf/util/internal/testdata/struct.proto",
    "google/protobuf/util/internal/testdata/timestamp_duration.proto",
    "google/protobuf/util/internal/testdata/wrappers.proto",
    "google/protobuf/util/json_format.proto",
    "google/protobuf/util/json_format_proto3.proto",
    "google/protobuf/util/message_differencer_unittest.proto",
]

TEST_PROTOS = ["src/" + s for s in RELATIVE_TEST_PROTOS]

GENERIC_RELATIVE_TEST_PROTOS = [
    "google/protobuf/map_proto2_unittest.proto",
    "google/protobuf/map_unittest.proto",
    "google/protobuf/unittest.proto",
    "google/protobuf/unittest_arena.proto",
    "google/protobuf/unittest_custom_options.proto",
    "google/protobuf/unittest_drop_unknown_fields.proto",
    "google/protobuf/unittest_embed_optimize_for.proto",
    "google/protobuf/unittest_empty.proto",
    "google/protobuf/unittest_enormous_descriptor.proto",
    "google/protobuf/unittest_import.proto",
    "google/protobuf/unittest_import_public.proto",
    "google/protobuf/unittest_lazy_dependencies.proto",
    "google/protobuf/unittest_lazy_dependencies_custom_option.proto",
    "google/protobuf/unittest_lazy_dependencies_enum.proto",
    "google/protobuf/unittest_lite_imports_nonlite.proto",
    "google/protobuf/unittest_mset.proto",
    "google/protobuf/unittest_mset_wire_format.proto",
    "google/protobuf/unittest_no_field_presence.proto",
    "google/protobuf/unittest_no_generic_services.proto",
    "google/protobuf/unittest_optimize_for.proto",
    "google/protobuf/unittest_preserve_unknown_enum.proto",
    "google/protobuf/unittest_preserve_unknown_enum2.proto",
    "google/protobuf/unittest_proto3.proto",
    "google/protobuf/unittest_proto3_arena.proto",
    "google/protobuf/unittest_proto3_arena_lite.proto",
    "google/protobuf/unittest_proto3_lite.proto",
    "google/protobuf/unittest_proto3_optional.proto",
    "google/protobuf/unittest_well_known_types.proto",
]

GENERIC_TEST_PROTOS = ["src/" + s for s in GENERIC_RELATIVE_TEST_PROTOS]

proto_library(
    name = "generic_test_protos",
    srcs = LITE_TEST_PROTOS + GENERIC_TEST_PROTOS,
    strip_import_prefix = "src",
    visibility = ["//:__subpackages__"],
    deps = [
        "//:any_proto",
        "//:api_proto",
        "//:descriptor_proto",
        "//:duration_proto",
        "//:empty_proto",
        "//:field_mask_proto",
        "//:source_context_proto",
        "//:struct_proto",
        "//:timestamp_proto",
        "//:type_proto",
        "//:wrappers_proto",
    ],
)

cc_proto_library(
    name = "cc_test_protos",
    srcs = LITE_TEST_PROTOS + TEST_PROTOS,
    include = "src",
    default_runtime = ":protobuf",
    protoc = ":protoc",
    deps = [":cc_wkt_protos"],
)

COMMON_TEST_SRCS = [
    # AUTOGEN(common_test_srcs)
    "src/google/protobuf/arena_test_util.cc",
    "src/google/protobuf/map_lite_test_util.cc",
    "src/google/protobuf/test_util_lite.cc",
    "src/google/protobuf/map_test_util.inc",
    "src/google/protobuf/reflection_tester.cc",
    "src/google/protobuf/test_util.cc",
    "src/google/protobuf/test_util.inc",
    "src/google/protobuf/testing/file.cc",
    "src/google/protobuf/testing/googletest.cc",
]

cc_binary(
    name = "test_plugin",
    testonly = True,
    srcs = [
        # AUTOGEN(test_plugin_srcs)
        "src/google/protobuf/compiler/mock_code_generator.cc",
        "src/google/protobuf/compiler/test_plugin.cc",
        "src/google/protobuf/testing/file.cc",
    ],
    deps = [
        ":protobuf",
        ":protoc_lib",
        "@com_google_googletest//:gtest",
    ],
)

cc_test(
    name = "win32_test",
    srcs = ["src/google/protobuf/io/io_win32_unittest.cc"],
    tags = [
        "manual",
        "windows",
    ],
    deps = [
        ":protobuf_lite",
        "@com_google_googletest//:gtest",
        "@com_google_googletest//:gtest_main",
    ],
)

cc_test(
    name = "protobuf_test",
    srcs = COMMON_TEST_SRCS + [
        # AUTOGEN(test_srcs)
        "src/google/protobuf/any_test.cc",
        "src/google/protobuf/arena_unittest.cc",
        "src/google/protobuf/arenastring_unittest.cc",
        "src/google/protobuf/arenaz_sampler_test.cc",
        "src/google/protobuf/compiler/annotation_test_util.cc",
        "src/google/protobuf/compiler/command_line_interface_unittest.cc",
        "src/google/protobuf/compiler/cpp/bootstrap_unittest.cc",
        "src/google/protobuf/compiler/cpp/metadata_test.cc",
        "src/google/protobuf/compiler/cpp/move_unittest.cc",
        "src/google/protobuf/compiler/cpp/plugin_unittest.cc",
        "src/google/protobuf/compiler/cpp/unittest.cc",
        "src/google/protobuf/compiler/cpp/unittest.inc",
        "src/google/protobuf/compiler/csharp/csharp_bootstrap_unittest.cc",
        "src/google/protobuf/compiler/csharp/csharp_generator_unittest.cc",
        "src/google/protobuf/compiler/importer_unittest.cc",
        "src/google/protobuf/compiler/java/doc_comment_unittest.cc",
        "src/google/protobuf/compiler/java/plugin_unittest.cc",
        "src/google/protobuf/compiler/mock_code_generator.cc",
        "src/google/protobuf/compiler/objectivec/objectivec_helpers_unittest.cc",
        "src/google/protobuf/compiler/parser_unittest.cc",
        "src/google/protobuf/compiler/python/plugin_unittest.cc",
        "src/google/protobuf/compiler/ruby/ruby_generator_unittest.cc",
        "src/google/protobuf/descriptor_database_unittest.cc",
        "src/google/protobuf/descriptor_unittest.cc",
        "src/google/protobuf/drop_unknown_fields_test.cc",
        "src/google/protobuf/dynamic_message_unittest.cc",
        "src/google/protobuf/extension_set_unittest.cc",
        "src/google/protobuf/generated_message_reflection_unittest.cc",
        "src/google/protobuf/generated_message_tctable_lite_test.cc",
        "src/google/protobuf/inlined_string_field_unittest.cc",
        "src/google/protobuf/io/coded_stream_unittest.cc",
        "src/google/protobuf/io/io_win32_unittest.cc",
        "src/google/protobuf/io/printer_unittest.cc",
        "src/google/protobuf/io/tokenizer_unittest.cc",
        "src/google/protobuf/io/zero_copy_stream_unittest.cc",
        "src/google/protobuf/map_field_test.cc",
        "src/google/protobuf/map_test.cc",
        "src/google/protobuf/map_test.inc",
        "src/google/protobuf/message_unittest.cc",
        "src/google/protobuf/message_unittest.inc",
        "src/google/protobuf/no_field_presence_test.cc",
        "src/google/protobuf/preserve_unknown_enum_test.cc",
        "src/google/protobuf/proto3_arena_lite_unittest.cc",
        "src/google/protobuf/proto3_arena_unittest.cc",
        "src/google/protobuf/proto3_lite_unittest.cc",
        "src/google/protobuf/proto3_lite_unittest.inc",
        "src/google/protobuf/reflection_ops_unittest.cc",
        "src/google/protobuf/repeated_field_reflection_unittest.cc",
        "src/google/protobuf/repeated_field_unittest.cc",
        "src/google/protobuf/stubs/bytestream_unittest.cc",
        "src/google/protobuf/stubs/common_unittest.cc",
        "src/google/protobuf/stubs/int128_unittest.cc",
        "src/google/protobuf/stubs/status_test.cc",
        "src/google/protobuf/stubs/statusor_test.cc",
        "src/google/protobuf/stubs/stringpiece_unittest.cc",
        "src/google/protobuf/stubs/stringprintf_unittest.cc",
        "src/google/protobuf/stubs/structurally_valid_unittest.cc",
        "src/google/protobuf/stubs/strutil_unittest.cc",
        "src/google/protobuf/stubs/template_util_unittest.cc",
        "src/google/protobuf/stubs/time_test.cc",
        "src/google/protobuf/text_format_unittest.cc",
        "src/google/protobuf/unknown_field_set_unittest.cc",
        "src/google/protobuf/util/delimited_message_util_test.cc",
        "src/google/protobuf/util/field_comparator_test.cc",
        "src/google/protobuf/util/field_mask_util_test.cc",
        "src/google/protobuf/util/internal/default_value_objectwriter_test.cc",
        "src/google/protobuf/util/internal/json_objectwriter_test.cc",
        "src/google/protobuf/util/internal/json_stream_parser_test.cc",
        "src/google/protobuf/util/internal/protostream_objectsource_test.cc",
        "src/google/protobuf/util/internal/protostream_objectwriter_test.cc",
        "src/google/protobuf/util/internal/type_info_test_helper.cc",
        "src/google/protobuf/util/json_util_test.cc",
        "src/google/protobuf/util/message_differencer_unittest.cc",
        "src/google/protobuf/util/time_util_test.cc",
        "src/google/protobuf/util/type_resolver_util_test.cc",
        "src/google/protobuf/well_known_types_unittest.cc",
        "src/google/protobuf/wire_format_unittest.cc",
        "src/google/protobuf/wire_format_unittest.inc",
    ] + select({
        "//conditions:default": [
            # AUTOGEN(non_msvc_test_srcs)
        ],
        ":msvc": [],
    }),
    copts = COPTS + select({
        ":msvc": [],
        "//conditions:default": [
            "-Wno-deprecated-declarations",
        ],
    }),
    data = [
        ":test_plugin",
    ] + glob([
        "src/google/protobuf/**/*",
        # Files for csharp_bootstrap_unittest.cc.
        "conformance/**/*",
        "csharp/src/**/*",
    ]),
    includes = [
        "src/",
    ],
    linkopts = LINK_OPTS,
    deps = [
        ":cc_test_protos",
        ":protobuf",
        ":protoc_lib",
        "@com_google_googletest//:gtest",
        "@com_google_googletest//:gtest_main",
    ] + PROTOBUF_DEPS,
)

################################################################################
# Java support
################################################################################

internal_gen_well_known_protos_java(
    name = "gen_well_known_protos_java",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [proto + "_proto" for proto in WELL_KNOWN_PROTO_MAP.keys()],
)

internal_gen_well_known_protos_java(
    name = "gen_well_known_protos_javalite",
    javalite = True,
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [proto + "_proto" for proto in LITE_WELL_KNOWN_PROTO_MAP.keys()],
)

internal_gen_kt_protos(
    name = "gen_well_known_protos_kotlin",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [proto + "_proto" for proto in WELL_KNOWN_PROTO_MAP.keys()],
)

internal_gen_kt_protos(
    name = "gen_well_known_protos_kotlinlite",
    lite = True,
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [proto + "_proto" for proto in LITE_WELL_KNOWN_PROTO_MAP.keys()],
)

alias(
    name = "protobuf_java",
    actual = "//java/core",
    visibility = ["//visibility:public"],
)

alias(
    name = "protobuf_javalite",
    actual = "//java/lite",
    visibility = ["//visibility:public"],
)

alias(
    name = "protobuf_java_util",
    actual = "//java/util",
    visibility = ["//visibility:public"],
)

alias(
    name = "java_toolchain",
    actual = "//java/core:toolchain",
    visibility = ["//visibility:public"],
)

alias(
    name = "javalite_toolchain",
    actual = "//java/lite:toolchain",
    visibility = ["//visibility:public"],
)

################################################################################
# Python support
################################################################################

py_library(
    name = "python_srcs",
    srcs = glob(
        [
            "python/google/protobuf/**/*.py",
        ],
    ),
    imports = ["python"],
    srcs_version = "PY2AND3",
    visibility = ["@upb//:__subpackages__"],
)

py_library(
    name = "python_test_srcs",
    srcs = glob(
        [
            "python/google/protobuf/internal/*_test.py",
            "python/google/protobuf/internal/test_util.py",
        ],
    ),
    imports = ["python"],
    srcs_version = "PY3",
    visibility = ["@upb//:__subpackages__"],
)

cc_binary(
    name = "python/google/protobuf/internal/_api_implementation.so",
    srcs = ["python/google/protobuf/internal/api_implementation.cc"],
    copts = COPTS + [
        "-DPYTHON_PROTO2_CPP_IMPL_V2",
    ],
    linkshared = 1,
    linkstatic = 1,
    tags = [
        # Exclude this target from wildcard expansion (//...) because it may
        # not even be buildable. It will be built if it is needed according
        # to :use_fast_cpp_protos.
        # https://docs.bazel.build/versions/master/be/common-definitions.html#common-attributes
        "manual",
    ],
    deps = select({
        "//conditions:default": [],
        ":use_fast_cpp_protos": ["//external:python_headers"],
    }),
)

cc_binary(
    name = "python/google/protobuf/pyext/_message.so",
    srcs = glob([
        "python/google/protobuf/pyext/*.cc",
        "python/google/protobuf/pyext/*.h",
    ]),
    copts = COPTS + [
        "-DGOOGLE_PROTOBUF_HAS_ONEOF=1",
    ] + select({
        "//conditions:default": [],
        ":allow_oversize_protos": ["-DPROTOBUF_PYTHON_ALLOW_OVERSIZE_PROTOS=1"],
    }),
    includes = [
        "python/",
        "src/",
    ],
    linkshared = 1,
    linkstatic = 1,
    tags = [
        # Exclude this target from wildcard expansion (//...) because it may
        # not even be buildable. It will be built if it is needed according
        # to :use_fast_cpp_protos.
        # https://docs.bazel.build/versions/master/be/common-definitions.html#common-attributes
        "manual",
    ],
    deps = [
        ":protobuf",
        ":proto_api",
    ] + select({
        "//conditions:default": [],
        ":use_fast_cpp_protos": ["//external:python_headers"],
    }),
)

config_setting(
    name = "use_fast_cpp_protos",
    values = {
        "define": "use_fast_cpp_protos=true",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

config_setting(
    name = "allow_oversize_protos",
    values = {
        "define": "allow_oversize_protos=true",
    },
    visibility = [
        # Public, but Protobuf only visibility.
        "//:__subpackages__",
    ],
)

# Copy the builtin proto files from src/google/protobuf to
# python/google/protobuf. This way, the generated Python sources will be in the
# same directory as the Python runtime sources. This is necessary for the
# modules to be imported correctly since they are all part of the same Python
# package.
internal_copied_filegroup(
    name = "protos_python",
    srcs = WELL_KNOWN_PROTOS,
    dest = "python",
    strip_prefix = "src",
)

# TODO(dzc): Remove this once py_proto_library can have labels in srcs, in
# which case we can simply add :protos_python in srcs.
COPIED_WELL_KNOWN_PROTOS = ["python/" + s[4:] for s in WELL_KNOWN_PROTOS]

py_proto_library(
    name = "well_known_types_py_pb2",
    srcs = COPIED_WELL_KNOWN_PROTOS,
    include = "python",
    default_runtime = "",
    protoc = ":protoc",
    srcs_version = "PY2AND3",
    visibility = ["@upb//:__subpackages__"],
)

py_library(
    name = "protobuf_python",
    deps = [
        ":well_known_types_py_pb2",
        ":python_srcs",
    ],
    data = select({
        "//conditions:default": [],
        ":use_fast_cpp_protos": [
            ":python/google/protobuf/internal/_api_implementation.so",
            ":python/google/protobuf/pyext/_message.so",
        ],
    }),
)

# Copy the test proto files from src/google/protobuf to
# python/google/protobuf. This way, the generated Python sources will be in the
# same directory as the Python runtime sources. This is necessary for the
# modules to be imported correctly by the tests since they are all part of the
# same Python package.
internal_copied_filegroup(
    name = "protos_python_test",
    srcs = LITE_TEST_PROTOS + TEST_PROTOS,
    dest = "python",
    strip_prefix = "src",
)

# TODO(dzc): Remove this once py_proto_library can have labels in srcs, in
# which case we can simply add :protos_python_test in srcs.
COPIED_LITE_TEST_PROTOS = ["python/" + s for s in RELATIVE_LITE_TEST_PROTOS]

COPIED_TEST_PROTOS = ["python/" + s for s in RELATIVE_TEST_PROTOS]

py_proto_library(
    name = "python_common_test_protos",
    srcs = COPIED_LITE_TEST_PROTOS + COPIED_TEST_PROTOS,
    include = "python",
    default_runtime = "",
    protoc = ":protoc",
    srcs_version = "PY2AND3",
    deps = [":well_known_types_py_pb2"],
    visibility = ["//visibility:public"],
)

py_proto_library(
    name = "python_specific_test_protos",
    srcs = glob([
        "python/google/protobuf/internal/*.proto",
        "python/google/protobuf/internal/import_test_package/*.proto",
    ]),
    include = "python",
    default_runtime = ":protobuf_python",
    protoc = ":protoc",
    srcs_version = "PY2AND3",
    deps = [":python_common_test_protos"],
    visibility = ["//visibility:public"],
)

py_library(
    name = "python_tests",
    srcs = glob(
        [
            "python/google/protobuf/internal/*_test.py",
            "python/google/protobuf/internal/test_util.py",
            "python/google/protobuf/internal/import_test_package/__init__.py",
        ],
    ),
    imports = ["python"],
    srcs_version = "PY2AND3",
    deps = [
        ":protobuf_python",
        ":python_common_test_protos",
        ":python_specific_test_protos",
    ],
)

internal_protobuf_py_tests(
    name = "python_tests_batch",
    data = glob([
        "src/google/protobuf/**/*",
    ]),
    modules = [
        "descriptor_database_test",
        "descriptor_pool_test",
        "descriptor_test",
        "generator_test",
        "json_format_test",
        "message_factory_test",
        "message_test",
        "proto_builder_test",
        "reflection_test",
        "service_reflection_test",
        "symbol_database_test",
        "text_encoding_test",
        "text_format_test",
        "unknown_fields_test",
        "wire_format_test",
    ],
    deps = [":python_tests"],
)

cc_library(
    name = "proto_api",
    hdrs = ["python/google/protobuf/proto_api.h"],
    visibility = ["//visibility:public"],
    deps = [
        "//external:python_headers",
    ],
)

proto_lang_toolchain(
    name = "cc_toolchain",
    blacklisted_protos = [proto + "_proto" for proto in WELL_KNOWN_PROTO_MAP.keys()],
    command_line = "--cpp_out=$(OUT)",
    runtime = ":protobuf",
    visibility = ["//visibility:public"],
)

alias(
    name = "objectivec",
    actual = "//objectivec",
    visibility = ["//visibility:public"],
)

alias(
    name = "protobuf_objc",
    actual = "//objectivec",
    visibility = ["//visibility:public"],
)

################################################################################
# Test generated proto support
################################################################################

genrule(
    name = "generated_protos",
    srcs = ["src/google/protobuf/unittest_import.proto"],
    outs = ["unittest_gen.proto"],
    cmd = "cat $(SRCS) | sed 's|google/|src/google/|' >  $(OUTS)",
)

proto_library(
    name = "generated_protos_proto",
    srcs = [
        "src/google/protobuf/unittest_import_public.proto",
        "unittest_gen_import.proto",
    ],
)

py_proto_library(
    name = "generated_protos_py",
    srcs = [
        "src/google/protobuf/unittest_import_public.proto",
        "unittest_gen_import.proto",
    ],
    default_runtime = "",
    protoc = ":protoc",
)

################################################################################
# Conformance tests
################################################################################

proto_library(
    name = "test_messages_proto2_proto",
    srcs = ["src/google/protobuf/test_messages_proto2.proto"],
    visibility = ["//visibility:public"],
)

proto_library(
    name = "test_messages_proto3_proto",
    srcs = ["src/google/protobuf/test_messages_proto3.proto"],
    visibility = ["//visibility:public"],
    deps = [
        ":any_proto",
        ":duration_proto",
        ":field_mask_proto",
        ":struct_proto",
        ":timestamp_proto",
        ":wrappers_proto",
    ],
)

cc_proto_library(
    name = "test_messages_proto2_proto_cc",
    srcs = ["src/google/protobuf/test_messages_proto2.proto"],
)

cc_proto_library(
    name = "test_messages_proto3_proto_cc",
    srcs = ["src/google/protobuf/test_messages_proto3.proto"],
    deps = [
        ":cc_wkt_protos",
    ],
)

proto_library(
    name = "conformance_proto",
    srcs = ["conformance/conformance.proto"],
    visibility = ["//visibility:public"],
)

cc_proto_library(
    name = "conformance_proto_cc",
    srcs = ["conformance/conformance.proto"],
)

cc_library(
    name = "jsoncpp",
    srcs = ["conformance/third_party/jsoncpp/jsoncpp.cpp"],
    hdrs = ["conformance/third_party/jsoncpp/json.h"],
    includes = ["conformance"],
)

cc_library(
    name = "conformance_test",
    srcs = [
        "conformance/conformance_test.cc",
        "conformance/conformance_test_runner.cc",
    ],
    hdrs = [
        "conformance/conformance_test.h",
    ],
    includes = [
        "conformance",
        "src",
    ],
    deps = [":conformance_proto_cc"],
)

cc_library(
    name = "binary_json_conformance_suite",
    srcs = ["conformance/binary_json_conformance_suite.cc"],
    hdrs = ["conformance/binary_json_conformance_suite.h"],
    deps = [
        ":conformance_test",
        ":jsoncpp",
        ":test_messages_proto2_proto_cc",
        ":test_messages_proto3_proto_cc",
    ],
)

cc_library(
    name = "text_format_conformance_suite",
    srcs = ["conformance/text_format_conformance_suite.cc"],
    hdrs = ["conformance/text_format_conformance_suite.h"],
    deps = [
        ":conformance_test",
        ":test_messages_proto2_proto_cc",
        ":test_messages_proto3_proto_cc",
    ],
)

cc_binary(
    name = "conformance_test_runner",
    srcs = ["conformance/conformance_test_main.cc"],
    visibility = ["//visibility:public"],
    deps = [
        ":binary_json_conformance_suite",
        ":conformance_test",
        ":text_format_conformance_suite",
    ],
)

# TODO: re-enable this test if appropriate, or replace with something that
# uses the new setup.
# sh_test(
#     name = "build_files_updated_unittest",
#     srcs = [
#         "build_files_updated_unittest.sh",
#     ],
#     data = [
#         "BUILD",
#         "cmake/extract_includes.bat.in",
#         "cmake/libprotobuf.cmake",
#         "cmake/libprotobuf-lite.cmake",
#         "cmake/libprotoc.cmake",
#         "cmake/tests.cmake",
#         "src/Makefile.am",
#         "update_file_lists.sh",
#     ],
# )

java_proto_library(
    name = "java_test_protos",
    visibility = ["//java:__subpackages__"],
    deps = [":generic_test_protos"],
)

java_lite_proto_library(
    name = "java_lite_test_protos",
    visibility = ["//java:__subpackages__"],
    deps = [":generic_test_protos"],
)

java_proto_library(
    name = "test_messages_proto2_java_proto",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [":test_messages_proto2_proto"],
)

java_proto_library(
    name = "test_messages_proto3_java_proto",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [":test_messages_proto3_proto"],
)

java_proto_library(
    name = "conformance_java_proto",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [":conformance_proto"],
)

java_lite_proto_library(
    name = "test_messages_proto2_java_proto_lite",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [":test_messages_proto2_proto"],
)

java_lite_proto_library(
    name = "conformance_java_proto_lite",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [":conformance_proto"],
)

java_lite_proto_library(
    name = "test_messages_proto3_java_proto_lite",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [":test_messages_proto3_proto"],
)

java_binary(
    name = "conformance_java",
    srcs = ["conformance/ConformanceJava.java"],
    main_class = "ConformanceJava",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [
        ":conformance_java_proto",
        ":test_messages_proto2_java_proto",
        ":test_messages_proto3_java_proto",
        "//:protobuf_java",
        "//:protobuf_java_util",
    ],
)

java_binary(
    name = "conformance_java_lite",
    srcs = ["conformance/ConformanceJavaLite.java"],
    main_class = "ConformanceJavaLite",
    visibility = [
        "//java:__subpackages__",
    ],
    deps = [
        ":conformance_java_proto_lite",
        ":test_messages_proto2_java_proto_lite",
        ":test_messages_proto3_java_proto_lite",
        "//:protobuf_java_util",
        "//:protobuf_javalite",
    ],
)

exports_files([
    "conformance/conformance_test_runner.sh",
    "conformance/failure_list_java.txt",
    "conformance/failure_list_java_lite.txt",
    "conformance/text_format_failure_list_java.txt",
    "conformance/text_format_failure_list_java_lite.txt",
])

filegroup(
    name = "bzl_srcs",
    srcs = glob(["**/*.bzl"]),
    visibility = ["//visibility:public"],
)

# Kotlin proto rules

proto_library(
    name = "kt_unittest_lite",
    srcs = [
        "src/google/protobuf/map_lite_unittest.proto",
        "src/google/protobuf/unittest_import_lite.proto",
        "src/google/protobuf/unittest_import_public_lite.proto",
        "src/google/protobuf/unittest_lite.proto",
    ],
    strip_import_prefix = "src",
)

internal_gen_kt_protos(
    name = "gen_kotlin_unittest_lite",
    lite = True,
    visibility = ["//java:__subpackages__"],
    deps = [":kt_unittest_lite"],
)

proto_library(
    name = "kt_unittest",
    srcs = [
        "src/google/protobuf/map_proto2_unittest.proto",
        "src/google/protobuf/unittest.proto",
        "src/google/protobuf/unittest_import.proto",
        "src/google/protobuf/unittest_import_public.proto",
    ],
    strip_import_prefix = "src",
)

internal_gen_kt_protos(
    name = "gen_kotlin_unittest",
    visibility = ["//java:__subpackages__"],
    deps = [":kt_unittest"],
)

proto_library(
    name = "kt_proto3_unittest",
    srcs = [
        "src/google/protobuf/unittest_import.proto",
        "src/google/protobuf/unittest_import_public.proto",
        "src/google/protobuf/unittest_proto3.proto",
    ],
    strip_import_prefix = "src",
)

internal_gen_kt_protos(
    name = "gen_kotlin_proto3_unittest_lite",
    lite = True,
    visibility = ["//java:__subpackages__"],
    deps = [":kt_proto3_unittest"],
)

internal_gen_kt_protos(
    name = "gen_kotlin_proto3_unittest",
    visibility = ["//java:__subpackages__"],
    deps = [":kt_proto3_unittest"],
)

################################################################################
# Packaging rules
################################################################################

# Files included in all source distributions
pkg_files(
    name = "common_dist_files",
    srcs = glob([
        "*.bzl",
        "cmake/*.cmake",
        "cmake/*.in",
        "editors/*",

        # Several of these files are generated by autogen.sh, so using
        # glob() lets us ignore them if they are missing. (This is not good
        # practice, though.)
        "Makefile.in",
        "aclocal.m4",
        "ar-lib",
        "compile",
        "config*",
        "depcomp",
        "install-sh",
        "ltmain.sh",
        "m4/*.m4",
        "missing",
        "protobuf*.pc.in",
        "test-driver",
    ]) + [
        "BUILD",
        "CHANGES.txt",
        "CMakeLists.txt",
        "CONTRIBUTORS.txt",
        "LICENSE",
        "Makefile.am",
        "README.md",
        "WORKSPACE",
        "autogen.sh",
        "build_files_updated_unittest.sh",
        "cmake/CMakeLists.txt",
        "cmake/README.md",
        "generate_descriptor_proto.sh",
        "maven_install.json",
        "update_file_lists.sh",
        "//third_party:zlib.BUILD",
        "//util/python:BUILD",
    ],
    strip_prefix = strip_prefix.from_root(""),
    visibility = ["//pkg:__pkg__"],
)

# Conformance tests
pkg_files(
    name = "conformance_dist_files",
    srcs = glob(
        ["conformance/**/*"],
        exclude = [
            # The following are not in autotools dist:
            "conformance/autoload.php",
            "conformance/conformance_nodejs.js",
            "conformance/conformance_test_runner.sh",
            "conformance/failure_list_java_lite.txt",
            "conformance/failure_list_jruby.txt",
            "conformance/text_format_failure_list_*.txt",
            "conformance/update_failure_list.py",
        ],
    ),
    strip_prefix = strip_prefix.from_root(""),
    visibility = ["//pkg:__pkg__"],
)

# C++ runtime
pkg_files(
    name = "cpp_dist_files",
    srcs = glob(
        ["src/**/*"],
        exclude = [
            "src/google/protobuf/compiler/objectivec/method_dump.sh",  # not in autotools dist
        ],
    ),
    strip_prefix = strip_prefix.from_root(""),
    visibility = ["//pkg:__pkg__"],
)

# Additional files for C#
pkg_files(
    name = "csharp_dist_files",
    srcs = [
        "global.json",
    ],
    visibility = ["//pkg:__pkg__"],
)

# Additional files for ObjC
pkg_files(
    name = "objectivec_dist_files",
    srcs = [
        "Protobuf.podspec",
    ],
    visibility = ["//pkg:__pkg__"],
)

# Additional files for PHP
pkg_files(
    name = "php_dist_files",
    srcs = [
        "composer.json",
    ],
    visibility = ["//pkg:__pkg__"],
)

# Python runtime
pkg_files(
    name = "python_dist_files",
    srcs = glob([
        "python/google/**/*.proto",
        "python/google/**/*.py",
        "python/google/protobuf/internal/*.cc",
        "python/google/protobuf/pyext/*.cc",
        "python/google/protobuf/pyext/*.h",
    ]) + [
        "python/MANIFEST.in",
        "python/README.md",
        "python/google/protobuf/proto_api.h",
        "python/google/protobuf/pyext/README",
        "python/google/protobuf/python_protobuf.h",
        "python/mox.py",
        "python/release.sh",
        "python/setup.cfg",
        "python/setup.py",
        "python/stubout.py",
        "python/tox.ini",
    ],
    strip_prefix = strip_prefix.from_root(""),
    visibility = ["//pkg:__pkg__"],
)
