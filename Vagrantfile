# -*- mode: ruby -*-
# vi: set ft=ruby :

# MobileInsight Vagrant Installation Script
# Copyright (c) 2020 MobileInsight Team
# Author: Zengwen Yuan, Yunqi Guo
# Version: 2.0


# --------------------------------Wireshark for Android---------------------------------------#

$INSTALL_BASE_WS = <<SCRIPT
apt-get update
apt-get -y install build-essential pkg-config
apt-get -y install autoconf automake zlib1g-dev libtool
apt-get -y install bison byacc flex ccache
apt-get -y install unzip
apt -y install python3.8
apt-get -y install libncurses5
apt-get -y install cmake pkg-config wget libglib2.0-dev bison flex libpcap-dev libgcrypt-dev qt5-default qttools5-dev qtmultimedia5-dev libqt5svg5-dev libc-ares-dev libsdl2-mixer-2.0-0 libsdl2-image-2.0-0 libsdl2-2.0-0

alias python=python3

SCRIPT


$CREATE_NDK_TOOLCHAIN = <<SCRIPT
# Download and setup Android NDK r15c
cd ~
wget https://dl.google.com/android/repository/android-ndk-r15c-linux-x86_64.zip
unzip android-ndk-r15c-linux-x86_64.zip
echo 'export ANDROID_NDK_HOME=/home/vagrant/android-ndk-r15c' >> ~/.bashrc
echo 'PATH=$PATH:$ANDROID_NDK_HOME' >> ~/.bashrc
source ~/.bashrc
rm android-ndk-r15c-linux-x86_64.zip

# Create MobileInsight dev folder at /home/vagrant/mi-dev
cd ~/android-ndk-r15c
python3 build/tools/make_standalone_toolchain.py \
    --arch arm \
    --api 26 \
    --stl gnustl \
    --unified-headers \
    --install-dir /home/vagrant/android-ndk-toolchain

SCRIPT


$DOWNLOAD_TARBALLS = <<SCRIPT
cd ~
wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz
tar xf libiconv-1.15.tar.gz
rm libiconv-1.15.tar.gz

wget http://ftp.gnu.org/pub/gnu/gettext/gettext-0.19.8.tar.gz
tar xf gettext-0.19.8.tar.gz
rm gettext-0.19.8.tar.gz

wget https://gnupg.org/ftp/gcrypt/libgpg-error/libgpg-error-1.37.tar.gz
tar xf libgpg-error-1.37.tar.gz
rm libgpg-error-1.37.tar.gz

wget https://gnupg.org/ftp/gcrypt/libgcrypt/libgcrypt-1.8.1.tar.bz2
tar xf libgcrypt-1.8.1.tar.bz2
rm libgcrypt-1.8.1.tar.bz2

wget http://ftp.gnome.org/pub/gnome/sources/glib/2.54/glib-2.54.3.tar.xz
tar xf glib-2.54.3.tar.xz
rm glib-2.54.3.tar.xz

wget https://github.com/c-ares/c-ares/releases/download/cares-1_15_0/c-ares-1.15.0.tar.gz
tar -xf c-ares-1.15.0.tar.gz
rm c-ares-1.15.0.tar.gz

wget https://github.com/libffi/libffi/releases/download/v3.3/libffi-3.3.tar.gz
tar -xf libffi-3.3.tar.gz
rm libffi-3.3.tar.gz

ws_ver=3.4.0
wget  http://www.mobileinsight.net/wireshark-3.4.0-rbc-dissector.tar.xz -O wireshark-3.4.0.tar.xz
tar -xf wireshark-3.4.0.tar.xz
rm wireshark-3.4.0.tar.xz


wget http://www.tcpdump.org/release/libpcap-1.9.1.tar.gz
tar -xf libpcap-1.9.1.tar.gz
rm libpcap-1.9.1.tar.gz

# Apply the patch to wireshark
cp /vagrant/ws_android.patch ./
cd wireshark-3.4.0
patch -p1 < ../ws_android.patch

# Compile wireshark first time
cd tools/lemon
cmake .
make
cp lemon ~/
echo "lemon is generated"

# Import the environment settings
cd ~
cp  /vagrant/envsetup.sh .
chmod +x envsetup.sh
source ~/envsetup.sh

SCRIPT


$COMPILE_LIBICONV = <<SCRIPT
source ~/envsetup.sh
cd ~/libiconv-1.15
./configure --build=${BUILD_SYS} --host=arm-eabi --prefix=${PREFIX} --disable-rpath
make
make install
SCRIPT


$COMPILE_GETTEXT = <<SCRIPT
source ~/envsetup.sh
cd ~/gettext-0.19.8
./configure --build=${BUILD_SYS} --host=arm-eabi  --prefix=${PREFIX} --disable-rpath --disable-java --disable-native-java --disable-libasprintf --disable-openmp --disable-curses
make
make install

SCRIPT


$COMPILE_LIBGPGERROR = <<SCRIPT
source ~/envsetup.sh
cd ~/libgpg-error-1.37
./configure --build=${BUILD_SYS} --host=${TOOLCHAIN} --prefix=${PREFIX} --enable-static --disable-shared
make
make install

SCRIPT


$COMPILE_LIBGCRYPT = <<SCRIPT
source ~/envsetup.sh
cd ~/libgcrypt-1.8.1
./configure --build=${BUILD_SYS} --host=${TOOLCHAIN} --prefix=${PREFIX} --enable-static --disable-shared
make
make install

SCRIPT


$COMPILE_PCAP = <<SCRIPT
source ~/envsetup.sh
cd ~/libpcap-1.9.1
./configure --build=${BUILD_SYS} --host=${TOOLCHAIN} --prefix=${PREFIX} --enable-static --disable-shared
make
make install
SCRIPT


$COMPILE_GLIB = <<SCRIPT
# Install libffi
source ~/envsetup.sh
cd ~/libffi-3.3
./configure --build=${BUILD_SYS} --host=${TOOLCHAIN} --prefix=${PREFIX} --enable-static --disable-shared
make
make install

cd ~/c-ares-1.15.0/
unset CFLAGS
./configure --build=${BUILD_SYS} --host=${TOOLCHAIN} --prefix=${PREFIX} --enable-static --disable-shared
make
make install

# Reset the CFLAGS
source ~/envsetup.sh
cd ~/glib-2.54.3
cp  /vagrant/android.cache .
./configure --build=${BUILD_SYS} --host=${TOOLCHAIN} --prefix=${PREFIX} --disable-dependency-tracking --cache-file=android.cache --enable-included-printf --enable-static --with-pcre=no --disable-libmount
make
make install

SCRIPT


$COMPILE_WIRESHARK = <<SCRIPT

cp /vagrant/build_ws.sh ./
chmod +x build_ws.sh
./build_ws.sh

SCRIPT


$COPY_LIBS = <<SCRIPT
# Copy MobileInsight apk to local folder
cd ~
mkdir ws_lib
cd ws_lib
cp ~/androidcc/lib/libgio-2.0.so .
cp ~/androidcc/lib/libglib-2.0.so .
cp ~/androidcc/lib/libgobject-2.0.so .
cp ~/androidcc/lib/libgmodule-2.0.so .
cp ~/androidcc/lib/libgthread-2.0.so .
cp /usr/local/lib/libwireshark.so .
cp /usr/local/lib/libwiretap.so .
cp /usr/local/lib/libwsutil.so .

cp -r ~/ws_lib  /vagrant/
SCRIPT


$COMPILE_WS_DISSECTOR = <<SCRIPT
# Compile android_ws_dissector
cd ~/
git clone -b dev-6.0 https://github.com/mobile-insight/mobileinsight-core.git
cd mobileinsight-core/ws_dissector/
make android
mkdir ws_bin
cp android_* ws_bin/
cp -r ws_bin /vagrant/
SCRIPT





Vagrant.configure(2) do |config|
    # config.vm.box = "bento/ubuntu-16.04"
    # config.vm.box_version = "201708.22.0"
    config.vm.box = "bento/ubuntu-20.04"
    config.vm.box_version = "202004.27.0"

    config.vm.provider "virtualbox" do |vb|
        # # Display the VirtualBox GUI when booting the machine
        # vb.gui = true

        # Customize the amount of memory and cpus on the VM:
        vb.memory = "8192"
        vb.cpus = 12
    end

    config.vm.provision :shell, inline: "echo Building Wireshark for Android"
    config.vm.provision "shell", privileged: true, inline: $INSTALL_BASE_WS
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $CREATE_NDK_TOOLCHAIN
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $DOWNLOAD_TARBALLS
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_LIBICONV
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_GETTEXT
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_LIBGPGERROR
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_LIBGCRYPT
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_PCAP
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_GLIB
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_WIRESHARK
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COPY_LIBS
    config.vm.provision "shell", privileged: false, keep_color: true, inline: $COMPILE_WS_DISSECTOR
  end
end