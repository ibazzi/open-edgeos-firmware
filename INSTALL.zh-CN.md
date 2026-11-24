# 固件刷机说明

[English](INSTALL.md) | 简体中文 | [项目 README](README.zh-CN.md)

请按发布说明选择适用于目标平台的镜像。第 1–6 节适用于 ER-4 的 EdgeOS 原生 `.tar` 系统镜像安装。[x86_64](#x86_64) 和 [ARM64 UEFI](#arm64-uefi) 另有首次安装及 A/B 升级步骤。

> 操作前请阅读[免责声明](README.zh-CN.md#免责声明)。本固件仍为实验性，安装可能导致业务中断、配置丢失或设备无法启动。请提前完成备份并准备恢复方案。

## 1. 确认版本适用性

- 仅下载本仓库 **GitHub Releases** 中明确发布的固件附件。GitHub 自动生成的“Source code”压缩包不是刷机固件。
- 确认版本说明明确支持 **ER-4**、你当前的起始固件版本，以及准备采用的安装方式。文件名带有 `e300` 并不足以证明兼容。
- 阅读已知问题、配置兼容性、首次启动的访问方式及恢复前提。如果版本说明未提供从当前系统迁入的方法，请先咨询。
- 按版本说明选择 debug 或 release 镜像。文件名中的 `release` 是打包模式，不代表已通过稳定性验收。

开发记录包含从官方 EdgeOS 3.0.1 基线进行的首次原生 debug 安装，之后还做过组件修复。这不能证明任意公开镜像均能从任意 EdgeOS 版本直接升级。当前运行 OpenWrt 或已经修改存储布局的设备不适用本文。项目固件之间的升级也需由目标版本明确说明支持。

### Bootloader 前置要求

ER-4 首次安装 Open EdgeOS 前，须将官方 EdgeOS 固件和 Bootloader 均更新至适用于本机型的官方最新版本。两项更新各自在官方 EdgeOS 环境中完成。[官方固件更新说明](https://help.uisp.com/hc/en-us/articles/22591203404311-EdgeRouter-How-to-Upgrade-the-EdgeOS-Firmware) · [官方 Bootloader 更新说明](https://help.uisp.com/hc/en-us/articles/22591243829911-EdgeRouter-How-to-Update-the-Bootloader)

在官方 EdgeOS 的操作模式 CLI 中，可通过以下命令检查：

```text
show system boot-image
```

该命令对比已安装版本与当前官方固件内附的 Bootloader，并非在线查询最新版本。请结合适用的官方固件和官方说明核对，不要将旧示例中的固定版本号当作最新版本。

如需更新，应先在官方环境完成更新、重启和复核，再安装本项目固件。Bootloader 写入期间不得断电或重启；版本或更新结果不明确时，应先停止并确认。不要在本实验性固件中尝试更新 Bootloader。

### 启动与存储保护措施

当前 ER-4 开发固件采取了以下措施，以降低设备无法启动或存储损坏的风险：

| 措施 | 保护范围与限制 |
| --- | --- |
| 跳过自动 Bootloader 写入 | 平台启动脚本跳过自动 Bootloader 更新和 Bootloader 损坏恢复分支，避免这些启动路径触发写入；不影响正常系统启动，也不代表关闭 Bootloader 自身的恢复入口 |
| Bootloader 分区只读 | ER-4 运行时设备树修正将 Linux MTD 中的 `boot0` 和 `boot1` 标记为只读 |
| 保留原存储布局 | 由内核挂载 SquashFS 系统镜像和可写 overlay，并使用已有配置目录 |
| 标准关机与重启 | 正常关机与重启由 systemd 管理 |
| 构建时镜像检查 | 打包时检查归档成员、文件类型、内核/根文件系统格式及校验值，帮助发现产物格式错误；不代表安装必然成功，也不能保证来源真实性或完整硬件兼容性 |

Bootloader 写保护适用于开发及测试阶段。请在安装本固件前完成官方 Bootloader 更新。

请在发布说明中核对具体产物已验证的启动和存储行为。

上述措施不提供自动回退，不保证设备内始终保有完整备用镜像，也不保证绝不变砖。原生升级器仍会修改存储中的镜像；内核无法启动、存储损坏或安装中断等情况仍可能需要独立恢复。请始终保留设备外备份，并准备串口及恢复入口。

## 2. 准备设备、访问方式和备份

| 项目 | 准备事项 |
| --- | --- |
| 维护窗口 | 预留网络中断及现场恢复时间，准备备用上网方式 |
| 供电 | 使用可靠电源，安装和首次启动期间不要断电 |
| 管理电脑 | 使用有线连接及当前配置下已确认可用的接口，不要依赖穿过待升级路由器的 VPN 管理连接 |
| 管理权限 | 确认现有 Web/SSH 管理账号可用，记录管理 IP、端口映射、VLAN 和地址配置 |
| 串口 | 实验性固件安装前应备好兼容的 Console 线并确认日志可读；ER-4 参数为 115200、8N1、无流控 |
| 恢复资料 | 将官方恢复说明、正确的恢复镜像及所需工具提前保存在电脑上，确保断网后仍可使用 |
| 备份 | 导出配置，并单独保存所需证书、密钥、脚本和其他自定义文件；安全存放在路由器之外，检查文件可读性 |

使用当前系统的配置备份功能导出配置。配置导出不一定是整机备份，也不一定包含自定义文件。不要将备份或秘密信息上传到 Issue。旧配置可能保留，但保留文件不代表迁移成功；即使首次启动正常，也应继续保存原始备份。

上传前，在路由器的 **EdgeOS 操作模式 CLI**（非 `configure` 配置模式）记录：

```text
show version
show system image
show system storage
```

核对机型、当前版本、镜像状态和剩余存储空间。空间要求以版本说明为准，不能只按压缩包大小估算上传、解包及安装所需空间。通过 CLI 上传至 `/tmp` 时，还应在路由器 shell 中用 `df -h /tmp` 检查该路径空间。空间不足时先停止并排查，不要盲目删除旧镜像或系统文件腾空间。

请将上述 Bootloader 准备作为独立的官方维护操作提前完成。下文系统镜像安装步骤**不更新 Bootloader**，不要在这些步骤中执行 `add system boot-image`、`ubnt-upgrade-boot`、`flash_erase`、直接写分区或重新分区等操作。本项目固件不是 Bootloader 更新包。

## 3. 校验下载的固件

下文 `firmware.tar` 表示你实际下载的固件附件，请替换为真实文件名。保留原始 `.tar` 归档，上传前不要解压或重新打包。

在**电脑上**，按系统选择命令计算 SHA-256：

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

将完整结果与该版本对应附件发布的校验值比较。缺少校验值或结果不一致时不要继续。校验值用于发现文件变化，固件及校验值都应从目标发布页面获取。旧升级器使用的内部 MD5 文件不能替代此检查，也不能证明发布者身份。

## 4. 选择一种受支持的安装方式

一次安装只使用 **Web 或 CLI 其中一种方式**，不要同时操作。对应版本必须说明支持所选入口；官方 EdgeOS 的接口说明本身不能证明其一定接受自定义固件。

### 方式 A：通过 Web 界面安装

1. 使用现有管理账号登录路由器**当前的管理地址**。
2. 打开 **System → Upgrade System Image → Upload a file**，选择已校验的固件 `.tar`。
3. 等待明确的安装结果。文件上传结束不等于固件安装成功。
4. 安装成功后按页面提示和该版本的首次启动说明重启。若出现错误，先保存报错并按下文排查，不要立即重试或重启。

原生 Web 菜单可参考[官方固件升级说明](https://help.uisp.com/hc/en-us/articles/22591203404311-EdgeRouter-How-to-Upgrade-the-EdgeOS-Firmware)，不同版本的界面文字可能有差异。

### 方式 B：通过 CLI 安装本地文件

1. 在**电脑上**上传已校验的归档。将 `USER` 和 `ROUTER_IP` 替换为管理用户名和当前管理 IP：

   ```sh
   scp ./firmware.tar USER@ROUTER_IP:/tmp/firmware.tar
   ```

2. 在**路由器 shell** 中再次计算校验值，并与 Release 发布值比较：

   ```sh
   sha256sum /tmp/firmware.tar
   ```

3. 在 **EdgeOS 操作模式 CLI** 中指定本地文件安装：

   ```text
   add system image /tmp/firmware.tar
   ```

   仔细阅读输出和确认提示。不要省略文件名：不带文件名的 `add system image` 可能调用其他更新源。不要绕过机型或完整性校验。

4. 仅在安装器明确报告成功后，检查已安装镜像：

   ```text
   show system image
   ```

   确认下次启动指向预期版本。状态不符合预期或含义不明确时，先停止并保存安装输出，不要反复切换镜像尝试修复。

5. 确认安装结果和下次启动状态后，正常重启：

   ```text
   reboot
   ```

逐步执行，不要将安装和重启拼成一条连续命令。重启期间 SSH/Web 断开属于预期现象，请持续记录串口日志，不要断电。

## 5. 首次启动后检查

访问地址和账号以对应版本说明为准。不要假定设备会重置为 `192.168.1.1` 或使用默认密码；原配置和账号可能继续保留。

- 用 `show version` 核对固件版本，并查看 `show system image`。
- 确认 CLI/SSH、Web 登录、物理端口对应关系、管理连通性及配置状态。
- 按实际需求测试地址、路由、DNS/DHCP、防火墙/NAT、VPN 和真实客户端流量。仅能打开 Web 页面不足以证明业务正常。
- 检查启动日志中的存储、驱动、服务及配置错误，保存配置前先记录不支持或未生效的项目。
- 确认配置兼容后，通过正常 EdgeOS 配置流程保存需要的修改，并继续保留设备外的原始备份。

逐步恢复业务连接，不要因为基础连通正常就假定尚未验证的 SFP、加速或高级协议功能均可用。

## 6. 安装失败、回退与恢复

| 现象 | 建议处理 |
| --- | --- |
| 上传或安装报错 | 保存完整报错、版本、镜像列表和空间信息，不要立即重试或重启 |
| 机型、兼容性或校验被拒绝 | 停止并核对固件附件及版本要求，不要修改归档或绕过校验 |
| 重启后无法管理 | 先查串口日志、原管理地址、链路/VLAN 和版本说明，再考虑重置 |
| 无法启动或出现存储错误 | 保留串口日志，使用已确认适用于该机型和现有 Bootloader 的恢复方式 |
| 需要恢复官方固件 | 按该版本已验证的回退流程或适用的官方恢复流程操作，配置还原单独处理 |

原生升级器可能在安装过程中删除旧镜像，甚至早于部分校验。列表中存在旧版本名称，不代表保留了完整且可启动的备份。`set system image default-boot` 会实际切换镜像文件，并非无副作用的菜单选择，本文不将其作为通用回退命令。

官方恢复资料：

- [官方 TFTP 恢复说明](https://help.uisp.com/hc/en-us/articles/22591244564887-EdgeRouter-TFTP-Recovery)：Reset 按键恢复取决于现有 Bootloader，使用专用恢复镜像，不能上传本项目的升级 `.tar` 代替。
- [官方手动 TFTP 恢复说明](https://help.uisp.com/hc/en-us/articles/22591244620311-EdgeRouter-Manual-TFTP-Recovery)：通过 Console 操作，需选择 ER-4/e300 对应章节及正确类型的恢复文件。

官方恢复会清除自定义配置，不是无损回退，也不能保证任意故障均可恢复。上述链接描述厂商流程，本项目固件的独立恢复尚未完成完整验收。Bootloader 前提应在安装前于官方环境确认完成，不要将临时更新 Bootloader 当作故障后的应急尝试。

反馈失败时，请提供准确版本、SHA-256、起始固件版本、安装方式、完整报错及脱敏后的串口日志，具体参阅[问题反馈说明](README.zh-CN.md#问题反馈与讨论)。

## x86_64

使用发布附件中的 `SHA256SUMS` 核对下载文件。虚拟机首次安装时，将 x86_64 的 `.qcow2` 导入为启动盘。物理机按目标磁盘的**逻辑扇区大小**选择 4 GiB GPT 整盘镜像。

| 逻辑扇区大小 | 镜像文件后缀 | 启动模式 |
| --- | --- | --- |
| 512 字节（含 512e） | `-release.img.gz` | Legacy BIOS 或 64 位 UEFI |
| 4096 字节（4Kn） | `-release-4kn.img.gz` | 64 位 UEFI |

使用 UEFI 时关闭 Secure Boot。备份目标盘后，将解压得到的镜像写到整块目标盘。写入会覆盖分区表和现有数据。首次启动通过屏幕或兼容串口（`ttyS0`，115200 8N1）访问。使用出厂配置时，首个识别到的以太网端口使用 `192.168.1.1/24`。

已有 x86_64 Open EdgeOS A/B 系统可用对应的 x86_64 `-release.tar` 升级。[A/B 升级](#ab-升级)

### 创建 VMware VMDK

先按 `SHA256SUMS` 核对发布的 x86_64 QCOW2。下列命令中的 `x86_64-release.qcow2` 需替换为实际下载的文件名。VMware Workstation 或 Intel Mac 上的 Fusion 可用以下命令生成 VMDK。[QEMU 镜像工具说明](https://www.qemu.org/docs/master/tools/qemu-img.html)

```sh
qemu-img convert -p -f qcow2 -O vmdk \
  -o subformat=monolithicSparse,adapter_type=lsilogic \
  x86_64-release.qcow2 edgeos-x86_64.vmdk
qemu-img compare -f qcow2 -F vmdk x86_64-release.qcow2 edgeos-x86_64.vmdk
```

创建 64 位 Linux 虚拟机，并选用该 VMDK 作为已有启动磁盘。若选择 UEFI，请关闭 Secure Boot。

使用 ESXi 时，先生成用于上传的 `streamOptimized` VMDK。

```sh
qemu-img convert -p -c -f qcow2 -O vmdk \
  -o subformat=streamOptimized,adapter_type=lsilogic \
  x86_64-release.qcow2 edgeos-x86_64-esxi-import.vmdk
```

把文件上传到虚拟机所在的数据存储目录。在 **ESXi 主机**上进入该目录，用以下命令导入为原生 VMFS 磁盘。[VMware vmkfstools 磁盘转换](https://knowledge.broadcom.com/external/article/343140/)

```sh
vmkfstools -i edgeos-x86_64-esxi-import.vmdk edgeos-x86_64-esxi.vmdk -d thin
```

导入后会生成描述文件 `edgeos-x86_64-esxi.vmdk` 和数据文件 `edgeos-x86_64-esxi-flat.vmdk`。将两者保留在同一目录，在虚拟机的「添加已有硬盘」中选择**描述文件**。使用 EFI 并关闭 Secure Boot。使用 SCSI 时可选 LSI Logic SAS。直接挂载上传用镜像可能报磁盘类型 22 不受支持。[VMware VMDK 描述文件与 flat 数据文件说明](https://knowledge.broadcom.com/external/article/321422)

## ARM64 UEFI

使用支持 64 位 UEFI 的 ARM64 虚拟机、VirtIO 磁盘和网卡，并关闭 Secure Boot。通过 `SHA256SUMS` 核对附件，将 ARM64 UEFI 的 `.qcow2` 导入为启动盘，或将解压后的 `.img.gz` 整盘镜像写入逻辑扇区为 512 字节的空启动盘。Oracle 云服务器请按下文的 [OCI Ampere A1 安装步骤](#oracle-cloud-infrastructure-oci-ampere-a1)操作。

出厂配置下，`eth0` 首次启动时通过 DHCP 获取地址。可用出厂账号登录 HTTPS 管理界面，也可通过 `ttyAMA0` 串口进入 CLI 或排查故障。

已有 ARM64 UEFI Open EdgeOS A/B 系统可使用对应的 `-release.tar` 升级。若从早期单槽磁盘迁移，先备份现有配置，再用新整盘镜像首次安装并恢复配置。

### Oracle Cloud Infrastructure (OCI Ampere A1)

以下步骤通过 Open EdgeOS 自定义镜像创建新的 `VM.Standard.A1.Flex` 实例。若从现有 OCI 实例迁移，请保留旧实例及启动卷，直到新实例验证可用。

#### 1. 校验并上传镜像

1. 从同一发布页面下载 `arm64_uefi.v0.2.0-dev.*-release.qcow2` 和 `SHA256SUMS`。用[校验命令](#3-校验下载的固件)计算 QCOW2 的 SHA-256，并与 `SHA256SUMS` 中对应文件名的一行比较。
2. 在 OCI 控制台选择将要创建实例的区域，进入 **存储 → 对象存储 → 存储桶（Buckets）**。创建或选择私有存储桶，上传完整的 QCOW2 文件。导入时选择同一区域及你有权访问的区间（Compartment）。[Oracle 自定义 Linux 镜像导入说明](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/importingcustomimagelinux.htm)

#### 2. 导入自定义镜像并设置启动能力

1. 进入 **计算 → 自定义镜像（Custom Images）→ 导入镜像（Import image）**。选择区间、填写镜像名称，并按下表设置。

   | 字段 | 选择 |
   | --- | --- |
   | 操作系统 | Linux |
   | 导入来源 | 对象存储桶，选择已上传的 QCOW2 对象 |
   | 镜像类型 | QCOW2 |
   | 启动模式 | 半虚拟化（Paravirtualized mode） |

2. 等待镜像状态变为 **可用（Available）**。打开该镜像，如果兼容规格中没有 `VM.Standard.A1.Flex`，通过 **Edit details** 添加该规格。若编辑列表中找不到 A1，可在 Cloud Shell 中使用以下 OCI CLI 命令，将 `IMAGE_OCID` 替换为该自定义镜像的 OCID。[Oracle 兼容规格设置](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/custom-images-edit.htm) · [OCI CLI 参考](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/compute/image-shape-compatibility-entry/add.html)

   ```sh
   oci compute image-shape-compatibility-entry add \
     --image-id 'IMAGE_OCID' --shape-name VM.Standard.A1.Flex
   ```

3. 在 **Edit image capabilities** 中，将固件及首选固件设为 **64 位 UEFI**（`UEFI_64`），关闭 Secure Boot。启动模式、启动卷类型和网卡连接类型均选 **Paravirtualized**。界面提供首选值时，也设为相同选项。[Oracle 镜像能力设置](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/configuringimagecapabilities-tasks.htm)

#### 3. 创建实例并设置管理网络

1. 进入 **计算 → 实例 → 创建实例（Create instance）**。在 **Change image → My images → Custom images** 中选择刚导入的镜像。在 **Change shape → Virtual machine → Ampere** 中选择 `VM.Standard.A1.Flex`，按租户配额选 OCPU 和内存。启动卷容量使用 OCI 提供的默认值，并确认启动卷和网卡仍为半虚拟化。[Oracle 实例创建说明](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
2. 为主 VNIC 选择已配置 IPv4 的 VCN 和子网。通过私网管理时，从能到达该子网的管理网络连接 VNIC 私有 IP。需要从互联网直连时，在公共子网为实例分配公网 IPv4，并确认该子网有一条指向 Internet Gateway 的 `0.0.0.0/0` 路由。在适用的 NSG 或安全列表中，仅允许管理网段访问入站 TCP 443。需要 SSH 时，再向同一网段开放 TCP 22。检查其他生效规则是否开放了更大的范围。[Oracle 公网子网连接](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/quick-action-internet-gateway.htm)
3. 创建实例并等待状态变为 **运行中（Running）**。在实例详情页查看主 VNIC 的私有或公网 IPv4 地址。

#### 4. 首次登录 Web 管理

1. 首次启动时，`eth0` 通过 DHCP 获取主 VNIC 的私有 IPv4 地址。从管理网络打开 `https://实例IP/`，使用可达的私有或公网 IPv4 地址。
2. 使用出厂用户名 `ubnt`、密码 `ubnt` 登录，随后在 **Users** 中修改出厂密码。[EdgeOS 用户账户说明](https://help.uisp.com/hc/en-us/articles/22591215486103-EdgeRouter-User-Accounts)
3. 如果无法打开页面，先检查实例地址和 OCI 入站 TCP 443。网络不可达时，通过串口排查。

#### 5. 串口与可选的 SSH 管理

在实例详情页的 **资源（Resources）** 或 **OS Management** 下打开 **Console connection**，选择 **Launch Cloud Shell connection**。如果 Cloud Shell 不可用，可创建本地串口连接，并使用 OCI 生成的 SSH 命令。看到 `ttyAMA0` 的系统登录提示后，用 EdgeOS 账号及其当前密码登录。[Oracle 串口控制台说明](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm)

SSH 默认关闭密码认证。如需 SSH 管理，请通过 EdgeOS 配置公钥。OCI 创建页面填写的 SSH 公钥作为实例元数据传入，仍需通过 EdgeOS 安装到相应账号。

1. 在**管理电脑**上使用已有的 Ed25519 公钥，或生成一对新密钥。

   ```sh
   ssh-keygen -t ed25519 -f ~/.ssh/edgeos_oci
   awk '{print $2}' ~/.ssh/edgeos_oci.pub
   ```

2. 在 **EdgeOS 串口控制台**执行以下命令。将 `BASE64_PUBLIC_KEY` 换成 `.pub` 文件的第二列。

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

   确认 `eth0` 通过 DHCP 获取主 VNIC 的私有 IPv4 地址，且存在默认路由。从互联网连接时，使用 OCI 显示的公网 IPv4。公网地址由 OCI 映射到 VNIC 的私有 IP。

3. 保持串口会话开启，在**管理电脑**上验证 SSH 登录。将 `INSTANCE_IP` 换成当前管理网络可达的 VNIC 地址。首次连接时，通过串口核对服务器提供的 SSH 主机公钥指纹。若显示 Ed25519 指纹，在实例上运行 `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`。

   ```sh
   ssh -o IdentitiesOnly=yes -i ~/.ssh/edgeos_oci ubnt@INSTANCE_IP
   ```

#### 6. 验证启动与配置保留

- 在 EdgeOS 操作模式运行 `show version` 和 `show system image`，确认 ARM64 UEFI 版本、当前运行镜像和默认启动镜像。若使用 Web 管理，从允许的管理网络打开 `https://INSTANCE_IP/`，登录前核对证书。
- 正常重启后确认 Web 仍可登录。若配置了 SSH，一并确认公钥登录。将已保存的配置备份到实例之外。
- 若实例承担其他实例的路由转发，先配置并验证 EdgeOS 路由及防火墙，再对承担转发的 VNIC 启用 **Skip source/destination check**，在 OCI 路由表中将该 VNIC 的**私有 IP**设为相应目标网段的下一跳。[Oracle VCN 路由表](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)

| 现象 | 检查项 |
| --- | --- |
| 导入失败，或选不到 A1 | 检查导入工作请求、QCOW2 校验值、对象读取权限、区域与区间、镜像兼容规格，以及 A1 容量或配额。 |
| 实例显示运行中，串口没有系统登录提示 | 核对串口连接及启动日志，确认镜像为 ARM64 UEFI QCOW2、固件为 `UEFI_64`、已关闭 Secure Boot，启动卷使用半虚拟化。 |
| 串口可登录，HTTPS 或 SSH 超时 | 比对 DHCP 地址和 VNIC 私有 IP，再检查连接地址、子网路由、OCI NSG 与安全列表，以及实例内防火墙。 |
| SSH 提示 `Permission denied (publickey)` | 确认使用 `ubnt` 用户和对应的私钥，并检查 EdgeOS 已提交和保存正确的公钥类型及 Base64 内容。 |

此实例后续使用对应的 ARM64 UEFI 升级包更新固件。[A/B 升级](#ab-升级)

## A/B 升级

备份配置，校验对应平台升级包的 SHA-256。在操作模式 CLI 中运行 `add system image /tmp/firmware.tar`，将路径替换为实际升级包。安装成功后，用 `show system image` 确认默认启动镜像，再正常重启并通过 `show version` 核对版本。两槽共享 `/config`。需要回到上一槽时，可在 GRUB 菜单选择上一槽，也可确认目标槽后在运行系统使用 `set system image default-boot`。
