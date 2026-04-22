## image vector table  
(arch/arm/include/asm/mach-imx/hab.h)  
```
struct __packed ivt_header {
	uint8_t		magic;
	uint16_t	length;
	uint8_t		version;
};

struct ivt {
	struct ivt_header hdr;	/* IVT header above */
	uint32_t entry;		/* Absolute address of first instruction */
	uint32_t reserved1;	/* Reserved should be zero */
	uint32_t dcd;		/* Absolute address of the image DCD */
	uint32_t boot;		/* Absolute address of the boot data */
	uint32_t self;		/* Absolute address of the IVT */
	uint32_t csf;		/* Absolute address of the CSF */
	uint32_t reserved2;	/* Reserved should be zero */
};
```

In the i.MX8M architecture, the **Image Vector Table (IVT)** acts like a "map" that the ROM bootloader reads first. The ROM uses this table to determine where to load data into memory and where to jump to begin execution.

Based on the structure defined in `arch/arm/include/asm/mach-imx/image.h`, here are the specific values and their meanings.

---

### 1. IVT Structure and Field Meanings

The IVT consists of the following fields, each being **4 bytes (32-bit)** in size:

| Field | Actual Value / Meaning |
| --- | --- |
| header | Tag (0xD1), Length (0x0020), Version (0x40/0x41)A header used to verify the validity of the IVT structure. |
| entry | Image Entry Point AddressThe absolute memory address the ROM jumps to after the copy process is complete. |
| reserved1 | Usually set to 0. |
| dcd | DCD (Device Configuration Data) AddressPoints to a table of commands for hardware initialization (e.g., DDR controller setup). |
| boot_data | Boot Data AddressPoints to a structure containing the total image size and its location in the flash memory. |
| self | Absolute Address of the IVT itselfUsed by the ROM to verify that the IVT was loaded into the correct memory location. |
| csf | CSF (Command Sequence File) AddressPoints to the digital signature data used for Secure Boot (HAB). |
| reserved2 | Usually set to 0. |

---

### 2. Visualization of Memory Layout

The ROM bootloader looks for the IVT at a specific offset within the flash memory (e.g., eMMC or SD card).

1. Header (0xD1...): Signals to the ROM, "The IVT starts here!"
2. DCD: Configures the DDR controller so that RAM can be used for the next steps.
3. Boot Data: Tells the ROM exactly how much more data to read from the flash.
4. Entry: Commands the ROM, "Execution starts at this address."

---

### 3. Checking the Source Code

If you open `image.h` in VS Code, you can see the `struct ivt` definition with detailed comments:

C```
/* Location: arch/arm/include/asm/mach-imx/image.h */
struct ivt {
    uint32_t header;       /* Header: Tag=0xd1, len=0x20, ver=0x40/0x41 */
    uint32_t entry;        /* Abs. address of the image entry point */
    uint32_t reserved1;
    uint32_t dcd;          /* Abs. address of the DCD table */
    uint32_t boot_data;    /* Abs. address of the boot data */
    uint32_t self;         /* Abs. address of the IVT itself */
    uint32_t csf;          /* Abs. address of the CSF */
    uint32_t reserved2;
};

```

### 💡 Pro Tip: Verification via Binary

Once your build is finished and a `flash.bin` is generated, you can inspect the actual values using `hexdump` in your WSL terminal:

Bash```
# View the first 32 bytes (IVT) in hexadecimal
hexdump -n 32 -C flash.bin

```

If the first byte is `d1`, you have successfully located the **IVT Header**.

Would you like to know how these addresses (like the `entry` point) are calculated during the build process?

