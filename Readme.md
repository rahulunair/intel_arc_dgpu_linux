# Configure Linux to utilize Intel Arc A370M Xe-HPG discrete GPU

These instructions are for Ubuntu 22.04 LTS based Oses. I am using Pop!_OS 22.04 LTS derivative of Ubuntu 22.04.

## Configure Linux kernel


1. Install the latest available Linux kernel, for example I have installed **kernel 6.0** using Mainline tool

Install a recent version of the kernel. The easiest way to install a different kernel is by using [Mainline](https://github.com/bkw777/mainline).

```bash
root@runnikri-mobl Coding → add-apt-repository ppa:cappelikan/ppa
root@runnikri-mobl Coding → apt update
root@runnikri-mobl Coding → apt install mainline
```
As seen in the sceenshot below, I am using Kernel 6.0.0:

![image](https://user-images.githubusercontent.com/45503355/195398959-c28fe9f3-47c9-46cd-b1a0-5e26d8e6228a.png)

After installing the kernel please restart the machine.

2. As Arc dGPUs support are still `experimental` in the kernel, you will have to force the dGPU to be detected. This can be done using the kernel force_probe parameter `i915.force_probe=<device_id>` for Intel i915 HD graphics driver. This will force probe the driver for new Intel graphics devices that are recognized by the kernel but not properly supported. Hopefully, with a newer version of the kernel you wouldn't have to do that. I am using a [Yoga 7i (16" Intel) with Intel Arc Graphics](https://www.lenovo.com/us/en/p/laptops/yoga/yoga-2-in-1-series/yoga-7i-gen-7-(16-inch-intel)) which has the Intel® Arc™ A370M discrete Graphics card (dgpu) along with an integrated Intel® UHD Graphics. Device id for this dgpu is `5693`, you can find the device id of the card either looking at i915 logs using `sudo dmesg | grep -i i915` or from Intel's gpu [hardware table](https://dgpu-docs.intel.com/devices/hardware-table.html).

Pop!_OS 22.04 LTS uses systemd to manage kernel boot params, to force i915 driver to enable the dgpu use `kernelstub` tool:

```bash
root@runnikri-mobl Coding → kernelstub -a "i915.force_probe=5693"
```

After this restart the machine and check i915 logs using `dmesg` to see if the graphics card has been detected:

```bash
root@runnikri-mobl Coding → dmesg | grep -i i915
```

You should see an output like:

```bash
root@runnikri-mobl Coding →  dmesg | grep -i i915
[    0.000000] Command line: initrd=\EFI\Pop_OS-97fe6a26-7d8a-4120-89db-8f2130b644b7\initrd.img root=UUID=97fe6a26-7d8a-4120-89db-8f2130b644b7 ro quiet loglevel=0 systemd.show_status=false splash i915.force_probe=5693
[    0.047217] Kernel command line: initrd=\EFI\Pop_OS-97fe6a26-7d8a-4120-89db-8f2130b644b7\initrd.img root=UUID=97fe6a26-7d8a-4120-89db-8f2130b644b7 ro quiet loglevel=0 systemd.show_status=false splash i915.force_probe=5693
[    1.775328] i915 0000:00:02.0: [drm] VT-d active for gfx access
[    1.775383] i915 0000:00:02.0: vgaarb: deactivate vga console
[    1.775416] i915 0000:00:02.0: [drm] Using Transparent Hugepages
[    1.775979] i915 0000:00:02.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=none:owns=io+mem
[    1.777234] i915 0000:00:02.0: [drm] Finished loading DMC firmware i915/adlp_dmc_ver2_16.bin (v2.16)
[    1.912397] i915 0000:00:02.0: [drm] GuC firmware i915/adlp_guc_70.1.1.bin version 70.1
[    1.912399] i915 0000:00:02.0: [drm] HuC firmware i915/tgl_huc_7.9.3.bin version 7.9
[    1.926153] i915 0000:00:02.0: [drm] HuC authenticated
[    1.926448] i915 0000:00:02.0: [drm] GuC submission enabled
[    1.926449] i915 0000:00:02.0: [drm] GuC SLPC enabled
[    1.927353] i915 0000:00:02.0: [drm] GuC RC: enabled
[    1.930736] i915 0000:00:02.0: [drm] Protected Xe Path (PXP) protected content support initialized
[    3.769893] [drm] Initialized i915 1.6.0 20201103 for 0000:00:02.0 on minor 0
[    3.775461] i915 0000:03:00.0: enabling device (0000 -> 0002)
[    3.775491] i915 0000:03:00.0: [drm] Incompatible option enable_guc=3 - HuC is not supported!
[    3.776346] i915 0000:03:00.0: [drm] VT-d active for gfx access
[    3.776472] i915 0000:03:00.0: [drm] Local memory IO size: 0x00000003fa000000
[    3.776476] i915 0000:03:00.0: [drm] Local memory available: 0x00000003fa000000
[    3.787361] fbcon: i915drmfb (fb0) is primary device
[    3.787366] i915 0000:00:02.0: [drm] fb0: i915drmfb frame buffer device
[    3.796583] i915 0000:03:00.0: [drm] Finished loading DMC firmware i915/dg2_dmc_ver2_06.bin (v2.6)
[    5.326692] i915 0000:03:00.0: [drm] failed to retrieve link info, disabling eDP
[    5.428126] i915 0000:03:00.0: [drm] GuC firmware i915/dg2_guc_70.1.2.bin version 70.1
[    5.442889] i915 0000:03:00.0: [drm] GuC submission enabled
[    5.442892] i915 0000:03:00.0: [drm] GuC SLPC enabled
[    5.443378] i915 0000:03:00.0: [drm] GuC RC: enabled
```

The dg2_guc is loaded as seen in the above line: **[    5.428126] i915 0000:03:00.0: [drm] GuC firmware i915/dg2_guc_70.1.2.bin version 70.1**. Both the igpu and dgpu is detected and also the firmware is loaded. If drm verification fails and [GuC](https://01.org/linuxgraphics/downloads/firmware) is not loaded, install the latest linux firmware from this [link](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/):

You can install the latest firmware files by cloning the repo and moving it to `/lib/firmware` (**warning**: this is a bruteforce approach):

```bash
root@runnikri-mobl Coding →  git clone https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/
root@runnikri-mobl Coding →  cd linux-firmware && yes | cp -r * /lib/firmware
```

For me the firmware that came with Pop!_OS 22.04 LTS worked, i didnt have to install the latest ones.

## Setup Open source Mesa 3d Graphics libraries for openGL and Vulkan

1. Install drivers

Now if you want media and graphics support beyond compute, install bleeding edge Mesa libraries from [oiabf ppa](https://launchpad.net/~oibaf/+archive/ubuntu/graphics-drivers) that provides open graphics drivers, you can do it by:

```bash
root@runnikri-mobl Coding → add-apt-repository ppa:oibaf/graphics-drivers
root@runnikri-mobl Coding → apt-update
root@runnikri-mobl Coding → apt-upgrade

root@runnikri-mobl Coding → dpkg -l | grep -i mesa
ii  libegl-mesa0:amd64                                                      22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        free implementation of the EGL API -- Mesa vendor library
ii  libgl1-mesa-dri:amd64                                                   22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        free implementation of the OpenGL API -- DRI modules
ii  libglapi-mesa:amd64                                                     22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        free implementation of the GL API -- shared library
ii  libglu1-mesa:amd64                                                      9.0.2-1                                                           amd64        Mesa OpenGL utility library (GLU)
ii  lib
-mesa0:amd64                                                      22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        free implementation of the OpenGL API -- GLX vendor library
ii  mesa-utils                                                              8.4.0-1ubuntu1                                                    amd64        Miscellaneous Mesa utilities -- symlinks
ii  mesa-utils-bin:amd64                                                    8.4.0-1ubuntu1                                                    amd64        Miscellaneous Mesa utilities -- native applications
ii  mesa-va-drivers:amd64                                                   22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        Mesa VA-API video acceleration drivers
ii  mesa-vdpau-drivers:amd64                                                22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        Mesa VDPAU video acceleration drivers
ii  mesa-vulkan-drivers:amd64                                               22.3~git2210120600.ddc5c3~oibaf~j                                 amd64        Mesa Vulkan graphics drivers

```
Although be careful as these drivers are bleeding edge and by design may not be stable. After installing the Mesa drivers you should be able to run glx and vulkan benchmarks. 

2. Check if opengl detects the dgpu using glxinfo

Install glxinfo from apt repo mesa-utils

To use the dGPU, set the env variable `DRI_PRIME=1`, [PRIME](https://wiki.archlinux.org/title/PRIME) is a technology in linux that uses open source graphics drivers to use `switchable graphics` and install glxinfo from apt repo mesa-utils.

Here is the output of glxinfo without setting `DRI_PRIME` environment variable:

```bash
root@runnikri-mobl Coding → glxinfo -B | grep -i device
    Device: Mesa Intel(R) Graphics (ADL GT2) (0x46a6)
```

After setting the environment variable for PRIME:

```bash
root@runnikri-mobl Coding → export DRI_PRIME=1
root@runnikri-mobl Coding → glxinfo -B | grep -i device
    Device: Mesa Intel(R) Arc(tm) A370M Graphics (DG2) (0x5693)
```

As  you can the dgpu is recognized by Mesa OpenGL. yay!

3. Let's try to run a benchmark on the Arc gpu to see how it performance. I am using glmark2, which can be installed on Ubuntu based Oses easily using:

```bash
root@runnikri-mobl Coding → apt-get install glmark2
```

```bash
root@runnikri-mobl Coding → export DRI_PRIME=1
root@runnikri-mobl Coding → glxinfo -B | grep -i device
    Device: Mesa Intel(R) Arc(tm) A370M Graphics (DG2) (0x5693)
root@runnikri-mobl Coding → glmark2
```

![glmark2](assets/arc370m_bench.gif)

4. Now let's move to installing the intel compute drivers. [Goto](https://github.com/intel/compute-runtime/releases) and get the latest release and install using the deb packages for opencl, level zero, igc etc. 

As of this writing the latest compute driver release version is [22.39.24347](https://github.com/intel/compute-runtime/releases/tag/22.39.24347):

```bash
root@runnikri-mobl Coding → cd /tmp && mkdir compute_drivers
root@runnikri-mobl Coding → cd compute_drivers
root@runnikri-mobl Coding → wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.12149.1/intel-igc-core_1.0.12149.1_amd64.deb
root@runnikri-mobl Coding → wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.12149.1/intel-igc-opencl_1.0.12149.1_amd64.deb
root@runnikri-mobl Coding → wget https://github.com/intel/compute-runtime/releases/download/22.39.24347/intel-level-zero-gpu_1.3.24347_amd64.deb
root@runnikri-mobl Coding → wget https://github.com/intel/compute-runtime/releases/download/22.39.24347/intel-opencl-icd_22.39.24347_amd64.deb
root@runnikri-mobl Coding → wget https://github.com/intel/compute-runtime/releases/download/22.39.24347/libigdgmm12_22.2.0_amd64.deb
root@runnikri-mobl Coding → dpkg -i *.deb
```


Now that we have the compute drivers installed, let's see if opencl can detect the dgpu. Install `clinfo` from apt and check using:

```bash
root@runnikri-mobl Coding → clinfo | grep "0x5690"
  Device Name                                     Intel(R) Graphics [0x5693]
    Device Name                                   Intel(R) Graphics [0x5693]
    Device Name                                   Intel(R) Graphics [0x5693]
    Device Name                                   Intel(R) Graphics [0x5693]

```
We can see that the dgpu has been detected.

## Install oneAPI basekit and device discovery using sycl

1. Finally install oneapi basekit so that we can use the dpcpp runtime, I used 2022.2.0 version of oneapi basekit.

```bash
04:37:56  |base|rahul@pop-os ~ → dpcpp -v
Intel(R) oneAPI DPC++/C++ Compiler 2022.2.0 (2022.2.0.20220730)
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /opt/intel/oneapi/compiler/2022.2.0/linux/bin-llvm
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/11
Selected GCC installation: /usr/lib/gcc/x86_64-linux-gnu/11
Candidate multilib: .;@m64
Selected multilib: .;@m64
```
2. Device discovery using sycl-ls see if sycl can detect the dgpu:

Intel dgpus like the A370m are represented as SYCL devices. `sycl-ls` is a tool that is part of the oneAPI basekit that can show all the detected devices and all the SYCL backends support by the runtime. Once oneapi basekit has been installed, source the environment using:

```bash
root@runnikri-mobl Coding → source /opt/intel/oneapi/setvars.sh 
```
Device discovery using syclto see if sycl can detect the dgpu:

```bash
root@runnikri-mobl Coding → sycl-ls
[opencl:acc:0] Intel(R) FPGA Emulation Platform for OpenCL(TM), Intel(R) FPGA Emulation Device 1.2 [2022.14.7.0.30_160000]
[opencl:cpu:1] Intel(R) OpenCL, 12th Gen Intel(R) Core(TM) i7-12700H 3.0 [2022.14.7.0.30_160000]
[opencl:gpu:2] Intel(R) OpenCL HD Graphics, Intel(R) Graphics [0x5693] 3.0 [22.40.024349]
[opencl:gpu:3] Intel(R) OpenCL HD Graphics, Intel(R) Graphics [0x46a6] 3.0 [22.40.024349]
[ext_oneapi_level_zero:gpu:0] Intel(R) Level-Zero, Intel(R) Graphics [0x5693] 1.3 [1.3.24349]
[ext_oneapi_level_zero:gpu:1] Intel(R) Level-Zero, Intel(R) Graphics [0x46a6] 1.3 [1.3.24349]
```

To get a more verbose output use, `sycl-ls --verbose`:

```bash
root@runnikri-mobl Coding → sycl-ls --verbose | grep -i name
    Name     : Intel(R) FPGA Emulation Platform for OpenCL(TM)
        Name       : Intel(R) FPGA Emulation Device
    Name     : Intel(R) OpenCL
        Name       : 12th Gen Intel(R) Core(TM) i7-12700H
    Name     : Intel(R) OpenCL HD Graphics
        Name       : Intel(R) Graphics [0x5693]
    Name     : Intel(R) OpenCL HD Graphics
        Name       : Intel(R) Graphics [0x46a6]
    Name     : Intel(R) Level-Zero
        Name       : Intel(R) Graphics [0x5693]
        Name       : Intel(R) Graphics [0x46a6]
    Name     : SYCL host platform
        Name       : SYCL host device
 ```
 
 As seen above 2 GPU devices are detected by the SYCL runtime and are supported using both OpenCL and Level-Zero drivers.
 
We now have configured the machine with all the required software stack to fully utilize the descrete gpu available on the laptop. These instructions can be used to enable any Arc discrete GPUs like the A370m, A770m, A770, A750 etc on Linux.

## Intel's System Monitoring Utility to monitor the dgpu 

1. Install sysmon

`sysmon` is a tool similar to `top` for cpu, that is part of Intel's [Platform Tools Interfaces for GPU](https://github.com/intel/pti-gpu/tree/master/tools/sysmon). `sysmon` helps in monitoring the Intel gpu parameters like frequency, memory etc. The tool can be installed using:

```bash
root@runnikri-mobl Coding → git clone https://github.com/intel/pti-gpu/
root@runnikri-mobl Coding → cd pti-gpu/tools/sysmon
root@runnikri-mobl Coding → mkdir build && cd build
root@runnikri-mobl Coding → cmake -DCMAKE_BUILD_TYPE=Release .. && make
```

2. After successfully building `sysmon` let's check the dgpu frequency:

```bash
./sysmon
```

![image](https://user-images.githubusercontent.com/45503355/195416715-923dc33a-acbd-411e-a774-7a38c13e72f3.png)

As seen above both the igpu and dgpu performance can be monitored using the tool.

## Pytorch and IPEX on dgpu

Checkout the `pytorch` [directory](https://github.com/unrahul/intel_arc_dgpu_linux/tree/main/pytorch) to know how to install PyTorch and IPEX with dGPU support.

## Acknowledgements

I wouldn't have been able to do this without the help of [@sanchitintel](https://github.com/sanchitintel) and [@gujingui](https://github.com/gujinghui). There were issues in compiling Pytorch and IPEX with gcc version 11.2.0 (Ubuntu 11.2.0-19ubuntu1) and also I talking both these folks I was able to understand the different modes of compilation for IPEX and build an AOT version for DG2 architecture (ATS-M, Arc Xe-HPG). Also, [Phoronix](https://www.phoronix.com/review/intel-arc-graphics-linux) has been publishing updates on what is the best way to enable Intel Arc dGPU, version of the kernel, mesa drivers etc, that was my start in setting up the software stack.

## Something not working?

Please create an [issue](https://github.com/unrahul/arc_dgpu_setup/issues) here to track any issues with these steps.

## Citation

If you are using this information, please cite using the below link:

Unnikrishnan Nair, R. (2022). dgpu_setup_pytorch (Version 1.0.0) [Computer software]. https://github.com/unrahul/dgpu_setup_pytorch

