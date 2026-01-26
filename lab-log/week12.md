# Week 12 – Basic Network Configuration

This week I spent some time brushing up on the core networking skills you need on a Linux system. So, this weeks lab log includes checking network info, using nmtui and nmcli, managing network services, setting up static routes, handling packet filtering, and even working with time service clients. Lets get into it!

## Some helpful commands to get started

Before we start changing any settings, let’s look at how we can query the system for more information! 
```bash
ip link show             # List all network interfaces
ip address show          # List IP addresses assigned to those interfaces
ip route show            # View the routing table

cat /etc/resolv.conf     # Check the DNS resolver
sudo vum /etc/hosts      # View (or add) static hostname mappings

```

## Managing configuration files with `nmtui` and `nmcli`

Network configuration files are the system’s stored settings that define how a Linux machine’s network interfaces behave, including IP addresses, DNS settings, routes, and connection parameters. 

On Red Hat–based distributions, these files are stored in the `/etc/sysconfig/network-scripts/` directory. 

```bash
# To list the active interface configuration files
ls /etc/sysconfig/network-scripts/

# To view the contents
cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

NetworkManager is the default network management tool on most Red Hat–based systems, offering both `nmtui` and `nmcli`.

`nmtui` is a simple, text‑based interface that lets you manage network settings without touching configuration files directly. You can use it to edit connection profiles, set static or dynamic IP addresses, manage DNS settings, activate or deactivate interfaces, and even change the system hostname, all through an easy, menu‑driven UI.

```bash
# Launch nmtui by executing:
sudo nmtui

# To apply the changes immediately, use nmcli:
sudo nmcli device reapply enp0s3
```

## Configure Network Services to Start Automatically at Boot

Before a Linux system can get an IP address, gateway, routes, or DNS settings, the network services need to start correctly during boot. On Red Hat–based systems, NetworkManager is the main service responsible for managing network connectivity, and if it doesn’t start automatically, your network configuration won’t come up properly.

```bash
# Confirm that NetworkManager is running and enabled at boot:
systemctl status NetworkManager.service

# Ensuring a Network Connection Autoconnects at Boot
nmcli connection show 
sudo nmcli connection modify enp0s3 autoconnect yes

```

## Inspecting Listening Services with `ss`

You can also which programs are actively listening for network connections. The modern tool for this is `ss`, which replaces the older `netstat`.

```bash
sudo ss -ltunp  # A simple mnemonic: l‑t‑u‑n‑p → listening, TCP, UDP, numeric, process.
```

This displays all services currently listening on the system. For example, `sshd` on port 22 and `chronyd` on port 323. It also tells you whether they accept only local connections (`127.0.0.1`) or external ones (`0.0.0.0`). Once you have the PID, you can dig deeper into the process using tools like `ps` or `lsof`.

You might remember from last week that we used `sudo systemctl stop chronyd.service` to stop a service. `ss` is useful alongside that because after stopping a service, you can verify that it’s no longer listening on its port by running `sudo ss -ltunp` on top of `sudo systemctl status chronyd.service`.

## Packet Filtering

Packet filtering controls which network packets are allowed into or out of a Linux system, helping protect against unwanted or malicious traffic. On Red Hat–based systems, FirewallD handles this using zones, and each zone defines what traffic is allowed for the interfaces assigned to it.

#### FirewallD Basics

Each network interface is assigned to a specific FirewallD zone, and each zone has its own rules—for example, a wireless interface might be placed in the Drop zone to block all traffic, while a wired office interface might be placed in the Trusted zone to allow everything. 

Most systems default to the Public zone, which blocks all incoming connections unless explicitly allowed.

```bash
# Check the default zones
firewall-cmd --get-default-zone

# Set the default zones
firewall-cmd --set-default-zone=public

# View all rules in the active zone
sudo firewall-cmd --list-all

```

#### Opening Traffic 

FirewallD lets you open traffic in two ways, by service or by port. It’s important to choose one method, either by service or by port—to avoid conflicting rules. You can also remove a rule the same way you added it, whether you’re removing the service name or the port number.

```bash
# Allow by service name:
sudo firewall-cmd --add-service=http

# Allow by port:
sudo firewall-cmd --add-port=80/tcp

# You can also sllow traffic from a specific network range:
sudo firewall-cmd --add-source=10.11.12.0/24 --zone=trusted
```

These rules are temporary by default, so always ensure to test first, then make them permanent like so:

```bash
sudo firewall-cmd --add-port=12345/tcp
sudo firewall-cmd --add-port=12345/tcp --permanent

```

## Configuring Static Routes

Adding temporary static routes:
```bash
# Use the `ip` command to add routes for testing:
sudo ip route add 192.168.0.0/24 via 10.0.0.100
sudo ip route add 192.168.0.0/24 via 10.11.12.100 dev enp0s3 # If you need to specify an interface:

# Remove a route:
sudo ip route del 192.168.0.0/24

# Add or remove a default route:
sudo ip route add default via 10.0.0.100
sudo ip route del default via 10.0.0.100
```
Note: Routes added with `ip` are temporary and disappear after reboot — perfect for quick testing. To persist routes, configure them through NetworkManager.

```bash
# List connections:
sudo nmcli connection show
```
```
Once you identify the active connection (e.g., `enp0s3`), add the route in its settings using either:

- nmcli (after editing, reload with `sudo nmcli connection reload`)
- nmtui (navigate to the routing section and add the route interactively)

NetworkManager applies the route after a reload or reboot.
```

## Configuring Time Service Clients

Accurate system time is essential for logging, scheduling, authentication, and overall server reliability. Linux uses `Chrony` to keep the system clock synchronized with trusted network time servers.

```bash
systemctl status chronyd.service                 # Check if the Chrony service is running
timedatectl                                      # View current time settings, time zone, and NTP sync status

timedatectl list-timezones                       # List all available time zones
sudo timedatectl set-timezone America/New_York   # Set the system time zone

timedatectl                                      # Check if the system clock is synchronized
sudo systemctl set-ntp true                      # Force NTP synchronization if needed
```

