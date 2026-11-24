# Open EdgeOS v0.2.0-dev

This experimental development release covers the Ubiquiti EdgeRouter 4 (ER-4), x86_64 systems, and ARM64 UEFI virtual machines. It uses Linux 6.18.60, Debian Buster user space, and `release` packaging mode.

## Changes since v0.1.0-dev.20261112

The previous public release covered the ER-4. [v0.1.0-dev.20261112 release notes](https://github.com/ibazzi/open-edgeos-firmware/releases/tag/v0.1.0-dev.20261112)

### Major shared component updates

| Component | v0.1.0-dev.20261112 | v0.2.0-dev |
| --- | --- | --- |
| Linux kernel | 6.12.112 | 6.18.60, with updated OpenWrt kernel inputs and module integration. |
| ZebOS routing backend | Vendor routing binaries | Routing daemons rebuilt from maintained source, with the EdgeOS configuration and operational CLI retained. |
| NHRP | — | Adds `nhrpd` 0.15 for IPv4 multipoint GRE and native route redistribution, with managed HA through `nhrpd-ha`. |
| TDTS traffic analysis | DPI support | Builds TDTS and `tm_crypto` from source, bundles signature rules, and adds QUIC v1/v2 Initial traffic classification. |
| IPv4 software flow offload | — | Adds nftables 0.9.0 and optional, policy-aware software flow offload for eligible connections, with TDTS classification and accounting integration. |

### Platform-specific changes

- **ER-4:** Restore QCA8511 switch behavior on Octeon and the kernel options used by ER-4 tunnel and QoS features.
- **x86_64:** First public firmware images, including GPT disk images for 512-byte and 4Kn logical sectors and a QCOW2 image for virtual machines. The 512-byte image supports Legacy BIOS and 64-bit UEFI.
- **ARM64 UEFI:** First public firmware images for 64-bit UEFI virtual machines with VirtIO storage and networking, including the OCI Ampere A1 installation path.
- **x86_64 and ARM64 UEFI:** GRUB A/B upgrades install to the alternate slot, with a separate writable system layer per slot and shared `/config`.

## Build information

All three platforms use build ID `221664003` from the `non-ubnt` branch, based on source revision `d365303c8686048be9170dbe8a70374b37f444d8`. The release inputs set the firmware version to 0.2.0-dev and preserve the factory dnsmasq helper ownership on the ER-4.

## Firmware assets

| Target | First installation | Upgrade archive | Build manifest |
| --- | --- | --- | --- |
| **EdgeRouter 4 (ER-4)** | Use the native upgrade archive from the official EdgeOS environment | `ER-e300.v0.2.0-dev.221664003.261124.1237.tar` | `manifest-er4.json` |
| **x86_64** | `x86_64.v0.2.0-dev.221664003.261124.1237-release.qcow2` for VMs, or `-release.img.gz` for 512-byte logical sectors and `-release-4kn.img.gz` for 4Kn disks | `x86_64.v0.2.0-dev.221664003.261124.1237-release.tar` for an existing x86_64 A/B installation | `manifest-x86_64.json` |
| **ARM64 UEFI** | `arm64_uefi.v0.2.0-dev.221664003.261124.1237-release.qcow2` for VirtIO VMs, or `-release.img.gz` for a 512-byte logical-sector boot disk | `arm64_uefi.v0.2.0-dev.221664003.261124.1237-release.tar` for an existing ARM64 UEFI A/B installation | `manifest-arm64_uefi.json` |

Verify the selected asset against `SHA256SUMS`. Use only an asset that the release lists for your exact platform.

## Installation

- On **ER-4**, update to the latest official EdgeOS firmware and official bootloader for this model before first installation. Complete the required reboots and verify both updates in official EdgeOS. Back up the configuration, then use the native system-image workflow with the ER-4 `.tar`. [ER-4 installation guide](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/INSTALL.md#1-confirm-that-the-release-is-suitable)
- On **x86_64**, import the QCOW2 into a VM or select the physical disk image by logical sector size. Disable Secure Boot for UEFI. Use the matching upgrade archive for existing A/B installations. [x86_64 installation guide](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/INSTALL.md#x86_64) · [VMware VMDK conversion](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/INSTALL.md#create-a-vmware-vmdk)
- On **ARM64 UEFI**, use 64-bit UEFI with Secure Boot disabled and VirtIO storage and networking. Earlier single-slot installations require a configuration backup, a fresh disk image, and configuration restoration. [ARM64 UEFI installation guide](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/INSTALL.md#arm64-uefi) · [OCI Ampere A1 installation](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/INSTALL.md#oracle-cloud-infrastructure-oci-ampere-a1)

[简体中文安装说明](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/INSTALL.zh-CN.md)

## Compatibility and validation

Release verification covered the native and A/B upgrade archives, embedded firmware versions and payload checksums, and the GPT sector layouts and QCOW2 image integrity.

Use the target manifest to identify the full firmware version and kernel release. The project README summarizes development testing on specific images and component builds. Hardware support is limited to the ER-4, x86_64 systems with the boot modes listed in the installation guide, and the ARM64 UEFI VirtIO configuration described above. [Development test summary](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/README.md#current-progress)

Open EdgeOS is an independent project. See the project README for reporting issues, source availability, licensing, and the disclaimer. [Project README](https://github.com/ibazzi/open-edgeos-firmware/blob/v0.2.0-dev/README.md)
