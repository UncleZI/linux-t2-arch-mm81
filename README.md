# linux-t2-arch-mm81

Minimalist Arch Linux kernel package tailored specifically for **Mac mini 2018 (`macmini8,1`)** with Apple T2 Security Chip support.

## Features

- Based on the official Arch Linux kernel (`linux`) with patches from [t2linux/linux-t2-patches](https://github.com/t2linux/linux-t2-patches).
- Trimmed kernel configuration (`localmodconfig`) built exclusively for `macmini8,1` hardware — laptop-specific drivers (Touch Bar, batteries, trackpads, discrete mobile GPUs) and `-docs` package are stripped out for significantly faster compile times.
- Includes essential T2 and Mac mini modules (`T2BCE_*`, `MACSMC_*`, `BT_HCIBCM4377`, `BRCMFMAC`, `TIGON3`, `AQTION`).

## Installation

### Option 1: Prebuilt Packages (Recommended)

Install the latest packages directly from the [Releases](https://github.com/UncleZI/linux-t2-arch-mm81/releases) page:

```sh
sudo pacman -U linux-t2-mm81-*.pkg.tar.zst linux-t2-mm81-headers-*.pkg.tar.zst
```

### Option 2: Build from Source

```sh
git clone https://github.com/UncleZI/linux-t2-arch-mm81
cd linux-t2-arch-mm81
makepkg -si
```

> **Tip:** To speed up compilation across all CPU cores, ensure `MAKEFLAGS="-j$(nproc)"` is enabled in `/etc/makepkg.conf`.

## Updating

To sync `PKGBUILD` with the latest Arch Linux kernel version and `t2linux` patches:

```sh
./update.sh
makepkg -si
```

## Notes

- **Hardware scope:** Because the config is trimmed for `macmini8,1`, external GPUs (eGPU) or uncommon PCIe/USB peripherals may require re-enabling their respective modules via `make menuconfig` before building.
- **Bootloader:** Make sure your bootloader passes the required T2 kernel parameters (`intel_iommu=on iommu=pt pcie_ports=compat`). See the [t2linux Wiki](https://wiki.t2linux.org) for full setup instructions.

## Credits

- [t2linux](https://github.com/t2linux) for the T2 kernel patches and documentation.
- [NoaHimesaka1873/linux-t2-arch](https://github.com/NoaHimesaka1873/linux-t2-arch) for the original Arch Linux `PKGBUILD` structure.
