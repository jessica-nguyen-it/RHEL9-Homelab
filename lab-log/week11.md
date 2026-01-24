# Week 11 – Advanced Package Management

This week we'll work on understanding subscriptions, repositories, and how YUM pulls everything together. So, let’s dive in!
> NOTE: If your system uses **DNF** instead of YUM, you can replace `yum` with `dnf` in almost all commands.

## Registering with Red Hat Subscription Management

RHEL uses subscription-manager to verify your system and grant access to official Red Hat repositories.  
If you skip this step, YUM will complain that your system isn’t registered, so it is VERY important!

```bash
# Register your system with your Red Hat Developer credentials
sudo subscription-manager register --username your-redhat-developer-username --password your-redhat-password

# Automatically attach an available subscription
sudo subscription-manager attach --auto
```

## Software Repositories

Repositories contain packages, metadata, configuration files, and libraries. They can be hosted by Red Hat, provided internally by an organization, or added manually from the internet or a local network.

```bash
# To View Enabled Repositories:
sudo yum repolist

# To View Enabled and Disabled Repositiories:
sudo yum repolist --all

# Example output:
repo id                           repo name
rhel-8-for-x86_64-appstream-rpms  Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
rhel-8-for-x86_64-baseos-rpms     Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)

# For more details (like repo URLs):
sudo yum repolist -v
```

#### Enabling and Disabling Repositories

```bash
# To enable an optional repository, use its repo ID (as listed in `sudo yum repolist --all` output):

sudo subscription-manager repos --enable codeready-builder-for-rhel-8-x86_64-rpms
sudo subscription-manager repos --disable codeready-builder-for-rhel-8-x86_64-rpms

# You can also use yum-utils:

sudo yum-config-manager --enable codeready-builder-for-rhel-8-x86_64-rpms
sudo yum-config-manager --disable codeready-builder-for-rhel-8-x86_64-rpms

# Verify:
sudo yum repolist --all
```

#### Adding Custom Repositories

```bash
# Before adding custom repos, install yum-utils:
sudo yum install yum-utils

# To Add an internet-based repo:
sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# To Add a repo from a local network:
sudo yum-config-manager --add-repo 192.168.1.220/BaseOS.repo

# Verify:
sudo yum repolist -v
```

#### Examining a Custom Repo File

```bash
# Look in the following directory
sudo vi /etc/yum.repos.d/docker-ce.repo

# To remove a custom repo, just delete its file from `/etc/yum.repos.d/`.
```

## Managing Packages with YUM

YUM is a powerful tool that can install, update, reinstall, remove, and search for packages. You may recall that it handles dependency resolution automatically, making sure all required libraries are installed or cleaned up as needed. Lets take a deeper look into what else we can do with the tool!

#### Searching for Packages

```bash
# This searches for packages containing either word:
sudo yum search web server

# For exact phrase matching:
sudo yum search 'web server'

# Once you find a package, you can get more details using:
sudo yum info nginx
```

#### Installing, Reinstalling, and Removing Packages

```bash
# Install:
sudo yum install nginx

# Reinstall (useful if configs get messed up):
sudo yum reinstall nginx

# Remove:
sudo yum remove package_name

# Clean up leftover dependencies:
sudo yum autoremove
```

## Installing Software from a Local RPM File

A remote RPM comes from a repository hosted somewhere else, usually on the internet or your company’s internal network. In contrast, a local RPM is a package file that already exists on your machine (or a USB drive, shared folder, etc.). You download it manually, then install it directly.

```bash
# Download the RPM:
sudo wget https://download.nomachine.com/download/7.7/Linux/nomachine_7.7.4_1_x86_64.rpm

# Install it:
sudo yum install ./nomachine_7.7.4_1_x86_64.rpm

# Remove it later:
sudo yum remove nomachine
sudo yum autoremove
```

## Managing Package Groups

YUM also supports package groups, which bundle related software together into a single installable unit. This makes it easy to set up entire environments—like a GUI, development tools, or server roles—with one command instead of installing each package individually.

```bash
# List groups:
sudo yum group list

# Include hidden groups:
sudo yum group list --hidden

# Install a group:
sudo yum group install 'Server with GUI'

# Remove it:
sudo yum group remove 'Server with GUI'
```

## Updating and Upgrading Packages

```bash
# Check for available updates:
sudo yum check-upgrade

# Update everything:
sudo yum update

# After updating critical components like the kernel, reboot to apply changes.
```
