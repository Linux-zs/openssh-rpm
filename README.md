# openssh-rpm

提供经过修改的 OpenSSH spec 文件，用于构建自包含静态 OpenSSL 库的 RPM 包。此包不依赖系统自带的 OpenSSL 共享库，提高安全性和部署便利性。

## 📦 仓库内容
- `openssh.spec`  
  1. 静态链接 OpenSSL 3.5.3（%global static_libcrypto 1 + 内联编译 openssl-3.5.3.tar.gz），不依赖系统 libcrypto.so
  2. 默认启用 ssh-copy-id 及其 man 页面
  3. 去掉 strip 方便调试；关闭 debuginfo 减少体积
  4. 其余逻辑（PAM、systemd、x11-askpass、gnome-askpass、Kerberos5）保持与官方一致，可直接替换 CentOS 7/8/9、RHEL 7/8/9 自带包。

## 🔧 自行打包方法
如果你希望自己打包 RPM，可以按照以下步骤操作：

0. 克隆仓库：
   ```bash
   git clone https://github.com/Linux-zs/openssh-rpm.git
   ```
1. 安装必要工具：
   ```bash
   yum groups install "Development Tools"
   yum install rpm-build rpmdevtools libXt-devel gcc make perl pam-devel krb5-devel gtk2-devel imake -y
   #el7需要额外安装
   yum install perl-IPC-Cmd perl-Time-Piece -y
   ```
   el8仓库如果没有imake的包，用下面链接安装：
   ```bash
   yum install -y https://www.rpmfind.net/linux/almalinux/8.10/PowerTools/x86_64/os/Packages/imake-1.0.7-11.el8.x86_64.rpm
   ```
2. 初始化打包目录：
   ```bash
   rpmdev-setuptree
   #初始化完成后把openssh.spec文件放到rpmbuild/SPECS/目录下
   cp openssh-rpm/openssh.spec rpmbuild/SPECS/
   ```
3. 下载源码包（根据 openssh.spec 中的定义）：
   ```bash
   cd rpmbuild/SPECS/
   spectool -g -R openssh.spec
   ```
4. 开始打包：
   ```bash
   rpmbuild -ba --nodebuginfo openssh.spec
   #el7不支持--nodebuginfo
   rpmbuild -ba openssh.spec
   ```
5. 打包完成后，生成的 RPM 包会出现在：
   ```bash
   ~/rpmbuild/RPMS/
   ```
⚠️ **升级建议必备份以下文件**  
> `/etc/ssh/sshd_config`  
> `/etc/ssh/ssh_config`  
> `/etc/pam.d/sshd`  
> 以及所有 `/etc/ssh/ssh_host_*key` 私钥文件
升级通常会尝试覆盖这些配置，比如默认不允许root登录等。

## 🚀 安装 / 升级流程

1. 下载已构建好的 RPM 包（或使用你自己打包生成的 RPM）。
2. 安装或升级：
   ```bash
   yun localinstall openssh-<version>.rpm openssh-server-<version>.rpm openssh-client-<version>.rpm
   ```
3. 重启sshd服务
   ```bash
   systemctl restart sshd
   ```
   如果你的配置文件修改过，升级后会新生成.rpmnew后缀的文件，如果没有修改过，升级会覆盖原本的配置文件
4. 验证
   ```bash
   ssh -V
   ```
## ❓ 常见问题（FAQ）

### 1. 为什么要静态链接 OpenSSL？
- 系统自带的 OpenSSL 版本可能过旧或存在兼容性问题。
- 静态链接可以确保 OpenSSH 使用的是打包时指定的 OpenSSL 版本，避免依赖系统库带来的不确定性。

### 2. 如何回退到系统自带的 OpenSSH？
- 如果需要回退，可以先卸载本项目的 RPM 包：
   ```bash
   yum remove openssh openssh-server openssh-clients
   yum install -y openssh openssh-server openssh-clients
   ```

## 免责声明
此软件包按原样提供，作者不承担任何直接或间接使用造成的风险。在生产环境部署前，请在测试环境中充分验证。
