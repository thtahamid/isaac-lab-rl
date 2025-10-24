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




