# 🛠 Yocto Project Build Guide for i.MX8M Quad

This document provides a step-by-step guide to building a custom Embedded Linux distribution for the NXP i.MX8M Quad using the Yocto Project (Mickledore release).

## 1\. Prerequisites

Before starting, ensure your host machine (Ubuntu 20.04/22.04 recommended) has the necessary build dependencies installed.

### Install Host Packages
```
sudo apt update  
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential \\  
chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils \\  
iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \\  
pylint3 xterm python3-subunit mesa-common-dev zstd liblz4-tool  
```
### Install Google Repo Tool
```
mkdir ~/bin  
curl \[https://storage.googleapis.com/git-repo-downloads/repo\](https://storage.googleapis.com/git-repo-downloads/repo) > ~/bin/repo  
chmod a+x ~/bin/repo  
export PATH=${PATH}:~/bin  
```
## 2\. Fetching Source Code

We use the repo tool to manage multiple layers including meta-freescale.

### Initialize Repository
```
mkdir imx8m-yocto-bsp  
cd imx8m-yocto-bsp  
  
\# Initialize for Mickledore release  
repo init -u \[https://github.com/nxp-imx/imx-manifest\](https://github.com/nxp-imx/imx-manifest) -b imx-linux-mickledore -m imx-6.1.36-2.1.0.xml  
repo sync  
```
## 3\. Environment Setup

Configure the build environment for the specific machine (i.MX8M Quad EVK).

### Setup Script
```
\# Accept EULA and set machine  
DISTRO=fsl-imx-xwayland MACHINE=imx8mqevk source imx-setup-release.sh -b build  
```
-   **DISTRO:** fsl-imx-xwayland (Supports Wayland display server)
-   **MACHINE:** imx8mqevk (Target hardware)
-   **\-b build:** Specifies the build directory name.

## 4\. Configuration (Local.conf)

Edit conf/local.conf to optimize the build process based on your CPU cores.
```
BB\_NUMBER\_THREADS = "16"  
PARALLEL\_MAKE = "-j 16"  
  
\# Accept Freescale EULA  
ACCEPT\_FSL\_EULA = "1"  
```
## 5\. Building the Image

Now, use bitbake to compile the entire distribution, including the toolchain, kernel, U-Boot, and RootFS.

### Build Minimal Console Image
```
bitbake core-image-minimal  
```
### Build Multimedia Image (with Qt6)
```
bitbake imx-image-multimedia  
```
## 6\. Deployment

After a successful build, the output images are located in:

tmp/deploy/images/imx8mqevk/

### Key Artifacts:

-   imx-boot-imx8mqevk-sd.bin-flash\_evk: Combined bootloader image.
-   core-image-minimal-imx8mqevk.wic.bz2: Full SD card image.

### Writing to SD Card
```
bzcat core-image-minimal-imx8mqevk.wic.bz2 | sudo dd of=/dev/sdX bs=1M conv=fsync  
```
## 🔍 Build Process Visualization

## 💡 Troubleshooting

-   **Disk Space:** Ensure at least 100GB of free space.
-   **Fetch Errors:** If a download fails, try re-running bitbake; it will resume from where it left off.  

## wsl2 build note  

<hr style="border: double 5px #000;">

## 💻 Building on WSL2 (Windows Subsystem for Linux)

If you are using Windows, WSL2 is a powerful environment for Yocto. However, you must follow these rules for a successful build:

### 1\. Work in the Linux Native Filesystem

**DO NOT** build in /mnt/c/. The Windows-Linux file system bridge is too slow for Yocto's intensive I/O operations.
```
-   **Good:** /home/username/imx8m-yocto-bsp
-   **Bad:** /mnt/c/Users/Name/Projects/...
```
### 2\. Configure WSL Resources

Create or edit %USERPROFILE%\\.wslconfig in Windows to allocate enough resources:
```
\[wsl2\]  
memory=16GB # Recommended minimum  
processors=8 # Adjust based on your CPU  
```
## 1\. Prerequisites

Before starting, ensure your host machine (Ubuntu 20.04/22.04 recommended) has the necessary build dependencies installed.

### Install Host Packages
```
sudo apt update  
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential \\  
chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils \\  
iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \\  
pylint3 xterm python3-subunit mesa-common-dev zstd liblz4-tool  
```
### Install Google Repo Tool
```
mkdir ~/bin  
curl \[https://storage.googleapis.com/git-repo-downloads/repo\](https://storage.googleapis.com/git-repo-downloads/repo) > ~/bin/repo  
chmod a+x ~/bin/repo  
export PATH=${PATH}:~/bin  
```
## 2\. Fetching Source Code

We use the repo tool to manage multiple layers including meta-freescale.

### Initialize Repository
```
mkdir imx8m-yocto-bsp  
cd imx8m-yocto-bsp  
  
\# Initialize for Mickledore release  
repo init -u \[https://github.com/nxp-imx/imx-manifest\](https://github.com/nxp-imx/imx-manifest) -b imx-linux-mickledore -m imx-6.1.36-2.1.0.xml  
repo sync  
```
## 3\. Environment Setup

Configure the build environment for the specific machine (i.MX8M Quad EVK).

### Setup Script
```
\# Accept EULA and set machine  
DISTRO=fsl-imx-xwayland MACHINE=imx8mqevk source imx-setup-release.sh -b build  
```
## 4\. Configuration (Local.conf)

Edit conf/local.conf to optimize the build process based on your CPU cores.
```
BB\_NUMBER\_THREADS = "16"  
PARALLEL\_MAKE = "-j 16"  
  
\# Accept Freescale EULA  
ACCEPT\_FSL\_EULA = "1"  
```
## 5\. Building the Image
```
bitbake core-image-minimal  
```
## 6\. Deployment

After a successful build, the output images are located in:
```
tmp/deploy/images/imx8mqevk/
```
### Key Artifacts:

-   imx-boot-imx8mqevk-sd.bin-flash\_evk: Combined bootloader image.
-   core-image-minimal-imx8mqevk.wic.bz2: Full SD card image.

## 🔍 Build Process Visualization

## 💡 Troubleshooting

-   **Disk Space:** Ensure at least 150GB of free space. WSL2 virtual disks (.vhdx) expand automatically but don't shrink easily.
-   **Build Speed:** Disable Windows Defender for the WSL folder to gain extra performance.
