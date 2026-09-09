# Installing Ubuntu Server: A Step-by-Step Guide

> Adapted from the official Ubuntu tutorial: [Install Ubuntu Server](https://ubuntu.com/tutorials/install-ubuntu-server)

Ubuntu Server is a variant of the standard Ubuntu you already know, tailored for networks and services. Unlike Ubuntu Desktop, it uses a text-based, menu-driven installer rather than a graphical one. This guide walks through the full installation process, step by step, with screenshots and explanations for each stage.

---

## 1. Overview

Ubuntu Server shares the same underlying system as Ubuntu Desktop but is optimized for running services rather than a desktop environment. Because there's no graphical interface during setup, the installer uses simple text menus that you navigate with the keyboard.

**Related resources:**
- [Install Ubuntu Desktop tutorial](https://ubuntu.com/tutorials/install-ubuntu-desktop)
- [Ubuntu Server community documentation](https://ubuntu.com/server/docs)

---

## 2. Requirements

Before you begin, make sure you have:

- At least **2 GB of free storage space** on the target machine
- A **DVD or USB flash drive** with the Ubuntu Server image written to it
- A **recent backup** of any existing data, if you're installing alongside data you want to keep

---

## 3. Boot from the Install Media

Insert your installation media (USB or DVD) and restart the computer. Most machines will automatically boot from the inserted media, but some require you to manually open a boot menu and select the drive — commonly done by pressing `Escape`, `F2`, `F10`, or `F12` during startup (the exact key depends on your hardware).

![Boot from install media](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/1/17ee449b2bd7c530d2f996215407fca5b722dcb2.png)

---

## 4. Choose Your Language

Once the installer loads, you'll be presented with a language selection menu. Use the `Up` and `Down` arrow keys to navigate, and press `Enter` to confirm your choice.

![Choose your language](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/e/e1d75e3584b6a3c23da39263fbf2f9ba6411de9a.png)

---

## 5. Choose the Correct Keyboard Layout

Next, select your keyboard layout and variant. If you're unsure which to pick, the tutorial recommends going with the suggested defaults — the layout can always be adjusted later after installation is complete.

![Keyboard layout selection](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/5/5c918dc341d92f647d6f1665ed2714922d5e688c.png)

![Keyboard variant selection](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/1/150b56850da9ca443d2a4842104ada1c1dfc264c.png)

---

## 6. Choose Your Install Type

You'll now see three menu options. The first, **"Install Ubuntu,"** is the standard choice for most users. The remaining two options are for **Metal As A Service (MAAS)** deployments, used in larger, automated data-center environments.

![Choose install type](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/7/7e92f38c56ca7b2fbb8323d51d6ee35457fe4fce.png)

---

## 7. Networking

The installer automatically detects and attempts to configure any available network connections via DHCP. This screen is purely informational — you don't need to do anything here, and installation can proceed even without network connectivity (you can configure networking manually afterward).

![Networking configuration](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/f/f3e24675ada8dec595905d3853766afdbe1a9039.png)

---

## 8. Configure Storage

The recommended approach is to dedicate an entire disk or partition to the installation. A manual option is also available for more complex partitioning setups. Note that modern Ubuntu no longer requires a separate swap partition — swap is now handled differently by default.

![Configure storage](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/4/48e6067fea81202b132da004d467ec9df20ecf4f.png)

---

## 9. Select a Device

Choose the target disk from the list of detected drives, identified by their system IDs. Use the arrow keys to move between options and `Enter` to select.

![Select a device](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/5/5be59d8fb261e3f4ed175d11fb660248c9215823.png)

---

## 10. Confirm Partitions

The installer displays the partition layout it has calculated for the selected disk. You can go back to choose a different drive, manually edit the partitions, or select **"Done"** to accept the layout and continue.

![Confirm partitions](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/5/5a1d73b678c2cdbd0a75225d0a583eb156e1b953.png)

---

## 11. Confirm Changes

This is the final confirmation before the installer makes destructive changes to the disk.

> ⚠️ **Warning:** There is no "Undo" for this step. The contents of the selected device may be permanently lost. Make sure you have backed up anything important before continuing.

![Confirm changes](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/c/c33fb0d6b4b9bfcb982eda42b21ced5187c38e7b.png)

---

## 12. Set Up a Profile

Create your system profile, which requires at minimum:

- One system **user**
- A **hostname** for the machine
- A **password**

You also have the option to import an SSH key from Launchpad, Ubuntu One, or GitHub, which lets you log in securely right after installation without setting up SSH separately.

![Set up a profile](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/2/22deb8bda1b7df8dc94a19f6a23bb07c5b7664b0.png)

---

## 13. Install Software

The installer now installs "a concise set of useful software required for servers." This keeps the base installation small and fast, while additional packages can be installed later as needed.

![Install software](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/4/4108bb9cbe9a52f5b39d266e78e0cf2668f58d80.png)

---

## 14. Installation Complete

Once installation finishes, a completion message appears. Remove the installation media (USB/DVD), then press `Enter` to reboot the machine and start your new Ubuntu Server installation.

![Installation complete](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/c/c36b4880b5e4291ca57e6a51b527c05449fec48e.png)

---

## 15. What Next?

Now that Ubuntu Server is installed, here's where to go for more help and information:

- **[Ubuntu Server Guide](https://ubuntu.com/server/docs)** — the official documentation covering everyday server administration tasks.
- **[Ubuntu Server pages](https://ubuntu.com/server)** — general information about Ubuntu Server, its features, and releases.
- **Community support:**
  - [Ubuntu Discourse](https://discourse.ubuntu.com/)
  - [Ask Ubuntu](https://askubuntu.com/)
  - IRC channels (e.g., `#ubuntu-server` on Libera.Chat)
- **Commercial support** is also available through **Ubuntu Advantage** for organizations that need SLA-backed assistance.

---

### Source

Full original tutorial: [https://ubuntu.com/tutorials/install-ubuntu-server](https://ubuntu.com/tutorials/install-ubuntu-server)
