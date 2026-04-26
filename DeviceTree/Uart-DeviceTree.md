Building on the hardware specifications for the i.MX8M series, here is the English version of the UART device tree example and its integration with the memory map.

---

### 1. i.MX8MQ UART1 Memory Map (Hardware Spec)

According to the i.MX8MQ Reference Manual, the hardware address information for the first serial port (**UART1**) is as follows:

| Device | Base Address | Size | Interrupt (IRQ) |
| --- | --- | --- | --- |
| UART1 | 0x30860000 | 64 KB | 26 |

This base address acts as the **physical gateway** for the CPU to access the internal registers of the UART controller.

---
<img src="../images/Uart-address.PNG" width="400px" alt="i.MX8M NXP MCIMX8M RM">

---

### 2. Step-by-Step Device Tree Construction

#### Step 1: SoC Level Definition (imx8mq.dtsi)

First, we define the absolute hardware address and properties at the SoC level. This is where the memory map address is implemented.

```
/ {
	soc {
		aips1: bus@30000000 {  // UART is attached under the AIPS1 bus
			uart1: serial@30860000 {
				compatible = "fsl,imx8mq-uart", "fsl,imx21-uart";
				reg = <0x30860000 0x10000>; // Base address and Size (64KB)
				interrupts = <GIC_SPI 26 IRQ_TYPE_LEVEL_HIGH>;
				clocks = <&clk IMX8MQ_CLK_UART1_ROOT>,
					 <&clk IMX8MQ_CLK_UART1_ROOT>;
				clock-names = "ipg", "per";
				status = "disabled"; // Disabled by default
			};
		};
	};
};

```

## 1. Hierarchy and Bus Structure

The Device Tree represents the hardware in a parent-child hierarchy, reflecting how components are physically wired.

- soc { ... }: This node encompasses all components within the System on Chip.
- aips1: bus@30000000: This represents the AIPS-1 (AHB to IP Interface Bridge).@30000000: This is the base address of the AIPS-1 bus controller.In the i.MX 8M architecture, many peripherals like UART, I2C, and SPI are connected via this bridge.

---

## 2. UART1 Node Analysis (uart1: serial@30860000)

This section defines the specific configuration for the first UART controller.

### 📍 Address and Size (reg)

- reg = <0x30860000 0x10000>;0x30860000: The Base Address where the UART1 control registers begin.0x10000: The size of the memory-mapped IO space, which is 64 KB (as we calculated earlier). This ensures the kernel reserves the entire address range for this peripheral.

### ⚙️ Driver Compatibility (compatible)

- compatible = "fsl,imx8mq-uart", "fsl,imx21-uart";This string tells the Linux kernel which driver to load.It first looks for a specific i.MX8MQ driver. If not found, it falls back to the i.MX21 driver, as NXP's UART hardware design is largely compatible across these generations.

### ⚡ Interrupts (interrupts)

- interrupts = <GIC_SPI 26 IRQ_TYPE_LEVEL_HIGH>;
  - GIC_SPI 26: It uses Interrupt ID 26 (Shared Peripheral Interrupt) managed by the ARM Generic Interrupt Controller.
  - IRQ_TYPE_LEVEL_HIGH: The interrupt is triggered when the signal level is "High."

### 🕒 Clock Configuration (clocks)

- clocks = <&clk IMX8MQ_CLK_UART1_ROOT>, ...;
  - ipg (Interface clock): Used for communication between the CPU and the peripheral registers.
  - per (Peripheral clock): The source clock used to generate the actual Baud Rate for serial communication.

---

## 3. Activation Status (status)

- status = "disabled";By default, this peripheral is turned off in the main SOC definition file (.dtsi).To use it on a specific board (like the i.MX 8M EVK), you must "override" this property to "okay" in your board-level file (.dts).

---

### 💡 Developer's Note

When analyzing your **U-Boot** or **Linux** source code, you will often find this address (`0x30860000`) referenced as `UART1_BASE`. Since you are studying the BSP, checking the **"System Boot"** section of the Reference Manual will explain how the ROM code uses this UART for the "Serial Download Mode" if the primary boot fails.



#### Step 2: U-Boot & Boot Environment Configuration (imx8mq-u-boot.dtsi)

To use this UART during the early boot stage (SPL), the `u-boot,dm-spl;` property must be added as seen in your provided files.

```
&soc {
	u-boot,dm-spl; // Enable SoC for SPL [cite: 4]
};

&aips1 {
	u-boot,dm-spl; // Enable AIPS1 Bridge for SPL [cite: 4]
};

&uart1 {
	u-boot,dm-spl; // Enable UART1 for SPL 
};

```

- Key Point: To access UART1, you must also open the "gateways" of its parent nodes, soc and aips1, for the SPL to recognize the path.

#### Step 3: Board Level Activation (imx8mq-evk.dts)

Finally, declare that UART1 will be used as the actual debug console for the specific board.

```
/ {
	chosen {
		stdout-path = &uart1; // Redirect system logs to UART1
	};
};

&uart1 {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_uart1>; // Link pin configurations 
	status = "okay"; // Finally activate the device
};

```

---

### 3. Summary of Memory Map & Device Tree Integration

1. reg = <0x30860000 0x10000>;: When the driver runs in the kernel, it maps this address range to virtual memory to control hardware registers directly.
2. interrupts = <26>;: When data arrives at the UART, it sends signal #26 to the CPU, prompting the OS to call the UART driver.
3. Hierarchical Path: Since the address 0x30860000 cannot be reached without going through the &aips1 node, the activation property u-boot,dm-spl; must appear repeatedly throughout the hierarchy to keep the "communication path" open.

This example demonstrates how **addresses in hardware documentation** are transformed into **software configuration data**, allowing the system to recognize hardware at boot time.
