# Open EdgeOS Firmware

English | [简体中文](README.zh-CN.md)

Community firmware development for the **Ubiquiti EdgeRouter 4 (ER-4)**, x86_64 systems, and ARM64 UEFI virtual machines.

This repository is the public home for firmware releases, release notes, bug reports, and project discussion. It is an independent project and is not affiliated with or endorsed by Ubiquiti.

> **Status: experimental.** Selected functions have been tested on an ER-4, but full hardware, compatibility, recovery, performance, and stability acceptance is still incomplete. A successful test of one feature does not establish production readiness.

## Project overview

The project aims to bring a newer Linux foundation to existing EdgeRouter devices while preserving the familiar EdgeOS CLI, configuration workflow, Web interface, and service behavior. It also aims to extend the useful life of these devices by introducing more functionality and new features over time.

**This project is entirely AI-driven.** AI drives the development process, including analysis, implementation, compatibility work, testing, and documentation.

The current implementation combines:

- Linux 6.18.60 with OpenWrt kernel inputs and platform-specific board support in v0.2.0-dev.
- Debian Buster user space for MIPS big-endian O32 (ER-4), amd64 (x86_64), and arm64 (ARM64 UEFI).
- The Vyatta/EdgeOS management layer, with compatibility work for the newer kernel and runtime.

Compatibility with existing EdgeOS behavior is a project goal. Complete feature parity and automatic migration of every existing configuration have not been established. A newer kernel alone also does not establish that every user-space component is current or that the firmware meets a security maintenance baseline.

## Supported devices

The table lists the v0.2.0-dev release targets. See the release notes for platform compatibility and validation scope.

| Device | Status |
| --- | --- |
| **EdgeRouter 4 (ER-4)** | Native EdgeOS system-image upgrade archive |
| **x86_64** | Disk images for PCs and VMs, plus an upgrade archive for existing Open EdgeOS A/B installations |
| **ARM64 UEFI** | Disk images for ARM64 UEFI VMs with VirtIO, plus an upgrade archive for existing Open EdgeOS A/B installations |
| EdgeRouter 6P (ER-6P), EdgeRouter 12 (ER-12), EdgeRouter 12P (ER-12P) | Not validated; board, port, and boot compatibility remain unverified |
| Other EdgeRouter models | No supported release at present |

A shared platform or firmware family name, including `e300`, does not establish cross-model compatibility. Use an image only when its release notes explicitly list your exact model.

## Current progress

Updated **2026-11-24 (Asia/Shanghai)**. The table summarizes development testing for the three release targets.

| Platform and area | Development results | Remaining validation |
| --- | --- | --- |
| ER-4 installation and management | On November 23, an ER-4 upgraded through the native CLI and booted a Linux 6.18.60 development image. Saved and active configurations were preserved, and HTTPS management was reachable. | First installation of the release image, configuration migration, rollback, and independent recovery |
| ER-4 routing, VPN, and DPI | Hardware tests covered dual-stack static routing, IPv4 BGP, and OSPFv2. A private mGRE topology verified authenticated NHRP registration, route advertisement and withdrawal, and forwarding over IKEv2 ESP. A module update was used to verify QUIC v1/v2 classification. | More external peers and configuration combinations, fragmented HTTP/3 classification, full port and SFP traffic coverage, acceleration, and sustained load |
| x86_64 and ARM64 UEFI | On November 24, development images on both platforms completed A/B upgrades and rebooted with saved and active configurations preserved and HTTPS management available. Subsequent candidate images booted to serial login in isolated UEFI VMs. | Deployment of the subsequent candidates, BIOS/UEFI and different disk sector sizes on x86_64 hardware, ARM64 cloud import, and rollback |
| x86_64 and ARM64 UEFI networking | On November 24, a private topology with an ER2 verified selected dual-stack routing, MPLS/LDP, GRE/NHRP, IKEv2, dual-stack firewall, IPv4 NAT, DNS forwarding, and DHCPv4 scenarios. Further checks covered NHRP Witness failover and failback, and DNS application blocking and recovery after rule removal with the candidate DPI component. | Broader device interoperability, failure combinations, sustained load, power loss, and recovery |

Each result applies to the development image or component update used for that test. See the release notes for the validation scope of the published assets.

## Firmware downloads and installation

Use the installation guide to choose the correct asset for your platform and check installation and recovery requirements. [Firmware installation guide](INSTALL.md)

See the release page for firmware assets, SHA-256 checksums, changes, and known issues for each version. [GitHub Releases](https://github.com/ibazzi/open-edgeos-firmware/releases)

Each release should provide:

- Exact supported models, version, build date, and debug/release mode.
- Firmware files and SHA-256 checksums.
- Changes, tested functions, known issues, and untested areas.
- Required starting firmware and bootloader, configuration compatibility, and installation steps.
- Backup requirements, recovery prerequisites, and the tested scope of upgrade/rollback.
- Component/source provenance and applicable licensing notices.

Before installing, back up the original configuration and keep the backup off the router. Confirm the image model and checksum, and prepare the recovery access described in that release. Preserving `config.boot` does not guarantee that every configuration item will migrate successfully. Do not assume automatic rollback.

Before first installation on ER-4, update to the latest official EdgeOS firmware and the latest official bootloader for that model. Complete the updates and verify them in the official EdgeOS environment. [Bootloader prerequisite](INSTALL.md#bootloader-prerequisite)

## Bug reports and discussion

Use **GitHub Issues** for reproducible bugs, device compatibility reports, and feature requests. Search existing issues first and keep each report focused on one problem. English and Chinese reports are welcome.

For general questions and ideas, use **GitHub Discussions** if enabled; otherwise, open an Issue and identify it as a question or proposal. Feature requests do not imply a delivery commitment or schedule.

**Upstream bugs:** Bugs confirmed to originate in upstream code need to be fixed by the respective upstream project. This project will then incorporate the upstream fix in a subsequent update; the timing depends on the upstream release and compatibility validation. When reporting such a bug here, include a link to the upstream issue or fix if available so its progress can be tracked.

Include the following when reporting a bug:

```text
Device model and hardware revision:
Firmware version / release tag and debug or release mode:
Previous firmware version and installation/upgrade method:
Relevant topology, interfaces, and configuration (redacted):
Steps to reproduce:
Expected behavior:
Actual behavior:
Frequency and impact:
Relevant logs / screenshots:
Recovery steps or workaround, if any:
```

Remove passwords, private keys, tokens, VPN secrets, and identifying network details before posting. Share the smallest relevant configuration and log excerpts rather than an unredacted backup. Throughput or acceleration reports should also describe the traffic direction, packet size, enabled features, peer setup, and CPU usage.

For a suspected security vulnerability, use GitHub's private **Report a vulnerability** option if it is available. Otherwise, request a private reporting channel without posting sensitive details publicly.

## Contributing

Useful contributions include reproducible reports, documented test results, translations, and device compatibility information. Describe exactly what was tested, on which firmware and hardware, and with what outcome. Separate physical hardware tests from emulation or isolated-peer tests.

## Source availability

The project's development source code is not publicly available during the current development phase. We plan to open-source it once the firmware reaches a reasonably stable state. The scope and timing of that release, along with the source repository and build instructions, will be announced here.

For now, this repository focuses on firmware releases, issue tracking, and community discussion. This source release plan does not change the applicable licenses of upstream or third-party components.

## Disclaimer

By downloading, installing, upgrading, or using this firmware, you acknowledge that you have read, fully understood, and agreed to this disclaimer, the applicable component licenses, and the terms and risk notices accompanying the relevant release. If you do not agree, do not download, install, upgrade, or use this firmware.

This is an independent, experimental project. Firmware, tools, and documentation are provided “as is,” without warranties of correctness, reliability, security, compatibility, or fitness for a particular purpose. AI-driven development does not guarantee that the software or documentation is free of errors.

Installing, upgrading, or using this firmware may cause loss of configuration or data, network outages, security issues, or an unbootable device requiring recovery. Evaluate these risks before use, keep backups off the device, and prepare a recovery method. Use in production or critical networks requires your own assessment and validation.

You are responsible for your decision to install and use this firmware. To the extent permitted by applicable law, the project maintainers and contributors disclaim liability for loss or damage arising from its use. The project does not promise ongoing maintenance, response times, or recovery assistance. This disclaimer does not override applicable component licenses or rights that cannot legally be excluded.

## Licensing and acknowledgments

The firmware draws on OpenWrt, Debian, Linux, Vyatta, and EdgeOS components. It includes reused vendor components, so the project name does not imply that every bundled component is open source. Component licenses and notices remain applicable; consult the materials accompanying each release for source availability and redistribution terms.

Ubiquiti, EdgeRouter, and EdgeOS names are used to identify the relevant hardware and software. Thanks to the upstream projects and community contributors whose work makes this effort possible.
