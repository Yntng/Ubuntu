# Install Nagios on Ubuntu 24.04

## What We Are Going to Build

```text
Ubuntu Server 24.04
        │
        ├── Nagios Core
        │       └── Monitoring Engine
        │
        ├── Nagios Plugins
        │       └── check_ping, check_http, etc.
        │
        └── Remote Hosts
                └── NRPE + Nagios Plugins
```

---

# Step 1: Prepare the Server

## 1. Set the Server IP Address

Before installing Nagios, configure the Ubuntu server with a **static IP address**.

First, check the network interface name:

```bash
ip a
```

Example:

```text
2: ens18:
    inet 172.28.7.12/24
```

Check the Netplan configuration:

```bash
ls /etc/netplan/
```

Open the Netplan configuration file:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Example configuration:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 172.28.7.12/24
      routes:
        - to: default
          via: 172.28.7.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

> Replace `ens18`, IP address, subnet, gateway, and DNS servers with the values appropriate for your network.

Apply the configuration:

```bash
sudo netplan apply
```

Verify the IP address:

```bash
ip a
```

Verify connectivity:

```bash
ping -c 4 8.8.8.8
```

---

## 2. Set the Timezone to Asia/Kolkata

Check the current timezone:

```bash
timedatectl
```

Set the timezone:

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

Verify:

```bash
timedatectl
```

The timezone should show:

```text
Time zone: Asia/Kolkata (IST, +0530)
```

---

## 3. Enable Time Synchronization

Enable NTP time synchronization:

```bash
sudo timedatectl set-ntp true
```

Check the synchronization status:

```bash
timedatectl
```

Look for:

```text
System clock synchronized: yes
NTP service: active
```

You can also check the current date and time:

```bash
date
```

> Correct time synchronization is important for monitoring, logs, alerts, certificates, and communication between the Nagios server and monitored hosts.

---

## 4. Verify IP Address, Date, Time and Timezone

Run:

```bash
ip a
```

Check the server IP address.

Then:

```bash
timedatectl
```

Confirm:

* Correct date
* Correct time
* `Asia/Kolkata` timezone
* System clock synchronized
* NTP service active

---

## 5. Update Ubuntu

Once the IP address, timezone, and time synchronization are correctly configured:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install Required Software

Install the necessary packages:

```bash
sudo apt install -y autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php
```

---

# Step 3: Download and Install Nagios Core

## 1. Go to Your Home Directory

```bash
cd ~
```

## 2. Create a New Folder

```bash
mkdir nagios-core && cd nagios-core
```

## 3. Download Nagios Core

```bash
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/refs/tags/nagios-4.5.14.tar.gz
```

## 4. Extract the File

```bash
tar xzf nagioscore.tar.gz
```

## 5. Go to the Nagios Folder

```bash
cd nagioscore-nagios-4.5.14/
```

## 6. Configure Nagios

```bash
sudo ./configure --with-httpd-conf=/etc/apache2/sites-enabled
```

## 7. Compile and Install

```bash
sudo make all
```

```bash
sudo make install-groups-users
```

```bash
sudo usermod -a -G nagios www-data
```

---

# Step 4: Install Nagios Components

Run:

```bash
sudo make install
```

```bash
sudo make install-daemoninit
```

```bash
sudo make install-commandmode
```

```bash
sudo make install-config
```

```bash
sudo make install-webconf
```

Enable the required Apache modules:

```bash
sudo a2enmod rewrite
```

```bash
sudo a2enmod cgi
```

## Create an Admin User for the Nagios Dashboard

```bash
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

Enter and confirm the password when prompted.

## Restart Apache

```bash
sudo systemctl restart apache2
```

---

# Step 5: Install Nagios Plugins

## 1. Install Required Packages

```bash
sudo apt install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc
```

## 2. Download Nagios Plugins

```bash
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/releases/download/release-2.5.0/nagios-plugins-2.5.0.tar.gz
```

## 3. Extract the File

```bash
tar zxf nagios-plugins.tar.gz
```

Go to the extracted directory:

```bash
cd nagios-plugins-release-2.5.0
```

## 4. Install the Plugins

```bash
sudo ./tools/setup
```

```bash
sudo ./configure
```

```bash
sudo make
```

```bash
sudo make install
```

---

# Step 6: Start and Enable Nagios

## 1. Check for Configuration Errors

Before starting Nagios, validate the configuration:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

The configuration should finish without errors.

## 2. Enable Nagios to Start Automatically

```bash
sudo systemctl enable nagios
```

## 3. Start Nagios

```bash
sudo systemctl start nagios
```

Check the service:

```bash
sudo systemctl status nagios
```

---

# Step 7: Access Nagios Web Dashboard

## 1. Allow SSH Through the Firewall

```bash
sudo ufw allow ssh
```

## 2. Allow HTTP

```bash
sudo ufw allow 80/tcp
```

## 3. Reload the Firewall

```bash
sudo ufw reload
```

## 4. Open Nagios in a Web Browser

```text
http://SERVER-IP/nagios
```

Example:

```text
http://172.28.7.12/nagios
```

## 5. Login

**Username:**

```text
nagiosadmin
```

**Password:**

```text
The password you created earlier
```

---

# Step 8: Install Nagios on Remote Host

## 1. SSH Into the Remote Host

```bash
ssh user@remote-host-ip
```

## 2. Install NRPE and Plugins

```bash
sudo apt install nagios-plugins nagios-nrpe-server -y
```

## 3. Enable NRPE Service

```bash
sudo systemctl enable --now nagios-nrpe-server
```

## 4. Edit NRPE Configuration

```bash
sudo nano /etc/nagios/nrpe.cfg
```

Add the Nagios server IP under `allowed_hosts`:

```ini
allowed_hosts=127.0.0.1,::1,NAGIOS_SERVER_IP
```

Replace:

```text
NAGIOS_SERVER_IP
```

with the actual IP address of the Nagios server.

## 5. Restart NRPE

```bash
sudo systemctl restart nagios-nrpe-server
```

---

# Step 9: Configure Nagios to Monitor Remote Host

## 1. SSH Into the Nagios Server

```bash
ssh user@nagios-server-ip
```

## 2. Create the Servers Directory

```bash
sudo mkdir -p /usr/local/nagios/etc/servers/
```

## 3. Create a Configuration File

```bash
sudo nano /usr/local/nagios/etc/servers/remotehost.cfg
```

Add:

```text
define host {
    use                     linux-server
    host_name               RemoteHost
    alias                   First Remote Host
    address                 remote-host-ip
    max_check_attempts      5
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}
```

Replace:

```text
remote-host-ip
```

with the actual IP address of the remote host.

## 4. Add Parent Host When Required

If the remote host is connected through another monitored host, you can define the parent relationship:

```text
define host {
    use                     linux-server
    host_name               RemoteHost
    alias                   First Remote Host
    parents                 ParentHost
    address                 remote-host-ip
    max_check_attempts      5
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}
```

> `parents` must contain the name of an already defined Nagios host. Use the correct directive syntax: `parents`, not `Parents`.

## 5. Edit the Nagios Main Configuration File

```bash
sudo nano /usr/local/nagios/etc/nagios.cfg
```

Add the following line:

```text
cfg_dir=/usr/local/nagios/etc/servers
```

This tells Nagios to read host and service configuration files from:

```text
/usr/local/nagios/etc/servers
```

## 6. Validate the Configuration

Always check the configuration before restarting Nagios:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

If there are no errors, restart Nagios:

```bash
sudo systemctl restart nagios
```

Check the service:

```bash
sudo systemctl status nagios
```

## 7. Check the Nagios Dashboard

Open:

```text
http://SERVER-IP/nagios
```

The remote host should now appear in the Nagios dashboard.

---

# Configuration Verification

Whenever you make changes to Nagios configuration files, run:

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

This checks the configuration before Nagios uses it.

If the validation reports errors, fix them before restarting the Nagios service.

---

# Useful Nagios Commands

## Check Nagios Configuration

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

## Start Nagios

```bash
sudo systemctl start nagios
```

## Stop Nagios

```bash
sudo systemctl stop nagios
```

## Restart Nagios

```bash
sudo systemctl restart nagios
```

## Check Nagios Status

```bash
sudo systemctl status nagios
```

## Enable Nagios at Boot

```bash
sudo systemctl enable nagios
```

## Check Nagios Version

```bash
/usr/local/nagios/bin/nagios -v
```

---

# Conclusion

Nagios is now installed on Ubuntu 24.04 and configured to monitor remote hosts.

To add additional remote hosts:

1. Install NRPE and Nagios plugins on the remote host.
2. Configure `allowed_hosts`.
3. Create a host configuration file on the Nagios server.
4. Make sure the `servers` directory is included in `nagios.cfg`.
5. Validate the configuration.
6. Restart Nagios.
7. Verify the host from the Nagios dashboard.
