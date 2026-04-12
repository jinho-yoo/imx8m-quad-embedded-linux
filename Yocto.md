# 🛠 Yocto Project Build Guide for i.MX8M Quad

This document provides a step-by-step guide to building a custom Embedded Linux distribution for the NXP i.MX8M Quad using the Yocto Project (Mickledore release).

## 1\. Prerequisites

Before starting, ensure your host machine (Ubuntu 20.04/22.04 recommended) has the necessary build dependencies installed.

### Install Host Packages
```
confirm your environment is Ubuntu-22.4
sudo apt update  
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1 libsdl1.2-dev pylint xterm python3-subunit mesa-common-dev zstd liblz4-tool  
```
### Install Google Repo Tool
```
mkdir ~/bin  
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo   
chmod a+x ~/bin/repo  
export PATH=${PATH}:~/bin  
```
## 2\. Fetching Source Code

We use the repo tool to manage multiple layers including meta-freescale.

### Initialize Repository
  
```
mkdir imx8m-yocto-bsp  
cd imx8m-yocto-bsp  

sudo apt install repo
  
\# Initialize for Mickledore release  
repo init -u \https://github.com/nxp-imx/imx-manifest\https://github.com/nxp-imx/imx-manifest -b imx-linux-mickledore -m imx-6.1.36-2.1.0.xml
or
repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-mickledore -m imx-6.1.36-2.1.0.xml  

repo sync  
```
## 3\. Environment Setup

Configure the build environment for the specific machine (i.MX8M Quad EVK).

### Setup Script(imx-setup-release.sh)  
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
BB_NUMBER_THREADS = "16"  
PARALLEL_MAKE = "-j 16"  
  
# Accept Freescale EULA  
ACCEPT_FSL_EULA = "1"  
```
## 5\. Building the Image

Now, use bitbake to compile the entire distribution, including the toolchain, kernel, U-Boot, and RootFS.

### 8GB Memory, 6GB GPU  
[local system preparation]  
<pre>
1.Windows 탐색기 주소창에 %USERPROFILE% 입력 후 엔터.  
2.해당 폴더에 .wslconfig 파일을 만들고(메모장 활용) 아래 내용을 넣으세요.  
  
[wsl2]  
memory=6GB      # 실제 RAM 8GB 중 6GB를 WSL에 할당  
swap=16GB       # 부족한 RAM을 대신할 가상 메모리를 16GB로 넉넉히 설정  
localhostForwarding=true  
</pre>  
<pre>
1. local.conf 최적화 설정 (가장 중요)  
build/conf/local.conf 파일을 열어 기존 설정을 지우거나 주석 처리하고, 아래 내용을 맨 아래에 추가하세요. 핵심은 "한 번에 하나씩 천천히" 빌드하는 것입니다.  
1. 병렬 빌드 개수 최소화 (8GB RAM 기준 최적화)  
프로세스 개수를 2개로 제한하여 메모리 점유율을 낮춥니다.  
BB_NUMBER_THREADS = "2"  
PARALLEL_MAKE = "-j 2"  
  
2. 불필요한 이미지 용량 줄이기  
빌드 도중 생성되는 중간 파일들을 즉시 삭제하여 디스크 및 메모리 부하를 줄입니다.  
INHERIT += "rm_work"  
  
3. 호스트 glibc 버전 이슈 우회  
8GB 환경에서 uninative 체크는 사치입니다. 일단 비활성화해서 부하를 줄입니다.  
INHERIT:remove = "uninative"  
  
4. 압축 방식 변경 (메모리 소모 절감)  
이미지를 압축할 때 메모리를 많이 쓰는 방식을 피합니다.  
XZ_THREADS = "1"  
</pre>

### Build Minimal Console Image  
<pre>
sudo apt-get update
sudo apt-get install locales
sudo locale-gen en_US.UTF-8

sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8  

bitbake core-image-minimal
위 명령이 안된다면 아래를 수행
source setup-environment <b>build</b>
</pre>
### Build Multimedia Image (with Qt6)  

<pre>
bitbake imx-image-multimedia  
위 명령이 안된다면 아래를 수행
source setup-environment <b>build</b>
</pre>

## 6\. Deployment

After a successful build, the output images are located in:

tmp/deploy/images/imx8mqevk/

### Key Artifacts:

-   imx-boot-imx8mqevk-sd.bin-flash\_evk: Combined bootloader image.
-   core-image-minimal-imx8mqevk.wic.bz2: Full SD card image.

### bitbake Key Artifacts:  
 scenario can be found in [scenario1.md](https://github.com/jinho-yoo/scenarioes/blob/main/scenario1.md)

- bitbake core-image-minimal 결과물  
가장 핵심적인 파일은 아래와 같습니다.  
파일 이름: core-image-minimal-imx8mqevk.wic.bz2  
설명: 부팅 가능한 전체 이미지 파일입니다. (압축된 형태)  
기타 파일: * core-image-minimal-imx8mqevk.manifest (포함된 패키지 목록)  
  
core-image-minimal-imx8mqevk.testdata.json (테스트용 데이터)

    
- bitbake imx-image-multimedia 결과물  
멀티미디어 전용 패키지들이 포함된 이미지입니다.  
파일 이름: imx-image-multimedia-imx8mqevk.wic.bz2  
설명: GPU/VPU 드라이버와 GStreamer 등이 포함된 대용량 이미지입니다.

### Writing to SD Card
```
bzcat core-image-minimal-imx8mqevk.wic.bz2 | sudo dd of=/dev/sdX bs=1M conv=fsync  
```
## 🔍 Build Process Visualization

## 💡 Troubleshooting

-   **Disk Space:** Ensure at least 100GB of free space.
-   **Fetch Errors:** If a download fails, try re-running bitbake; it will resume from where it left off.  


<hr style="border: double 5px #000;">
<hr style="border: double 5px #000;">


## 💻 Building on WSL2 (Windows Subsystem for Linux) same as above  

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

| **Artifact Type** | **Filename Example** | **Description** |
| --- | --- | --- |
| **Bootloader** | flash.bin | Combined image of SPL, ATF, and U-Boot Proper. |
| --- | --- | --- |
| **Kernel** | Image | The uncompressed Linux Kernel binary. |
| --- | --- | --- |
| **Device Tree** | imx8mq-evk.dtb | Binary description of the hardware layout. |
| --- | --- | --- |
| **RootFS** | core-image-minimal-\*.tar.bz2 | The complete root filesystem directory structure. |
| --- | --- | --- |
| **Full Image** | core-image-minimal-\*.wic.bz2 | **The all-in-one image** for SD card flashing. |
| --- | --- | --- |

### Key Artifacts:

-   imx-boot-imx8mqevk-sd.bin-flash\_evk: Combined bootloader image.
-   core-image-minimal-imx8mqevk.wic.bz2: Full SD card image.

## 🔍 Build Process Visualization

## 💡 Troubleshooting

-   **Disk Space:** Ensure at least 150GB of free space. WSL2 virtual disks (.vhdx) expand automatically but don't shrink easily.
-   **Build Speed:** Disable Windows Defender for the WSL folder to gain extra performance.
