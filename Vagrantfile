# -*- mode: ruby -*-
# vi: set ft=ruby :

# This Vagrantfile is essentially a build script: Creating the machine with
# `vagrant up` will build a GCC toolchain for building Zcash and save it to
# zcash_cc_toolchain.tar.xz in the same directory as this file. It tries to
# make a reproducible build: Running it again on another machine should produce
# bit-identical output.
#
# The toolchain built is a GCC toolchain where both the host and the target is
# x86_64 linux using glibc. It does not use headers or libraries from
# system-wide search paths, making it suitable for build systems that perform
# reproducible builds.

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provision "shell", inline: <<-SHELL
    #!/bin/bash
    set -e

    apt-get install -y gcc g++ gperf bison flex texinfo help2man make \
        libncurses5-dev python-dev libstdc++-4.8-dev git autoconf

    su vagrant <<'INNER_SHELL'
      #!/bin/bash
      set -e

      git clone https://github.com/ReproducibleBuilds/strip-nondeterminism.git
      cd strip-nondeterminism
      git checkout 3bff3ec0e4393863ceeacc19093fdec23522e1b4
      cd ..

      git clone https://github.com/crosstool-ng/crosstool-ng.git
      cd crosstool-ng
      git checkout adaa3a5d8b4e2834a8c2c79efcdd3c718236ba5a

      ./bootstrap
      ./configure --enable-local
      make
      cp /vagrant/crosstool.config .config
      ./ct-ng build

      TOOLCHAIN=x86_64-unknown-linux-gnu

      mkdir ~/zcash_cc_toolchain
      mv ~/x-tools/$TOOLCHAIN ~/zcash_cc_toolchain/
      cd ~/zcash_cc_toolchain

      # Prepare Bazel package
      touch WORKSPACE
      cp /vagrant/tmpl/CROSSTOOL .
      # A bug in Bazel requires this BUILD file to not have a .bazel extension.
      cp /vagrant/tmpl/BUILD .
      cp /vagrant/tmpl/crosstool.bazel $TOOLCHAIN/BUILD.bazel

      # Remove the log of building the toolchain, it's nondeterministic.
      mv $TOOLCHAIN/build.log.bz2 ~/

      # Strip timestamps from .a static libraries.
      find $TOOLCHAIN -name '*.a' | PERLLIB=~/strip-nondeterminism/lib xargs ~/strip-nondeterminism/bin/strip-nondeterminism

      cd ~
      tar cJf zcash_cc_toolchain.tar.xz zcash_cc_toolchain
      mv zcash_cc_toolchain.tar.xz /vagrant/
INNER_SHELL

  SHELL

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
  end
end
