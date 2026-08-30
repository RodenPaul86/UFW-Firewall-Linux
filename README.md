# UFW Firewall Setup Guide for Linux

This guide explains how to install and configure **UFW (Uncomplicated Firewall)** on a Raspberry Pi or other Debian/Ubuntu-based Linux system.

UFW provides a simple command-line interface for managing Linux firewall rules. It is available in the standard repositories on Raspberry Pi OS, Debian, and Ubuntu.

> **Tested on:** Raspberry Pi OS, Debian, Ubuntu
> **Requires:** `sudo` privileges

---

## ⚠️ Important: SSH Warning

If you are connected to your Linux machine using **SSH**, do **not** enable UFW until you have allowed SSH through the firewall.

Otherwise, you may lock yourself out of the machine.

The safe order is:

1. Install UFW
2. Allow SSH
3. Configure your firewall rules
4. Enable UFW
5. Verify the firewall

Raspberry Pi specifically recommends allowing remote access before enabling UFW.

---

# 1. Update the Package List

First, update your system's package list:

```bash
sudo apt update
```

Optionally, upgrade installed packages:

```bash
sudo apt upgrade
```

---

# 2. Install UFW

Install UFW using `apt`:

```bash
sudo apt install ufw
```

If prompted to continue, type:

```text
Y
```

and press **Enter**.

Installing UFW does **not** automatically activate the firewall. UFW remains inactive until you explicitly enable it.

---

# 3. Check the UFW Status

Verify that UFW was installed successfully:

```bash
sudo ufw status
```

You should see:

```text
Status: inactive
```

This is normal.

UFW is installed but has not yet been enabled.

For additional information, use:

```bash
sudo ufw status verbose
```

---

# 4. Allow SSH

### If you connect using SSH

Before enabling UFW, allow SSH:

```bash
sudo ufw allow ssh
```

Alternatively, you can allow the standard SSH port directly:

```bash
sudo ufw allow 22
```

You can verify the rule with:

```bash
sudo ufw status
```

You should see something similar to:

```text
To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
```

### If SSH uses a custom port

If your SSH server uses a different port, replace `22` with your SSH port.

For example, if SSH uses port `2222`:

```bash
sudo ufw allow 2222
```

> **Important:** Only allow the SSH port that your system actually uses.

---

# 5. Set the Default Firewall Policies

A common server configuration is:

* Deny incoming connections by default
* Allow outgoing connections by default

Run:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

This means that incoming connections must be explicitly allowed while normal outgoing connections remain available.

---

# 6. Allow HTTP and HTTPS

If your Raspberry Pi or Linux machine is running a web server, you will normally want to allow HTTP and HTTPS.

### HTTP

Port `80`:

```bash
sudo ufw allow 80
```

### HTTPS

Port `443`:

```bash
sudo ufw allow 443
```

You can also use the service names:

```bash
sudo ufw allow http
sudo ufw allow https
```

Raspberry Pi's documentation lists HTTP (`80`) and HTTPS (`443`) as the standard web-server ports.

---

# 7. Allow Other Services

Only open ports that your server actually needs.

For example:

### Port 3000

```bash
sudo ufw allow 3000
```

### Port 8080

```bash
sudo ufw allow 8080
```

### DNS

If your machine needs to accept DNS queries:

```bash
sudo ufw allow 53
```

You can specify the protocol when necessary:

```bash
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
```

---

# 8. Restrict Access to Your Local Network

For services that should only be accessible from your local network, you can restrict access to your LAN.

For example, if your network is:

```text
192.168.1.0/24
```

You could allow SSH only from that network:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
```

This is more restrictive than allowing SSH from anywhere.

You can also restrict a web application:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 3000 proto tcp
```

---

# 9. Enable UFW

Once your rules are configured, enable the firewall:

```bash
sudo ufw enable
```

You may receive a warning similar to:

```text
Command may disrupt existing ssh connections.
Proceed with operation (y|n)?
```

If you already allowed SSH, type:

```text
y
```

and press **Enter**.

UFW will then become active and will be configured to start when the system boots.

---

# 10. Verify the Firewall

Check the firewall status:

```bash
sudo ufw status
```

A typical server configuration may look like:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
443                        ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
443 (v6)                   ALLOW       Anywhere (v6)
```

---

# 11. View Numbered Rules

To see all firewall rules with numbers:

```bash
sudo ufw status numbered
```

Example:

```text
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    Anywhere
[ 2] 80                         ALLOW IN    Anywhere
[ 3] 443                        ALLOW IN    Anywhere
[ 4] 22 (v6)                    ALLOW IN    Anywhere (v6)
[ 5] 80 (V6)                    ALLOW IN    Anywhere (v6)
[ 6] 443 (v6)                   ALLOW IN    Anywhere (v6)
```

Numbered rules make it easier to remove individual rules.

---

# 12. Delete a Firewall Rule

You can delete a rule using its rule number.

For example:

```bash
sudo ufw delete 3
```

Or delete a rule by specifying the original rule:

```bash
sudo ufw delete allow 443
```

Then verify:

```bash
sudo ufw status numbered
```

---

# 13. Enable Firewall Logging

UFW can log firewall activity.

Enable logging:

```bash
sudo ufw logging on
```

Check the logging configuration:

```bash
sudo ufw status
```

You can disable logging with:

```bash
sudo ufw logging off
```

UFW supports multiple logging levels, including `low`, `medium`, `high`, and `full`.

For most systems, the default `low` level is a reasonable starting point.

---

# 14. Test a Rule Before Applying It

UFW supports `--dry-run`, which allows you to see what a command would do without actually changing the firewall.

For example:

```bash
sudo ufw --dry-run allow 443
```

This is useful when you're unsure about a firewall rule. Raspberry Pi and Ubuntu both document `--dry-run` as a way to preview firewall changes.

---

# 15. Check Available Application Profiles

Some applications provide UFW profiles.

List them with:

```bash
sudo ufw app list
```

Example:

```text
Available applications:
  OpenSSH
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
```

You can then allow an application profile:

```bash
sudo ufw allow OpenSSH
```

or:

```bash
sudo ufw allow 'Nginx Full'
```

To view information about a profile:

```bash
sudo ufw app info 'Nginx Full'
```

---

# 16. Rate-Limit SSH

UFW includes a `limit` rule that can help reduce brute-force SSH connection attempts.

Use:

```bash
sudo ufw limit ssh
```

Or:

```bash
sudo ufw limit 22/tcp
```

This allows SSH while rate-limiting repeated connection attempts. UFW's documentation describes `limit` as useful for protecting services such as SSH from brute-force attacks.

---

# 17. Check the Firewall After Reboot

UFW should remain enabled after restarting the machine.

Reboot:

```bash
sudo reboot
```

After reconnecting, check:

```bash
sudo ufw status
```

You should still see:

```text
Status: active
```

---

# 18. Disable UFW

If you need to temporarily disable the firewall:

```bash
sudo ufw disable
```

Check the status:

```bash
sudo ufw status
```

You should see:

```text
Status: inactive
```

---

# 19. Reset UFW

If you need to completely reset UFW's configuration:

```bash
sudo ufw reset
```

This removes the firewall rules and returns UFW to its default configuration.

> **Warning:** Resetting UFW removes your existing firewall rules. If you are connected through SSH, make sure you understand the consequences before resetting the firewall.

After resetting, you will need to configure your rules again.

---

# Recommended Web Server Configuration

For a basic Raspberry Pi web server, a good starting configuration is:

```bash
sudo apt update
sudo apt install ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22
sudo ufw limit 22

sudo ufw allow 80
sudo ufw allow 443

sudo ufw logging on

sudo ufw enable

sudo ufw status
```

This configuration:

* 🔒 Blocks unsolicited incoming connections
* 🌐 Allows HTTP
* 🔐 Allows HTTPS
* 🖥️ Allows SSH
* 🛡️ Rate-limits SSH connections
* 📤 Allows outgoing connections
* 📝 Enables firewall logging

---

# Quick Reference

| Command                        | Description               |
| ------------------------------ | ------------------------- |
| `sudo ufw status`              | Check firewall status     |
| `sudo ufw status verbose`      | Show detailed status      |
| `sudo ufw status numbered`     | Show numbered rules       |
| `sudo ufw enable`              | Enable firewall           |
| `sudo ufw disable`             | Disable firewall          |
| `sudo ufw reset`               | Reset firewall rules      |
| `sudo ufw allow 22`            | Allow SSH                 |
| `sudo ufw allow 80`            | Allow HTTP                |
| `sudo ufw allow 443`           | Allow HTTPS               |
| `sudo ufw deny 80/tcp`         | Block HTTP                |
| `sudo ufw delete allow 80`     | Remove HTTP rule          |
| `sudo ufw limit ssh`           | Rate-limit SSH            |
| `sudo ufw logging on`          | Enable logging            |
| `sudo ufw logging off`         | Disable logging           |
| `sudo ufw app list`            | List application profiles |
| `sudo ufw --dry-run ...`       | Preview a firewall change |

---

## Example: Complete Basic Setup

For a new Raspberry Pi or Linux web server, the following is a simple starting point:

```bash
# Update package information
sudo apt update

# Install UFW
sudo apt install ufw

# Configure default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH BEFORE enabling the firewall
sudo ufw allow ssh

# Rate-limit SSH connections
sudo ufw limit ssh

# Allow web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall logging
sudo ufw logging on

# Enable UFW
sudo ufw enable

# Verify configuration
sudo ufw status verbose
```

> **Remember:** Never enable UFW on a remotely managed machine until you have allowed the port you use for remote access. Otherwise, you risk locking yourself out of the server.
