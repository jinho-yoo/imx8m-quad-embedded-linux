Building on the hardware specifications for the i.MX8M series, here is the English version of the UART device tree example and its integration with the memory map.

---

### 1. i.MX8MQ UART1 Memory Map (Hardware Spec)

According to the i.MX8MQ Reference Manual, the hardware address information for the first serial port (**UART1**) is as follows:

| Device | Base Address | Size | Interrupt (IRQ) |
| --- | --- | --- | --- |
| UART1 | 0x30860000 | 64 KB | 26 |

This base address acts as the **physical gateway** for the CPU to access the internal registers of the UART controller.

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
