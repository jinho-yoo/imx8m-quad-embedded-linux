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
