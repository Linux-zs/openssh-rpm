# openssh-rpm

本仓库用于管理 **OpenSSH 打包成 RPM 的 spec 文件**，以及在 Release 中发布打包好的 RPM 包。

## 📦 仓库内容
- `openssh.spec`  
  唯一的 spec 文件，基于 OpenSSH 官方自带的 spec 修改而来，主要改动包括：
  - **内置静态 OpenSSL**：打包时会编译并静态链接 OpenSSL，不依赖系统本地的 OpenSSL 库。
  - **新增 ssh-copy-id**：方便用户快速分发公钥。
- **Releases**  
  提供不同版本的 OpenSSH RPM 包，用户可直接下载使用。

## 🔧 自行打包方法
如果你希望自己打包 RPM，可以按照以下步骤操作：

1. 安装必要工具：
   ```bash
   yum install rpm-build rpmdevtools spectool -y
   ```
2. 初始化打包目录、下载源码包、：
   ```bash
   rpmdev-setuptree
   ```
3. 下载源码包（根据 openssh.spec 中的定义）：
  ```bash
  spectool -g -R openssh.spec
```
4. 开始打包：
  ```bash
  rpmbuild -ba --nodebuginfo openssh.spec
```
5. 打包完成后，生成的 RPM 包会出现在：
  ```bash
  ~/rpmbuild/RPMS/
```
⚠️ 注意事项
升级前请务必备份关键配置文件，例如：

/etc/ssh/sshd_config
/etc/ssh/ssh_config
/etc/pam.d/sshd

## 🚀 安装 / 升级流程

1. 下载已构建好的 RPM 包（或使用你自己打包生成的 RPM）。
2. 安装或升级：
   ```bash
   # 安装
   rpm -ivh openssh-<version>.rpm

   # 升级
   rpm -Uvh openssh-<version>.rpm
   ```
## ❓ 常见问题（FAQ）

### 1. 为什么要静态链接 OpenSSL？
- 系统自带的 OpenSSL 版本可能过旧或存在兼容性问题。
- 静态链接可以确保 OpenSSH 使用的是打包时指定的 OpenSSL 版本，避免依赖系统库带来的不确定性。

### 2. 如何回退到系统自带的 OpenSSH？
- 如果需要回退，可以先卸载本项目的 RPM 包：
  ```bash
  rpm -e openssh
  yum install -y openssh openssh-server
```
