# FlickerDS Kernel — Poco F3 / Mi 11X (alioth)

Kernel **completo e próprio** para Xiaomi POCO F3 / Mi 11X (alioth), mantido
neste repositório. Base MagicTime (4.19.325) + DroidSpaces + **EEVDF pleno**,
compilado direto desta árvore pelo GitHub Actions.

## Identidade
- Nome: `uname -r` → `4.19.325-perf-FlickerDS` *(LOCALVERSION `-FlickerDS`)*
- Banner AnyKernel: FlickerDS ASCII
- Fonte completa vendorizada em [`kernel/`](kernel/) — sem dependência de repos de terceiros

## Scheduler — EEVDF pleno
- Backport completo do EEVDF (`pick_eevdf`, delayed dequeue, slice protection)
- **`ENFORCE_ELIGIBILITY=true`** — fairness imposta: tarefas que consumiram acima da cota perdem a vez
- Placement features ON (`PLACE_LAG`, `PLACE_DEADLINE_INITIAL`, `PLACE_REL_DEADLINE`, `RUN_TO_PARITY`) — defaults do 6.6+
- **CASS** para wake-up balancing + **Uclamp Assist**
- WALT removido

## DroidSpaces / LXC
- Namespaces completos: PID, UTS, IPC, NET e **USER**
- MEMCG + PSI (lmkd com sinal fino de pressão)
- `CGROUP_DEVICE/PIDS/FREEZER` + **FAIR_GROUP_SCHED** (CPU shares por container)
- Netfilter/NAT/VETH/Bridge para isolamento de rede
- Patch cgroup NOPREFIX (symlinks `<subsys>.<file>` que o LXC espera)

## Extras
- KernelSU-Next (submódulo, `CONFIG_KSU=y`)
- ZRAM writeback; NTSYNC presente na base MagicTime

> ℹ️ MGLRU não faz parte desta base por desenho. O porte MGLRU+MEMCG/PSI viveu na
> geração anterior (base e404), disponível no histórico do repo.

## Building

### GitHub Actions (recomendado)
Push em `main` dispara o build:
1. Checkout com submódulos recursivos (KernelSU-Next)
2. Sanity-check de branding + flags EEVDF
3. `alioth_defconfig` + fragments `magictime-common` + `droidspaces` via `merge_config.sh`
   (com relatório de símbolos não-aplicados)
4. Validação de configs críticas (KSU, MEMCG, PSI, USER_NS, FAIR_GROUP_SCHED, VETH…)
5. Clang 20 (ZyCromerZ) + GCC 4.9, swap 8 GB
6. Empacota zip AnyKernel3 (`Image` + `dtb` concatenado p/ vendor_boot + `dtbo.img`)
7. Artifact + GitHub Release

### Manual
```bash
git clone --recurse-submodules https://github.com/otaviomorais/Flicker-alioth.git
cd Flicker-alioth/kernel

export ARCH=arm64
make O=out ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
  CC=clang LD=ld.lld AR=llvm-ar NM=llvm-nm \
  OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip HOSTCC=gcc \
  alioth_defconfig
./scripts/kconfig/merge_config.sh -m -O out out/.config \
  arch/arm64/configs/vendor/xiaomi/magictime-common.config \
  arch/arm64/configs/vendor/xiaomi/droidspaces.config
make O=out ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=clang LD=ld.lld \
  AR=llvm-ar NM=llvm-nm OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip \
  HOSTCC=gcc olddefconfig

make -j$(nproc) O=out ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
  CC=clang LD=ld.lld AR=llvm-ar NM=llvm-nm \
  OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip HOSTCC=gcc \
  Image dtbo.img dtbs
```

Defconfig: `arch/arm64/configs/alioth_defconfig`
(+ fragments em `arch/arm64/configs/vendor/xiaomi/`)

## Installation
1. Baixe o zip da última release
2. Flash via TWRP/custom recovery (AnyKernel3)
3. Reboot

## Estrutura
```
├── kernel/            # fonte completa do kernel (+ KernelSU-Next submodule)
├── anykernel/         # template AnyKernel3 (banner FlickerDS)
└── .github/workflows/ # build + release automáticos
```

## Credits
- [TIMISONG-dev/MagicTime](https://github.com/TIMISONG-dev/kernel_xiaomi_sm8250) — base EEVDF+CASS
- [kvsnr113/e404](https://github.com/kvsnr113/xiaomi_sm8250_kernel_e404) — referências EEVDF/DroidSpaces
- [ravindu644](https://github.com/ravindu644) — fragmento DroidSpaces original
- [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next) — root solution
- [osm0sis/AnyKernel3](https://github.com/osm0sis/AnyKernel3) — flashable zip
