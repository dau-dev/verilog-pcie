
# FPGA settings
FPGA_PART = xcvu9p-fsgd2104-2L-e
FPGA_TOP = fpga
FPGA_ARCH = virtexuplus

# Files for synthesis
SYN_FILES = rtl/fpga_au200.v
SYN_FILES += rtl/fpga_core.v
SYN_FILES += rtl/common/example_core_pcie_us.v
SYN_FILES += rtl/common/example_core_pcie.v
SYN_FILES += rtl/common/example_core.v
SYN_FILES += rtl/common/axi_ram.v
SYN_FILES += lib/pcie/rtl/pcie_us_if.v
SYN_FILES += lib/pcie/rtl/pcie_us_if_rc.v
SYN_FILES += lib/pcie/rtl/pcie_us_if_rq.v
SYN_FILES += lib/pcie/rtl/pcie_us_if_cq.v
SYN_FILES += lib/pcie/rtl/pcie_us_if_cc.v
SYN_FILES += lib/pcie/rtl/pcie_us_cfg.v
SYN_FILES += lib/pcie/rtl/pcie_axil_master.v
SYN_FILES += lib/pcie/rtl/pcie_axi_master.v
SYN_FILES += lib/pcie/rtl/pcie_axi_master_rd.v
SYN_FILES += lib/pcie/rtl/pcie_axi_master_wr.v
SYN_FILES += lib/pcie/rtl/pcie_tlp_demux_bar.v
SYN_FILES += lib/pcie/rtl/pcie_tlp_demux.v
SYN_FILES += lib/pcie/rtl/pcie_tlp_mux.v
SYN_FILES += lib/pcie/rtl/pcie_tlp_fifo.v
SYN_FILES += lib/pcie/rtl/pcie_tlp_fifo_raw.v
SYN_FILES += lib/pcie/rtl/pcie_msix.v
SYN_FILES += lib/pcie/rtl/dma_if_pcie.v
SYN_FILES += lib/pcie/rtl/dma_if_pcie_rd.v
SYN_FILES += lib/pcie/rtl/dma_if_pcie_wr.v
SYN_FILES += lib/pcie/rtl/dma_psdpram.v
SYN_FILES += lib/pcie/rtl/priority_encoder.v
SYN_FILES += lib/pcie/rtl/pulse_merge.v

# XDC files
XDC_FILES = fpga_au200.xdc

# IP
IP_TCL_FILES = ip/pcie4_uscale_plus_0.tcl

# Configuration
CONFIG_TCL_FILES = ./config.tcl

include ../common/vivado.mk

%_fallback.bit: %.bit
	echo "open_project $*.xpr" > generate_fallback_bit.tcl
	echo "open_run impl_1" >> generate_fallback_bit.tcl
	echo "startgroup" >> generate_fallback_bit.tcl
	echo "set_property BITSTREAM.CONFIG.CONFIGFALLBACK ENABLE [current_design]" >> generate_fallback_bit.tcl
	echo "set_property BITSTREAM.CONFIG.TIMER_CFG 0x03000000 [current_design]" >> generate_fallback_bit.tcl
	echo "set_property BITSTREAM.CONFIG.NEXT_CONFIG_REBOOT ENABLE [current_design]" >> generate_fallback_bit.tcl
	echo "set_property BITSTREAM.CONFIG.NEXT_CONFIG_ADDR 0x04000000 [current_design]" >> generate_fallback_bit.tcl
	echo "endgroup" >> generate_fallback_bit.tcl
	echo "write_bitstream -verbose -force $*_fallback.bit" >> generate_fallback_bit.tcl
	echo "undo" >> generate_fallback_bit.tcl
	echo "exit" >> generate_fallback_bit.tcl
	vivado -nojournal -nolog -mode batch -source generate_fallback_bit.tcl
	mkdir -p rev
	EXT=bit; COUNT=100; \
	while [ -e rev/$*_rev$$COUNT.$$EXT ]; \
	do COUNT=$$((COUNT+1)); done; \
	COUNT=$$((COUNT-1)); \
	cp $@ rev/$*_fallback_rev$$COUNT.$$EXT; \
	echo "Output: rev/$*_fallback_rev$$COUNT.$$EXT";

program: $(FPGA_TOP).bit
	echo "open_hw" > program.tcl
	echo "connect_hw_server" >> program.tcl
	echo "open_hw_target" >> program.tcl
	echo "current_hw_device [lindex [get_hw_devices] 0]" >> program.tcl
	echo "refresh_hw_device -update_hw_probes false [current_hw_device]" >> program.tcl
	echo "set_property PROGRAM.FILE {$(FPGA_TOP).bit} [current_hw_device]" >> program.tcl
	echo "program_hw_devices [current_hw_device]" >> program.tcl
	echo "exit" >> program.tcl
	vivado -nojournal -nolog -mode batch -source program.tcl

%.mcs %.prm: %.bit
	echo "write_cfgmem -force -format mcs -size 128 -interface SPIx4 -loadbit {up 0x04000000 $*.bit} -checksum -file $*.mcs" > generate_mcs.tcl
	echo "exit" >> generate_mcs.tcl
	vivado -nojournal -nolog -mode batch -source generate_mcs.tcl
	mkdir -p rev
	COUNT=100; \
	while [ -e rev/$*_rev$$COUNT.bit ]; \
	do COUNT=$$((COUNT+1)); done; \
	COUNT=$$((COUNT-1)); \
	for x in .mcs .prm; \
	do cp $*$$x rev/$*_rev$$COUNT$$x; \
	echo "Output: rev/$*_rev$$COUNT$$x"; done;

%_fallback.mcs %_fallback.prm: %_fallback.bit
	echo "write_cfgmem -force -format mcs -size 128 -interface SPIx4 -loadbit {up 0x00000000 $*_fallback.bit} -checksum -file $*_fallback.mcs" > generate_fallback_mcs.tcl
	echo "exit" >> generate_fallback_mcs.tcl
	vivado -nojournal -nolog -mode batch -source generate_fallback_mcs.tcl
	mkdir -p rev
	COUNT=100; \
	while [ -e rev/$*_rev$$COUNT.bit ]; \
	do COUNT=$$((COUNT+1)); done; \
	COUNT=$$((COUNT-1)); \
	for x in .mcs .prm; \
	do cp $*_fallback$$x rev/$*_fallback_rev$$COUNT$$x; \
	echo "Output: rev/$*_fallback_rev$$COUNT$$x"; done;

%_full.mcs %_full.prm: %_fallback.bit %.bit
	echo "write_cfgmem -force -format mcs -size 128 -interface SPIx4 -loadbit {up 0x00000000 $*_fallback.bit up 0x04000000 $*.bit} -checksum -file $*_full.mcs" > generate_full_mcs.tcl
	echo "exit" >> generate_full_mcs.tcl
	vivado -nojournal -nolog -mode batch -source generate_full_mcs.tcl
	mkdir -p rev
	COUNT=100; \
	while [ -e rev/$*_rev$$COUNT.bit ]; \
	do COUNT=$$((COUNT+1)); done; \
	COUNT=$$((COUNT-1)); \
	for x in .mcs .prm; \
	do cp $*_full$$x rev/$*_full_rev$$COUNT$$x; \
	echo "Output: rev/$*_full_rev$$COUNT$$x"; done;

flash: $(FPGA_TOP).mcs $(FPGA_TOP).prm
	echo "open_hw" > flash.tcl
	echo "connect_hw_server" >> flash.tcl
	echo "open_hw_target" >> flash.tcl
	echo "current_hw_device [lindex [get_hw_devices] 0]" >> flash.tcl
	echo "refresh_hw_device -update_hw_probes false [current_hw_device]" >> flash.tcl
	echo "create_hw_cfgmem -hw_device [current_hw_device] [lindex [get_cfgmem_parts {mt25qu01g-spi-x1_x2_x4}] 0]" >> flash.tcl
	echo "current_hw_cfgmem -hw_device [current_hw_device] [get_property PROGRAM.HW_CFGMEM [current_hw_device]]" >> flash.tcl
	echo "set_property PROGRAM.FILES [list \"$(FPGA_TOP).mcs\"] [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.PRM_FILES [list \"$(FPGA_TOP).prm\"] [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.ERASE 1 [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.CFG_PROGRAM 1 [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.VERIFY 1 [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.CHECKSUM 0 [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.ADDRESS_RANGE {use_file} [current_hw_cfgmem]" >> flash.tcl
	echo "set_property PROGRAM.UNUSED_PIN_TERMINATION {pull-none} [current_hw_cfgmem]" >> flash.tcl
	echo "create_hw_bitstream -hw_device [current_hw_device] [get_property PROGRAM.HW_CFGMEM_BITFILE [current_hw_device]]" >> flash.tcl
	echo "program_hw_devices [current_hw_device]" >> flash.tcl
	echo "refresh_hw_device [current_hw_device]" >> flash.tcl
	echo "program_hw_cfgmem -hw_cfgmem [current_hw_cfgmem]" >> flash.tcl
	echo "boot_hw_device [current_hw_device]" >> flash.tcl
	echo "exit" >> flash.tcl
	vivado -nojournal -nolog -mode batch -source flash.tcl

flash%: $(FPGA_TOP)%.mcs $(FPGA_TOP)%.prm
	echo "open_hw" > flash$*.tcl
	echo "connect_hw_server" >> flash$*.tcl
	echo "open_hw_target" >> flash$*.tcl
	echo "current_hw_device [lindex [get_hw_devices] 0]" >> flash$*.tcl
	echo "refresh_hw_device -update_hw_probes false [current_hw_device]" >> flash$*.tcl
	echo "create_hw_cfgmem -hw_device [current_hw_device] [lindex [get_cfgmem_parts {mt25qu01g-spi-x1_x2_x4}] 0]" >> flash$*.tcl
	echo "current_hw_cfgmem -hw_device [current_hw_device] [get_property PROGRAM.HW_CFGMEM [current_hw_device]]" >> flash$*.tcl
	echo "set_property PROGRAM.FILES [list \"$(FPGA_TOP)$*.mcs\"] [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.PRM_FILES [list \"$(FPGA_TOP)$*.prm\"] [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.ERASE 1 [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.CFG_PROGRAM 1 [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.VERIFY 1 [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.CHECKSUM 0 [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.ADDRESS_RANGE {use_file} [current_hw_cfgmem]" >> flash$*.tcl
	echo "set_property PROGRAM.UNUSED_PIN_TERMINATION {pull-none} [current_hw_cfgmem]" >> flash$*.tcl
	echo "create_hw_bitstream -hw_device [current_hw_device] [get_property PROGRAM.HW_CFGMEM_BITFILE [current_hw_device]]" >> flash$*.tcl
	echo "program_hw_devices [current_hw_device]" >> flash$*.tcl
	echo "refresh_hw_device [current_hw_device]" >> flash$*.tcl
	echo "program_hw_cfgmem -hw_cfgmem [current_hw_cfgmem]" >> flash$*.tcl
	echo "boot_hw_device [current_hw_device]" >> flash$*.tcl
	echo "exit" >> flash$*.tcl
	vivado -nojournal -nolog -mode batch -source flash$*.tcl

