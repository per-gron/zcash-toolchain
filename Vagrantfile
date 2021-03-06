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
  config.vm.box_version = "= 20180110.0.0"

  config.vm.provision "shell", inline: <<-SHELL
    #!/bin/bash
    set -e

    apt-get install -y gcc g++ gperf bison flex texinfo help2man make \
        libncurses5-dev python-dev libstdc++-4.8-dev git autoconf faketime

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

      # Pull in a GCC patch to get deterministic builds
      cp /vagrant/1000-disable-genchecksum.patch packages/gcc/7.2.0/

      ./bootstrap
      ./configure --enable-local
      make
      cp /vagrant/crosstool.config .config
      faketime '2018-01-01 00:00:00' ./ct-ng build

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
      # Remove the nscd binary; the old-ish version of glibc used here uses
      # __DATE__ and __TIME__ in a .c file there (nscd_stat.c) which breaks
      # determinism. nscd is not used anyway so just remove it.
      rm $TOOLCHAIN/$TOOLCHAIN/sysroot/usr/sbin/nscd

      # Strip timestamps from .a static libraries. This is still needed even
      # though libfaketime is used because the mtimes of files in the archive
      # include the difference between when they were built and when the archive
      # was made.
      find $TOOLCHAIN -name '*.a' | PERLLIB=~/strip-nondeterminism/lib xargs ~/strip-nondeterminism/bin/strip-nondeterminism

      cd ~
      find zcash_cc_toolchain -print0 | LC_ALL=C sort -z |
          tar --null -T - --no-recursion \
              --owner=root --group=root --numeric-owner \
              --mode=go=rX,u+rw,a-s \
              --mtime='2018-01-01' \
              -c | xz > /vagrant/zcash_cc_toolchain.tar.xz
INNER_SHELL

  SHELL

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
  end
end
