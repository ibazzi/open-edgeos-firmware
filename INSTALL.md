# Firmware installation guide

English | [简体中文](INSTALL.zh-CN.md) | [Project README](README.md)

Choose an image explicitly listed for your platform in the release notes. Sections 1–6 cover the ER-4 native EdgeOS `.tar` system-image installation. See [x86_64](#x86_64) and [ARM64 UEFI](#arm64-uefi) for fresh installs and A/B upgrades on those platforms.

> Read the [disclaimer](README.md#disclaimer) before proceeding. The firmware is experimental. Installation may interrupt service, lose configuration, or leave the device unable to boot. A working backup and recovery plan are essential.

## 1. Confirm that the release is suitable

- Download only a firmware asset explicitly published in this repository's **GitHub Releases**. GitHub's automatically generated “Source code” archives are not firmware.
- Confirm that the release lists **ER-4**, your starting firmware version, and the installation method you intend to use. An `e300` filename alone is insufficient.
- Read the known issues, configuration compatibility, first-boot access details, and recovery prerequisites. If the release does not define an installation path from your current system, ask before proceeding.
- Choose the documented debug or release image. The filename's `release` mode does not mean the firmware has passed stability acceptance.

Development records include a first native debug installation from an official EdgeOS 3.0.1 baseline, followed by component fixes. They do not establish that every published image works from every EdgeOS version. Upgrades from OpenWrt or a changed storage layout are outside this guide. Project-to-project upgrades must also be explicitly covered by the target release.

### Bootloader prerequisite

Before first installing Open EdgeOS on ER-4, update to the latest official EdgeOS firmware and the latest official bootloader for the exact model. Complete both updates in the official EdgeOS environment, where firmware and bootloader updates are separate operations. [Official firmware update](https://help.uisp.com/hc/en-us/articles/22591203404311-EdgeRouter-How-to-Upgrade-the-EdgeOS-Firmware) · [Official bootloader update](https://help.uisp.com/hc/en-us/articles/22591243829911-EdgeRouter-How-to-Update-the-Bootloader)

In the official EdgeOS operational CLI, inspect the bootloader with:

```text
show system boot-image
```

This compares the installed bootloader with the one bundled in the current official firmware; it is not an online check for the latest version. Use the applicable official firmware and guidance, rather than a fixed version number copied from an old example.

Complete any required update, reboot, and verification in the official environment before installing this project's firmware. Bootloader writes must not be interrupted. If the version or update result is unclear, stop and resolve it first. Do not attempt the update from this experimental firmware.

### Boot and storage safeguards

The current ER-4 development firmware includes the following measures to reduce the risk of an unbootable device or storage damage:

| Measure | Protection and limits |
| --- | --- |
| Skip automatic bootloader writes | The platform startup script skips its automatic bootloader update and bootloader-corruption recovery branches. This avoids those startup-triggered writes; it does not disable normal system boot or the bootloader's own recovery entry |
| Read-only bootloader partitions | The ER-4 runtime device-tree fixup marks `boot0` and `boot1` read-only through Linux MTD |
| Preserve the native storage layout | The kernel mounts the SquashFS system image with a writable overlay and uses the existing configuration directory |
| Standard shutdown and reboot | systemd manages the normal shutdown and reboot sequence |
| Build-time image checks | Packaging checks archive members, file types, kernel/rootfs formats, and checksums. These checks help catch malformed artifacts, but do not guarantee successful installation, authenticity, or complete hardware compatibility |

The bootloader write protection applies during development and testing. Complete the official bootloader update before installing this firmware.

Check the release notes for the boot and storage behavior verified on the exact artifact.

These measures do not provide automatic rollback, a guaranteed intact backup image, or a guarantee against bricking. The native upgrader can still modify stored images, and a failed kernel, damaged storage, or interrupted installation may require independent recovery. Keep off-device backups and console/recovery access ready.

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

Complete the bootloader prerequisite above as a separate official maintenance task. The system-image installation steps below do **not** update the bootloader: do not run `add system boot-image`, `ubnt-upgrade-boot`, `flash_erase`, raw partition writes, or repartitioning commands during those steps. This project's firmware is not a bootloader update package.

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

Official recovery erases custom configuration. It is not a lossless rollback, and recovery is not guaranteed for every failure. These links describe vendor procedures; independent recovery after this project's firmware has not completed full acceptance. Resolve bootloader prerequisites in the official environment before installation, rather than attempting a bootloader update as an improvised recovery step.

When reporting a failure, include the exact release, SHA-256, starting firmware, chosen installation method, complete error output, and redacted console logs. Follow the [bug report guidance](README.md#bug-reports-and-discussion).

## x86_64

Verify the downloaded file against the release's `SHA256SUMS`. For a fresh VM installation, import the x86_64 `.qcow2` as its boot disk. For a physical installation, choose the 4 GiB GPT disk image by the destination disk's **logical** sector size:

| Logical sector size | Image suffix | Boot mode |
| --- | --- | --- |
| 512 bytes, including 512e | `-release.img.gz` | Legacy BIOS or 64-bit UEFI |
| 4096 bytes (4Kn) | `-release-4kn.img.gz` | 64-bit UEFI |

Disable Secure Boot for UEFI. Back up the destination disk, decompress the chosen image, and write it to the entire target disk. This replaces its partition table and data. On first boot, use the screen or a compatible serial console (`ttyS0`, 115200 8N1). With factory settings, the first detected Ethernet port uses `192.168.1.1/24`.

To upgrade an existing x86_64 Open EdgeOS A/B installation, use the matching x86_64 `-release.tar` as described under [A/B upgrades](#ab-upgrades).

### Create a VMware VMDK

Verify the release's x86_64 QCOW2 against `SHA256SUMS`. In the commands below, replace `x86_64-release.qcow2` with the downloaded filename. For VMware Workstation or Fusion on an Intel Mac, create a VMDK on a computer with `qemu-img`. [QEMU image tools](https://www.qemu.org/docs/master/tools/qemu-img.html)

```sh
qemu-img convert -p -f qcow2 -O vmdk \
  -o subformat=monolithicSparse,adapter_type=lsilogic \
  x86_64-release.qcow2 edgeos-x86_64.vmdk
qemu-img compare -f qcow2 -F vmdk x86_64-release.qcow2 edgeos-x86_64.vmdk
```

Create a 64-bit Linux VM and attach the VMDK as its existing boot disk. If you select UEFI, disable Secure Boot.

For ESXi, create a compressed `streamOptimized` VMDK for transfer to the host.

```sh
qemu-img convert -p -c -f qcow2 -O vmdk \
  -o subformat=streamOptimized,adapter_type=lsilogic \
  x86_64-release.qcow2 edgeos-x86_64-esxi-import.vmdk
```

Upload that file to the VM's datastore directory. From that directory **on the ESXi host**, import it into a native VMFS disk. [VMware disk conversion with vmkfstools](https://knowledge.broadcom.com/external/article/343140/)

```sh
vmkfstools -i edgeos-x86_64-esxi-import.vmdk edgeos-x86_64-esxi.vmdk -d thin
```

The import creates the descriptor `edgeos-x86_64-esxi.vmdk` and the data file `edgeos-x86_64-esxi-flat.vmdk`. Keep them together. In the VM settings, add the **descriptor** as an existing hard disk. Use EFI with Secure Boot disabled. If using SCSI, select LSI Logic SAS. Directly attaching the transfer image can fail with an unsupported disk type 22 error. [VMware VMDK descriptor and flat file](https://knowledge.broadcom.com/external/article/321422)

## ARM64 UEFI

Use an ARM64 VM with 64-bit UEFI, Secure Boot disabled, and VirtIO storage and networking. Verify the downloaded file against `SHA256SUMS`. Import the ARM64 UEFI `.qcow2` as the VM boot disk, or decompress the `.img.gz` and write it to an empty 512-byte logical-sector boot disk. For Oracle Cloud, follow [the OCI Ampere A1 instructions](#oracle-cloud-infrastructure-oci-ampere-a1).

With factory settings, `eth0` obtains its address through DHCP. Open the Web UI over HTTPS and sign in with the factory account. The `ttyAMA0` serial console is available for troubleshooting or initial CLI access.

Use the ARM64 UEFI `-release.tar` to upgrade an existing ARM64 UEFI Open EdgeOS A/B installation. To migrate an earlier single-slot ARM64 installation, back up the configuration, install the new disk image, then restore the configuration.

### Oracle Cloud Infrastructure (OCI Ampere A1)

This procedure creates a new `VM.Standard.A1.Flex` instance from an Open EdgeOS custom image. If you are migrating from an existing OCI instance, keep its instance and boot volume until the new instance is working.

#### 1. Verify and upload the image

1. Download the `arm64_uefi.v0.2.0-dev.*-release.qcow2` asset and `SHA256SUMS` from the same release. Compare the QCOW2 file's SHA-256 against its entry in the checksum file. Use the [download verification commands](#3-verify-the-firmware-download) with the QCOW2 filename.
2. In the OCI Console, select the region where you will create the instance. Open **Storage → Object Storage → Buckets**. Create or select a private bucket and upload the intact QCOW2 file. Use the same region and a compartment you can access for the import. [Oracle custom Linux image import](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/importingcustomimagelinux.htm)

#### 2. Import the custom image and set its capabilities

1. Go to **Compute → Custom Images → Import image**. Select the compartment, give the image a name, and use the following settings.

   | Field | Value |
   | --- | --- |
   | Operating system | Linux |
   | Import source | Object Storage bucket, then the uploaded QCOW2 object |
   | Image type | QCOW2 |
   | Launch mode | Paravirtualized mode |

2. Wait for the custom image to show **Available**. Open it and use **Edit details** to add `VM.Standard.A1.Flex` to the compatible shapes if needed. If the editor does not list A1, try the OCI CLI in Cloud Shell with the custom image's OCID. [Edit compatible shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/custom-images-edit.htm) · [OCI CLI reference](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/compute/image-shape-compatibility-entry/add.html)

   ```sh
   oci compute image-shape-compatibility-entry add \
     --image-id 'IMAGE_OCID' --shape-name VM.Standard.A1.Flex
   ```

3. Under **Edit image capabilities**, check that the firmware and preferred firmware are **UEFI 64-bit** (`UEFI_64`), Secure Boot is disabled, and launch mode, boot volume type, and network attachment type are **Paravirtualized**. Set the corresponding preferred values to the same options where shown. [OCI image capabilities](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/configuringimagecapabilities-tasks.htm)

#### 3. Create the instance and configure management access

1. Go to **Compute → Instances → Create instance**. Under **Change image → My images → Custom images**, select the image you imported. Under **Change shape → Virtual machine → Ampere**, select `VM.Standard.A1.Flex`. Choose OCPUs and memory within your tenancy limits. Keep the default boot volume size offered by OCI and verify that the launch options still use paravirtualized storage and networking. [Create an OCI instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
2. Select a VCN and subnet with IPv4 available for the primary VNIC. For private management, use a subnet reachable from your administration network and connect to the VNIC's private IP. For direct internet access, assign a public IPv4 address in a public subnet with an Internet Gateway and a `0.0.0.0/0` route to it. In the applicable NSG or security list, allow inbound TCP 443 for Web management only from your administration CIDR. If you use SSH, restrict TCP 22 to that same range. Check all applicable rules for broader access. [Oracle public subnet connectivity](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/quick-action-internet-gateway.htm)
3. Create the instance and wait for **Running**. In the instance details, find the primary VNIC's private or public IPv4 address.

#### 4. First sign-in through the Web UI

1. At first boot, `eth0` obtains the primary VNIC's private IPv4 address via DHCP. From the administration network, open `https://INSTANCE_IP/` using the reachable private or public IPv4 address.
2. Sign in with username `ubnt` and password `ubnt`. Change the factory password in **Users** after signing in. [EdgeOS user accounts](https://help.uisp.com/hc/en-us/articles/22591215486103-EdgeRouter-User-Accounts)
3. If the Web UI is unreachable, check the instance's address and inbound TCP 443 in OCI. Use the serial console to check the guest when network access is unavailable.

#### 5. Serial console and optional SSH access

In the instance details, open **Console connection** under **Resources** or the **OS Management** tab, then choose **Launch Cloud Shell connection**. If Cloud Shell is unavailable, create a local serial console connection and use the SSH command provided by OCI. At the `ttyAMA0` guest login prompt, use the EdgeOS account with its current password. [OCI serial console](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm)

SSH password authentication is disabled by default. If you need SSH access, install a public key through EdgeOS. OCI passes keys from the instance creation form as metadata.

1. On your **administration computer**, use an existing Ed25519 key or create one.

   ```sh
   ssh-keygen -t ed25519 -f ~/.ssh/edgeos_oci
   awk '{print $2}' ~/.ssh/edgeos_oci.pub
   ```

2. In the **EdgeOS serial console**, replace `BASE64_PUBLIC_KEY` with the second field from the `.pub` file.

   ```text
   configure
   set system login user ubnt authentication public-keys oci-admin type ssh-ed25519
   set system login user ubnt authentication public-keys oci-admin key 'BASE64_PUBLIC_KEY'
   commit
   save
   exit
   show interfaces
   show ip route
   ```

   Confirm that `eth0` received the primary VNIC's private IPv4 address via DHCP and that a default route is present. For internet connections, use the public IPv4 shown by OCI, which maps to the VNIC's private IP.

3. Keep the serial session open and connect from the **administration computer**. Replace `INSTANCE_IP` with the reachable VNIC address. On first connection, verify the presented host-key fingerprint via the serial console. For an Ed25519 host key, use `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` on the guest.

   ```sh
   ssh -o IdentitiesOnly=yes -i ~/.ssh/edgeos_oci ubnt@INSTANCE_IP
   ```

#### 6. Check boot, persistence, and recovery

- Check `show version` and `show system image` in the EdgeOS operational CLI. Confirm the ARM64 UEFI build and the running and default image. For the Web UI, open `https://INSTANCE_IP/` from the authorized administration network and verify the certificate before signing in.
- Reboot normally and confirm you can sign in through the Web UI again. If you configured SSH, verify public-key login. Keep a copy of the saved configuration outside the instance.
- If you will route traffic for other instances, first configure and verify EdgeOS routing and firewall rules. Then enable **Skip source/destination check** on the forwarding VNIC and use its **private IP** as the OCI route target for the intended destination CIDRs. [Oracle VCN route targets](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)

| Symptom | Check |
| --- | --- |
| Import fails or A1 is unavailable | Check the import work request, the QCOW2 checksum, Object Storage read permissions, region and compartment, A1 shape compatibility, and tenancy capacity or quota. |
| Instance is running with no guest login prompt | Check the serial connection and boot output. Confirm `UEFI_64`, Secure Boot disabled, paravirtualized boot volume, and that the image is the ARM64 UEFI QCOW2. |
| Serial login works but HTTPS or SSH times out | Compare the private DHCP address with the VNIC. Check the target IP, subnet routes, OCI NSG and security list rules, and the guest firewall. |
| SSH responds with `Permission denied (publickey)` | Check that you are connecting as `ubnt` with the matching private key, and that the key type and base64 value were committed and saved in EdgeOS. |

Use the ARM64 UEFI upgrade archive for subsequent firmware updates on this instance. [A/B upgrades](#ab-upgrades)

## A/B upgrades

Back up the configuration and verify the SHA-256 of the platform-specific upgrade archive. In the operational CLI, run `add system image /tmp/firmware.tar` with the actual archive path. After installation succeeds, use `show system image` to confirm the default boot image, then reboot normally and check `show version`. Both slots share `/config`. If needed, select the previous slot from GRUB or use `set system image default-boot` from a running system after checking which slot it will select.
