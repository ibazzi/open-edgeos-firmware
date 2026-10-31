# Firmware installation guide

English | [简体中文](INSTALL.zh-CN.md) | [Project README](README.md)

Last updated: **2026-10-31**. This guide covers native EdgeOS `.tar` system-image installation on **ER-4 only**, subject to the requirements of the exact release. It is not an OpenWrt sysupgrade or raw-disk flashing guide.

> Read the [disclaimer](README.md#disclaimer) before proceeding. The firmware is experimental. Installation may interrupt service, lose configuration, or leave the device unable to boot. A working backup and recovery plan are essential.

## 1. Confirm that the release is suitable

- Download only a firmware asset explicitly published in this repository's **GitHub Releases**. GitHub's automatically generated “Source code” archives are not firmware.
- Confirm that the release lists **ER-4**, your starting firmware version, and the installation method you intend to use. An `e300` filename alone is insufficient.
- Read the known issues, configuration compatibility, first-boot access details, and recovery prerequisites. If the release does not define an installation path from your current system, ask before proceeding.
- Choose the documented debug or release image. The filename's `release` mode does not mean the firmware has passed stability acceptance.

Development records include a first native debug installation from an official EdgeOS 3.0.1 baseline, followed by component fixes. They do not establish that every published image works from every EdgeOS version. Upgrades from OpenWrt or a changed storage layout are outside this guide. Project-to-project upgrades must also be explicitly covered by the target release.

## 2. Prepare equipment, access, and backups

| Item | Preparation |
| --- | --- |
| Maintenance window | Allow for network downtime and hands-on recovery; arrange alternative connectivity |
| Power | Use a reliable power supply; do not interrupt power during installation or first boot |
| Management computer | Use a wired connection and an interface known to work with the current configuration; avoid depending on a VPN through the router being upgraded |
| Administrator access | Verify current Web/SSH credentials and record the management IP, port mapping, VLANs, and addressing |
| Console | For experimental installation, prepare a compatible console cable and verify readable output before starting; ER-4 uses 115200 baud, 8N1, no flow control |
| Recovery resources | Keep the official recovery instructions, correct recovery image, and necessary tools available offline |
| Backups | Export the configuration and separately preserve required certificates, keys, scripts, and other custom files; keep them securely off the router and check readability |

Use the current system's configuration backup function. A configuration export is not necessarily a full-device backup and may omit custom files. Do not post backups or secrets in Issues. Existing configuration may be retained, but retention does not guarantee successful migration; keep the original backup even after a successful boot.

Before uploading, record these commands from the router's **EdgeOS operational CLI** (outside `configure` mode):

```text
show version
show system image
show system storage
```

Confirm the model, current version, image state, and available storage. The release must define its space requirements; the archive size alone does not account for upload, extraction, and installed files. For CLI uploads to `/tmp`, also check its available space with `df -h /tmp` in the router shell. If space is insufficient, stop and assess it. Do not blindly delete the previous image or other system files to make room.

This guide does **not** update the bootloader. Do not run `add system boot-image`, `ubnt-upgrade-boot`, `flash_erase`, raw partition writes, or repartitioning commands as part of these steps. If a release or recovery method requires a bootloader change, stop: that prerequisite is outside this guide.

## 3. Verify the firmware download

In the examples, `firmware.tar` means the actual firmware asset you downloaded. Substitute its filename; keep the original `.tar` archive intact. Do not extract or repack it before uploading.

On your **computer**, calculate SHA-256 using the command for your platform:

```sh
# Linux
sha256sum firmware.tar

# macOS
shasum -a 256 firmware.tar
```

```powershell
# Windows PowerShell
Get-FileHash .\firmware.tar -Algorithm SHA256
```

Compare the complete result with the value published for that exact asset. A missing or mismatched checksum is a reason to stop. A checksum detects file changes; obtain both the file and its checksum from the intended release. Internal MD5 files used by the old upgrader do not replace this check or authenticate the publisher.

## 4. Install using one supported method

Use **either** Web or CLI for one installation attempt. Do not start both. The release notes must confirm support for the selected method; official EdgeOS interface documentation alone does not establish acceptance of a custom image.

### Method A: Web interface

1. Sign in at the router's **current** management address using your existing administrator account.
2. Open **System → Upgrade System Image → Upload a file** and select the verified firmware `.tar`.
3. Wait for an explicit installation result. Upload completion alone is not installation success.
4. If installation succeeds, follow the reboot prompt and the release's first-boot instructions. If an error appears, save it and follow the troubleshooting section before retrying or rebooting.

The native Web menu is described in [Ubiquiti's firmware upgrade guide](https://help.uisp.com/hc/en-us/articles/22591203404311-EdgeRouter-How-to-Upgrade-the-EdgeOS-Firmware). Labels can vary by version.

### Method B: Local file through the CLI

1. From your **computer**, copy the verified archive to the router. Replace `USER` and `ROUTER_IP` with your administrator username and current management IP:

   ```sh
   scp ./firmware.tar USER@ROUTER_IP:/tmp/firmware.tar
   ```

2. In the **router shell**, verify the uploaded file against the release's SHA-256 value:

   ```sh
   sha256sum /tmp/firmware.tar
   ```

3. In the **EdgeOS operational CLI**, install the explicit local file:

   ```text
   add system image /tmp/firmware.tar
   ```

   Read all output and prompts. Do not omit the filename: bare `add system image` can invoke a different update source. Do not bypass model or integrity checks.

4. Only after the installer reports success, inspect the installed images:

   ```text
   show system image
   ```

   Verify that the intended version is listed for the next boot. If the state is unexpected or ambiguous, stop and retain the installation output. Do not repeatedly switch images to try to repair it.

5. When installation and next-boot state are confirmed, reboot normally:

   ```text
   reboot
   ```

Run each step separately. Do not chain installation and reboot into one command. The SSH/Web connection will disconnect during reboot; keep the console log running and do not interrupt power.

## 5. Check the first boot

Use the release's documented address and credentials. Do not assume a reset to `192.168.1.1` or a default password; the device may retain its previous configuration and accounts.

- Confirm the intended firmware with `show version` and inspect `show system image`.
- Verify CLI/SSH and Web login, physical-port mapping, management reachability, and configuration state.
- Test the functions you need: addressing, routes, DNS/DHCP, firewall/NAT, VPN, and actual client traffic as applicable. An accessible Web page alone is insufficient.
- Check startup logs for storage, driver, service, or configuration errors. Record any unsupported or failed configuration items before saving changes.
- After confirming configuration compatibility, save intended changes through the normal EdgeOS configuration workflow. Keep the original off-device backup.

Restore business connections gradually. Do not assume untested SFP, acceleration, or advanced protocol behavior works because basic connectivity succeeds.

## 6. Installation failures, rollback, and recovery

| Symptom | Next action |
| --- | --- |
| Upload or installation error | Save the exact message, version, image list, and storage information; do not immediately retry or reboot |
| Model, compatibility, or checksum rejection | Stop and recheck the asset and release requirements; do not modify the archive or bypass checks |
| No management access after reboot | Check console output, previous addressing, link/VLAN settings, and the release's first-boot instructions before considering a reset |
| Boot failure or storage errors | Preserve console logs and use a recovery method confirmed for your model and existing bootloader |
| Need to return to official firmware | Follow a release-specific tested rollback path or the applicable official recovery procedure; keep configuration restoration separate |

The native upgrader can remove an old image during installation, including before some validation steps. The presence of an old version name does not guarantee a complete, bootable backup. `set system image default-boot` changes image files; it is not merely a harmless menu selection. This guide does not prescribe it as a universal rollback command.

Official recovery references:

- [Ubiquiti: TFTP recovery](https://help.uisp.com/hc/en-us/articles/22591244564887-EdgeRouter-TFTP-Recovery) — reset-button recovery depends on the existing bootloader and uses a recovery image, not this project's upgrade `.tar`.
- [Ubiquiti: manual TFTP recovery](https://help.uisp.com/hc/en-us/articles/22591244620311-EdgeRouter-Manual-TFTP-Recovery) — console-based recovery; follow the ER-4/e300 section and the correct recovery file type.

Official recovery erases custom configuration. It is not a lossless rollback, and recovery is not guaranteed for every failure. These links describe vendor procedures; independent recovery after this project's firmware has not completed full acceptance. Do not update the bootloader just to satisfy a recovery prerequisite under this guide.

When reporting a failure, include the exact release, SHA-256, starting firmware, chosen installation method, complete error output, and redacted console logs. Follow the [bug report guidance](README.md#bug-reports-and-discussion).
