#
# Copyright 2018 Asylo authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

load(
    "@com_google_asylo//asylo/bazel:asylo.bzl",
    "enclave_loader",
    "sim_enclave",
    "enclave_test",
)
load(
    "@com_google_asylo//asylo/bazel:proto.bzl",
    "asylo_proto_library",
    "asylo_grpc_proto_library",
)

licenses(["notice"])  # Apache v2.0

# Example demonstrating gRPC in Asylo.

package(
    default_visibility = ["//visibility:public"],
)

# The generated code for the translator server (and client).
asylo_grpc_proto_library(
    name = "translator_server_grpc_proto",
    srcs = ["translator_server.proto"],
)

# The implementation of the translation server.
cc_library(
    name = "translator_server",
    srcs = ["translator_server.cc"],
    hdrs = ["translator_server.h"],
    deps = [
        ":translator_server_grpc_proto",
        "@com_github_grpc_grpc//:grpc++",
        "@com_google_absl//absl/base:core_headers",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/synchronization",
    ],
)

# Contains extensions to enclave protos.
asylo_proto_library(
    name = "grpc_server_config_proto",
    srcs = ["grpc_server_config.proto"],
    deps = ["@com_google_asylo//asylo:enclave_proto"],
)

# The enclave hosting the translation server.
sim_enclave(
    name = "grpc_server_enclave.so",
    srcs = ["grpc_server_enclave.cc"],
    config = "@com_google_asylo//asylo/grpc/util:grpc_enclave_config",
    deps = [
        ":grpc_server_config_proto_cc",
        ":translator_server",
        "@com_github_grpc_grpc//:grpc++",
        "@com_github_grpc_grpc//:grpc++_reflection",
        "@com_google_absl//absl/base:core_headers",
        "@com_google_absl//absl/memory",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
        "@com_google_asylo//asylo:enclave_runtime",
        "@com_google_asylo//asylo/util:status",
    ],
)

# The driver for the gRPC server enclave.
enclave_loader(
    name = "grpc_server",
    srcs = ["grpc_server_driver.cc"],
    enclaves = {"enclave": ":grpc_server_enclave.so"},
    loader_args = ["--enclave_path='{enclave}'"],
    deps = [
        ":grpc_server_config_proto_cc",
        "@com_github_gflags_gflags//:gflags_nothreads",
        "@com_google_asylo//asylo:enclave_client",
        "@com_google_asylo//asylo/util:logging",
    ],
)

# A test of the example using the generated client stubs rather than the gRPC
# command-line interface.
enclave_test(
    name = "grpc_server_test",
    srcs = ["grpc_server_test.cc"],
    data = [":grpc_server_host_loader"],
    enclaves = {"server_enclave": ":grpc_server_enclave.so"},
    test_args = ["--enclave_path='{server_enclave}'"],
    deps = [
        ":translator_server_grpc_proto",
        "@com_github_gflags_gflags//:gflags_nothreads",
        "@com_github_grpc_grpc//:grpc++",
        "@com_google_absl//absl/base:core_headers",
        "@com_google_absl//absl/memory",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/synchronization",
        "@com_google_asylo//asylo/test/util:exec_tester",
        "@com_google_asylo//asylo/test/util:status_matchers",
        "@com_google_asylo//asylo/test/util:test_main",
        "@com_google_asylo//asylo/util:status",
        "@com_google_googletest//:gtest",
    ],
)
