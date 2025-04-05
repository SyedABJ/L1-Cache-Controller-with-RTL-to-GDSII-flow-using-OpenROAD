# RISC V CPU with L1-Cache-Controller-with-RTL-to-GDSII-flow-using-OpenROAD (ongoing project)
Repository for the RTL to GDSII flow run on OpenROAD for Mini Project in VLSI Design

## Block diagram of Cache Controller Interconnections with CPU and Main Memory
![image](https://github.com/user-attachments/assets/c4ca9518-c8e4-4207-a944-090e634288d0)

## Specifications of Picorv32
+ 250 Mhz
+ Small (750-2000 LUTs in 7-Series Xilinx Architecture)
+ Selectable native memory interface or AXI4-Lite master
+ Optional IRQ support
+ Optional Co-Processor Interface

## I/O Ports of Picorv32
	input clk, resetn,
	output reg trap,

	output reg        mem_valid,
	output reg        mem_instr,
	input             mem_ready,

	output reg [31:0] mem_addr,
	output reg [31:0] mem_wdata,
	output reg [ 3:0] mem_wstrb,
	input      [31:0] mem_rdata,

	// Look-Ahead Interface
	output            mem_la_read,
	output            mem_la_write,
	output     [31:0] mem_la_addr,
	output reg [31:0] mem_la_wdata,
	output reg [ 3:0] mem_la_wstrb,

	// Pico Co-Processor Interface (PCPI)
	output reg        pcpi_valid,
	output reg [31:0] pcpi_insn,
	output     [31:0] pcpi_rs1,
	output     [31:0] pcpi_rs2,
	input             pcpi_wr,
	input      [31:0] pcpi_rd,
	input             pcpi_wait,
	input             pcpi_ready,

	// IRQ Interface
	input      [31:0] irq,
	output reg [31:0] eoi,
    `ifdef RISCV_FORMAL
        output reg        rvfi_valid,
        output reg [63:0] rvfi_order,
        output reg [31:0] rvfi_insn,
        output reg        rvfi_trap,
        output reg        rvfi_halt,
        output reg        rvfi_intr,
        output reg [ 1:0] rvfi_mode,
        output reg [ 1:0] rvfi_ixl,
        output reg [ 4:0] rvfi_rs1_addr,
        output reg [ 4:0] rvfi_rs2_addr,
        output reg [31:0] rvfi_rs1_rdata,
        output reg [31:0] rvfi_rs2_rdata,
        output reg [ 4:0] rvfi_rd_addr,
        output reg [31:0] rvfi_rd_wdata,
        output reg [31:0] rvfi_pc_rdata,
        output reg [31:0] rvfi_pc_wdata,
        output reg [31:0] rvfi_mem_addr,
        output reg [ 3:0] rvfi_mem_rmask,
        output reg [ 3:0] rvfi_mem_wmask,
        output reg [31:0] rvfi_mem_rdata,
        output reg [31:0] rvfi_mem_wdata,

        output reg [63:0] rvfi_csr_mcycle_rmask,
        output reg [63:0] rvfi_csr_mcycle_wmask,
        output reg [63:0] rvfi_csr_mcycle_rdata,
        output reg [63:0] rvfi_csr_mcycle_wdata,

        output reg [63:0] rvfi_csr_minstret_rmask,
        output reg [63:0] rvfi_csr_minstret_wmask,
        output reg [63:0] rvfi_csr_minstret_rdata,
        output reg [63:0] rvfi_csr_minstret_wdata,
    `endif

	// Trace Interface
	output reg        trace_valid,
	output reg [35:0] trace_data

## Parameters in Picorv32

### COMPRESSED_ISA
This enables support for the RISC-V Compressed Instruction Set. By default it is 0.

### ENABLE_PCPI
Set this to 1 to enable the Pico Co-Processor Interface (PCPI). By default it is 0.

### ENABLE_MUL
This parameter internally enables PCPI and instantiates the picorv32_pcpi_mul core that implements the MUL[H[SU|U]] instructions. The external PCPI interface only becomes functional when ENABLE_PCPI is set as well. By default it is 0.

### ENABLE_FAST_MUL
This parameter internally enables PCPI and instantiates the picorv32_pcpi_fast_mul core that implements the MUL[H[SU|U]] instructions. The external PCPI interface only becomes functional when ENABLE_PCPI is set as well. By default it is 0.

### ENABLE_DIV
This parameter internally enables PCPI and instantiates the picorv32_pcpi_div core that implements the DIV[U]/REM[U] instructions. The external PCPI interface only becomes functional when ENABLE_PCPI is set as well.

### TWO_CYCLE_COMPARE
This relaxes the longest data path a bit by adding an additional FF stage at the cost of adding an additional clock cycle delay to the conditional branch instructions. By default it is 0. Note: Enabling this parameter will be most effective when retiming (aka "register balancing") is enabled in the synthesis flow.

### TWO_CYCLE_ALU
This adds an additional FF stage in the ALU data path, improving timing at the cost of an additional clock cycle for all instructions that use the ALU. By default it is 0.

### Pico Co-Processor Interface (PCPI)
The thing that makes PicoRV32 ideal for use in microcontrollers is its Pico Co-Processor Interface (PCPI) feature. The PCPI is an interface that can be enabled by changing verilog parameters as mentioned above. PCPI helps adding additional functionality to the core easier provided they are non-branching instructions.

### PCPI

When an unsupported instruction is found by PicoRV32 occurs it asserts pcpi_valid. The unsopported instruction is sent to pcpi_insn for the co-processor to recognise it. The decoded values of registers is made available through pcpi_rs1 and pcpi_rs2 and its output can be sent to pcpi_rd. The pcpi_ready needs to asserted when the execution of the instruction is over.

When no external PCPI core acknowledges the instruction within 16 clock cycles, then an illegal instruction exception is raised and the respective interrupt handler is called. A PCPI core that needs more than a couple of cycles to execute an instruction, should assert pcpi_wait as soon as the instruction has been decoded successfully and keep it asserted until it asserts pcpi_ready. This will prevent the PicoRV32 core from raising an illegal instruction exception.

## Specification of L1 Cache Controller
![image](https://github.com/user-attachments/assets/c63c9cfe-d6f3-46e3-a3e7-56fa1938a5ac)

