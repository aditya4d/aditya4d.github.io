---
layout: default
---

# Setup NVIDIA M40

Setting up M40 to run CUDA code is not trivial. NVIDIA has deprecated software support (NVIDIA Drivers, CUDA toolkit) for it a while ago. It took me a while for me to figure out which OS and Driver to use.

## OS

I have picked Fedora as a personal choice. At the time of writing, Fedora 43 is the latest version. But, the issue is NVIDIA deprecated or last NVIDIA driver that supported Maxwell, Pascal and Volta is r575. As NVIDIA driver builds kernel modules and change initramfs, it requires specific headers. Unfortunately, these headers are not present in Fedora 43. I got the following error:

```sh
make[4]: *** [/usr/src/kernels/6.17.10-100.fc41.x86_64/scripts/Makefile.build:287: nvidia-drm/nv-kthread-q.o] Error 1
make[4]: *** [/usr/src/kernels/6.17.10-100.fc41.x86_64/scripts/Makefile.build:287: nvidia-drm/nvidia-drm-gem-nvkms-memory.o] Error 1
  CC [M]  nvidia-drm/nvidia-drm-os-interface.o
In file included from nvidia-drm/nvidia-drm-gem-user-memory.c:23:
nvidia-drm/nvidia-drm-conftest.h:26:10: fatal error: conftest.h: No such file or directory
   26 | #include "conftest.h"
      |          ^~~~~~~~~~~~
compilation terminated.
```

The `conftest.h` is not present in Fedora 43. So, to make life easier, I tried Fedora 42, which didn't work (some other issue). So, I finally tried Fedora 41.

I installed [Fedora 41](https://dl.fedoraproject.org/pub/fedora/linux/releases/41/Workstation/x86_64/iso/)

After installation, run `lspci | grep NVIDIA` to make sure the **M40** is visible to the OS. If not, most likely, you don't have enough PCIe lanes connecting the card or some BIOS issue.

To be specific, I always load into linux kernel 6.11 rather than the latest 6.17 as the conftest.h is removed from 6.17 and 6.11 works well with the NVIDIA driver

## NVIDIA Drivers

As NVIDIA deprecated Maxwell support after r575, we get a cuda toolkit that works well with it which is CUDA 12.9

[CUDA toolkit 12.9](https://developer.nvidia.com/cuda-12-9-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Fedora&target_version=41&target_type=runfile_local).

I typically extract the driver first from the `cuda_12.9xxx.run` file and run it (it saves extraction time).

The toolkit requires more kernel headers and tools

```sh
sudo dnf install kernel-devel-$(uname -r) kernel-headers
```

Once the installation is done, before installing CUDA toolkit, I run `nvidia-smi` to check if the **M40** is accessible by the driver.

There is a very high chance (at least for me) that you can't see **M40** in `nvidia-smi`. This is most likely that **Above 4G Decoding** could be disabled.

Even after enabling >4G decoding, I couldn't see the **M40**.

Looking at the `dmesg` log, I found:

```
[    5.637363] NVRM: This PCI I/O region assigned to your NVIDIA device is invalid:
[    5.638722] nvidia 0000:67:00.0: probe with driver nvidia failed with error -1
[    5.638865] nvidia 0000:68:00.0: vgaarb: VGA decodes changed: olddecodes=io+mem,decodes=none:owns=io+mem
[    5.858636] NVRM: The NVIDIA probe routine failed for 1 device(s).
[    5.858655] NVRM: loading NVIDIA UNIX x86_64 Kernel Module  575.51.03  Wed Apr 16 14:22:51 UTC 2025
[    5.900031] nvidia-modeset: Loading NVIDIA Kernel Mode Setting Driver for UNIX platforms  575.51.03  Wed Apr 16 13:46:27 UTC 2025
[    5.909232] [drm] [nvidia-drm] [GPU ID 0x00006800] Loading driver
[    5.909236] [drm] Initialized nvidia-drm 0.0.0 for 0000:68:00.0 on minor 0
[   62.466316] nvidia_uvm: module uses symbols nvUvmInterfaceDisableAccessCntr from proprietary module nvidia, inheriting taint
```

This shows that NVIDIA driver couldn't use the GPU properly.

The fix I found for it is, edit the `grub` file and add `pci=realloc` to `GRUB_CMDLINE_LINUX`

```
sudo vim /etc/default/grub
# Find GRUB_CMDLINE_LINUX
# Add pci=realloc to the end of it
```

After rebooting, I was able to see the **M40** in `nvidia-smi`

```sh
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 575.51.03              Driver Version: 575.51.03      CUDA Version: 12.9     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla M40                      Off |   00000000:67:00.0 Off |                    0 |
| N/A   40C    P0             69W /  250W |       0MiB /  11520MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
```

## CUDA

Once `nvidia-smi` is running, we can install the CUDA toolkit. Once installed, get [cuda-samples](https://github.com/NVIDIA/cuda-samples), build them and run `deviceQuery` to check if the CUDA driver, NVIDIA driver, OS and hardware are working properly.

```
Device 0: "Tesla M40"
  CUDA Driver Version / Runtime Version          12.9 / 12.9
  CUDA Capability Major/Minor version number:    5.2
  Total amount of global memory:                 11436 MBytes (11991515136 bytes)
  (024) Multiprocessors, (128) CUDA Cores/MP:    3072 CUDA Cores
  GPU Max Clock rate:                            1112 MHz (1.11 GHz)
  Memory Clock rate:                             3004 Mhz
  Memory Bus Width:                              384-bit
  L2 Cache Size:                                 3145728 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total shared memory per multiprocessor:        98304 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     No
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Enabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Managed Memory:                Yes
  Device supports Compute Preemption:            No
  Supports Cooperative Kernel Launch:            No
  Supports MultiDevice Co-op Kernel Launch:      No
  Device PCI Domain ID / Bus ID / location ID:   0 / 103 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >
```

## CUTLASS

To test the GPU properly, I get [cutlass v2.11.0](https://github.com/NVIDIA/cutlass/tree/v2.11.0) 

I build it using

```sh
cd cutlass
mkdir build
cd build
cmake .. -G Ninja -DCUTLASS_NVCC_ARCHS=52
ninja
```

Once built, run `./profiler/cutlass_profiler` to run different GEMM operations on the GPU. (Make sure the binary is running on the correct GPU by checking nvidia-smi for process running on specific GPU or use `--device=<cuda device id>` after the profiler. 

To see what GPUs are visible to `cutlass_profiler`, run `./profiler/cutlass_profiler --help`

