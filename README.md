# zcash-toolchain

This repository contains scripts for building compiler toolchains for building
Zcash. The scripts use [Vagrant](https://www.vagrantup.com/) to ensure a stable
environment for the script to run in and
[crosstool-NG](http://crosstool-ng.github.io/) for actually building the
toolchain.

The scripts here attempt to build the toolchain deterministically; building
multiple times, even on different machines, should produce bit-identical output.
This makes it easier to verify that the build was done correctly and was not
tampered with.

To build, run `vagrant up`. This will take some time and eventually produce
a `zcash_cc_toolchain.tar.xz` file in the root directory of the repo. Currently
only Linux is supported as a target platform.
