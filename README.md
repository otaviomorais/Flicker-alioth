# Flicker Kernel — Poco F3 (alioth)

Custom kernel build for Xiaomi Poco F3 based on
[Flicker-Android-Devices/kernel_xiaomi_sm8250](https://github.com/Flicker-Android-Devices/kernel_xiaomi_sm8250)
(sixteen-qpr2 branch).

## Features

- **KernelSU** integrated (`drivers/kernelsu`) with root access
- **DroidSpaces** support: full cgroup, namespace, overlayfs, and networking stack
- **Uclamp + PELT** scheduler for better CPU load balancing
- **Full LTO** (Link-Time Optimization) via Clang
- **MGLRU** (Multi-Gen LRU) for improved memory management
- **NTSync** for Wine/Proton compatibility
- **BBR/TCP** congestion control
- **ZRAM** with zstd compression
- **io_uring** support

## Building

### GitHub Actions (recommended)

Push to `main` or trigger manually from the Actions tab. The workflow will:

1. Clone the Flicker kernel source
2. Enable KernelSU + DroidSpaces in the defconfig
3. Compile with Clang 20 (ZyCromerZ)
4. Package as AnyKernel3 zip
5. Create a GitHub Release

### Manual build

```bash
# Install dependencies
sudo apt install flex bison bc zip unzip libssl-dev

# Clone
git clone --depth 1 --branch sixteen-qpr2 \
  https://github.com/Flicker-Android-Devices/kernel_xiaomi_sm8250.git kernel

# Enable KernelSU
echo "CONFIG_KSU=y" >> kernel/arch/arm64/configs/alioth_defconfig

# Build
cd kernel
export ARCH=arm64
make O=out CC=clang LD=ld.lld LLVM=1 LLVM_IAS=1 HOSTCC=gcc \
  CROSS_COMPILE=aarch64-linux-gnu- alioth_defconfig
make -j$(nproc) O=out CC=clang LD=ld.lld LLVM=1 LLVM_IAS=1 HOSTCC=gcc \
  CROSS_COMPILE=aarch64-linux-gnu- Image dtbs
```

## Installation

1. Download the latest release zip
2. Flash via TWRP or any custom recovery
3. Reboot

## Credits

- [Flicker-Android-Devices](https://github.com/Flicker-Android-Devices) — kernel source
- [KernelSU](https://github.com/tiann/KernelSU) — root solution
- [AnyKernel3](https://github.com/osm0sis/AnyKernel3) — flashable zip template
