# Week 13 - Users and Groups

Today’s focus is on how Linux handles users, groups, and pluggable authentication modules!

## Create delete and modify local user accounts

Every person who uses a Linux server should have their own account. It protects their files, lets them personalize their environment, and gives administrators tighter control to prevent mistakes or misuse. 

That being said, this is mostly review, so we'll go over this topic very briefly!

#### Creating a New User Account

```bash
# Creating a new user account
sudo useradd john

# When you create a new user, the operating system will preform a set of default actions during account creation.
# You can review these default settings using:
useradd --defaults

GROUP=100                          # default user GID
HOME=/home                         # base directory where new users’ home directories will be created
INACTIVE=-1                        # number of days after a password expires before the account is disabled
EXPIRE=                            # account expiration date
SHELL=/bin/bash                    # login shell
SKEL=/etc/skel                     # skeleton directory where default files will be copied from 
CREATE_MAIL_SPOOL=yes              # creates mail spool in /var/mail/username

```

#### Setting a Password and Deleting an Account

```bash
# Users do no have passwords by default, set one with:
sudo passwd john

# Deleting an account
sudo userdel -r john               # Including the -r option also deletes the user's home directory and mail spool
```

#### Viewing User Account settings 

```bash
# View user account details
cat /etc/passwd
john:x:1001:1001::/home/otherdirectory/:/bin/othershell    # username, password placeholder, UID, GID, GECOS, home directory, login shell

# You can also review the current user's details using:
id
whoami 

```

#### Modifying User Accounts

```bash
# Changing a user's home directory
sudo usermod -d /home/newdirectory -m john

# Assigning a specific uid
sudo useradd --uid 110 smith  

# Changing the username
sudo usermod -l jack john 

# Changing the login shell
usermod -s /usr/bin/zsh  # OR
chsh -s /usr/bin/zsh  

# Locking and Unlocking an account
sudo usermod --lock jack
sudo usermod --unlock jack

# Setting an expiration date
sudo usermod --expiredate 2021-12-10 jack

# Setting a password expiration
sudo chage --lastday 0 jack

# Setting a password change interval
sudo chage --maxdays 20 jack

# Check using:
sudo chage --list jack
```

#### System Accounts

Linux also accommodates system accounts designed for programs and daemons. These accounts typically have numeric IDs less than 1000 and do not require a home directory.

System accounts are ideal for running background services such as database servers or web servers that do not need interactive logins.

```bash
# Creating a system account
sudo useradd --system sysacc
```

## Create delete and modify local groups and group memberships

Managing groups in Linux helps organize permissions and shared access. By placing users into the right groups, you can easily control who can work with certain files or perform specific system tasks. 

Again, this is mostly review, so we'll go over this topic very briefly. Here's am example of what you can do to manage your groups:

```bash
# Creating a new group
sudo groupadd developers

# Adding a user to your group
sudo gpasswd --add john developers

# Confirm
groups john
```
```bash
# Changing a user's primary group
sudo usermod -g developers john

# Adding a secondary group
sudo usermod -G administrators john
```
```bash
# Renaming a group
sudo groupmod -n programmers developers    # new name first, old name after

# Removing a group
sudo usermod -g john john    # make sure to change the user's primary group back to an existing group
sudo groupdel programers

```

## Managing Access to the Root Account

Managing root access in Linux comes down to choosing when and how users can act as root. Most of the time, you use sudo for temporary admin commands or to start a full root session. 

```bash
# Execute an individual command with root privileges
sudo ls /root/

# Initiating a full root session
sudo --login  # OR
sudo -i

# Log out
logout

```

If you know the root password, you can switch to the root account with `su -`, `su -l`, or `su --login`, but this only works if the root account isn’t locked. 

Some systems lock root by default, meaning you must rely on sudo unless you set or unlock the root password yourself. You can also lock the root account again for security, as long as your user still has sudo access.

```bash
# Setting a new root password
sudo passwd root

# Locking the root account
sudo passwd --lock root
sudo passwd -l root

# Unlocking the root account
sudo passwd --unlock root
sudo passwd -u root

```

# Configuring PAM (aka Pluggable Authentication Modules)

When you run su, the system asks for the root password. Entering it proves who you are and gives you root privileges. This is the basic idea behind authentication: confirming identity before granting access. PAM extends this idea to programs too, not just humans.

#### So what is PAM?

PAM is a system of libraries that programs use to handle authentication, letting Linux stack different checks to decide how and when a user is allowed to access something.

It's flexible enough to customize how commands like `su` behave, require extra authentication steps such as smartcards or tokens, and stack multiple checks together to control how users authenticate.

All PAM configs live in `/etc/pam.d`. Each file corresponds to a specific service or command. For example:  
- `su` → `/etc/pam.d/su`  
- `sudo` → `/etc/pam.d/sudo`  
- login services → `/etc/pam.d/login`, `system-auth`, etc.


#### Looking Inside a PAM Configuration File 

`/etc/pam.d/su` defines how the `su` command authenticates and sets up a user session. 

```bash
ls /etc/pam.d/

auth    required      pam_env.so
auth    sufficient    pam_rootok.so
auth    sufficient    pam_wheel.so trust use_uid
auth    substack      system-auth
account sufficient    pam_succeed_if.so uid = 0 use_uid quiet
session include       system-auth
session optional      pam_xauth.so
```

Column 1: PAM modules fall into four categories:

```
- auth – verify identity  
- account – check account status (expired? locked?)  
- password – handle password changes  
- session – run tasks at login/logout  
```

Column 2: Control Flags determine how PAM reacts to success/failure:

```
- required – must pass; failures are reported *after* all modules run  
- requisite – must pass; failure stops everything immediately  
- sufficient – if it passes, PAM may skip the rest  
- include/substack – pull in rules from other files  
```

Column 3: Exploring Available Modules

Using `man pam` or `man pam.conf` shows all available modules. This is helpful for understanding what each module does and how to use it.

> This is where the authentication logic takes shape: if you want a user to pass both a password check and a USB token check, you mark both modules as required, but if you want access to be granted when either method succeeds, you mark both modules as sufficient instead.
