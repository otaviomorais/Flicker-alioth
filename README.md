# E404-MGLRU Kernel — Poco F3 / Mi 11X (alioth)

Custom kernel for Xiaomi POCO F3 / Mi 11X (alioth) built on the
[e404 kernel](https://github.com/kvsnr113/xiaomi_sm8250_kernel_e404)
(`staging-bpf` branch, Linux 4.19.404-R) with the **MGLRU memory management
backported from Flicker** and **memory fixes ported from MagicTime**.

## Why this kernel?

The stock e404 ships with `MEMCG=n` and `PSI=n`. Under sustained
[DroidSpaces](https://github.com/AndroidDroidSpaces) usage this degrades
lmkd pressure signals: background apps get killed indiscriminately and the
camera HAL dies. This build enables both (like MagicTime/Flicker do) and adds
MGLRU on top.

## Features

### Base (e404 staging-bpf, 4.19.404-R)
- **EEVDF + CASS** scheduler (WALT removed upstream)
- **KernelSU** (xxksu by backslashxx) integrated via submodule
- **SBalance** IRQ balancer (`IRQ_SBALANCE=y`)
- **Uclamp Assist** (`UCLAMP_ASSIST=y`)
- **CPU Input Boost** (`CPU_INPUT_BOOST=y`)
- **ZRAM zstd** default compression + writeback
- **NTSync** (Wine/Proton)
- **BPF backport 5.15** + uname spoof (Android 16/17 ROM support)
- **ThinLTO**

### Ported in this repo (see `patches/`)
| Patch | Content |
|-------|---------|
| `0001-alioth-defconfig...` | `MEMCG`, `MEMCG_SWAP`, `PSI`, `LRU_GEN`, `LRU_GEN_ENABLED`, `CPU_INPUT_BOOST`, zram zstd |
| `0002-mm-backport-MGLRU...` | Full MGLRU series from Flicker (27 commits) adapted to the e404 baseline: modern pagewalk ops API (new `p4d_entry`), XArray swap-cache shadow handling, `mmap_sem` naming, 2-arg lru list helpers, arm64 `arch_has_hw_pte_young` |

## Building

### GitHub Actions (recommended)

Push to `main` or trigger manually from the Actions tab. The workflow will:

1. Clone the e404 kernel source **with the KernelSU submodule**
2. Apply `patches/*.patch`
3. Validate critical configs (`MEMCG`, `PSI`, `LRU_GEN`, `THINLTO`, `KSU`)
4. Compile with Clang 20 (ZyCromerZ) + 8 GB swap for ThinLTO
5. Package as AnyKernel3 zip (single concatenated `dtb` → vendor_boot,
   `dtbo.img` → dtbo partition)
6. Upload artifact and create a GitHub Release

### Manual build

```bash
# Dependencies
sudo apt install flex bison bc zip unzip libssl-dev

# Clone with KernelSU submodule
git clone --depth 1 --branch staging-bpf \
  https://github.com/kvsnr113/xiaomi_sm8250_kernel_e404.git kernel
cd kernel && git submodule update --init --depth 1

# Apply patches from this repo
for p in ../patches/*.patch; do git apply "$p"; done

# Build
export ARCH=arm64
make O=out ARCH=arm64 CC=clang LD=ld.lld AR=llvm-ar NM=llvm-nm \
  OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip HOSTCC=gcc \
  vendor/alioth_defconfig
make -j$(nproc) O=out ARCH=arm64 CC=clang LD=ld.lld AR=llvm-ar NM=llvm-nm \
  OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip HOSTCC=gcc \
  Image dtbo.img dtbs
```

> Note: the defconfig lives at `arch/arm64/configs/vendor/alioth_defconfig`.

## Installation

1. Download the latest release zip
2. Flash via TWRP or any custom recovery (AnyKernel3)
3. Reboot

⚠️ MGLRU + MEMCG change reclaim behavior significantly — test on your device
before daily driving, especially if you use DroidSpaces.

## Credits

- [kvsnr113/e404](https://github.com/kvsnr113/xiaomi_sm8250_kernel_e404) — base kernel (EEVDF/CASS, KSU, BPF A16/A17)
- [Flicker-Android-Devices](https://github.com/Flicker-Android-Devices/kernel_xiaomi_sm8250) — MGLRU backport source
- [TIMISONG-dev/MagicTime](https://github.com/TIMISONG-dev/MagicTime-alioth) — MEMCG/PSI reference config + AnyKernel template
- [osm0sis/AnyKernel3](https://github.com/osm0sis/AnyKernel3) — flashable zip
- [backslashxx/KernelSU](https://github.com/backslashxx/KernelSU) — root solution
