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















