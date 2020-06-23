Simple 3-stage pipeline RISC-V processor
========================================

This is a simple RISC-V 3-stage pipeline processor.
I wrote this code to understand the RV32I instruction set, just for fun.
This is not a RISC-V core available for production

> Notes: add RV32M supporting now.

## Building toolchains

Install RV32IM toolchains.

    # Ubuntu packages needed:
    sudo apt-get install autoconf automake autotools-dev curl libmpc-dev \
        libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo \
        gperf libtool patchutils bc zlib1g-dev git libexpat1-dev

    git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
    cd riscv-gnu-toolchain

    mkdir build; cd build
    ../configure --with-arch=rv32im --prefix=/opt/riscv32im
    make -j$(nproc)

## Files list

| Folder    | Description                    |
| --------- | ------------------------------ |
| doc       | Instruction sets document      |
| rtl       | RTL files                      |
| sim       | Icarus verilog simulation env  |
| syn       | Syntheis env for Yosys         |
| testbench | testbench, memory model        |
| tool      | software simulator             |

## RTL Simulation

    # Ubuntu package needed to run the RTL simulation
    sudo apt install iverilog

Only running make without paramets will get help.

    $ make
    make all         build all diags and run the RTL sim
    make build       build all diags and the RTL
    make hello       build hello diag and run the RTL sim
    make dhrystone   build Dhrystone diag and run the RTL sim
    make coremark    build Coremark diag and run the RTL sim
    make qsort       build qsort diag and run the RTL sim
    make clean       clean

Supports following parameter when running the simulation.

    +meminit    memory intialize zero
    +dumpvcd    dump VCD file
    +trace      generate tracelog

For example, following command will generate the VCD dump.

    cd sim && ./riscsim +dumpvcd

Use +trace to generate a trace log, which can be compared with the log file of the software simulator to ensure that the RTL simulation is correct.

    cd sim && ./riscsim +trace

The RTL testbench read the memory initialize file from sw/imem.bin and sw/dmem.bin.

## Software Simulator

Tracegen is a software simulator that can generate trace logs for comparison with RTL simulation results. It can also set parameters of branch penalty to run benchmarks to see the effect of branch penalty. The branch instructions of hardware is two instructions delay for branch penalties.

    Usage: tracegen [-h] [-b n] [-l logfile]

           --help, -n              help
           --branch n, -b n        branch penalty (default 2)
           --log file, -l file     generate log file

The software simulator and hardware supports RV32IM instruction sets.

## Bechmark

Here is the RV32I simulation result of benchmarks.

### Dhrystone

<img src="https://github.com/kuopinghsu/simple-riscv/blob/master/images/dhrystone.png" alt="Dhrystone" width=640>

### Coremark

<img src="https://github.com/kuopinghsu/simple-riscv/blob/master/images/coremark.png" alt="Coremark" width=640>

> Note: Coremark requires a total time of more than 10 seconds, but this will result in a longer simulation time. This Coremark value provides a reference when the iteration is 4.

## Cycles per Instruction Performance

Two instructions brach panelty if branch taken, CPI is 1 for other instructions. The average Cycles per Instruction (CPI) is approximately 1.2 on Dhrystone diag.

This core is three-stage pipeline processors, which is Fetch & Decode (F/D), execution (E) and write back (WB).

* Register Forwarding

The problem with data hazards, introduced by this sequence of instructions can be solved with a simple hardware technique called forwarding. When the execution result accesses the same register, the execution result is directly forwarded to the next instruction.

<img src="https://github.com/kuopinghsu/simple-riscv/blob/master/images/forwarding.svg" alt="Register Forwarding" width=480>

* Branch Penalty

When the branch is taken during the execute phase, it needs to flush the instructions that have been fetched into the pipeline, which causes a delay of two instructions, so the extra cost of the branch is two.

<img src="https://github.com/kuopinghsu/simple-riscv/blob/master/images/branch.svg" alt="Branch Penalty" width=480>

## Memory Interface

One instruction memory and one data memory. The instuction memory is read-only for one read port, while data memory is two port, one for reading and one for writing.

## TODO

* pass formal verification
* merge the memory interface into one memory for one port
* support ECALL and EBREAK instructions
* interrupt handling
* support RV32C compress extension
* support FreeRTOS
* branch predictor??

# License

MIT license

