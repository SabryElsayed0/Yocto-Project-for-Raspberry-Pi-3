# Yocto Project for Raspberry Pi 3

Yocto-based build for Raspberry Pi 3 with SSH, WiFi support, Nano editor, Qt5 for GUI apps, RPiPlay for screen mirroring, audio support, a native Hello Bullet app, and a custom observability layer. Built with kernel 5.15.x and systemd, designed for embedded systems development and testing.

---

## Project Overview

This repository provides a Yocto-based build environment for Raspberry Pi 3, including essential packages such as:

- SSH for remote access.
- WiFi support for wireless connectivity.
- Nano editor for basic text editing.
- Qt5 for GUI application development.
- RPiPlay for screen mirroring.
- Audio playback/recording support (ALSA utilities).
- A native Hello Bullet sample application.
- A custom observability layer for system monitoring.

The project is built using **kernel version 5.15.x** and uses **systemd** as the init system, making it ideal for embedded systems development and testing.

![Project Overview](./project_overview.png)

---

## Steps of the Project

### 1. Pre-development Stage

#### 1 - Software Preparation

- Download Yocto Project extension for VS Code (**Yocto Project BitBake**).

  ![Yocto Project BitBake](./Yocto_Project_Bitbake.png)

#### 2- Install Dependencies

- Prepare **Environment** on the host machine(install dependencies):

  ```bash
  sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev python3-subunit mesa-common-dev zstd liblz4-tool file locales libacl1
  sudo locale-gen en_US.UTF-8
  ```


#### 3 - Choose YOCTO Release (Kirkstone Release)

The Yocto Project release process is predictable and consists of both major and minor (point) releases:

- **4.1 (“Langdale”)**
- **4.0 (“Kirkstone”)** (selected for this project)
- **3.4 (“Honister”)**

See all releases in this [link](https://wiki.yoctoproject.org/wiki/Releases).

![yocto_releases](./yocto_releases.png)

---

#### 4 - Cloning Poky Kirkstone

To clone the Poky repository for the Kirkstone release, run the following command:

```bash
git clone -b kirkstone https://github.com/yoctoproject/poky
```
#### 5- Understanding Poky

Poky is the Yocto Project reference system and is composed of a collection of tools and metadata. It is platform-independent and performs cross-compiling, using the BitBake tool, OpenEmbedded Core, and a default set of metadata, as shown in the following figure. It provides the mechanism to build and combine thousands of distributed open source projects to form a fully customizable, complete, and coherent Linux software stack.

Poky's main objective is to provide all the features an embedded developer needs.

![understanding_poky](./understanding_poky.png)





### 2. Development stage

#### Integrate BSP for raspberry pi
1- go to  (https://layers.openembedded.org/layerindex/branch/master/layers/)

2- search for raspberrypi , and select meta-raspberrypi
![meta-raspberrypi](./meta-raspberrypi.png)

3- clone meta-raspberrypi from here

git clone -b kirkstone git://git.yoctoproject.org/meta-raspberrypi
![meta-raspberrypi-git](./meta-raspberrypi-git.png)

4- add the layer to your yocto project bitbake-layers add-layer /PATH/TO/meta-raspberrypi

#### Integrate Qt-5
1- go to  (https://layers.openembedded.org/layerindex/branch/master/layers/)

2- search for qt , and select meta-qt5
![meta-qt5](./meta-qt5.png.png)

3- you will encounter this error
![qt_error](./qt_error.png)

4- to solve this error you should integrate qt-5 dependencies

![qt-5-dependncies](./qt-5-dependncies.png)

5- clone the openembedded-core
```bash
git clone -b kirkstone git://git.openembedded.org/openembedded-core
```
6- so now after cloning both you will find meta-oe layer inside openembedded-core layer by default
7- add meta-oe to the bblayers.conf

```bash
bitbake-layers add-layer /PATH/TO/meta-customRaspi/meta-qt5
```

### 3-Creating SW Layer meta-IVI Layer

1- create layer
```bash
bitbake create-layer meta-IVI
```

2- add meta-IVI to the bblayers.conf

```bash
bitbake add-layer meta-IVI
```
### 4-creating DISTRO Layer
1- create meta-info-distro layer
```bash
bitbake create-layer meta-info-distro
```

2- add meta-info-distro to the bblayers.conf

``` bash 
bitbake add-layer meta-info-distro
```

3 - go to meta-info-distro , and in conf directory , create distro directory

4- go to distro , create infotainment.conf

![info-distro](./info-distro.png)

5 - add this at infotainment.conf to add the features (ex- systemd)
```bash
# Distibution Information.
DISTRO="infotainment"
DISTRO_NAME="Bullet-Infotainment"
DISTRO_VERSION="1.0"

MAINTAINER="sabryelsayedfarg@gmail.com"


# SDK Information.
SDK_VENDOR = "-bulletSDK"
SDK_VERSION = "${@d.getVar('DISTRO_VERSION').replace('snapshot-${METADATA_REVISION}', 'snapshot')}"
SDK_VERSION[vardepvalue] = "${SDK_VERSION}"

SDK_NAME = "${DISTRO}-${TCLIBC}-${SDKMACHINE}-${IMAGE_BASENAME}-${TUNE_PKGARCH}-${MACHINE}"
# Installation path --> can be changed to ${HOME}-${DISTRO}-${SDK_VERSION}
SDKPATHINSTALL = "/opt/${DISTRO}/${SDK_VERSION}" 

# Disribution Feature --> NOTE: used to add customize package (for package usage).

# infotainment --> INFOTAINMENT

INFOTAINMENT_DEFAULT_DISTRO_FEATURES = "largefile opengl ptest multiarch vulkan x11 bluez5 bluetooth wifi qt5"
INFOTAINMENT_DEFAULT_EXTRA_RDEPENDS = "packagegroup-core-boot"
INFOTAINMENT_DEFAULT_EXTRA_RRECOMMENDS = "kernel-module-af-packet"

# TODO: to be org.

DISTRO_FEATURES ?= "${DISTRO_FEATURES_DEFAULT} ${INFOTAINMENT_DEFAULT_DISTRO_FEATURES} userland"

require conf/distro/include/systemd.inc 

# prefered version for packages.
PREFERRED_VERSION_linux-yocto ?= "5.15%"
PREFERRED_VERSION_linux-yocto-rt ?= "5.15%"


# Build System configuration.

LOCALCONF_VERSION="2"

# add poky sanity bbclass
INHERIT += "poky-sanity"
```

6 - this distro has bluetooth,wifi,qt5 
![distro_features](./distro_features.png)

7 - add those to DISTRO_FEATURES
![distro_Var](./distro_Var.png)

8- configure linux version 

![linux_version](./linux_version.png)


9-add systemd to the distro
- go to meta-info-distro/conf/distro
- create include directory
- create systemd.inc

 ![systemd](./systemd.png) 

 - at systemd.inc at this 
```bash
DISTRO_FEATURES:append = " systemd "
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscript = "systemd-compat-units"
```

- go to infotainment.conf and include systemd.inc
  ```bash
  require conf/distro/include/systemd.inc
  ```


![include_systemd](./include_systemd.png)

### 5-create your own image recipe (ivi-test-image)
 1- go to meta-IVI directory
 2- create receipes-core directory
 3- go to receipes-core 
 4- create images directory 
 5- create ivi-test-image.bb
 6- add this to ivi-test-image.bb

 ```bash
# include base image ----------->
# 1-poky
# 2- BSP layer

require recipes-core/images/rpi-test-image.bb


#2 - set of local variables
SUMMARY = " IVI Testing Image that include rpi func + helloworld package recipes "


inherit audio 

#3- customize the image 
IMAGE_INSTALL:append=" helloworld " 



# IMAGE_INSTALL ssh
# allow root access through ssh 
# access root through through ssh using empty password 
IMAGE_FEATURES:append=" debug-tweaks"
```
### 6- Access root through through ssh using empty password
1- go to ivi-test-image.bb
2- add debug-tweaks
```bash
# IMAGE_INSTALL ssh
# allow root access through ssh 
# access root through through ssh using empty password 
IMAGE_FEATURES:append=" debug-tweaks"
```

### 7-Inegrate ssh 
1-go to ivi-test-image.bb
2- add openssh
```bash
#3- customize the image 
IMAGE_INSTALL:append=" openssh "
```

### 8- Inegrate nano
1- go to meta-IVI directory
2- create receipes-editor
3- go to receipes-editor
4- create nano directory 
5- create nano receipe using this 

```bash
recipetool create -o nano_1.0.bb  https://git.savannah.gnu.org/git/nano.git
```
![nano](./nano.png)

7- bitbake -c fetch nano  ---> here I only run specifc task of nano recipe (fetch)
8- bitbake -c unpack nano  ---> here I only run specifc task of nano recipe (unpack)

6- add do_configre and do_build to nano receipe
```bash
do_configure(){
    oe_runconf
}

do_build(){

    oe_runmake
}
```
7-assure to install autopoint ----> (sudo apt install autopoint)

8-run ./autogen.sh from sources in the "nano" directory,-------> to generate ./configure file to excute by oe_runconf
```bash
## here to get the source dir
bitbake -e nano | grep "^S="  ----> to the path of the source file of nano after downloading
## go to this path
run ./autogen.sh
```
9-run bitbake -c configure nano
10- run bitbake nano

### 9- integrate rpiplay
 1- go to meta-IVI directory
 2- create receipes-info and go to it
 3- create rpi-play directory
 4- create rpiplay.bb
```bash
recipetool create -o rpi-play_1.0.bb https://github.com/FD-/RPiPlay      
 3- create audio.bbclass
```

5-replace the content of rpi-play_1.0.bb to include some depencies
```bash
LICENSE = "Unknown"
LIC_FILES_CHKSUM = "file://LICENSE;md5=1ebbd3e34237af26da5dc08a4e440464 \
                    file://lib/llhttp/LICENSE-MIT;md5=f5e274d60596dd59be0a1d1b19af7978 \
                    file://lib/playfair/LICENSE.md;md5=c7cd308b6eee08392fda2faed557d79a"

SRC_URI = "git://github.com/FD-/RPiPlay;protocol=https;branch=master \
           file://0001_fix_include_dir_gstreamer.patch"           
# Modify these as desired
PV = "1.0+git${SRCPV}"
SRCREV = "64d0341ed3bef098c940c9ed0675948870a271f9"

S = "${WORKDIR}/git"


# NOTE: the following library dependencies are unknown, ignoring: brcmEGL plist bcm_host plist-2 brcmGLESv2 vchiq_arm openmaxil vcos
#       (this is based on recipes that have previously been built and packaged)
DEPENDS = " userland  avahi libplist mdns openssl glib-2.0 gst-devtools gstreamer1.0 gstreamer1.0-plugins-base gst-examples gstreamer1.0-libav gstreamer1.0-plugins-bad gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly "

# TODO: check libplist and avahi is runtime and compile or runtime only.
RDEPENDS_${PN} = " avahi libplist gstreamer1.0-plugins-base gstreamer1.0-plugins-good "

inherit cmake pkgconfig

# Specify any options you want to pass to cmake using EXTRA_OECMAKE:
EXTRA_OECMAKE        = ""
TARGET_LDFLAGS      += "-Wl,--copy-dt-needed-entries"
EXTRA_OEMAKE:append  = 'LDFLAGS="${TARGET_LDFLAGS}"'
```

### 10- integrate AUDIO
- note this is the view of the audio stack
  
  ![audio](audio_stack.png)

 1- go to meta-IVI directory
 2- create classes directory and go to it
 3- create audio.bbclass
 ![audio_class](audio_class.png)
 
 4- add this to audio.bbclass
 ```bash
IMAGE_INSTALL:append = " pavucontrol pulseaudio pulseaudio-module-dbus-protocol pulseaudio-server \
                        pulseaudio-module-loopback pulseaudio-module-bluez5-device pulseaudio-module-bluez5-discover \
                        alsa-utils alsa-plugins bluez5 packagegroup-tools-bluetooth alsa-tools \
                        packagegroup-rpi-test rpi-play can-utils net-tools gstreamer1.0 alsa-topology-conf \
                        alsa-ucm-conf alsa-state alsa-lib qtbase-plugins libsocketcan qtquickcontrols \
                        qtquickcontrols2 qtgraphicaleffects qtmultimedia qtserialbus qtquicktimeline \
                        qtvirtualkeyboard i2c-tools hostapd iptables iproute2 iputils qtbase-examples \
                        gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
                            "
```

 

   


 
 















