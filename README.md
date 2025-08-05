👥 Audience

This guide is intended for system administrators, DevOps engineers, and security-conscious developers looking to harden their RedHat-based Linux systems (RHEL, CentOS, Rocky Linux) using best practices.
It includes step-by-step instructions with copy-paste friendly commands to help secure your Linux systems.

📁 Project Structure

<pre> ``` linux-hardening/
 ┣ 📜 README.md        # This guide
 ┣ 📁 scripts/         # Optional: automation scripts for hardening tasks
 ┣ 📁 docs/            # Detailed sub-guides or references
 ┗ 📄 .gitignore ``` </pre>

 📑 **Table of Contents**
- [System Updates and Kernel Patching](#system-updates-and-kernel-patching)
- [SSH Hardening](#ssh-hardening)
- [Firewall Configuration](#firewall-configuration)
- [Check Listening Network Ports](#check-listening-network-ports)
- [Disable Unnecessary Services](#disable-unnecessary-services)
- [User Account Security and Password Policy](#user-account-security-and-password-policy)
- [Partitioning Best Practices](#partitioning-best-practices)
- [Enforcing SELinux](#enforcing-selinux)
- [Logging and Auditing](#logging-and-auditing)
- [Configure Time and NTP](#configure-time-and-ntp)
- [Install and Configure Fail2Ban](#install-and-configure-fail2ban)


## 1. System Updates and Kernel Patching

Keep the system and kernel updated to reduce vulnerabilities

- Check current kernel
  
`uname -r`

- Update all packages
  
`sudo dnf update -y`

- Reboot if kernel was updated
  
`sudo reboot`

- Verify kernel after reboot
  
`uname -r`

## SSH Hardening

Restrict remote access and improve SSH security

- Backup sshd_config
  
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

- Edit SSH config
  
sudo vi /etc/ssh/sshd_config

- Recommended settings to modify/add:
  
<pre> ``` PermitRootLogin no PasswordAuthentication no AllowUsers your_user Protocol 2 MaxAuthTries 3 LoginGraceTime 30 ClientAliveInterval 300 ClientAliveCountMax 2 ``` </pre>

- Restart SSH
  
`sudo systemctl restart sshd`

- NB: Make sure your user has SSH key-based access before disabling password login.

## Firewall Configuration

Allow only specific services using firewalld

- Enable and start firewalld
  
`sudo systemctl enable firewalld --now`

- List zones
  
`sudo firewall-cmd --get-active-zones`

- List current rules
  
`sudo firewall-cmd --list-all`

- Allow specific services
  
`sudo firewall-cmd --permanent --add-service=ssh`
`sudo firewall-cmd --permanent --add-service=http`
`sudo firewall-cmd --permanent --add-service=https`

- Reload firewall
  
`sudo firewall-cmd --reload`

## Check Listening Network Ports

Review open ports to detect unnecessary services.

- Check listening ports
  
`sudo ss -tuln`

- OR use netstat if available
  
`sudo netstat -tulnp`

- Optional: check running services and their ports
  
`sudo lsof -i -P -n`

## Disable Unnecessary Services

Reduce attack surface by disabling unneeded services.

- List enabled services
  
`sudo systemctl list-unit-files --type=service | grep enabled`

- Disable unused service (example)
  
`sudo systemctl disable avahi-daemon --now`

## User Account Security and Password Policy

Configure password aging and enforce strong password policies.

- Check user password info
  
`sudo chage -l username`

- Force password change every 90 days
  
`sudo chage -M 90 username`

- Set account expiration
  
`sudo chage -E 2025-12-31 username`

Edit /etc/login.defs:

`PASS_MAX_DAYS   90
PASS_MIN_DAYS   10
PASS_WARN_AGE   7`

- Edit /etc/security/pwquality.conf for password strength rules:
  
`minlen = 12
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1`

## Partitioning Best Practices

Mount sensitive directories with restrictive options (if applicable):

- Edit /etc/fstab to include:
  
`tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0
/dev/sdX /var tmpfs defaults,nodev,nosuid 0 2`

- Apply the changes:

`sudo mount -a`

## Enforcing SELinux

Use SELinux to enforce mandatory access controls.

- Check SELinux status
- 
`sestatus`

- Check mode
- 
`getenforce`

- To set to enforcing mode, edit config:
- 
`sudo vi /etc/selinux/config`

- Change to:
`SELINUX=enforcing`

- Reboot for changes to take effect
`sudo reboot`

- Verify
`getenforce`

## Logging and Auditing

Enable and configure auditd for event logging.

- Install auditd
  
`sudo dnf install audit -y`

- Enable and start service
  
`sudo systemctl enable auditd --now`

- View logs
  
`sudo ausearch -x /usr/bin/passwd`

- Optional: log file changes
  
`sudo auditctl -w /etc/passwd -p wa -k passwd_changes`

## Configure Time and NTP

Ensure your server has correct time settings.

- Set timezone

`sudo timedatectl set-timezone UTC`

- Install chrony

`sudo dnf install chrony -y`

- Enable and start

`sudo systemctl enable chronyd --now`

- Check sync status

`chronyc tracking`

## Install and Configure Fail2Ban

Protect SSH and other services from brute-force attacks.

- Install fail2ban

`sudo dnf install epel-release -y`

`sudo dnf install fail2ban -y`

- Create SSH jail

`sudo vi /etc/fail2ban/jail.local`

- Add configuration:

[sshd]
`enabled = true
port    = ssh
logpath = %(sshd_log)s
maxretry = 5`

- Enable and start fail2ban

`sudo systemctl enable fail2ban --now`

- Check status
  
`sudo fail2ban-client status sshd`

- Always test changes on a staging environment before applying to production.

- Review logs regularly.

- Monitor new vulnerabilities using tools like Lynis or OpenSCAP.

- Keep this guide up-to-date!










