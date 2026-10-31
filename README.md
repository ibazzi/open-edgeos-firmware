# Open EdgeOS Firmware

English | [简体中文](README.zh-CN.md)

Community firmware development for Ubiquiti EdgeRouter devices, currently focused on the **EdgeRouter 4 (ER-4)**.

This repository is the public home for firmware releases, release notes, bug reports, and project discussion. It is an independent project and is not affiliated with or endorsed by Ubiquiti.

> **Status: experimental.** Selected functions have been tested on an ER-4, but full hardware, compatibility, recovery, performance, and stability acceptance is still incomplete. A successful test of one feature does not establish production readiness.

## Project overview

The project aims to bring a newer Linux foundation to existing EdgeRouter devices while preserving the familiar EdgeOS CLI, configuration workflow, Web interface, and service behavior. It also aims to extend the useful life of these devices by introducing more functionality and new features over time.

**This project is entirely AI-driven.** AI drives the development process, including analysis, implementation, compatibility work, testing, and documentation.

The current implementation combines:

- An OpenWrt-based Octeon kernel and board support, currently using Linux 6.12.112.
- A Debian Buster MIPS big-endian O32 user space.
- The Vyatta/EdgeOS management layer, with compatibility work for the newer kernel and runtime.

Compatibility with existing EdgeOS behavior is a project goal. Complete feature parity and automatic migration of every existing configuration have not been established. A newer kernel alone also does not establish that every user-space component is current or that the firmware meets a security maintenance baseline.

## Supported devices

“Current target” identifies the device under development; it does not mean all functions have passed acceptance.

| Device | Status |
| --- | --- |
| **EdgeRouter 4 (ER-4)** | Current development and hardware testing target; experimental support |
| EdgeRouter 6P (ER-6P), EdgeRouter 12 (ER-12), EdgeRouter 12P (ER-12P) | Not validated; board, port, and boot compatibility remain unverified |
| Other EdgeRouter models | No supported release at present |

A shared platform or firmware family name, including `e300`, does not establish cross-model compatibility. Use an image only when its release notes explicitly list your exact model.

## Current progress

Last updated: **2026-10-31 (Asia/Shanghai)**. This summary reflects recorded development and test results.

The project has moved from pre-hardware development to hardware acceptance. Selected ER-4 debug-firmware tests have passed; full product and release acceptance remain open. The latest hardware test round is paused.

| Area | Current progress | Remaining validation |
| --- | --- | --- |
| Build and packaging | Firmware images and Debian package sets have been built and checked; debug and release packaging modes exist | Final release artifact binding and clean rebuild verification |
| Boot and management | ER-4 boot, CLI/SSH, Web login and selected live statistics, configuration persistence, and normal reboot tested in defined scenarios | Full management, migration, failure, and recovery matrix |
| Networking and routing | Selected static routing, bridge/VLAN, firewall/NAT, BGP, and OSPFv2 flows tested on ER-4, including isolated peers | Full physical topology, IPv6 protocol variants, and feature combinations |
| VPN and services | Selected WireGuard, GRE, IPsec, OpenVPN, DHCP, and DNS scenarios tested | Complete configuration, lifecycle, physical-peer, and interoperability coverage |
| Physical ports and SFP | Physical eth0 traffic tested; limited SFP error handling and shared statistics behavior checked | eth1/eth2/eth3 physical traffic, SFP data traffic, hot-plug, and full port matrix |
| DPI, acceleration, MPLS, and extensions | Implementations and selected software/native checks exist; some control, classification, and statistics paths tested | Full forwarding paths, hardware acceleration, SMP/load behavior, and end-to-end feature acceptance |
| Stability, upgrade, and recovery | Normal reboot and selected restoration scenarios tested | Sustained traffic/load, power loss, independent recovery, upgrade/rollback, and final release acceptance |

Some device tests use isolated or synthetic peers. They do not establish external-device interoperability, physical-port coverage, or hardware acceleration performance. Packaging mode `release` describes an artifact build mode; it is not a stability certification.

The test device also contains incremental component updates beyond earlier complete images. Earlier image checks do not cover those later updates automatically. Each public release must state the validation scope for its exact artifacts.

## Firmware downloads and installation

See the [firmware installation guide](INSTALL.md) for preparation, checksum verification, Web/CLI installation, first-boot checks, and recovery considerations.

Published firmware belongs in this repository's **GitHub Releases** section. Use the notes attached to the exact version for device support, known issues, and installation instructions. This README does not announce a stable release or imply that downloadable firmware is already available.

Each release should provide:

- Exact supported models, version, build date, and debug/release mode.
- Firmware files and SHA-256 checksums.
- Changes, tested functions, known issues, and untested areas.
- Required starting firmware, configuration compatibility, and installation steps.
- Backup requirements, recovery prerequisites, and the tested scope of upgrade/rollback.
- Component/source provenance and applicable licensing notices.

Before installing, back up the original configuration and keep the backup off the router. Confirm the image model and checksum, and prepare the recovery access described in that release. Preserving `config.boot` does not guarantee that every configuration item will migrate successfully. Do not assume automatic rollback. Bootloader updates are outside the current testing scope; this README provides no bootloader modification procedure.

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
