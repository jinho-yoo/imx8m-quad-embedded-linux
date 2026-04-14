# 🛠 Yocto Project Build Guide for NXP MCIMX8M-EVKB

This document provides a step-by-step guide to building a custom Embedded Linux distribution for the NXP i.MX8M Quad using the Yocto Project (Mickledore release).
## 0.1 Out of box
https://www.nxp.com/document/guide/getting-started-with-the-i-mx-8m-mini-evkb:GS-iMX-8M-Mini-EVK  

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

Configure the build environment for the specific machine (NXP MCIMX8M-EVKB).

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

<!-- - bitbake core-image-minimal 결과물  
 가장 핵심적인 파일은 아래와 같습니다.  
 파일 이름: core-image-minimal-imx8mqevk.wic.bz2  
 설명: 부팅 가능한 전체 이미지 파일입니다. (압축된 형태)  
 기타 파일: * core-image-minimal-imx8mqevk.manifest (포함된 패키지 목록)  
  
 core-image-minimal-imx8mqevk.testdata.json (테스트용 데이터)
-->
<!--  
- 가장 핵심적인 파일: 부팅 이미지  
리스트에서 가장 중요한 파일은 .wic.zst 확장자를 가진 파일입니다. 조사하신 .wic.bz2 대신 현재 빌드에서는 .zst 압축 방식을 사용하고 있습니다.  
core-image-minimal-imx8mqevk.wic.zst (심볼릭 링크)  
실제 파일: core-image-minimal-imx8mqevk-20260402073451.rootfs.wic.zst  
설명: SD 카드나 eMMC에 바로 구울 수 있는 통합 이미지입니다. 부트로더, 커널, 루트 파일시스템이 모두 포함되어 있습니다.  
사용법: 이 파일의 압축을 풀면 .wic 파일이 나오며, 이를 dd 명령어나 Etcher 툴을 사용해 보드에 플래싱합니다.
-->
- bitbake core-image-minimal(Key Output File: Boot Image)  
The most critical file in the output list is the one with the .wic.zst extension. While your initial research suggested .wic.bz2, the current build environment utilizes the .zst (Zstandard) compression method.  
-- core-image-minimal-imx8mqevk.wic.zst (Symbolic Link)  
-- Actual File: core-image-minimal-imx8mqevk-20260402073451.rootfs.wic.zst  
-- Description: This is an integrated system image ready to be flashed onto an SD card or eMMC. It contains the bootloader, kernel, and root filesystem (RootFS).  
-- Usage: After decompressing this file to obtain the .wic file, use the dd command or flashing tools like BalenaEtcher to write the image to your target hardware.  
<!--    
- bitbake imx-image-multimedia 결과물  
멀티미디어 전용 패키지들이 포함된 이미지입니다.  
파일 이름: imx-image-multimedia-imx8mqevk.wic.bz2  
설명: GPU/VPU 드라이버와 GStreamer 등이 포함된 대용량 이미지입니다.
-->
- bitbake imx-image-multimedia Output
This image includes a suite of packages optimized for multimedia performance.  
File Name: imx-image-multimedia-imx8mqevk.wic.zst (or .bz2 depending on configuration)  
Description: A high-capacity image that includes GPU/VPU drivers and the GStreamer framework, designed for hardware-accelerated video and graphics processing.

<!--  
### Writing to SD Card
```
bzcat core-image-minimal-imx8mqevk.wic.bz2 | sudo dd of=/dev/sdX bs=1M conv=fsync  
```
-->
# Image Fusing and Boot process

i.MX8M EVB에 이미지를 플래싱하고 부팅하는 과정을 기술 문서 형식의 영어로 정리해 드립니다.

---

## 1. Prepare the Image File (Host PC)

First, move the generated `.wic.zst` file from your WSL environment to your Windows local drive. You can access the WSL file system by typing `\\wsl$` in the Windows Explorer address bar.

- Source Path: ~/imx8m-yocto-bsp/build/tmp/deploy/images/imx8mqevk/
- Target File: core-image-minimal-imx8mqevk.wic.zst

---

## 2. Flash the SD Card

Use a flashing utility like **balenaEtcher** to write the image to the MicroSD card.

1. Insert SD Card: Connect the MicroSD card to your PC using a card reader.
2. Select Image: In balenaEtcher, click 'Flash from file' and select the .wic.zst file. (Etcher handles the .zst decompression automatically).
3. Select Target: Choose your SD card and click 'Flash!'.

---

## 3. Hardware Configuration (Boot Mode Switches)

Set the **DIP switches** on the i.MX8M EVB to enable booting from the SD card.

- Boot Mode Switch (SW801):SD Card Booting: 0011 (1:OFF, 2:OFF, 3:ON, 4:ON)
- Power: Connect the USB Type-C power cable, but keep the power switch in the OFF position for now.

---

## 4. Connect Serial Console and Booting

To monitor the boot process, you need to establish a serial console connection.

1. Connect Debug Cable: Connect the Micro-USB cable from your PC to the Debug UART port on the EVB.
2. Launch Terminal Emulator: Open Tera Term or PuTTY.Port: Select the corresponding COM port.Baud Rate: 115200 bps.
3. Power ON: Flip the power switch on the board to ON.
4. Verification: You should see the U-Boot logs followed by the Linux kernel boot sequence.Default Login: root (No password required).

- install VCP Driver
https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=overview  

---

### 💡 Troubleshooting

- No Log Output: Ensure the cable is connected to the specific port labeled "Debug" and verify that your baud rate is set exactly to 115200.
- Boot Loop or Hang: This can often be a power supply issue. Ensure you are using a 12V adapter if the Type-C power is insufficient.




<!--
i.MX8M EVB에 빌드된 이미지를 플래싱하고 부팅하는 과정은 크게 **1) 이미지 준비, 2) SD 카드 플래싱, 3) 보드 스위치 설정, 4) 부팅 및 확인** 단계로 나뉩니다.

---

## 1. 이미지 준비 (Host PC)

WSL에서 생성된 `.wic.zst` 파일을 윈도우로 가져와야 합니다. 윈도우 탐색기를 열고 주소창에 `\\wsl$`를 입력하여 아래 경로의 파일을 윈도우 폴더(예: `D:\Temp`)로 복사하세요.

- 경로: ~/imx8m-yocto-bsp/build/tmp/deploy/images/imx8mqevk/
- 파일명: core-image-minimal-imx8mqevk.wic.zst

---

## 2. SD 카드 플래싱 (Flashing)

가장 안전하고 간편한 방법은 **BalenaEtcher** 툴을 사용하는 것입니다.

1. SD 카드 삽입: PC에 마이크로 SD 카드를 연결합니다.
2. Etcher 실행: Flash from file을 클릭하고 복사해온 .wic.zst 파일을 선택합니다. (Etcher는 .zst 압축을 자동으로 해제하며 구워줍니다.)
3. Target 선택: SD 카드를 선택한 후 Flash!를 누릅니다.

---

## 3. 하드웨어 설정 (SW Switch)

i.MX8M EVB가 SD 카드로부터 부팅될 수 있도록 보드 상의 **딥 스위치(DIP Switch)**를 설정해야 합니다. 보드 모델마다 조금씩 다르지만, 일반적인 i.MX8M EVK의 설정은 다음과 같습니다.

- Boot Mode 스위치 (SW801):SD Card 부팅: 0011 (1:OFF, 2:OFF, 3:ON, 4:ON)
- 전원: USB Type-C 전원 케이블을 연결하되, 아직 스위치는 켜지 마세요.

---

## 4. 시리얼 콘솔 연결 및 부팅

보드가 부팅되는 과정을 확인하려면 시리얼 디버그 포트를 연결해야 합니다.

1. 디버그 케이블 연결: 보드의 Micro-USB(또는 전용 디버그 포트)와 PC를 연결합니다.
2. 터미널 실행: Tera Term이나 PuTTY를 실행합니다.Port: 해당 COM 포트 선택Speed: 115200 bps
3. 전원 ON: 보드의 전원 스위치를 켭니다.
4. 확인: 터미널 창에 U-Boot 로그가 올라오고, 잠시 후 리눅스 커널이 부팅되며 로그인 프롬프트가 뜨면 성공입니다.ID: root (비밀번호 없음)

---

### 💡 문제 해결 팁 (Troubleshooting)

- 로그가 전혀 안 뜰 때: 시리얼 케이블이 'Debug'라고 적힌 포트에 정확히 꽂혔는지, 보레이트(115200)가 맞는지 확인하세요.
- 부팅 중 멈출 때: 전력 공급이 부족할 수 있습니다. 12V 전원 어댑터가 있다면 별도로 연결해 보세요.

이제 보드에서 직접 리눅스 명령어를 입력하실 수 있습니다. 혹시 하드웨어 변형 보드 설계 전에 이 기본 보드에서 먼저 테스트해보고 싶은 특정 기능이 있으신가요?
-->

## 🔍 Build Process Visualization

## 💡 Troubleshooting

-   **Disk Space:** Ensure at least 100GB of free space.
-   **Fetch Errors:** If a download fails, try re-running bitbake; it will resume from where it left off.  


<hr style="border: double 5px #000;">
<hr style="border: double 5px #000;">

<!--
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
-->
