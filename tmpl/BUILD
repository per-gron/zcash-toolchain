# This is the entry point for --crosstool_top.
#
# The cc_toolchain rule used is found by:
#
# 1. Finding the appropriate toolchain in the CROSSTOOL file based on the --cpu
#    and --compiler command line flags (if they exist, otherwise using the
#    "default_target_cpu" / "default_toolchain" fields in the CROSSTOOL file)
# 2. Concatenating the "target_cpu" and "compiler" fields of the toolchain in
#    use and using that as a key in the map in the "toolchains" attribute

cc_toolchain_suite(
    name = "cc_toolchain",
    visibility = ["//visibility:public"],
    toolchains = {
        "k8|compiler": ":cc-compiler-x86_64-unknown-linux-gnu",
    },
)

filegroup(
    name = "linux_gcc_all_files",
    srcs = [
        ":linux_gcc_linker_files",
        ":linux_gcc_compiler_files",
        "//x86_64-unknown-linux-gnu:compiler_pieces",
    ],
)

filegroup(
    name = "linux_gcc_linker_files",
    srcs = [
        "//x86_64-unknown-linux-gnu:gcc",
        "//x86_64-unknown-linux-gnu:ld",
        "//x86_64-unknown-linux-gnu:ar",
        "//x86_64-unknown-linux-gnu:compiler_pieces",
    ],
)

filegroup(
    name = "linux_gcc_compiler_files",
    srcs = [
        "//x86_64-unknown-linux-gnu:gcc",
        "//x86_64-unknown-linux-gnu:ld",
        "//x86_64-unknown-linux-gnu:as",
    ],
)

cc_toolchain(
    name = "cc-compiler-x86_64-unknown-linux-gnu",
    visibility = ["//visibility:public"],
    all_files = ":linux_gcc_all_files",
    compiler_files = ":linux_gcc_compiler_files",
    cpu = "k8",
    dwp_files = ":empty",
    dynamic_runtime_libs = [":empty"],
    linker_files = ":linux_gcc_linker_files",
    objcopy_files = "//x86_64-unknown-linux-gnu:objcopy",
    static_runtime_libs = [":empty"],
    strip_files = "//x86_64-unknown-linux-gnu:strip",
    supports_param_files = 1,
)
