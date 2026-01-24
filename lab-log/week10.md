# Week 10 - Linux Startup Processes and Services

Linux uses systemd to control how services start, stop, reload, and recover, and by understanding service units and the related management commands, you can keep essential applications running smoothly and your system stable.

## systemd (Linux's Init System)

When Linux starts, several important applications launch automatically. If one service depends on another (e.g., app2 depends on app1), systemd ensures app1 starts first. Even better, if a critical service crashes, systemd can automatically restart it to keep the system stable.

Systemd reads configuration files called unit files, which tell it how to:

- Start services
- Handle crashes
- Reload configurations
- Restart applications

To explore all available options for service units, run `man systemd.service`. This manual page explains how `.service` files are structured and details all of their configuration options.

## Managing the SSH Daemon

SSH is essential for remote access, so it’s a great example of how systemd manages a service. 

```bash
# To view the SSH service unit:
systemctl cat sshd.service

# And what you might see:
[Unit]
Description=OpenSSH server daemon
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

[Service]
Type=notify
ExecStart=/usr/sbin/sshd -D $OPTIONS $CRYPTO_POLICY
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target

#   ExecStart — the command systemd uses to start the SSH daemon
#   ExecReload — how to reload the SSH configuration
#   Restart=on-failure — automatically restart SSH if it crashes

```

```bash
# To edit the service file
sudo systemctl edit --full sshd.service

# To revert your changes
sudo systemctl revert sshd.service

# To check the service status
sudo systemctl status sshd.service
```

## Controlling Service States

Systemd provides a consistent set of commands for managing services:
```bash
# Check status and recent logs
sudo systemctl status sshd.service

# Start the service
sudo systemctl start sshd.service

# Stop the service
sudo systemctl stop sshd.service

# Restart the service (stop + start)
sudo systemctl restart sshd.service

# Reload configuration without restarting
sudo systemctl reload sshd.service
```

## Enabling and Disabling Services

These commands control whether a service starts automatically at boot
```bash
# Disable a service (won’t start at boot)
sudo systemctl disable sshd.service

# Check if a service is enabled
sudo systemctl is-enabled sshd.service

# Enable a service (starts at boot)
sudo systemctl enable sshd.service
```

```bash
# You can also use the --now flag to enable or disable a service and apply the change immediately.
sudo systemctl enable --now sshd.service
sudo systemctl disable --now sshd.service
```

## Masking Services
Disabling a service doesn’t always prevent it from being started by another component. Masking ensures the service cannot be started at all.

```bash
# Mask a service
sudo systemctl mask atd.service

# To unmask
sudo systemctl unmask atd.service
```

## Listing All Service Units
Service names aren’t always intuitive (e.g., Apache = httpd.service).

```bash
# To list all service units:
sudo systemctl list-units --type service --all
```
This output shows whether each service is loaded, active, running, and includes a brief description, which is especially helpful when you’re not sure what a service is called.





