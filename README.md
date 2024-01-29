# Pi-Pico-on-Ubuntu22.04
Installation steps for initial setup of the Raspberry pi pico with debugprobe on Ubuntu 22.04

## install pico sdk
Normal install as the pico sdk getting started guide instructs: https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf

    sudo apt update
    sudo apt install cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential libstdc++-arm-none-eabi-newlib binutils-multiarch gdb-multiarch git
    ln -s /usr/bin/objdump objdump-multiarch
    ln -s /usr/bin/nm nm-multiarch 
    mkdir -p  ~/pico && cd ~/pico/
    git clone -b master https://github.com/raspberrypi/pico-sdk.git
    cd pico-sdk && git submodule update --init
    echo 'export PICO_SDK_PATH=$HOME/pico/pico-sdk' >> ~/.bashrc 
    cd ~/pico && git clone https://github.com/raspberrypi/pico-examples.git --branch master
    cp -r ~/pico/pico-examples/ide/vscode ~/pico/pico-examples/.vscode
    cp -r ~/pico/pico-examples/.vscode/launch-probe-swd.json  ~/pico/pico-examples/.vscode/launch.json

- afterwards logout

## install openocd
We need a special version of openocd so we have to build it ourself:

    sudo apt install automake autoconf build-essential texinfo libtool libftdi-dev pkg-config
    sudo apt install libusb-1.0-0 libusb-1.0-0-dev
    cd ~/pico
    git clone https://github.com/raspberrypi/openocd.git --branch rp2040-v0.12.0 --depth=1 --no-single-branch    cd openocd
    ./bootstrap
    ./configure --enable-picoprobe 
    make -j4
    sudo make install
    sudo cp openocd/contrib/60-openocd.rules /etc/udev/rules.d/

## install picotool
We also need picotool:

    cd ~/pico
    git clone https://github.com/raspberrypi/picotool.git --branch master
    cd picotool && mkdir build && cd build
    cmake ../
    make

## Install vscode
We will install vs code through apt, since this makes less problems later on:

    sudo apt update
    sudo apt install apt-transport-https
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
    sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
    sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
    rm -f packages.microsoft.gpg
    sudo apt update
    sudo apt install code

## install vscode extensions

    code --install-extension marus25.cortex-debug
    code --install-extension ms-vscode.cmake-tools
    code --install-extension ms-vscode.cpptools

    cd ~/pico/pico-examples && code
    
## Inside VScode

- Add after first line of CMakeLists.txt
    set(PICO_BOARD pico_w)
    set(WIFI_SSID "TITUS-WLAN")
    set(WIFI_PASSWORD "password")
    set(TEST_TCP_SERVER_IP "10.0.0.1")
- Select cmaketools in the left bar
- Configure --> [no kit selected] --> edit (pen symbol next to Caption)
- select "GCC 10.3.1 arm-none-eabi" 
- Build --> click on Build Symbol (right to next to it)
- Select Run/Debug in the left bar
- Run --> select "picow_blink"


## Update Picoprobe
In case this is not working, it could be that the firmware of the porbe is outdated.

- get .uf2 from https://github.com/raspberrypi/picoprobe/releases/tag/picoprobe-cmsis-v1.1
- unplug the picoprobe
- press and hold the boot button on the picoprobe
- plug the picoprobe while holding the boot button
- place the .uf2 file on the pi
- wait for the device to disappear
- replug probe
