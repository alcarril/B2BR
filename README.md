
# Born to bee Root
<div align="center">

### **Virtual machine deployment and Linux server hardening**


<a href="https://42.fr/">
    <img src="https://img.shields.io/badge/42-Project-black" alt="42" />
</a>
<a href="https://www.linux.org/">
    <img src="https://img.shields.io/badge/Linux-System_Administration-yellow?logo=linux" alt="Linux" />
</a>
<a href="https://www.virtualbox.org/">
    <img src="https://img.shields.io/badge/VirtualBox-Virtualization-blue?logo=virtualbox" alt="VirtualBox" />
</a>
<a href="https://www.gnu.org/software/bash/">
    <img src="https://img.shields.io/badge/Bash-Scripting-green?logo=gnu-bash" alt="Bash" />
</a>

<br />

</div>


## 🖥️ Project Overview

Born2beRoot is a **System Administration** project focused on creating and configuring **virtual machines**, with emphasis on **operating system setup**, **administration**, and **hardening**. The goal is to install a minimal Linux server (Debian or Rocky) inside a VirtualBox (or UTM) virtual machine following strict rules, configure services, enforce security policies, and demonstrate understanding of **system administration fundamentals**.

## ✨ Features

- **Understanding hypervisor and virtual machines and ISO manual managing** — Manually download, verify, and attach operating system ISOs to VirtualBox/UTM, understanding the virtualization layer and how hypervisors abstract hardware.
- **Understanding the Linux filesystem hierarchy and directory tree** — Navigate and manage the standard FHS directory structure (`/etc`, `/var`, `/usr`, `/home`, etc.).
- **User, group, file, and permission management** — Create and manage users and groups, assign permissions, and understand ownership (`chmod`, `chown`, `usermod`).
- **Superuser (root) access control and sudo hardening** — Restrict privileged access with sudo rules, TTY mode, attempt limits, and audit logging.
- **Basic system-level hardening** — Configure firewalls (UFW), SSH on non-standard ports, strong password policies (PAM), and MAC (AppArmor/SELinux).
- **Remote connections via SSH on a custom port** — Set up and connect to the VM securely using OpenSSH on port 4242.
- **Package management with APT** — Install, update, and remove software using `apt`, understand repositories, and keep the system up to date.
- **Logical Volume Manager (LVM) with encrypted partitions** — Create encrypted LUKS volumes and LVM logical volumes for flexible and secure disk management.
- **Bash scripting fundamentals and task automation with cron jobs** — Write a monitoring script in bash and schedule it with cron to run every 10 minutes.
- **Daemon management with systemctl** — Manage services (start, stop, enable, disable, status) using `systemctl` and understand systemd units and targets.
- **Familiarization with Linux administration commands and formatting tools** — Use commands like `lsblk`, `ss`, `ufw`, `systemctl`, and `journalctl` for everyday sysadmin tasks.

## 📚 Background and Related Repositories

The following are two of my personal repositories, used as guides and sources for carrying out this project.

**[Repo 1 — Infrastructure Foundations](https://github.com/alcarril/Infrastructure-Foundations-Handbook)** — Covers **VM creation and configuration** from the hypervisor (VirtualBox), **OS installation and partitioning**, **user management**, **SSH protocol**, and **automated provisioning** — the foundations for **Infrastructure as Code (IaC)** and **DevOps** automation.

**[Repo 2 — Linux & Systems Basics](https://github.com/alcarril/HBBSO-Docs)** — Covers **hardware, BIOS, ISOs, kernel boot process**, **Linux directory tree**, **management commands**, **daemons**, and **system administration best practices** — from the ground up.

## ⚙️ Requirements and Solutions

### 1. Operating System Download and Installation

**Requirement:** Download the latest stable version of Debian (no testing/unstable) or the latest stable version of Rocky Linux. No graphical interface (X.org, Wayland, or equivalents) is permitted.

**Solution:** Download the latest stable Debian netinstall ISO from the official site and create a virtual machine in VirtualBox.

```bash
# Debian ISO URL (check for latest version)
https://www.debian.org/distrib/
# Example: debian-12.x.x-amd64-netinst.iso
```

During installation:
1. Attach the ISO to the VM and boot
2. Select "Install" (not graphical install)
3. Choose language, location, and keyboard layout
4. Configure encrypted LVM when partitioning
5. Set the root password and create a regular user with your login
6. At software selection, deselect everything except "SSH server" and "standard system utilities"

### 2. Package Management and Sudo Installation

**Requirement:** The system must be up to date, and `sudo` must be installed and configured to allow privileged command execution for the regular user.

**Solution:** The Debian minimal install may not include sudo by default. Install it and update the system.

```bash
# Update package lists and upgrade the system
su -
apt update && apt upgrade -y

# Install sudo
apt install sudo -y

# Add your user to the sudo group
usermod -aG sudo alejandro

# Log out and back in for group changes to take effect, or use:
su - alejandro

# Verify sudo works
sudo whoami
# Expected output: root
```

### 3. Encrypted Partitions with LVM

**Requirement:** Create at least 2 encrypted partitions using LVM.

**Solution:** During the Debian installer partitioning step, select "Guided - use entire disk and set up encrypted LVM". Configure volume groups and logical volumes as needed (e.g., separate partitions for root, home, var, swap).

Manual partitioning steps in the installer:
1. Select "Manual" partitioning method
2. Create a physical partition for `/boot` (unencrypted, ~500MB)
3. Configure encrypted volume (LUKS) on the remaining space
4. Set up LVM on the decrypted volume
5. Create logical volumes: `lv_root`, `lv_home`, `lv_var`, `lv_swap`

Post-installation verification:

```bash
lsblk
sudo fdisk -l
sudo blkid | grep crypto
sudo pvs && sudo vgs && sudo lvs
```

### 4. Hostname

**Requirement:** The hostname must be your login followed by 42 (e.g., `wil42`). It must be possible to change the hostname during peer evaluation.

**Solution:** The hostname can be set via command or by editing configuration files directly.

**Option 1 — Using `hostnamectl`:**

```bash
# Set hostname
sudo hostnamectl set-hostname alejandro42

# Verify
hostnamectl
```

**Option 2 — Editing files directly:**

```bash
# Edit /etc/hostname
sudo nano /etc/hostname
# Change the content to: alejandro42

# Edit /etc/hosts
sudo nano /etc/hosts
# Update any line referencing the old hostname:
# 127.0.1.1    alejandro42
```

**Option 3 — Using `sed` (one-liner for both files):**

```bash
sudo sed -i 's/old-hostname/alejandro42/g' /etc/hostname /etc/hosts
```

To change the hostname during evaluation:

```bash
# Using hostnamectl
sudo hostnamectl set-hostname newlogin42

# Or by editing files directly
sudo sed -i 's/alejandro42/newlogin42/g' /etc/hostname /etc/hosts
```

### 5. User and Groups

**Requirement:** A user with your login as username must exist, belonging to the `user42` and `sudo` groups.

**Solution:**

```bash
# Create the user (if not created during installation)
sudo adduser alejandro

# Create the user42 group if it doesn't exist
sudo groupadd user42

# Add user to groups
sudo usermod -aG sudo,user42 alejandro

# Verify
groups alejandro
id alejandro
```

To create a new user during evaluation:

```bash
sudo adduser newuser
sudo usermod -aG sudo newuser
```

### 6. Password Policy

**Requirement:** Enforce a strong password policy with the following rules:
- Password expires every 30 days
- Minimum 2 days before password modification
- Warning 7 days before expiration
- Minimum length of 10 characters
- Must contain uppercase, lowercase, and a number
- No more than 3 consecutive identical characters
- Must not include the username
- At least 7 characters must differ from the previous password
- Root password must also comply

**Solution:** Edit `/etc/login.defs` and `/etc/pam.d/common-password`.

```bash
# Edit /etc/login.defs
sudo nano /etc/login.defs
```

Set the following:

```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

Install and configure `libpam-pwquality`:

```bash
sudo apt update && sudo apt install libpam-pwquality -y
```

Edit `/etc/pam.d/common-password`:

```
password        requisite       pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

Apply the policy to existing users:

```bash
# Force password change for all users
sudo passwd -e alejandro
sudo passwd -e root
```

### 7. Sudo Configuration

**Requirement:**
- Limited to 3 authentication attempts
- Custom error message on wrong password
- All sudo actions logged (inputs and outputs) to `/var/log/sudo/`
- TTY mode enabled
- Restricted sudo paths

**Solution:** Create `/etc/sudoers.d/sudo_config`:

```bash
sudo visudo -f /etc/sudoers.d/sudo_config
```

Add the following content:

```
Defaults        passwd_tries=3
Defaults        badpass_message="Incorrect password. Access denied."
Defaults        log_input,log_output
Defaults        iolog_dir="/var/log/sudo"
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

Verify the configuration:

```bash
sudo -V | grep -E "tries|message|log|tty|path"
```

### 8. SSH Service

**Requirement:** SSH must run on port 4242. Root login via SSH must be disabled.

**Solution:** Edit the SSH configuration file `/etc/ssh/sshd_config`:

**Option 1 — Editing with nano:**

```bash
sudo nano /etc/ssh/sshd_config
```

**Option 2 — Using sed to modify directly:**

```bash
# Change port
sudo sed -i 's/#Port 22/Port 4242/' /etc/ssh/sshd_config

# Disable root login
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
# Or if the line doesn't match, append it:
echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config
```

Modified lines in `/etc/ssh/sshd_config`:

```
Port 4242
PermitRootLogin no
```

Restart the SSH service:

```bash
sudo systemctl restart sshd
sudo systemctl status sshd
```

Verify the SSH port:

```bash
ss -tlnp | grep 4242
```

### 9. UFW Firewall

**Requirement:** Configure UFW (Uncomplicated Firewall) to leave only port 4242 open. The firewall must be active at startup.

**Solution:**

```bash
# Enable UFW
sudo ufw enable

# Allow port 4242
sudo ufw allow 4242

# Deny all other incoming connections by default
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Verify
sudo ufw status verbose
```

To check the rules:

```bash
sudo ufw status numbered
```

### 10. AppArmor

**Requirement:** AppArmor must be running at startup.

**Solution:**

```bash
# Verify AppArmor status
sudo aa-status

# Ensure AppArmor is enabled at boot
sudo systemctl enable apparmor
sudo systemctl start apparmor
```

### 11. Partitions Structure

**Requirement:** At least 2 encrypted partitions using LVM must be configured.

**Solution verification:**

```bash
# Show partition table
lsblk

# Show LVM layout
sudo lvmdiskscan
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay

# Verify LUKS encryption
sudo cryptsetup status <device>
```

## 🤖 Monitoring Script and Cron Configuration

**Requirement:** Create a bash script called `monitoring.sh` that runs at server startup and every 10 minutes via cron, broadcasting system information to all terminals using the `wall` command. No errors must be visible. The script must display the following information:

- Architecture of the operating system and its kernel version
- Number of physical processors
- Number of virtual processors
- Current available RAM and its utilization rate as a percentage
- Current available storage and its utilization rate as a percentage
- Current CPU utilization rate as a percentage
- Date and time of the last reboot
- Whether LVM is active or not
- Number of active TCP connections
- Number of users logged into the server
- IPv4 address and MAC address
- Number of commands executed with sudo

**Cron configuration:** Cron is a time-based job scheduler in Unix-like systems. It allows tasks (scripts, commands) to run automatically at specified intervals. To schedule the monitoring script, edit the root crontab with `sudo crontab -e` and add:

```
*/10 * * * * /root/monitoring.sh
@reboot /root/monitoring.sh
```

- `*/10 * * * *` — runs the script every 10 minutes (minute, hour, day, month, weekday)
- `@reboot` — runs the script once at system startup

Verify with `sudo crontab -l`.

**Expected output:**

The script broadcasts via `wall` in the following exact format:

```
Broadcast message from root@alejandro42 (tty1) (Sun Apr 25 15:45:00 2021):

#Architecture: Linux alejandro42 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-29) x86_64 GNU/Linux
#CPU physical: 1
#vCPU: 1
#Memory Usage: 74/987MB (7.50%)
#Disk Usage: 1009/2Gb (49%)
#CPU load: 6.7%
#Last boot: 2021-04-25 14:45
#LVM use: yes
#Connections TCP: 1 ESTABLISHED
#User log: 1
#Network: IP 10.0.2.15 (08:00:27:51:9b:a5)
#Sudo: 42 cmd
```

## ​ℹ️​ Resources

### Official Documentation

- [Debian Administrator's Handbook](https://www.debian.org/doc/manuals/debian-handbook/)
- [Debian Wiki](https://wiki.debian.org/)
- [AppArmor Documentation](https://apparmor.net/)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)
- [LVM How-To](https://tldp.org/HOWTO/LVM-HOWTO/)
- [OpenSSH Manual](https://www.openssh.com/manual.html)


## 👨‍💻 Author
**Alejandro Carrillo** — [alcarril](https://github.com/alcarril)
