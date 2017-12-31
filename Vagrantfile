# -*- mode: ruby -*-
# vi: set ft=ruby :

# This Vagrantfile is essentially a build script: Creating the machine with
# `vagrant up` will build a GCC toolchain for building Zcash and save it to
# x86_64-linux-musl.tar.xz in the same directory as this file. It tries to
# make a reproducible build: Running it again on another machine should produce
# bit-identical output.
#
# The toolchain built is a GCC toolchain where both the host and the target is
# x86_64 linux using musl as libc. It is tailored for creating fully statically
# linked PIE binaries. It does not use headers or libraries from system-wide
# search paths, making it suitable for build systems that perform reproducible
# builds.

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provision "shell", inline: <<-SHELL
    #!/bin/bash
    set -e

    apt-get install -y gcc g++ bison make git autoconf

    su vagrant <<'INNER_SHELL'
      #!/bin/bash
      set -e

      ### Configure everything

      git clone https://github.com/richfelker/musl-cross-make.git
      cd musl-cross-make
      git checkout b85e29c00d35c8c8c196d6713505b837816ad47f
      cat << 'EOF' > config.mak
TARGET = x86_64-linux-musl
GCC_VER = 7.2.0
GMP_VER = 6.1.1
MPC_VER = 1.0.3
MPFR_VER = 3.1.4
ISL_VER = 0.15
COMMON_CONFIG += --disable-nls  # Support only English, to save space
GCC_CONFIG += --enable-gold
GCC_CONFIG += --disable-multilib
GCC_CONFIG += --enable-languages=c,c++
GCC_CONFIG += --enable-default-pie

# Keep the local build path out of the toolchain binaries and target libraries
COMMON_CONFIG += --with-debug-prefix-map=$(CURDIR)=
EOF
      cd ..
      cp -R musl-cross-make musl-cross-make-stage2
      cd musl-cross-make-stage2
      cat << 'EOF' >> config.mak
COMMON_CONFIG += CC="/home/vagrant/stage1/bin/x86_64-linux-musl-gcc -static --static"
COMMON_CONFIG += CXX="/home/vagrant/stage1/bin/x86_64-linux-musl-g++ -static --static"
COMMON_CONFIG += LD="/home/vagrant/stage1/bin/x86_64-linux-musl-ld.gold"
COMMON_CONFIG += CFLAGS="-g0" CXXFLAGS="-g0" LDFLAGS="-s"
EOF
      cd ..

      ### Stage one build
      cd musl-cross-make
      make -j4
      make install
      mv output ../stage1
      cd ..
      # Steal over the downloads to not have to do them again
      cp -R musl-cross-make/sources musl-cross-make-stage2/
      rm -rf musl-cross-make

      ### Stage two build
      # The first stage compiler has known behavior, but its actual executable
      # files are dynamically linked and their contents depend on the built-in
      # compiler. The second stage compiler behaves exactly like stage one but
      # because it's built by a known compiler should be reproducible, hopefully
      # to the point of the output having the same hash if built again by
      # someone else.
      cd musl-cross-make-stage2
      make -j4
      make install
      mv output ../x86_64-linux-musl
      cd ..
      rm -rf musl-cross-make-stage2
      rm -r stage1

      ### Package
      tar cJf x86_64-linux-musl.tar.xz x86_64-linux-musl
      mv x86_64-linux-musl.tar.xz /vagrant/

INNER_SHELL

  SHELL

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
  end
end
