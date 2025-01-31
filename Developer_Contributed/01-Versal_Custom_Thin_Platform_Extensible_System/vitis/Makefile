 #
 # Copyright 2021 Xilinx, Inc.
 #
 # Licensed under the Apache License, Version 2.0 (the "License");
 # you may not use this file except in compliance with the License.
 # You may obtain a copy of the License at
 #
 #     http://www.apache.org/licenses/LICENSE-2.0
 #
 # Unless required by applicable law or agreed to in writing, software
 # distributed under the License is distributed on an "AS IS" BASIS,
 # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 # See the License for the specific language governing permissions and
 # limitations under the License.
 #

ECHO=@echo

TARGET            := hw
PLATFORM_CUSTOM   ?= vck190_thin
BASE_PFM          ?= ./platform/sw/build/${PLATFORM_CUSTOM}/export/${PLATFORM_CUSTOM}/${PLATFORM_CUSTOM}.xpfm
SYSTEM_CFG        := ./src/system.cfg
XPFM              := ../${BASE_PFM}
PETALINUX_IMAGES  := ../petalinux/linux/images/linux
KERNEL            := vadd
IP_DIR            := ../ip
XO                := $(wildcard ${IP_DIR}/*.xo)
GRAPH_O           := ${IP_DIR}/aie/libadf.a
XCLBIN            := my.xclbin
PS_APPS_DIR       := ../ps_apps
PS_EXE            := $(wildcard ${PS_APPS_DIR}/*.exe)
PS_APPS           := $(addprefix --package.sd_file=, ${PS_EXE})
SD_CARD           := ../package_output

.PHONY: clean all

help::
	$(ECHO) "Makefile Usage:"
	$(ECHO) "  make all"
	$(ECHO) "      Command to generate the sd_card in this design, including platform/petalinux/ip/ps_apps"
	$(ECHO) ""
	$(ECHO) "  make clean"
	$(ECHO) "      Command to remove all the generated files."

all: ${XCLBIN} ${SD_CARD}

${XCLBIN}: ${XO}
	v++ -l -t ${TARGET} --save-temps --platform ${XPFM} --config ${SYSTEM_CFG} ${XO} ${GRAPH_O} -o ${XCLBIN} |& tee ${VPP}_link.log

${SD_CARD}: ${XCLBIN} ${PS_EXE} ${GRAPH_O}
	v++ -p --package.out_dir ${SD_CARD} -t ${TARGET} --save-temps -f ${XPFM} --package.image_format=fat32 --package.boot_mode=sd --package.rootfs=${PETALINUX_IMAGES}/rootfs.cpio.gz.u-boot  --package.kernel_image=${PETALINUX_IMAGES}/Image ${XCLBIN} ${GRAPH_O} ${PS_APPS}
	echo ${PLATFORM_CUSTOM} > ${SD_CARD}/sd_card/platform_desc.txt

clean:
	rm -f *.log *.xclbin *summary *info 
	rm -rf .Xil _x/ ${SD_CARD} .ipcache

