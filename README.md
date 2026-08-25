# Debian Cloud Image Builder for Proxmox VE

本项目使用 [CircleCI](https://circleci.com/) 自动构建适用于 Proxmox VE 的自定义 [Debian 13 Cloud Image](https://cdimage.debian.org/images/cloud/trixie/latest/)，并自动上传至 GitHub Release。

This project uses [CircleCI](https://circleci.com/) to automatically build a customized [Debian 13 Cloud Image](https://cdimage.debian.org/images/cloud/) suitable for Proxmox VE, and publishes it to GitHub Releases.

---

## ✅ 镜像特性 | Image Features

- 基于官方 `debian-13-genericcloud-amd64.qcow2` (Trixie)
- **预装实用软件**：`qemu-guest-agent`, `curl`, `vim`, `htop`, `dnsutils`, `duf`, `tcping` 等
- **系统优化**：默认开启串行控制台（Serial Console），支持 Proxmox 实时监控
- **网络支持**：支持 Cloud-init、DHCP 获取 IP
- **远程登录**：默认开启 root 密码登录（密码由 Cloud-Init 注入，见下文）
- **软件源**：APT 源已切换至中科大（USTC）镜像，国内更新更快
- **即插即用**：可直接导入至 Proxmox VE 作为 VM 模板

- Based on official `debian-13-genericcloud-amd64.qcow2` (Trixie)
- Includes useful packages like `qemu-guest-agent`, `curl`, `vim`, `htop`, `dnsutils`, etc.
- Serial Console enabled for Proxmox VE monitoring
- Cloud-init ready, DHCP enabled
- Easily importable as a Proxmox VE template

---

## 📥 下载最新镜像 | Download Latest Image

```bash
# 下载 Debian 13 定制版镜像
wget https://github.com/xixi-zhao/debian-cloud-build/releases/download/latest/debian-13-custom.qcow2
```

---

## 💻 导入到 Proxmox VE | Import into Proxmox VE

```bash
# 1. 创建 VM 基础配置
qm create 9000 --name debian-13-template --memory 1024 --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-pci --serial0 socket --vga serial0 --agent enabled=1

# 2. 导入并挂载磁盘
# 注意：import-from 后面必须是文件的绝对路径
qm set 9000 --scsi0 local-lvm:0,import-from=$(pwd)/debian-13-custom.qcow2

# 3. 添加 Cloud-Init 驱动器并设置引导顺序
qm set 9000 --ide2 local-lvm:cloudinit
qm set 9000 --boot order=scsi0

# 4. 配置 root 密码（镜像默认开启 root 密码登录，密码在此注入）
qm set 9000 --ciuser root
qm set 9000 --cipassword "你的密码"

# 5. 转为模板
qm template 9000
```

> **关于 root 登录**：镜像已默认开启 `PermitRootLogin yes` 与 `PasswordAuthentication yes`，
> 并在 Cloud-Init 配置中设置 `disable_root: false`、`ssh_pwauth: true`。
> root 的实际密码不在镜像内预设（避免镜像泄露密码），需在创建 VM 时通过
> `--ciuser root` + `--cipassword` 由 Cloud-Init 注入。

---

## 🔁 自动构建流程 | Build Workflow

每次打 tag（如 `v3.0`）时，系统会自动执行以下操作：

1. **拉取**：从 Debian 官方源获取最新的 Trixie 云镜像。
2. **定制**：通过 `virt-customize` 注入驱动和常用工具。
3. **压缩**：使用 `virt-sparsify` 极致压缩体积。
4. **发布**：上传 `.qcow2` 至 GitHub Release，并同步更新 `latest` 标签。

---

## 🧩 修改建议 | Customization

- **增减软件**：修改 `.circleci/config.yml` 中的 `--install` 列表即可。
- **调整配置**：可以在 `virt-customize` 步骤中添加 `--run-command` 来执行自定义脚本。
- **额外二进制工具**：`duf`、`tcping` 通过「下载额外工具」步骤从 GitHub Release 拉取，
  再经 `--upload` 注入镜像内安装，版本号在下载步骤中修改。

---

## 🤝 参考资源 | References

- [Debian Cloud 官方镜像站](https://cdimage.debian.org/images/cloud/)
- [Proxmox VE 官方文档 (Cloud-Init)](https://pve.proxmox.com/wiki/Cloud-Init_Support)

---

## 🛠️ License | 许可证

本项目使用 MIT 许可证。  
This project is licensed under the MIT License.
