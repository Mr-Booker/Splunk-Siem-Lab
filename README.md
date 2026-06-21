# Splunk SIEM Lab Setup

> A multi-source Security Information and Event Management (SIEM) lab built with Splunk Enterprise to collect, monitor, and analyze security logs from Windows and Linux systems.

---

## 🎯 Project Overview

Built a functional SIEM lab that centralizes log collection from multiple operating systems. This setup demonstrates real-world security monitoring capabilities including failed login detection, SSH attempt tracking, and system event analysis.

---

## 🏗️ Lab Architecture

### Components

| Machine | Role | IP Address | OS | Purpose |
|---------|------|------------|----|---------|
| Ubuntu VM | Splunk Server (Indexer/Receiver) | `192.168.15.3` | Ubuntu 24.04 | Central log aggregation, indexing, and search |
| Windows VM | Client (Universal Forwarder) | `192.168.15.6` | Windows 10/11 | Forwarding Windows Event Logs |
| Linux VM | Client (Universal Forwarder) | `192.168.15.8` | Ubuntu 24.04 | Forwarding Linux system logs |

### Network Configuration

- **VirtualBox Network Mode**: Host-only Adapter (`192.168.15.X`)
- **Adapter 1**: NAT (for internet access)
- **Adapter 2**: Host-only (for VM-to-VM communication)

---

## 📦 Installation & Configuration

### 1. Splunk Enterprise (Ubuntu Server)

**Installation Process:**
- Downloaded Splunk Enterprise `.deb` file from Splunk website
- Installed via SSH from host terminal:
  ```bash
  sudo dpkg -i splunk-<version>.deb
  sudo /opt/splunk/bin/splunk start --accept-license --answer-yes
  sudo /opt/splunk/bin/splunk enable boot-start
  ```

**Configure Receiver:**
- Accessed Splunk web at `http://192.168.15.3:8000`
- Navigate to: **Settings** → **Forwarding and receiving** → **New receiver**
- Port: `9997`

**Firewall Configuration:**
```bash
sudo ufw allow 9997/tcp
```

---

### 2. Windows Universal Forwarder

**Installation:**
- Downloaded Splunk Universal Forwarder (`.msi`) from Splunk website
- Configured server address during installation: `192.168.15.3:9997`

**Configuration File (`inputs.conf`):**
```ini
[default]
host = Windows-Client

[WinEventLog://Security]
disabled = false

[WinEventLog://System]
disabled = false

[WinEventLog://Application]
disabled = false
```

**Configuration File (`outputs.conf`):**
```ini
[tcpout]
tcpout = tcpout-server

[tcpout-server://192.168.15.3:9997]
disabled = false
```

**Services:**
- Service Name: `SplunkForwarder`
- Monitors: Windows Security, System, and Application event logs

---

### 3. Linux Universal Forwarder

**Installation:**
```bash
sudo dpkg -i splunkforwarder-<version>-linux-2.6-amd64.deb
sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.15.3:9997
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

**Configuration:**
```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
sudo /opt/splunkforwarder/bin/splunk restart
```

**SSH Setup:**
```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo ufw allow ssh
```

---

## 📁 Key Configuration Files

| Component | File | Location |
|-----------|------|----------|
| Windows | `inputs.conf` | `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf` |
| Windows | `outputs.conf` | `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf` |
| Linux | `inputs.conf` | `/opt/splunkforwarder/etc/system/local/inputs.conf` |
| Linux | `outputs.conf` | `/opt/splunkforwarder/etc/system/local/outputs.conf` |

---

## ✅ Verification & Testing

### Test Events Generated

**Windows Test Event (PowerShell):**
```powershell
New-EventLog -LogName Application -Source "ProjectTest"
Write-EventLog -LogName Application -Source "ProjectTest" -EntryType Information -EventId 9999 -Message "Splunk Lab Test Event - Windows Client"
```

**Linux Test Event (Bash):**
```bash
sudo logger -t "SplunkTest" "Splunk Lab Test Event - Linux Client"
```

### Searches Verified in Splunk Web

Access Splunk web at: `http://192.168.15.3:8000`

| Search Query | Purpose |
|------------|---------|
| `index=* earliest=-1h` | All logs from last hour |
| `source=WinEventLog:*` | Windows event logs |
| `source=*syslog*` | Linux syslog entries |
| `source=WinEventLog:Security EventCode=4625` | Failed Windows login attempts |
| `source=*auth.log* sshd` | SSH login attempts (Linux) |

---

## 🔒 Security Monitoring Capabilities

### Windows Event Logs Collected
- **Security**: Authentication events, login attempts, account changes
- **System**: Service startups, driver loads, system errors
- **Application**: Program installations, application errors

### Linux Logs Collected
- **syslog**: System messages, service events
- **auth.log**: SSH login attempts, sudo usage, authentication events

---

## 🛠️ Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| VM Crashes on Startup | Increased RAM to 4GB and CPU cores to 2 |
| Firewall Blocking Port 9997 | Added `sudo ufw allow 9997/tcp` on Ubuntu |
| Permission Denied Writing to Syslog | Used `sudo logger` instead of direct file write |
| Copy/Paste Not Working in SSH | Used right-click to paste in terminal |
| SSH Service Inactive | Installed openssh-server and enabled with `sudo systemctl enable --now ssh` |

---

## 🎓 Skills Demonstrated

- **SIEM Implementation**: Splunk Enterprise deployment and configuration
- **Log Forwarding**: Universal Forwarder setup on Windows and Linux
- **Network Configuration**: VirtualBox host-only networking
- **Security Monitoring**: Failed login detection, SSH tracking
- **Linux Administration**: SSH setup, firewall configuration, service management
- **Windows Administration**: Event log configuration, service management
- **Troubleshooting**: VM issues, network connectivity, permission problems

---

## 🚀 Conclusion

Successfully built a functional multi-source SIEM lab with Splunk Enterprise that:

- ✅ Collects logs from Windows and Linux systems
- ✅ Centralizes all logs on a single Splunk server
- ✅ Enables real-time monitoring and searching
- ✅ Supports security event analysis (failed logins, SSH attempts)
- ✅ Provides foundation for future alerting and dashboard creation

**Status**: ✅ Lab is complete and fully operational

---

## 📚 References

- [Splunk Official Documentation](https://docs.splunk.com/)
- [Splunk Universal Forwarder Download](https://www.splunk.com/en_us/download/universal-forwarder.html)
- [VirtualBox Documentation](https://www.virtualbox.org/wiki/Documentation)

---

## 📞 Contact

Check out my LinkedIn for more projects: https://www.linkedin.com/in/devon-booker-3118531b6/

---

*Built with ❤️ for cybersecurity learning*
