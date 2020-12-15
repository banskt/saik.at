---
title: "How to control RGB colors of Wraith Prism in Ubuntu"
date: 2020-12-05T22:46:18+02:00
draft: false
---

I found three options to control the RGB colors of Wraith Prism on a Linux platform: 

1. [OpenRGB](https://gitlab.com/CalcProgrammer1/OpenRGB). RGB control application for multiple devices from diverse manufacturers. "One app to rule them all". Works on both Windows and Linux.
2. [Wraith Master](https://serebit.com/posts/wraith-master-hits-version-1.1/). Open-source Linux-only RGB control application for the Wraith Prism, written in Kotlin.
3. [cm-rgb](https://github.com/gfduszynski/cm-rgb). RGB control application for the Wraith Prism on Linux, Mac OS and Windows, written in Python and allows showing realtime CPU utilization.

I used OpenRGB as it will at least have a single application for all RGB devices if I plan to upgrade some components in the future. I used the USB connector for the RGB fan, because the RGB header on MSI MAG B550M Mortar motherboard is not yet supported by OpenRGB. Configuring OpenRGB from source requires the following dependencies: 

```
sudo apt install build-essential libusb-1.0-0-dev libhidapi-dev pkgconf
```

Installing OpenRGB also requires [Qt5](https://www.qt.io/) - a framework for cross-platform software development. Installing Qt5 from the Ubuntu package manager is [easy](https://stackoverflow.com/questions/48147356/install-qt-on-ubuntu):
```bash
sudo apt install qt5-default qtcreator
```
However, I often find pleasure in making things complicated, 
so I compiled Qt5 from source and kept it as a separate module which can be called only when required (see details below).
This is absolutely not required, it only gave me a sense of keeping things cleaner. 


Then, I downloaded OpenRGB from the [gitlab repository](https://gitlab.com/CalcProgrammer1/OpenRGB) 
and built it using qmake in a different `openrgb-build` directory. 
I prefer to keep the build separate from the source.
```bash
# clone from git; this should create a directory called 'OpenRGB'
git clone https://gitlab.com/CalcProgrammer1/OpenRGB.git
mkdir openrgb-build
cd openrgb-build
module load qt/5.14 # need the qmake
qmake ../OpenRGB/OpenRGB.pro
make -j8
sudo ln -s /home/saikat/Downloads/apps/openrgb/openrgb-build/openrgb /usr/local/bin/openrgb
```

There are two important steps after the installation: providing USB access and SMBus access (for controlling RGB RAM and certain motherboard on-board LEDs). 

**USB access**. The root user could now access the Wraith Prism using `sudo openrgb` . To make it accessible to the normal user, I copied the `60-openrgb.rules` file provided in the OpenRGB source code to `/etc/udev/rules.d` and reloaded the udev rules.

```
sudo cp 60-openrgb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules 
sudo udevadm trigger
```

**SMBus access**. I do not have any RGB via SMBus, so I did not need any configuration. However, detailed instructions are provided in the [OpenRGB Readme](https://gitlab.com/CalcProgrammer1/OpenRGB) and requires patching the kernel to include modules that provide `i2c` driver for several chipsets. One has to be careful to load the correct chipset driver. For example, the motherboard that I am using has Nuvoton NCT6687D-R chipset, which apparently does not have any `i2c` driver yet. There is also a [Github repository](https://github.com/CalcProgrammer1/openrgb-dkms-drivers) which provide DKMS (dynamic kernel module support) for some of these drivers.

## Install Qt5 from source

I have `environment-modules` already set up on my system and it would be simple to add another module for Qt. Therefore, I decided to install it from source. The [Qt5 wiki](https://wiki.qt.io/Main) is an awesome resource and provides [detailed instructions for building from source code](https://wiki.qt.io/Building_Qt_5_from_Git#Getting_the_source_code). After a few attempts of the whole process, I figured that the following were the prerequisites for configuring and installing Qt5 on my system. Note that this is not an exhaustive list of dependencies because some other dependencies are probably already installed on my system.

```
sudo apt install libudev-dev libxcb-xinerama0-dev
sudo apt install libgl1-mesa-dev
sudo ln -s /usr/lib/x86_64-linux-gnu/libGL.so.1 /usr/lib/libGL.so # Provide OpenGL support
sudo ldconfig
```

There's a long list of `xcb` dependencies [here](https://doc.qt.io/qt-5/linux-requirements.html). Here is a one-liner to install all of them:

```bash
sudo apt install libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync0-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-xinerama0-dev  libxkbcommon-dev libxkbcommon-x11-dev
```

Next, I cloned the git repository and installed Qt5. 

```
git clone https://code.qt.io/qt/qt5.git
cd qt5
git checkout 5.14 # the latest stable version
#./init-repository # clone the submodules
./init-repository --module-subset=default,-qtwebengine # skip the web module (as per wiki suggestion)
qmake -query # check. should not refer to any other Qt versions

mkdir ../qt5-build
cd ../qt5-build
../qt5/configure -prefix /opt/qt/5.14 -qpa xcb -opensource -confirm-license -nomake examples -nomake tests 
make -j8
sudo make install
```

`-opensource`: Build the open source edition of Qt

`-nomake examples -nomake tests`: exclude `examples` and `tests` from the build. Also see `-skip`  and `-no-feature-xxxxxx`

**Note**: In the configuration step, it showed that `WARNING: QDoc will not be compiled, probably because libclang could not be located. This means that you cannot build the Qt documentation.` but I do not need the documentation, so I ignored this warning. There was another error message:

```bash
ERROR: The OpenGL functionality tests failed!
You might need to modify the include and library search paths by editing QMAKE_INCDIR_OPENGL[_ES2],
QMAKE_LIBDIR_OPENGL[_ES2] and QMAKE_LIBS_OPENGL[_ES2] in the mkspec for your platform.
```

I tried to specify the `-platform linux-g++-64` as suggested [here](https://forum.qt.io/topic/78224/opengl-test-failed-when-configure-qt-as-source/5) but this did not work. I linked the `libGL.so.1` in the standard `/usr/lib` as documented [here](https://stackoverflow.com/questions/18406369/qt-cant-find-lgl-error/32184137#32184137) but it did not work. I had to install `libgl1-mesa-dev` and link the file -- then only it worked (only installing `libgl1-mesa-dev` without the link also did not work). It is helpful to check the configuration output to know if  there were any errors. 

Finally, I created the modulefile for loading Qt5 as and when required.

## The RGB madness

This is a rant, ignore if you will. I recently built a computer and it was astonishingly difficult to avoid an RGB explosion. To make matters worse, every manufacturer has their own proprietary applications for controlling these hardware. Most of them are Windows-only and compete for background resources. Life as a Linux user and random RGB lights is not much of a fun. I am truly amazed how much money, time and resources are being wasted by everyone involved in this warped concept of strobing aesthetics, sometimes leading to [rainbow pukes](https://i.imgur.com/gRRH5fQ.jpg). Although I managed to get most of the stuff without the obnoxious RGB elements, there was still the AMD Wraith Prism cooler which I had to deal with. Honestly, I had to spend quite some of my valuable time to switch off the RGB and I wish this stupid problem was not there to begin with. Rant over.

