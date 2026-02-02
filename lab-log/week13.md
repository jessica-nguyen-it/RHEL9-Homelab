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

#### Customizing User Account settings 

```bash
# Changing a user's home directory
sudo usermod --home /home

```
