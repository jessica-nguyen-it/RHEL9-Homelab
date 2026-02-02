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
