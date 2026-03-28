# 🛠 U-Boot Architecture for i.MX8M Quad

This document describes the bootloader structure and initialization flow for the i.MX8M Quad customized system.

## 🏗 Bootloader Hierarchy

The i.MX8M Quad (Cortex-A53) requires a complex boot sequence compared to older ARMv7 systems. The boot image (flash.bin) is a container that includes multiple components:

1.  **SPL (Secondary Program Loader):** Initial hardware setup, DDR training.
2.  **ATF (Arm Trusted Firmware):** Secure world initialization (EL3).
3.  **OP-TEE (Optional):** Trusted Execution Environment.
4.  **U-Boot Proper:** Full bootloader with network and shell support.

### Image Structure Diagram

## 🔄 Boot Flow

The boot process follows these stages:

1.  **ROM Code:** Hardcoded in SoC; loads SPL from the boot media (SD/eMMC) to Internal SRAM.
2.  **SPL Stage:**
    -   Initialize IOMUX for UART and I2C.
    -   Execute **DDR Training** using the LPDDR4 firmware binaries.
    -   Load ATF, U-Boot proper, and DTB into DRAM.
3.  **ATF Stage:** Switches the processor to the secure monitor mode and jumps to U-Boot.
4.  **U-Boot Proper Stage:**
    -   Initializes high-level peripherals (Ethernet, USB, HDMI).
    -   Loads Linux Kernel and RootFS.

## 📂 Source Code Map (Key Files)

Based on the board\_imx8mq\_evk.c implementation:

| **File Path** | **Description** |
| board/freescale/imx8mq\_evk/ | Board-specific initialization (C code) |
| configs/imx8mq\_evk\_defconfig | Build configuration and enabled features |
| arch/arm/dts/imx8mq-evk.dts | Hardware description for the bootloader stage |
| include/configs/imx8mq\_evk.h | Legacy macro-based configurations |

## ⚡ DDR Training Details

The i.MX8M Quad requires specific firmware to train the LPDDR4 interface. This project utilizes:

-   lpddr4\_pmu\_train\_1d\_imem.bin
-   lpddr4\_pmu\_train\_1d\_dmem.bin
-   lpddr4\_pmu\_train\_2d\_imem.bin
-   lpddr4\_pmu\_train\_2d\_dmem.bin

These are combined during the final image generation using the imx-mkimage tool.

## 🛠 Customization Highlights

### PMIC Configuration (BD71837)

As seen in board\_imx8mq\_evk.c, the PMIC is initialized via I2C to ensure stable voltage levels for the Cortex-A53 cores during the frequency scaling phase.
```
/\* Example: Setting VDD\_ARM to 0.9V for stable boot \*/  
pmic\_reg\_write(dev, BD71837\_BUCK1\_VOLT\_RUN, 0x1a);  
```
### UART Pad Settings

Configured UART1 as the primary debug console with a baud rate of 115200.
