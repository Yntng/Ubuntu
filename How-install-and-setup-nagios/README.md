# Nagios Core Installation and Setup — Ubuntu Server 24.04 LTS

## What We Are Going to Build

```text
Ubuntu Server 24.04
        │
        ├── Nagios Core
        │       └── Monitoring engine
        │
        ├── Nagios Plugins
        │       └── check_ping, check_http, check_ssh, check_load, etc.
        │
        ├── Apache
        │       └── Web interface
        │
        └── PHP
                └── Web interface support
```

At the end, you should be able to open:

```text
http://YOUR_SERVER_IP/nagios
```

Example:

```text
http://192.168.1.50/nagios
```

Nagios Core itself is free/open source.

---

# STEP 1 — Check Your Ubuntu Server

First log in to your Ubuntu server.

Run:

```bash
lsb_release -a
```

You should see something similar to:

```text
Distributor ID: Ubuntu
Description:    Ubuntu 24.04 LTS
Release:        24.04
Codename:       noble
```

```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

In TCRC you will need to bypass the IP to get apt update / upgrade 
```text
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```
Also check your IP:

```bash
ip addr
```

A simpler command:

```bash
hostname -I
```

Example:

```text
192.168.10.50
```

Write this IP down. You will use it later to access Nagios.

and also fix time before apt update and upgrade.
```text
date
sudo timedatectl set-ntp true
sudo timedatectl set-timezone Asia/Kolkata
sudo systemctl restart systemd-timesyncd
timedatectl timesync-status
```

---

# STEP 2 — Update Ubuntu

Run:

```bash
sudo apt update
```

Then:

```bash
sudo apt upgrade -y
```

After that, reboot:

```bash
sudo reboot
```

Reconnect to the server after it restarts.

---

# STEP 3 — Install Required Packages

Nagios Core requires Apache, PHP, GCC/build tools, GD libraries, OpenSSL and development packages.

Run:

```bash
sudo apt install -y autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php libgd-dev
```

Then install OpenSSL development libraries:

```bash
sudo apt install -y openssl libssl-dev
```

Install additional packages required for Nagios Plugins:

```bash
sudo apt install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext iputils-ping
```

Verify Apache:

```bash
systemctl status apache2
```

You should see:

```text
Active: active (running)
```

Press:

```text
q
```

to exit.

---

# STEP 4 — Make Sure Apache Starts Automatically

Run:

```bash
sudo systemctl enable apache2
```

Then:

```bash
sudo systemctl start apache2
```

Check:

```bash
sudo systemctl status apache2
```

---

# STEP 5 — Download Nagios Core

Go to `/tmp`:

```bash
cd /tmp
```

Download the latest Nagios Core release:

```bash
sudo wget -O nagioscore.tar.gz $(wget -q -O - https://api.github.com/repos/NagiosEnterprises/nagioscore/releases/latest | grep '"browser_download_url":' | grep -o 'https://[^"]*')
```

Check that the file exists:

```bash
ls -lh nagioscore.tar.gz
```

---

# STEP 6 — Extract Nagios Core

Run:

```bash
sudo tar xzf nagioscore.tar.gz
```

Check:

```bash
ls -d /tmp/nagios-*
```

You should get something similar to:

```text
/tmp/nagios-4.5.10
```

Enter the directory:

```bash
cd /tmp/nagios-*
```

Confirm:

```bash
pwd
```

You should be inside the Nagios source directory.

---

# STEP 7 — Configure Nagios Core

Run:

```bash
sudo ./configure --with-httpd-conf=/etc/apache2/sites-enabled
```

At the end, you want to see something similar to:

```text
*** Configuration summary for nagios 4.5.10 ...
```

and no major errors.

---

# STEP 8 — Compile Nagios

Run:

```bash
sudo make all
```

If successful, you should return to the command prompt without an error.

---

# STEP 9 — Create Nagios User and Group

Run:

```bash
sudo make install-groups-users
```

Add Apache's user to the Nagios group:

```bash
sudo usermod -a -G nagios www-data
```

Verify:

```bash
id nagios
```

And:

```bash
id www-data
```

---

# STEP 10 — Install Nagios Core

Run:

```bash
sudo make install
```

This installs the Nagios binaries, CGI files and web interface files.

---

# STEP 11 — Install Nagios Service

Run:

```bash
sudo make install-daemoninit
```

Validate the Nagios configuration:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Do not start Nagios if this shows errors.

Ideally:

```text
Total Warnings: 0
```

---

# STEP 12 — Install Command Mode

Run:

```bash
sudo make install-commandmode
```

---

# STEP 13 — Install Sample Configuration

Run:

```bash
sudo make install-config
```

Verify:

```bash
sudo ls -la /usr/local/nagios/etc/
```

You should now see files such as:

```text
nagios.cfg
cgi.cfg
resource.cfg
objects/
```

Validate Nagios:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Do not start Nagios if this shows errors.

Ideally:

```text
Total Warnings: 0
Total Errors:   0
```

Then start Nagios:

```bash
sudo systemctl start nagios
```

Check:

```bash
sudo systemctl status nagios
```

---

# STEP 14 — Install Apache Configuration

Run:

```bash
sudo make install-webconf
```

Enable Apache rewrite:

```bash
sudo a2enmod rewrite
```

Enable CGI:

```bash
sudo a2enmod cgi
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

---

# STEP 15 — Create the Nagios Web Login

Create the `nagiosadmin` account:

```bash
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

Enter a strong password when prompted:

```text
New password:
Re-type new password:
```

Use:

```text
Username: nagiosadmin
Password: YOUR_STRONG_PASSWORD
```

Do not forget this password.

If you later create another user, do **not** use `-c` because it recreates the password file.

Example:

```bash
sudo htpasswd /usr/local/nagios/etc/htpasswd.users admin2
```

---

# STEP 16 — Configure Nagios Email Address

Open the contacts configuration:

```bash
sudo nano /usr/local/nagios/etc/objects/contacts.cfg
```

Find:

```text
email
```

You should see something similar to:

```text
email                           nagios@localhost
```

Change it to your actual monitoring/admin email:

```text
email                           your-email@example.com
```

Save:

```text
Ctrl + O
```

Press:

```text
Enter
```

Exit:

```text
Ctrl + X
```

---

# STEP 17 — Install Nagios Plugins

Go to `/tmp`:

```bash
cd /tmp
```

Download the latest Nagios Plugins:

```bash
sudo wget -O nagios-plugins.tar.gz $(wget -q -O - https://api.github.com/repos/nagios-plugins/nagios-plugins/releases/latest | grep '"browser_download_url":' | grep -o 'https://[^"]*')
```

Check:

```bash
ls -lh nagios-plugins.tar.gz
```

---

# STEP 18 — Extract Nagios Plugins

Run:

```bash
sudo tar zxf nagios-plugins.tar.gz
```

Check:

```bash
ls -d /tmp/nagios-plugins-*
```

Enter the directory:

```bash
cd /tmp/nagios-plugins-*
```

---

# STEP 19 — Configure Nagios Plugins

Run:

```bash
sudo ./configure
```

Then compile:

```bash
sudo make
```

Then install:

```bash
sudo make install
```

Plugins should now be installed under:

```text
/usr/local/nagios/libexec/
```

Check:

```bash
sudo ls -l /usr/local/nagios/libexec/
```

You should see plugins such as:

```text
check_ping
check_http
check_ssh
check_load
check_disk
check_users
check_procs
```

---

# STEP 20 — Verify Nagios Configuration

Do not start Nagios yet.

First check the configuration:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

At the bottom you want:

```text
Total Warnings: 0
Total Errors:   0
```

Most importantly:

```text
Things look okay - No serious problems were detected during the pre-flight check
```

If you get errors, stop here.

---

# STEP 21 — Enable Nagios Service

Run:

```bash
sudo systemctl enable nagios
```

Then:

```bash
sudo systemctl start nagios
```

Check:

```bash
sudo systemctl status nagios
```

You want:

```text
Active: active (running)
```

Press:

```text
q
```

to exit.

---

# STEP 22 — Restart Apache

Run:

```bash
sudo systemctl restart apache2
```

Check:

```bash
sudo systemctl status apache2
```

You want:

```text
Active: active (running)
```

---

# STEP 23 — Check Nagios Service

Run:

```bash
sudo systemctl status nagios
```

You want:

```text
Active: active (running)
```

Also check:

```bash
sudo systemctl is-enabled nagios
```

Expected:

```text
enabled
```

---

# STEP 24 — Configure Ubuntu Firewall

If UFW is being used, first check:

```bash
sudo ufw status
```

If it says:

```text
Status: inactive
```

you don't need to do anything for UFW.

If UFW is active, allow Apache:

```bash
sudo ufw allow Apache
```

You can also allow HTTP port 80:

```bash
sudo ufw allow 80/tcp
```

If you also use SSH:

```bash
sudo ufw allow 22/tcp
```

Then enable UFW:

```bash
sudo ufw enable
```

Check:

```bash
sudo ufw status
```

You should see something similar to:

```text
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
```

> **Important:** Do not enable UFW until SSH access is allowed, especially if you are connected to the Ubuntu VM through SSH. Otherwise, you can lock yourself out.

---

# STEP 25 — Find Your Server IP

Run:

```bash
hostname -I
```

Suppose it returns:

```text
192.168.10.50
```

From your Windows PC, open Chrome/Edge and enter:

```text
http://192.168.10.50/nagios
```

You should get the Nagios login.

Enter:

```text
Username: nagiosadmin
Password: your password
```

---

# STEP 26 — Verify Nagios Web Interface

The web interface should show:

```text
Home
Current Status
Reports
System
```

You should also see:

```text
Hosts
Services
Host Groups
Service Groups
```

At this point Nagios Core is installed successfully.

---

# STEP 27 — Check Nagios from Terminal

Check the Nagios service:

```bash
sudo systemctl status nagios
```

Check Apache:

```bash
sudo systemctl status apache2
```

Check the Nagios process:

```bash
ps aux | grep nagios
```

Check installed plugins:

```bash
ls /usr/local/nagios/libexec/
```

Check Nagios configuration:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

---

# STEP 28 — Important Nagios Directories

| Directory                        | Purpose                  |
| -------------------------------- | ------------------------ |
| `/usr/local/nagios/`             | Main Nagios installation |
| `/usr/local/nagios/bin/`         | Nagios executable        |
| `/usr/local/nagios/etc/`         | Configuration            |
| `/usr/local/nagios/etc/objects/` | Host/service definitions |
| `/usr/local/nagios/libexec/`     | Monitoring plugins       |
| `/usr/local/nagios/share/`       | Web interface            |
| `/usr/local/nagios/var/`         | Logs and runtime data    |

The most important directory is:

```text
/usr/local/nagios/etc/
```

---

# Installation Complete

Your basic Nagios environment is now:

```text
                    NAGIOS SERVER
                         │
          ┌──────────────┼──────────────┐
          │              │              │
      Nagios Core      Apache          PHP
          │
    Nagios Plugins
          │
    Monitoring Checks
```

The next phase is adding your infrastructure:

```text
                    NAGIOS
                      │
        ┌─────────────┼─────────────┐
        │             │             │
     Servers       Network      Infrastructure
        │             │             │
   ┌────┴────┐    ┌───┴────┐    ┌───┴────┐
   │         │    │        │    │        │
Windows    Linux Switches Routers UPS   Printers
   │         │
 NCPA       NRPE
```

For network devices, SNMP can be used to monitor:

```text
Sophos Firewall
Core Switch
Access Switches
Wi-Fi
SD-WAN
```

For example:

```text
Nagios
   │
   │ SNMP
   ▼
Sophos Firewall
   │
   ├── CPU
   ├── Memory
   ├── Interfaces
   ├── Traffic
   └── Uptime
```

The first goal is to confirm that:

```text
http://SERVER-IP/nagios
```

opens successfully and that the Nagios web interface is working.
