OUTPUT := bitstream
PACKAGE := CABGA256
TOP_MODULE = top_icepi_zero_c64

VERILOG_FILES = \
VHDL_FILES = \

include files.mk

all: debug

# NEXTPNR_OPTIONS = --timing-allow-fail
%.json: $(VERILOG_SOURCES) $(VHDL_FILES) build_wave_mem
	rm -f $(OUTPUT).bit $(OUTPUT).config $(OUTPUT).json
	yosys -m ghdl \
		  -p "ghdl --ieee=synopsys --std=08 -fexplicit -frelaxed-rules $(VHDL_FILES) -e $(TOP_MODULE)" \
	      -p "read_verilog -sv $(VERILOG_FILES)" \
	      -p "hierarchy -top ${TOP_MODULE}" \
	      -p 'synth_ecp5 -json $@'

build_wave_mem:
	make -C ../rtl/sid8580

%.config: %.json icepi-zero.lpf
	nextpnr-ecp5 --25k --package $(PACKAGE) --lpf icepi-zero.lpf --router router2 --json $< --textcfg $@ # 2> log.txt
	# cat log.txt | grep Device -A 28

%.bit: %.config
	ecppack $< $@

build: $(OUTPUT).bit

debug: build
	openFPGALoader -cft231X --pins=7:3:5:6 $(OUTPUT).bit

install: build
	openFPGALoader -cft231X --pins=7:3:5:6 $(OUTPUT).bit --write-flash

install-bitstream:
	openFPGALoader -cft231X --pins=7:3:5:6 $(OUTPUT).bit --write-flash

lint:
	verilator --lint-only -Wall -Wno-DECLFILENAME -Wno-WIDTHEXPAND $(VERILOG_SOURCES)

help:
	echo "Usage: make [option]"
	echo "Options:"
	echo "- install: install to flash"
	echo "- debug: install to chip's temp memory (bitstream lost on power loss)"
	echo "- build: builds the bitstream"
	echo "- clean: delete all temparary files"

clean:
	rm -f $(OUTPUT).bit $(OUTPUT).config $(OUTPUT).json

.PHONY: build clean install
