# Ubuntu 26.04 LTS Hardening and Workstation Setup

An ordered checklist for building a secure personal Ubuntu workstation for
software development and authorized security research.

> [!IMPORTANT]
> This guide targets **Ubuntu Desktop 26.04 LTS on amd64** and was last reviewed
> on **2026-07-14**. Read every command before running it. Commands that modify
> disks, firmware, firewall rules, user groups, or system repositories can cause
> data loss or lock you out of the machine.

This is a practical daily-driver baseline, not proof of compliance with CIS or
another security standard. Apply only the optional tools you actually need;
every additional package, repository, browser extension, and background service
increases the attack surface.

## Priority order

| Priority | Timing | Purpose |
| --- | --- | --- |
| P0 | Installation | Establish a trusted base |
| P1 | First boot | Patch and harden the OS |
| P2 | After P1 | Install development tools |
| P3 | As needed | Add optional applications |
| Ongoing | Regularly | Maintain security and recovery |

## Contents

- [P0: Before installation](#p0-before-installation)
- [P0: Install Ubuntu securely](#p0-install-ubuntu-securely)
- [P1: First boot and security baseline](#p1-first-boot-and-security-baseline)
- [P1: Backup and recovery](#p1-backup-and-recovery)
- [P1: Optional CIS audit](#p1-optional-cis-audit)
- [P2: Development environment](#p2-development-environment)
- [P3: Optional desktop applications](#p3-optional-desktop-applications)
- [P3: Authorized security-research tools](#p3-authorized-security-research-tools)
- [Ongoing maintenance](#ongoing-maintenance)

## P0: Before installation

### Back up existing data

- [ ] Copy every file that cannot be replaced to an encrypted external or
      off-site backup.
- [ ] Export browser bookmarks, password-manager recovery material, SSH public
      keys, and application settings that are not already synchronized.
- [ ] Open several files from the backup to confirm that it is readable.
- [ ] Save any Windows BitLocker recovery key before changing partitions on a
      dual-boot machine.

Ubuntu's installer can erase the selected disk permanently. A backup is not
complete until a restore has been tested.

### Download and verify Ubuntu

- [ ] Download [Ubuntu Desktop 26.04 LTS](https://ubuntu.com/download/desktop)
      from Canonical.
- [ ] Download `SHA256SUMS` and `SHA256SUMS.gpg` from the matching directory on
      [releases.ubuntu.com](https://releases.ubuntu.com/).
- [ ] Follow Canonical's
      [download verification guide](https://ubuntu.com/tutorials/how-to-verify-ubuntu)
      to authenticate the checksum file and verify the ISO.

After authenticating the Ubuntu signing key as described in the official guide:

```bash
gpg --verify SHA256SUMS.gpg SHA256SUMS
sha256sum --check SHA256SUMS --ignore-missing
```

Do not continue if either verification fails.

### Create the installation USB

- [ ] Back up the USB drive; writing an image erases it.
- [ ] On Windows, use [Rufus](https://rufus.ie/).
- [ ] On Ubuntu or another GNOME desktop, prefer **Disks > Restore Disk Image**.
- [ ] Use `dd` only when a graphical image writer is unavailable.

The following is the advanced Linux command-line path. Replace `/dev/sdX` with
the whole USB device, not a partition such as `/dev/sdX1`.

```bash
lsblk --scsi
sudo umount /dev/sdX*
sudo dd \
  if="$HOME/Downloads/ubuntu-26.04-desktop-amd64.iso" \
  of=/dev/sdX \
  bs=4M \
  conv=fsync \
  status=progress
sudo partprobe /dev/sdX
```

> [!CAUTION]
> A wrong `of=` device destroys the selected disk. Re-run `lsblk` immediately
> before `dd` and confirm the model, size, transport, and device name.

## P0: Install Ubuntu securely

- [ ] Update the device firmware/UEFI before installation when the hardware
      vendor provides a trusted update mechanism.
- [ ] Enable UEFI mode, TPM 2.0, Secure Boot, virtualization support, and the
      CPU's execute-disable/NX feature in firmware where supported.
- [ ] Boot the verified USB and select the default interactive installation.
- [ ] Enable third-party drivers only when the hardware requires them.
- [ ] Prefer **Erase disk and install Ubuntu > Encrypt with a passphrase** for a
      single-OS workstation.
- [ ] Store the disk-encryption passphrase outside the laptop in a password
      manager or another protected recovery location.
- [ ] Create a strong, unique login password and keep **Require my password to
      log in** enabled.
- [ ] Review the installation summary and disk name before selecting Install.

Passphrase-based encrypted LVM is the practical default. Ubuntu also supports
[TPM-backed disk encryption](https://documentation.ubuntu.com/desktop/en/26.04/how-to/encrypt-your-disk-with-tpm/),
but it has stricter hardware requirements and known limitations. Use it only
after reviewing those limitations, add a PIN or passphrase, and store its
recovery key away from the machine.

Manual partitioning is for an explicit multi-boot, multi-drive, or storage
layout requirement. Fixed sizes such as an 80 GB root partition are not a safe
general default; size partitions from the actual disk capacity and workload.

## P1: First boot and security baseline

### Patch the system

- [ ] Install all available operating-system updates before signing into
      personal services.

```bash
sudo apt update
apt list --upgradable
sudo apt full-upgrade
sudo reboot
```

Review autoremove changes before accepting them:

```bash
sudo apt --dry-run autoremove --purge
sudo apt autoremove --purge
```

- [ ] Confirm that automatic security updates are enabled. Ubuntu Desktop ships
      with `unattended-upgrades`; verify the configuration instead of replacing
      it blindly.

```bash
dpkg-query --show unattended-upgrades
systemctl list-timers 'apt-daily*'
sudo unattended-upgrade --dry-run --debug
```

### Update firmware safely

Keep the backup and disk-encryption recovery key available before a firmware
update, especially on a TPM-encrypted installation.

```bash
fwupdmgr get-devices
fwupdmgr refresh
fwupdmgr get-updates
sudo fwupdmgr update
```

### Verify the boot and storage protections

```bash
sudo apt install mokutil
mokutil --sb-state
lsblk --fs
```

- [ ] Confirm that Secure Boot reports `enabled`.
- [ ] Confirm that the root filesystem is backed by an encrypted mapping.
- [ ] Confirm that automatic login is disabled.

### Enforce least privilege

- [ ] Use the normal user account for daily work and `sudo` only for specific
      administration tasks.
- [ ] Do not add `NOPASSWD` package-management rules to `/etc/sudoers`.
- [ ] Review administrative group membership and remove accounts that no longer
      need it.

```bash
getent group sudo
sudo -l
```

Never edit `/etc/sudoers` directly. Use `sudo visudo` when a justified change is
required.

### Enable the host firewall

For a workstation that does not provide network services, deny unsolicited
incoming traffic and allow outgoing traffic:

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
```

Add a narrowly scoped allow rule only when a service is intentionally exposed.
Docker-published ports can bypass normal UFW rules; review the Docker section
before using container port publishing.

### Verify AppArmor

AppArmor is installed and loaded by default on Ubuntu. Verify it; do not switch
profiles to complain mode merely to silence an unexplained denial.

```bash
sudo aa-status
systemctl status apparmor --no-pager
```

Investigate unexpected denials in the journal before changing a profile:

```bash
sudo journalctl --since today | grep -i apparmor
```

### Review exposed services

```bash
sudo ss --tcp --udp --listening --processes --numeric
systemctl --type=service --state=running
```

- [ ] Identify every listening port and running service.
- [ ] Disable or uninstall only services you understand and do not need.
- [ ] Re-run the checks after installing Docker, remote-desktop, proxy, or
      development tools.

### Secure SSH only if it is needed

Ubuntu Desktop does not need an SSH server for normal local use. If remote SSH
access is required:

1. Install `openssh-server` and allow port 22 only from the required local
   network or source address.
2. Create an Ed25519 key with a passphrase and verify key-based login.
3. Keep the existing SSH session open while testing a second session.
4. Only then disable password and root login in a file under
   `/etc/ssh/sshd_config.d/`.

```bash
ssh-keygen -t ed25519
sudo ufw allow proto tcp from 192.168.1.0/24 to any port 22
sudo sshd -t
sudo systemctl reload ssh
```

Recommended server overrides after key authentication has been tested:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
```

### Lock and minimize the desktop

- [ ] Enable **Settings > Privacy & Security > Screen Lock > Automatic Screen
      Lock** with a short delay.
- [ ] Disable lock-screen notification content if it may reveal sensitive data.
- [ ] Disable location services, Bluetooth, sharing, remote desktop, and media
      sharing when they are not required.
- [ ] Remove unused browser extensions and keep the browser's sandbox and safe
      browsing protections enabled.
- [ ] Use a password manager and multi-factor authentication for important
      accounts.

## P1: Backup and recovery

- [ ] Install or open **Backups** (Déjà Dup).
- [ ] Configure automatic encrypted backups to a separate device or trusted
      off-site destination.
- [ ] Include personal files, source code not already pushed to a remote, and
      selected configuration from `~/.config` and `~/.local`.
- [ ] Keep disk-encryption, password-manager, and MFA recovery material in a
      separate protected location.
- [ ] Perform a test restore into a temporary directory.

```bash
sudo apt install deja-dup
```

## P1: Optional CIS audit

[Ubuntu Security Guide (USG)](https://documentation.ubuntu.com/security/compliance/usg/)
is available through Ubuntu Pro. Compliance profiles can lag a new Ubuntu
release, so confirm that `cis_level1_workstation` is offered for Ubuntu 26.04
before running it.

```bash
pro status --all
sudo pro enable usg
sudo apt install usg
sudo usg audit cis_level1_workstation
```

The audit writes reports under `/var/lib/usg/`.

> [!WARNING]
> Do not run `sudo usg fix cis_level1_workstation` on a daily workstation without
> reviewing and tailoring every failed rule. Automated remediation can disable
> features, change mount behavior, or break development workflows.

## P2: Development environment

Install only the groups required by the current workload.

### Base command-line tools

```bash
sudo apt update
sudo apt install \
  build-essential \
  ca-certificates \
  clang \
  cmake \
  curl \
  git \
  gnupg \
  htop \
  jq \
  make \
  ninja-build \
  pkg-config \
  traceroute \
  tree \
  unzip \
  vim \
  wget \
  xz-utils \
  zip
```

Python and native-extension build dependencies:

```bash
sudo apt install \
  libbz2-dev \
  libffi-dev \
  liblzma-dev \
  libncurses-dev \
  libreadline-dev \
  libsqlite3-dev \
  libssl-dev \
  libxml2-dev \
  libxmlsec1-dev \
  llvm \
  tk-dev \
  zlib1g-dev
```

Flutter Linux desktop dependencies:

```bash
sudo apt install libglu1-mesa libgtk-3-dev libstdc++-dev
```

### Git

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global color.ui auto
git config --global pull.rebase false
git config --global core.editor vim
git config --global --list
```

Do not place access tokens or private keys directly in Git configuration. Use a
credential helper appropriate for the account and threat model.

### Java

Use Ubuntu's default supported JDK unless a project explicitly requires another
version:

```bash
sudo apt install default-jdk
java --version
javac --version
```

Android Studio includes its own JetBrains Runtime, so a separate JDK is not
required solely to launch Android Studio.

### Node.js with NVM

Use the current tagged release from the
[official NVM repository](https://github.com/nvm-sh/nvm). The example tag below
was current when this guide was reviewed; check the releases page before use.

```bash
git clone --branch v0.40.4 --depth 1 https://github.com/nvm-sh/nvm.git "$HOME/.nvm"
```

Add the following to `~/.bashrc`:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

Open a new shell, then install the current LTS release:

```bash
command -v nvm
nvm install --lts
nvm alias default 'lts/*'
node --version
npm --version
corepack enable
yarn --version
pnpm --version
```

Avoid `sudo npm install --global`; NVM's global packages belong to the user.

### Python with pyenv

Use the [official pyenv repository](https://github.com/pyenv/pyenv) instead of
piping a remote installer directly into a shell:

```bash
git clone https://github.com/pyenv/pyenv.git "$HOME/.pyenv"
cd "$HOME/.pyenv"
src/configure
make -C src
```

Add the following to both `~/.profile` and `~/.bashrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d "$PYENV_ROOT/bin" ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - bash)"
```

Open a new shell and choose a currently supported Python version:

```bash
pyenv --version
pyenv install --list
# Replace 3.x.y with a currently supported release shown above.
pyenv install 3.x.y
pyenv global 3.x.y
python --version
```

Use a virtual environment for each project; never install project dependencies
into the system Python with `sudo pip`.

### Go

Download the appropriate archive and checksum from
[go.dev/dl](https://go.dev/dl/), then follow the official
[Linux installation guide](https://go.dev/doc/install). Do not extract a new
release over an existing `/usr/local/go` tree.

Add Go and user-installed Go binaries to `~/.profile`:

```bash
export PATH="/usr/local/go/bin:$HOME/go/bin:$PATH"
```

Open a new shell and verify:

```bash
go version
go env GOPATH GOROOT
```

### Visual Studio Code

Download the official Debian package from
[code.visualstudio.com](https://code.visualstudio.com/docs/setup/linux). Installing
the package with APT resolves dependencies and offers to configure Microsoft's
signed update repository.

```bash
cd "$HOME/Downloads"
sudo apt install ./code_*.deb
code --version
```

Review extension publishers and permissions before installation. Extensions can
read workspace files and execute code with the editor user's privileges.

### Sublime Text

If Sublime Text is preferred, follow its
[signed APT repository instructions](https://www.sublimetext.com/docs/linux_repositories.html)
and verify the result:

```bash
subl --version
```

Installing multiple editors is optional; choose the smallest set needed.

### Android Studio

1. Check virtualization support and the current requirements in the
   [official Linux installation guide](https://developer.android.com/studio/install).
2. Download the Linux archive from
   [developer.android.com/studio](https://developer.android.com/studio).
3. Extract it to `/opt` and start the setup wizard:

```bash
sudo tar -xzf "$HOME"/Downloads/android-studio-*.tar.gz -C /opt
/opt/android-studio/bin/studio
```

- [ ] Install only the Android SDK platforms and emulators required by current
      projects.
- [ ] Enable KVM acceleration and create an Android Virtual Device if needed.
- [ ] In Android Studio, select **Tools > Create Desktop Entry**.

### Flutter stable

Use Flutter's
[official Linux installation page](https://docs.flutter.dev/install/manual) to
download the current stable archive. Keep the SDK in a user-owned directory:

```bash
mkdir -p "$HOME/develop"
tar -xf "$HOME"/Downloads/flutter_linux_*-stable.tar.xz -C "$HOME/develop"
```

Add Flutter to `~/.profile`:

```bash
export PATH="$HOME/develop/flutter/bin:$PATH"
```

Open a new shell and verify all required components:

```bash
flutter --version
dart --version
flutter doctor -v
```

Resolve only the `flutter doctor` findings relevant to the target platforms.

### Docker Engine

Follow Docker's current
[Ubuntu installation documentation](https://docs.docker.com/engine/install/ubuntu/).
The signed repository flow for Ubuntu 26.04 is:

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Create `/etc/apt/sources.list.d/docker.sources`:

```text
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
```

Then install and verify:

```bash
sudo apt update
sudo apt install \
  containerd.io \
  docker-buildx-plugin \
  docker-ce \
  docker-ce-cli \
  docker-compose-plugin
sudo systemctl status docker --no-pager
sudo docker run --rm hello-world
docker --version
docker compose version
```

> [!WARNING]
> Membership in the `docker` group is equivalent to root access. Prefer `sudo
> docker` or evaluate
> [rootless mode](https://docs.docker.com/engine/security/rootless/). If you
> intentionally accept the risk, `sudo usermod -aG docker "$USER"` takes effect
> after logging out and back in.

Published container ports can bypass UFW. Bind development services to
`127.0.0.1` whenever remote access is unnecessary and review Docker's firewall
guidance before exposing a port.

### Codex and local agent tooling

Install Codex under the NVM-managed Node.js environment, without `sudo`:

```bash
npm install --global @openai/codex
codex --version
codex
```

Follow the interactive sign-in flow. See the
[official Codex getting-started page](https://openai.com/codex/get-started/).

Optional supporting tools:

- [RTK](https://github.com/rtk-ai/rtk): install from a verified release or with
  Homebrew after Homebrew is configured.
- [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp): download
  its installer, inspect it, and then execute it explicitly.

```bash
curl -fsSLo /tmp/install-codebase-memory-mcp.sh \
  https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/install.sh
less /tmp/install-codebase-memory-mcp.sh
bash /tmp/install-codebase-memory-mcp.sh --ui
codebase-memory-mcp config set auto_index true
```

Optional Ponytail plugin marketplace:

```bash
codex plugin marketplace add DietrichGebert/ponytail
codex plugin marketplace list
codex
```

Inside Codex, run `/plugins`, inspect the marketplace entry, install Ponytail,
and start a new session. Adding a marketplace does not by itself install or
enable every plugin it contains. Review third-party plugin code, hooks, MCP
servers, and requested permissions before enabling them.

### Safe maintenance commands

Do not weaken `sudo` authentication to support an update alias. Keep operating
system and third-party updates separate so a failure remains visible:

```bash
sudo apt update
sudo apt full-upgrade
sudo snap refresh

command -v brew >/dev/null && brew update && brew upgrade
command -v codex >/dev/null && codex --upgrade
command -v codebase-memory-mcp >/dev/null && codebase-memory-mcp update
```

Review release notes before major SDK, runtime, container-engine, or agent-tool
upgrades.

## P3: Optional desktop applications

### GNOME customization

```bash
sudo apt install gnome-tweaks gnome-shell-extensions
```

Install extensions sparingly and only from trusted publishers. GNOME Shell
extensions execute in the desktop session.

### Persian fonts

Noto provides broad Persian and Arabic-script coverage:

```bash
sudo apt install fonts-noto-core fonts-noto-extra fonts-noto-color-emoji
fc-cache --force
fc-list :lang=fa family | sort --unique
```

### Google Chrome

Download the Debian package from
[google.com/chrome](https://www.google.com/chrome/) and install it with APT:

```bash
cd "$HOME/Downloads"
sudo apt install ./google-chrome-stable_current_amd64.deb
google-chrome --version
```

The package configures Google's signed update repository. Review any new
third-party repository periodically.

### Snap applications

Snap is preinstalled on Ubuntu Desktop. Verify it instead of reinstalling
`snapd` or the `core` snap:

```bash
snap version
sudo snap refresh
```

Optional applications:

```bash
sudo snap install telegram-desktop
```

For Discord, prefer the current package from
[discord.com/download](https://discord.com/download) and install it with APT:

```bash
cd "$HOME/Downloads"
sudo apt install ./discord-*.deb
```

### Media tools

```bash
sudo apt install ffmpeg vlc
```

`ubuntu-restricted-extras` includes software with additional license terms;
review those terms before installing it. For `yt-dlp`, prefer its
[official installation methods](https://github.com/yt-dlp/yt-dlp/wiki/Installation)
when the Ubuntu package is too old for a supported website.

### Remote desktop

```bash
sudo apt install remmina remmina-plugin-rdp remmina-plugin-vnc
```

Do not save privileged remote credentials unless the desktop keyring is locked
with a strong password. Remove unused saved hosts and disable listening remote
desktop services.

### JetBrains IDEs, XMind, and Cline

- Install PyCharm or PhpStorm through the
  [JetBrains Toolbox App](https://www.jetbrains.com/toolbox-app/) so updates and
  IDE versions are managed consistently.
- Install XMind from its [official Linux download](https://xmind.app/download/).
- Install Cline from its [official website](https://cline.bot/) or its linked
  editor marketplace entry. Review its model provider, data-sharing behavior,
  workspace permissions, and secret handling before use.

Avoid installing both PyCharm and PhpStorm unless both are actively used.

### Homebrew on Linux

Prefer Ubuntu packages first. Homebrew adds another package manager and another
update channel; install it only for tools that are unavailable or unsuitable in
APT.

```bash
curl -fsSLo /tmp/install-homebrew.sh \
  https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh
less /tmp/install-homebrew.sh
/bin/bash /tmp/install-homebrew.sh
brew doctor
```

Follow the shell-environment instructions printed by the installer. Do not run
Homebrew with `sudo`.

### v2rayN

Download the correct Linux package from the
[official v2rayN releases](https://github.com/2dust/v2rayN/releases), verify any
published checksum, and install the local package with APT:

```bash
cd "$HOME/Downloads"
sudo apt install ./v2rayN-linux-*.deb
```

- [ ] Import configuration only from a trusted provider.
- [ ] Restrict configuration files containing credentials to the user.
- [ ] Enable autostart only after confirming the listening interfaces and ports
      with `ss --tcp --udp --listening --processes --numeric`.
- [ ] Obtain GeoIP/geosite data from a trusted, maintained release and verify
      its checksum when provided.

### Hermes Agent

Review the current requirements at
[hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/) before
installing. Download and inspect the installer rather than piping it to Bash:

```bash
curl -fsSLo /tmp/install-hermes.sh \
  https://hermes-agent.nousresearch.com/install.sh
less /tmp/install-hermes.sh
bash /tmp/install-hermes.sh
```

The optional [Hermes Web UI](https://github.com/nesquena/hermes-webui) is another
network-facing component; bind it to localhost unless remote access is
deliberately configured and protected.

### Xtreme Download Manager

Download the Linux archive from the
[official XDM website](https://xtremedownloadmanager.com/#downloads), verify a
published checksum if available, inspect the extracted files, and then run its
installer:

```bash
mkdir -p /tmp/xdm-installer
tar -xf "$HOME"/Downloads/xdm*.tar.xz -C /tmp/xdm-installer
cd /tmp/xdm-installer
less install.sh
sudo ./install.sh
```

### witr

Review the [witr repository](https://github.com/pranshuparmar/witr) and its
installer before execution:

```bash
curl -fsSLo /tmp/install-witr.sh \
  https://raw.githubusercontent.com/pranshuparmar/witr/main/install.sh
less /tmp/install-witr.sh
bash /tmp/install-witr.sh
witr --help
```

### Wine and a legacy 32-bit prefix

Install Wine only for an application that cannot be replaced with a maintained
native or web client:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine wine32:i386 winetricks
wine --version
```

Create a separate prefix instead of exporting one prefix globally:

```bash
WINEPREFIX="$HOME/.wine-mt4" WINEARCH=win32 winecfg
WINEPREFIX="$HOME/.wine-mt4" winetricks corefonts vcrun2010
```

Legacy Windows runtimes expand the attack surface. Do not open untrusted
documents or executables in Wine, and back up important prefix data separately.

## P3: Authorized security-research tools

> [!WARNING]
> Use these tools only on systems you own or have explicit permission to test.
> Keep testing targets, wordlists, captures, cookies, tokens, and generated
> reports out of public repositories. Prefer an isolated lab VM for intrusive
> testing.

### Wireshark

```bash
sudo apt install wireshark
wireshark --version
```

Packet capture normally requires elevated capture privileges. If non-root
capture is needed, explicitly allow it and add only the required user:

```bash
sudo dpkg-reconfigure wireshark-common
sudo usermod -aG wireshark "$USER"
```

Log out and back in, then verify with `groups`. Membership in the `wireshark`
group permits capturing network traffic; do not grant it to unrelated users.

### Burp Suite

Use only PortSwigger's
[official combined Community/Professional installer](https://portswigger.net/burp/downloads).
Select the correct architecture with `uname -m`, compare the download's SHA-256
checksum with the value published by PortSwigger, and run the installer.

```bash
uname -m
sha256sum "$HOME"/Downloads/burpsuite_*.sh
chmod u+x "$HOME"/Downloads/burpsuite_*.sh
"$HOME"/Downloads/burpsuite_*.sh
```

Use Community Edition or activate Professional with a legitimate license. Do
not install loader/keygen JAR files. Install Burp's CA certificate only in a
dedicated testing browser profile, then remove it when it is no longer needed.

### crunch

```bash
sudo apt install crunch
crunch --help
```

Wordlists can consume significant disk space. Generate them in a dedicated
directory and check the expected size first.

### ffuf

After Go is installed:

```bash
go install github.com/ffuf/ffuf/v2@latest
ffuf -V
```

For reproducible environments, replace `@latest` with a reviewed release tag.

### jwt_tool

Install [jwt_tool](https://github.com/ticarpi/jwt_tool) into an isolated Python
environment:

```bash
mkdir -p "$HOME/.local/share/security-tools"
git clone https://github.com/ticarpi/jwt_tool.git \
  "$HOME/.local/share/security-tools/jwt_tool"
python3 -m venv "$HOME/.local/share/security-tools/jwt_tool/.venv"
"$HOME/.local/share/security-tools/jwt_tool/.venv/bin/pip" install \
  -r "$HOME/.local/share/security-tools/jwt_tool/requirements.txt"
"$HOME/.local/share/security-tools/jwt_tool/.venv/bin/python" \
  "$HOME/.local/share/security-tools/jwt_tool/jwt_tool.py" --help
```

### sqlmap

Use the maintained [sqlmap repository](https://github.com/sqlmapproject/sqlmap)
without installing it into the system Python:

```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git \
  "$HOME/.local/share/security-tools/sqlmap"
python3 "$HOME/.local/share/security-tools/sqlmap/sqlmap.py" --version
```

After reviewing upstream changes, update the checkout deliberately:

```bash
git -C "$HOME/.local/share/security-tools/sqlmap" pull --ff-only
```

### Jython

Jython is needed only by legacy Java extensions that still require Python 2.7
semantics. Prefer maintained Java or Python 3 extensions. If a specific Burp
extension requires Jython, download the standalone JAR from
[jython.org](https://www.jython.org/download), verify its published checksum,
and configure that exact file in the extension settings.

The old `tplmap` entry is intentionally omitted because it depends on an
obsolete Python 2 environment and unmaintained dependencies.

## Ongoing maintenance

### Weekly

- [ ] Apply APT, Snap, browser, and application security updates.
- [ ] Reboot when `/var/run/reboot-required` exists.
- [ ] Check that the latest backup completed successfully.
- [ ] Review unexpected AppArmor, authentication, and firewall log entries.

```bash
sudo apt update
sudo apt full-upgrade
sudo snap refresh
test -f /var/run/reboot-required && cat /var/run/reboot-required.pkgs
sudo journalctl --priority=warning --since '7 days ago'
```

### Monthly

- [ ] Test restoring at least one file from backup.
- [ ] Review listening ports, running services, UFW rules, sudo users, Docker
      containers, and third-party APT repositories.
- [ ] Remove unused packages, browser extensions, SDK versions, container images,
      editor extensions, SSH keys, and saved remote sessions.
- [ ] Check for firmware updates with `fwupdmgr get-updates`.

```bash
sudo ss --tcp --udp --listening --processes --numeric
sudo ufw status numbered
getent group sudo
docker ps --all
grep -Rhs '^URIs:' /etc/apt/sources.list.d/
```

### After a major change

- [ ] Re-run the port, service, firewall, AppArmor, Secure Boot, and backup
      checks after installing a container engine, VPN/proxy, remote-access tool,
      kernel driver, or new third-party repository.
- [ ] Record why the new privileged group, repository, service, or firewall rule
      is needed and remove it when that need ends.

Security hardening is a maintenance process, not a one-time installation step.
