# Setting up AWS DCV for Ubuntu 22.04
Doc: https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-installing-linux-prereq.html

# Prerequisites

# Install a desktop environment and desktop manager
- time sudo apt update
- time sudo apt install ubuntu-desktop
- sudo apt install gdm3
  # Verify GDM3 is set as default
  - cat /etc/X11/default-display-manager
    output: /usr/sbin/gdm3
- sudo apt upgrade
- sudo reboot
- 
# Disable the Wayland protocol (GDM3 only)
- sudo nano /etc/gdm3/custom.conf
  - [daemon]
    WaylandEnable=false
  - ctl+o, enter, ctl+x]
- sudo systemctl restart gdm3

# Configure the X Server
- sudo systemctl get-default (graphical.targe)
- sudo systemctl set-default graphical.target
- sudo systemctl isolate graphical.target
- ps aux | grep X | grep -v grep

# Install the glxinfo utility
- sudo apt install mesa-utils

# verify that OpenGL software rendering is available
- sudo DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*\-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') glxinfo | grep -i "opengl.*version"

# generate an updated xorg.conf
-  sudo nvidia-xconfig --preserve-busid --enable-all-gpus

# Restart the X server
- sudo systemctl isolate multi-user.target
- sudo systemctl isolate graphical.target

# verify that OpenGL hardware rendering is available
-   sudo DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*\-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') glxinfo | grep -i "opengl.*version"



# Install the Amazon DCV Server
wget https://d1uj6qtbmh3dt5.cloudfront.net/NICE-GPG-KEY
gpg --import NICE-GPG-KEY

wget https://d1uj6qtbmh3dt5.cloudfront.net/2024.0/Servers/nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz
tar -xvzf nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz && cd nice-dcv-2024.0-19030-ubuntu2204-x86_64

sudo apt install ./nice-dcv-server_2024.0.19030-1_amd64.ubuntu2204.deb

sudo apt install ./nice-dcv-web-viewer_2024.0.19030-1_amd64.ubuntu2204.deb


sudo usermod -aG video dcv


# Post-installation checks

# verify that the dcv user can access the X server
- for errors
sudo systemctl isolate multi-user.target
sudo systemctl isolate graphical.target

sudo DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*\-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') xhost | grep "SI:localuser:dcv$"

# DCV GL is properly installed
sudo dcvgldiag (it's okay to move forward)

# create session
sudo systemctl status dcvserver
sudo systemctl start dcvserver
sudo systemctl enable dcvserver


sudo dcv create-session isaac-session --type=console --owner ubuntu
dcv list-sessions

# Auto-Create DCV Session on Boot








