+++
title = 'One-liner to add new privileged user to Azure Linux VM using run command'
date = '2024-03-27T11:02:42+00:00'
draft = false
summary = "One-liner to add new privileged user to Azure Linux VM using run command"
+++
## One-liner to add new privileged user to Azure Linux VM using run command

When executed on an Azure VM running Linux (e.g from Run Command), this command effectively creates a new user named `recovery`, sets their password, and grants them sudo privileges.
```
sudo useradd -m recovery -s /bin/bash -p $(echo testpasswd1! | openssl passwd -1 -stdin) && sudo usermod -aG sudo recovery
```
