# Week 15 - Kernel Runtime Parameters

Kernel runtime parameters are simply live settings inside the Linux kernel that control how the system behaves while it’s running. In other words, they control how the Linux kernel manages core functions such as memory allocation, network traffic, and file system operations.

## Viewing Current Kernel Parameters

You can view all active kernel runtime parameters with `sudo sysctl -a`. The output might look something like this, showing a long list of kernel parameters along with their current values:

```bash
sudo sysctl -a

net.ipv6.conf.default.addr_gen_mode = 0
net.ipv6.conf.default.autoconf = 1
net.ipv6.conf.default.dad_transmits = 1
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.default.disable_policy = 0
vm.admin_reserve_kbytes = 8192

```

> Notice how these parameters follow a naming convention where networking parameters start with “net”, memory settings with “vm”, and file system configurations with “fs”.

## Editing Kernel Parameters

Take an example from above— the parameter `net.ipv6.conf.default.disable_ipv6` controls whether IPv6 is enabled for the system’s default network settings. A value of `0` means IPv6 is turned on, while a value of `1` disables it. 

To change this setting temporarily (so it only lasts until the next reboot) you can write a new value using the `-w` flag. Just make sure there are no spaces around the equals sign when setting the parameter.


```bash
# Changing the Kernel Parameter
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1

# Checking it
sudo sysctl net.ipv6.conf.default.disable_ipv6

```

## Making Changes Persistent

To make your changes persistent, create a `.conf` file in `/etc/sysctl.d/`. The kernel reads all files in this directory at boot.

```bash
# Example: adjusting `vm.swappiness`, which controls how aggressively the system uses swap (0–100; higher = swap sooner).

# Check the current value of memory-related parameters
sysctl -a | grep vm

# Create a config file:
sudo vim /etc/sysctl.d/swap-less.conf

# Add:
vm.swappiness=29

# Reload immediately without rebooting:
sudo sysctl -p /etc/sysctl.d/swap-less.conf

# This applies the new value now and ensures it persists after reboot.
```
