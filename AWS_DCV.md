# Setting up AWS DCV for Ubuntu 22.04
Doc: https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-installing-linux-prereq.html

# Prerequisites

# Install a desktop environment and desktop manager
- time sudo apt update
## Setting up AWS NICE DCV on Ubuntu 22.04

This document collects commands and steps to install and configure NICE DCV on Ubuntu 22.04 (desktop + DCV server). It follows the AWS/NICE documentation and includes copyable command blocks.

Reference:

- AWS DCV prerequisites: https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-installing-linux-prereq.html

---

## Prerequisites

- An Ubuntu 22.04 instance (with NVIDIA drivers installed if you want hardware acceleration).
- Sudo/administrator access.

Tip: Run the following to update package lists before starting:

```sh
sudo apt update
```

## 1) Install a desktop environment and display manager

Install the Ubuntu desktop and GDM3 (GNOME display manager):

```sh
sudo apt install -y ubuntu-desktop gdm3
sudo apt upgrade -y
```

After installation, you may want to reboot:

```sh
sudo reboot
```

Verify GDM3 is the default display manager:

```sh
cat /etc/X11/default-display-manager
# expected output: /usr/sbin/gdm3
```

## 2) Disable Wayland for GDM (recommended for some GPU setups)

Edit the GDM config and disable Wayland:

```sh
sudo nano /etc/gdm3/custom.conf
# In the [daemon] section set:
# WaylandEnable=false
# Save and exit (Ctrl+O, Enter, Ctrl+X)

sudo systemctl restart gdm3
```

## 3) Ensure the system boots to the graphical target

```sh
sudo systemctl set-default graphical.target
sudo systemctl isolate graphical.target
```

Confirm X server is running:

```sh
ps aux | grep X | grep -v grep
```

## 4) Install OpenGL utilities (glxinfo)

```sh
sudo apt install -y mesa-utils
```

To check OpenGL software rendering (example):

```sh
sudo DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') \ 
  glxinfo | grep -i "opengl.*version"
```

If you have NVIDIA drivers and want to generate/update an X configuration:

```sh
sudo nvidia-xconfig --preserve-busid --enable-all-gpus

# Restart X if needed (switch to multi-user then back)
sudo systemctl isolate multi-user.target
sudo systemctl isolate graphical.target
```

Check hardware rendering again (same pattern as above):

```sh
sudo DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') \ 
  glxinfo | grep -i "opengl.*version"
```

## 5) Install the NICE DCV Server

Download and import the NICE GPG key, then download and extract the server package.

```sh
wget https://d1uj6qtbmh3dt5.cloudfront.net/NICE-GPG-KEY
gpg --import NICE-GPG-KEY

wget https://d1uj6qtbmh3dt5.cloudfront.net/2024.0/Servers/nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz
tar -xvzf nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz
cd nice-dcv-2024.0-19030-ubuntu2204-x86_64
```

Install the DCV server and web viewer packages (adjust filenames if versions differ):

```sh
sudo apt install -y ./nice-dcv-server_2024.0.19030-1_amd64.ubuntu2204.deb
sudo apt install -y ./nice-dcv-web-viewer_2024.0.19030-1_amd64.ubuntu2204.deb
```

Add the `dcv` user to the `video` group so it can access GPU devices:

```sh
sudo usermod -aG video dcv
```

## 6) Post-installation checks

Start and enable the DCV server service:

```sh
sudo systemctl start dcvserver
sudo systemctl enable dcvserver
sudo systemctl status dcvserver
```

Verify the `dcv` user can access the X server (example check):

```sh
sudo DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') \ 
  xhost | grep "SI:localuser:dcv$"
```

Run the DCV GL diagnostic (optional):

```sh
sudo dcvgldiag
# The command reports GL status. If it returns without critical errors you can proceed.
```

## 7) Create and list DCV sessions

Create a console session:

```sh
sudo dcv create-session isaac-session --type=console --owner ubuntu

# List sessions
dcv list-sessions
```

## ✅ Step-by-Step : Auto-Create DCV Session on Boot

### 1. Create a systemd service for auto-starting your session

Run:

```bash
sudo nano /etc/systemd/system/dcv-autosession.service
```

Paste this content:

```ini
[Unit]
Description=Auto-create AWS DCV console session
After=network-online.target dcvserver.service
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/dcv-autosession.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Save and exit.

---

### 2. Create the session script

Now create the script that actually creates the session:

```bash
sudo nano /usr/local/bin/dcv-autosession.sh
```

Paste this:

```bash
#!/bin/bash

SESSION_NAME="isaac-session"
OWNER="ubuntu"

# Wait for DCV server to be ready
sleep 10

# Check if session already exists
if ! sudo dcv list-sessions | grep -q "$SESSION_NAME"; then
  echo "Creating DCV session: $SESSION_NAME"
  sudo dcv create-session "$SESSION_NAME" --type=console --owner "$OWNER"
else
  echo "DCV session $SESSION_NAME already exists."
fi
```

Save and make it executable:

```bash
sudo chmod +x /usr/local/bin/dcv-autosession.sh
```

---

### 3. Enable the service

Run:

```bash
sudo systemctl enable dcv-autosession.service
```

This makes it run automatically on every reboot.

---

### 4. Reboot and verify

After reboot, check:

```bash
sudo systemctl status dcv-autosession
sudo dcv list-sessions
```

You should see a session named `isaac-session` created automatically.

---

---

## Notes and troubleshooting

- Replace package filenames/versions with the latest NICE DCV release you downloaded.
- If DCV sessions fail to start, check `sudo journalctl -u dcvserver` for logs.
- Ensure NVIDIA drivers and CUDA (if used) are correctly installed for hardware acceleration.

If you'd like, I can also:

- add a quick systemd unit file into this repo as an example,
- or create a short troubleshooting checklist for common errors.








