# Week 14 - SELinux and Firewall Tuning

This week we'll go over security configs, including SELinux file and process contexts, kernel runtime parameters, and key‑based SSH configuration.

## Configuring Key-Based Authentication for SSH

SSH can authenticate users with cryptographic key pairs instead of passwords, using a private key on the client and a public key stored on the server. This method is the industry standard because it greatly reduces attack surface, prevents brute‑force attempts, and enables secure automation.

```bash
# 1. Generate an SSH key pair (client)
ssh-keygen    # This will create two files: a private key stored at `~/.ssh/id_rsa` and a matching public key at `~/.ssh/id_rsa.pub`.

```

```bash
# 2. Copy your public key to the server
ssh-copy-id user@server

# If ssh-copy-id isn't available:

cat ~/.ssh/id_rsa.pub        # copy this
ssh user@server
mkdir -p ~/.ssh
vi ~/.ssh/authorized_keys   # paste key
chmod 600 ~/.ssh/authorized_keys

```

```bash
# 3. Edit SSH server config
sudo vim /etc/ssh/sshd_config

# Recommended settings:
Port 22                     # or custom port
PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no   # disable passwords
Match User RockLee          # allow password login for one user
    PasswordAuthentication yes
ChallengeResponseAuthentication no
```

```bash
# 4. Reload the SSH service
sudo systemctl reload sshd

```

```bash
# 5. Client-side SSH config
vi ~/.ssh/config

Host myserver
    HostName 10.11.12.9
    User RockLee
    Port 22

chmod 600 ~/.ssh/config

# 6. Connect using the alias
ssh myserver

```

```bash
# 7. Fix known_hosts issues
ssh-keygen -R server_ip      # remove one entry
rm ~/.ssh/known_hosts        # remove all entries

```

## List and Identify SELinux file and process contexts

You may recall that the basic Linux command `ls -l` can be used to display the standard file and directory permissions. This shows the basic read, write, and execute permissions for a file. 

SELinux labels, which you can view with `ls -Z`, serve a similar function while providing a more granular form of security.

#### Understanding SELinux Context Labels

SELinux assigns each file and process a security context label. This label putlines four components: user, role, type, and level. Consider the example label below:

```bash
unconfined_u:object_r:user_home_t:s0

SELinux context fields:

    User --> unconfined_u      # SELinux user (not always the same as Linux user)
    Role --> object_r          # Helps determine what operations are allowed
    Type --> user_home_t       # The most important field; defines what domain can access this file
    Level --> s0               # Sensitivity level

```

Processes also carry SELinux security contexts. You can check the SELinux labels for running processes using the command `ps -Z`.

```bash
ps axZ
system_u:system_r:accountsd_t:s0       995 ?    Ssl    0:00 /usr/libexec/accoun
system_u:system_r:NetworkManager_t:s0   1024 ?    Ssl    0:00 /usr/sbin/NetworkMa
system_u:system_r:sshd_t:s0-s0:c0.c1023 1030 ?    Ss     0:00 /usr/sbin/sshd -D
system_u:system_r:tuned_t:s0            1032 ?    Ssl    0:00 /usr/libexec/platfo
system_u:system_r:cupsd_t:s0-s0:c0.c1023 1033 ?    Ss     0:00 /usr/sbin/cupsd -l

```

You can also view your own SELinux security context using the command `id -Z`.

```bash
id -Z
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
```bash
# To see how Linux users are mapped to SELinux users, execute:

sudo semanage login -l
Login Name    SELinux User      MLS/MCS Range    Service
__default__   unconfined_u      s0-s0:c0.c1023   *
root          unconfined_u      s0-s0:c0.c1023   *

```

#### Checking SELinux Enforcement Status

To check whether SELinux is currently enforcing its security policies, you can run the `getenforce` command. 

When executed, it returns one of three possible modes. If the output is *Enforcing*, SELinux is actively applying its policies and blocking any unauthorized actions. If it reports *Permissive*, SELinux is not enforcing restrictions but will still log any actions that would have been denied. If the mode is *Disabled*, SELinux is completely turned off and no access control is being performed.






