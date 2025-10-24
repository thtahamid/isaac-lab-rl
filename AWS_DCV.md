# Setting up AWS 
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


# Disable the Wayland protocol (GDM3 only)

- sudo nano /etc/gdm3/custom.conf
  - [daemon]
    WaylandEnable=false
  - ctl+o, enter, ctl+x]
- sudo systemctl restart gdm3

# Configure the X Server
- sudo systemctl get-default
