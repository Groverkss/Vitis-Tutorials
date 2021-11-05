<table class="sphinxhide" width="100%">
 <tr width="100%">
    <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>AI Engine Development</h1>
    <a href="https://www.xilinx.com/products/design-tools/vitis.html">See Vitis™ Development Environment on xilinx.com</br></a>
    <a href="https://www.xilinx.com/products/design-tools/vitis/vitis-ai.html">See Vitis-AI™ Development Environment on xilinx.com</a>
    </td>
 </tr>
</table>

# AI Engine Source Code Debug with Hardware Emulator

A system project includes PS, PL, and AI Engine development and additionally a system wide configuration that requires to be configured correctly. Hardware emulator supports the system project debug needs and is capable of debug PS and AI Engine domains. PL domain debug is not supported currently.

The following showcase features of the hardware emulator:

[ Hardware Emulation Source Code Debug](#Hardware-emulation-source-code-debug)
* [Hardware emulation source code debug limitations](#Hardware-emulation-source-code-debug-limitations)

[ Command-Line Project Source Code Debug with Hardware Emulator](#Command-line-project-source-code-debug-with-hardware-emulator)


## Hardware Emulation Source Code Debug

### Step 1. Build

Select the build configuration to be **Emulation-HW**.

![missing image](./images/he_run_cfg.png)

Highlight beamformer system project and right-click to select **build project**.
It takes over 60 minutes to complete the build.

### Step 2. Launch Hardware Emulator

![missing image](./images/he_run_init.png)

**Note:** It takes a couple of minutes to launch the emulator and boot up Petalinux.

### Step 3. During Debug

#### Step 3.1. Debug - PS

![missing image](./images/he_run_ps.png)

#### Step 3.2. Debug - AI Engine

![missing image](./images/he_run_aie.png)

![missing image](./images/he_run_aie_1.png)

AI Engine only debugs within system project via hardware emulator, AI Engine emulator debug still applies. <a href="Debug2_ai.md">AI Engine debug with AIE emulator</a> are applicable at this condition.

## Hardware Emulation Source Code Debug Limitations

1. Limitations from <a href="Debug2_ai.md">AI Engine debug with AIE emulator</a> applies.
2. PL debug is not supported from Vitis™ IDE.


## Command-Line Project Source Code Debug with Hardware Emulator

Use cases that involve using Vitis IDE functionalities without migrating to Vitis IDE are permitted.

### Step 1. Build Project to be Debugged

If the project is not built already, use this tutorial's `Makefile.emu` that has `--package.enable_aie_debug` option in the packaging step. This option inserts configuration data object (CDO) that generates stop requests for the AI Engine cores, so that they stop at the reset vector. This option is required to add debug capabilities. Issue command `make package_dbg` with this tutorial's `Makefile.emu` to package binaries that can be debugged.
Note: The packaged binaries that have debug capabilities require to run with debugger. Run without debugger will see execution hang due to wait for debugger invocation. Run command `make package` to package back regular binaries.

```bash
cp Makefile.emu Makefile
make
make package_dbg
```

### Step 2. Launch Hardware Emulator and Boot Petalinux

From terminal 1, setup tool path properly and issue this command to launch hardware emulator and boot up Petalinux.

```bash
./launch_hw_emu.sh -add-env ENABLE_RDWR_DEBUG=true -add-env RDWR_DEBUG_PORT=10100 -pid-file emulation.pid -no-reboot -forward-port 1440 1534
# Run next command in emulator shell after emulator is up and ready to debug
source /mnt/sd-mmcblk0p1/init.sh
```

Command option explanation:

1. `-add-env RDWR_DEBUG_PORT=${aie_mem_sock_port}`: Defines the port for communicating with the AI Engine domain. In the previous example, it is 10100.
2. `-forward-port ${linux_tcf_agent_port} 1534`: Defines the port for the Linux TCF agent. In the previous example, it is 1440, which is the default.

Note:
1. `launch_hw_emu.sh` is generated properly when the project under debug is built and packaged with hardware emulator correctly. Update repository's Makefile line 3 from "TARGET = hw" to "TARGET = hw_emu".
2. Previous command takes a few minutes to complete due to both hardware emulator and Petalinux are required to boot up properly.
3. Wait until both hardware server and Petalinux boot up BEFORE moving to next steps.

### Step 3. Launch Vitis IDE and XRT server

From terminal 2, setup tool path properly and issue this command to launch XRT server and Vitis IDE.

```bash
vitis -debug -flow embedded_accel -target hw_emu -exe ./host.exe -program-args ./a.xclbin -port 1440
xrt_server -I30000 -S -s tcp::4352
```

Command option explanation:

1. `vitis -debug`: Launches the Vitis IDE in stand-alone debug mode.
2. `-flow embedded_accel`: Specifies the embedded processor application acceleration flow.
3. `-target hw_emu`: Indicates the target build being debugged.
4. `-exe ./ps_app`: Indicates the PS application to run and debug.
5. `-program-args ${xcl_bin_dir}/binary_container_1.xclbin`: Refers to the location of the xclbin file to be loaded as an argument to the executable.
6. `-port 1440`: Specifies the ${linux_tcf_agent_port} as discussed previously.
7. `-I30000`: Defines an idle timeout in seconds, in which the server will quit if there is no response.
8. `-S`: Specifies print server properties in JSON format to stdout.
9. `-s tcp::${xrt_server_port}`: Defines the agent listening protocol and port. it is 4352 in example, but can be any free port.

Expected result
![missing image](./images/he_temp_run.png)

**Note:** The previously listed step takes a couple of minutes to complete.

### Step 4. Setup Vitis AI Engine Debug Configuration

#### Step 4.1. Configure Debug using Click-on Debug Icon

![missing image](./images/he_cl_config0.png)

#### Step 4.2. Under "Single Application Debug" to Create a New Debug Configuration

![missing image](./images/he_cl_config1.png)

#### Step 4.3. Setup New Debug Type Name and Connection

![missing image](./images/he_cl_config2.png)

#### Step 4.4. Setup New Connection Name, Host, and Port

![missing image](./images/he_cl_config3.png)

#### Step 4.5. Setup Execute Script of Newly Created Debug Type

![missing image](./images/he_cl_config4.png)

**Note:** The script, **aie_app_debug_em.tcl** is provided in this tutorial and needs to be updated to match your environment settings. The `aie_work_dir` variable should be the Work folder inside this lab –e.g. `set aie_work_dir "${PROJECT_PATH}/beamformer/Work/"` and `set vitis_install "${XILINX_VITIS_PATH}/Xilinx/Vitis/2021.1"`

#### Step 4.6. Close the debug configuration

![missing image](./images/he_cl_config5.png)

#### Step 4.7. Run PS (by resume) Application so AI Engine can be Initialized and Debugged

![missing image](./images/he_cl_config6.png" width="450" >

#### Step 4.8. Wait until PS has Initialized AI Engine and Observe "Starting graph run" from Console

![missing image](./images/he_cl_config6_1.png)

#### Step 4.9. Launch AI Engine Debugger to Debug the AI Engine Sub-project

![missing image](./images/he_cl_config0.png)

![missing image](./images/he_cl_config7.png)

**Note:**
1. The script, **aie_app_debug_em.tcl** is provided in this tutorial and needs to be updated to match your environment settings. The `aie_work_dir` variable should be the Work folder inside this lab. For example, `set aie_work_dir ${PROJECT_PATH}/beamformer/Work/` and `set vitis_install ${XILINX_VITIS_PATH}/Xilinx/Vitis/2021.1`
2. Step 4.7 requires to start PS application BEFORE launching debugger to debug AI Engine sub-project. Will see issues if above step sequence is not followed.
3. It takes some time depends on number of times in design to launch AI Engine debugger completely. Beamformer design contains 64 tiles that needs several minutes to complete.

### Step 5. Expected Vitis IDE

![missing image](./images/he_cl_run.png)

### Step 6. Clean up Launched Processes

```bash
killall -9 pllauncher; killall -9 qemu-system-aarch64; killall -9 xrt_server
```

These commands kill leftover processes for this debug run.


# License

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0


Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

<p align="center"><sup>XD005 | &copy; Copyright 2021 Xilinx, Inc.</sup></p>