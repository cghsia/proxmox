# proxmox
Just some Proxmox notes.

## Proxmox 8 and Realtek RTL8168 NIC
If you happen to have a computer with a Realtek RTL8168 network interface card and have been running Proxmox 7 for some time and planning to upgrade to Proxmox 8, you may have noticed discussions regarding Proxmox 8 issues with this NIC. Of particular note, Proxmox 7 used the RTL8169 driver for RTL8168, which did work for the 5.x kernel. However, with Proxmox 8 and the 6.2 kernel, there (at time of writing this in November 2023) isn't yet driver from Realtek that can be used. As such, many have suggested installing the `r8168-dkms` package and there is varied discussion on how to achieve this.

While every situation will vary, the steps documented below may work, but your mileage may vary. This worked for me on a used Dell Wyse 5070 thin client with an RTL8168 NIC.

### Step 1: Upgrade to Proxmox 8
Pre-flight check:
* Back up all VMs and LXCs, validate they are working.
* Make sure there's at least 5GB of free disk space on the Proxmox host. Use `df -h` to verify
* Make sure you're running at least Proxmox VE 7.4.13 or later. Run tteck's Proxmox VE post-install script to update Proxmox to the latest 7.4.x version available: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"`

In the Proxmox host's shell:
1. Run `pve7to8 --full`
Make note of any WARNINGS and resolve if possible
2. Automate Proxmox 7.4.x to 8.0 upgrade with tteck's Proxmox 8 upgrade script: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/pve8-upgrade.sh)"`<br><br>
    > Skip to Step 2 if you run this automatic script.<br><br>
    > Otherwise, continue onwards manually...
3. Stop all containers (VM, LXC, Docker, etc)
4. Run `sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list`
5. Run `cat /etc/apt/sources.list.d/pve-enterprise.list`
6. Run `cat /etc/apt/sources.list`
7. Run `apt update`
8. Run `apt dist-upgrade`
9. Press `Q`
10. If there are any `/etc/issue` errors, press `N` to overwrite the older versions
11. Restart
12. Press `N` and press the `Enter` key at pve-enterprise.list changes
13. Run `pve7to8` to check on any post-install WARNINGS and other statuses
14. Run `reboot`

### Step 2: Install `r8168-dkms`
1. Run `nano /etc/apt/sources.list`
2. Add in the following lines to the bottom:\
`deb http://ftp.debian.org/debian bookworm main contrib non-free non-free-firmware`<br>
`deb http://ftp.debian.org/debian bookworm-updates main contrib non-free non-free-firmware`
3. Save and exit
4. Run `apt update`
5. Run `apt dist-upgrade`
6. Run `apt install pve-headers`
7. Run `apt install r8168-dkms`
8. Run `echo blacklist r8169 >> /etc/modprobe.d/blacklist-r8169.conf`
9. Reboot
10. Run `ethtool -i enp1s0`
11. Verify r8168 driver is being used in the output (typically the very last line)
