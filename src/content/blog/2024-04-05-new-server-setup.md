---
title: "Setting up a new Ubuntu server"
pubDate: "2024-04-05"
state: "draft"
---

## Table of Contents

## Background

I primarily made this guide to remind myself of the steps I should go through
when setting up a server. [Ubuntu server](https://ubuntu.com/server) is my go-to
Linux server OS and these steps are what I find myself doing every time I setup
a server.

I've tried to make this guide unopinionated - what I've listed is what I believe
is the bare minimum for the majority of servers, _especially_ if exposed to the
internet. The exception to this is the items under
[Nice to haves](#nice-to-haves). These items are generally what I use on
servers, but the reader may choose not to.

## Download, install and update Ubuntu server

If you are using a cloud provider that installs Ubuntu for you, you can skip
this step!

Ubuntu Server can be downloaded from the Ubuntu website at
https://ubuntu.com/download/server

You should make sure to download the Ubuntu LTS (Long-Term Support) version
which includes a total of 10 years of updates and security from when it was
first released. The non-LTS versions only have 9 months of security maintenance!

At the time of writing, the current LTS version of Ubuntu is `22.04 LTS` with
the new version `24.04 LTS` due to come out next week.

### Installing Ubuntu Server

The installation of Ubuntu Server used a step by step wizard to guide you
through the process.

Ubuntu has a great tutorial on each of the steps here,
[click here to read it](https://ubuntu.com/tutorials/install-ubuntu-server).

### Update and Upgrade

Congratulations! You've installed Ubuntu Server. The first thing you should do
is update and upgrade the system.

You can do this by running the following commands:

```bash
sudo apt update
sudo apt upgrade
```

## Setting up user accounts

### Reset all passwords

If you are using a cloud provider, you should reset **_all_** passwords to
ensure that you are the only one with access to the server. Often cloud
providers will give you a root password or a password for the default user
account which can be sent over an insecure media such as email.

You can reset the password for a user by running the
[`passwd`](https://linux.die.net/man/1/passwd) command as the user. You will be
prompted to enter the new password.

### Create a non-root user account with sudo access

If the user account you are using is the root account, you should create a new
user account to use for day-to-day tasks and only use the root account when
necessary.

Not using the root account protects you from accidentally running commands that
could harm your system and acts as a good
[defence in depth](<https://en.wikipedia.org/wiki/Defense_in_depth_(computing)>)
tactic.

The built-in command called [`sudo`](https://linux.die.net/man/8/sudo) allows
you to run commands as the root user without having to log in as the root user.
This is a more secure way of running commands as the root user.

To create a new user account with sudo access, you can run the following
commands:

```bash
adduser <username>
usermod -aG sudo <username>
```

The [`adduser`](https://linux.die.net/man/8/adduser) command will create a new
user account with the username you provide and prompt you for a password.

The [`usermod`](https://linux.die.net/man/8/usermod) command will add the user
to the `sudo` group which gives them access to the `sudo` command.

## Configuring and securing SSH access

If you are using a cloud provider, SSH will typically be enabled by default.
When setting Ubuntu up on your own hardware, you maybe need to install the SSH
server using the following command:

```bash
sudo apt install openssh-server
```

### Public Key Authentication

SSH commonly uses password authentication to log in to a server. However, this
suffers from a number of security vulnerabilities, such as brute force attacks.
Public key authentication is a more secure method of logging in to a server.

Public key authentication uses a pair of keys: a public key and a private key.

The public key is placed on the server and the private key is kept on your local
machine. The public key is stored in the
[`~/.ssh/authorized_keys`](https://linux.die.net/man/8/sshd#:~:text=AUTHORIZED_KEYS%20FILE%20FORMAT)
file on the server in the user's home directory. It will look something like
this (Either with `ssh-rsa` or `ssh-ed25519`):

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZl ... 8J5z user@host
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI ... 8J5z user@host
```

Depending on whether you are using MacOS, Linux, or Windows, you can follow the
guides below to generate a key pair:

- [Linux, MacOS and Windows Subsystem for Linux (WSL)](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/create-with-openssh/)
- [Windows](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/create-with-putty/)
- [1Password](https://developer.1password.com/docs/ssh/manage-keys)

Once you have generated a key pair, you can copy the public key to the server
using one of the methods in this guide:
[How to copy SSH keys to a server](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04).

Once the public key is copied to the server, you can use an SSH terminal of your
choice in combination with an SSH agent to login to the server using public key
authentication. You will know you are using public key authentication if the
server doesn't prompt you for a password when logging in.

### Configuring SSH Server

By default, the SSH server on Ubuntu isn't locked down which means that if you
are exposing your server to the internet, you are at risk of being attacked.

There are a number of steps you can take to secure your SSH server:

- [Disable SSH access for the `root` account](#disable-ssh-access-for-the-root-account)
- [Disable password authentication](#disable-password-authentication)
- [Change the default SSH port](#change-the-default-ssh-port)
- [Disable `DebianBanner`, `PermitEmptyPasswords`](#disable-debianbanner-permitemptypasswords)

First, open the OpenSSH server config file
([`/etc/ssh/sshd_config`](https://linux.die.net/man/5/sshd_config)) in your
editor of choice. I will use [`nano`](https://linux.die.net/man/1/nano) in this
example:

```bash
sudo nano /etc/ssh/sshd_config
```

For each of the following changes, the configuration line might already be in
the file, but commented out. If it is, you can uncomment it by removing the `#`
at the start of the line. If the line isn't in the file, you can add it to the
end of the file.

Remember, security is best done in layers ("defence in depth") so it's best to
configure as many of these options as you can. The more layers of security you
have, the harder it is for an attacker to compromise your server.

#### Disable SSH access for the `root` account

Disabling network access via SSH for the `root` account is encouraged as it
means that direct access to the most privileged account must go through another
less privileged account.

To disable SSH access for the `root` account, you can set the `PermitRootLogin`
option to `no`.

#### Disable password authentication

As mentioned earlier, public key authentication is more secure than password
authentication. For this reason, it's preferred to completed disable password
authentication if you are not going to use it.

**Note: Make sure you have public key authentication working before disabling
password authentication.**

To disable password authentication, you can set the `PasswordAuthentication`
option to `no`.

#### Change the default SSH port

While this step isn't strictly necessary, it can help provide some security
through obscurity. Changing the default SSH port from `22` to something else can
help reduce the number of automated attacks on your server.

**Note: This step should not be used as security measure on its own! Some
automated attacks will not be fooled with a changed port**

To change the default SSH port, you can set the `Port` option to a port of your
choice. Make sure to choose a port that isn't already in use by another service.

I recommend using a port in a high range outside of the
[TCP/IP "Well known" ports](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Well-known_ports)
(From `0` to `1023`) to avoid collisions with existing services.

Make sure that you allow this port through your firewall if you are using one.

#### Disable `DebianBanner`, `PermitEmptyPasswords`

Finally, you can disable the `DebianBanner` and `PermitEmptyPasswords` options
which provide an extra layer of security.

Setting `DebianBanner` to `no` will prevent the SSH server from sending the
Debian banner to the client. This can help prevent attackers from knowing what
version of the SSH server you are running. While this isn't really a security
measure, it can help prevent attackers from targeting known vulnerabilities in
the SSH server.

Setting `PermitEmptyPasswords` to `no` will prevent users from logging in with
an empty password. This protects against the mistake of giving an empty password
to a user which can be used to login to the SSH server.

### Setup Fail2Ban

- SSH Jail

## Nice to haves

### Ubuntu Pro (Free for personal use)

### Install Docker

- https://docs.docker.com/engine/install/ubuntu/
- https://docs.docker.com/engine/install/linux-postinstall/
- Configure log rotation
  https://docs.docker.com/config/containers/logging/json-file/

### Configure Shell

- ZSH
- OhMyZSH
- `PROMPT="$fg[cyan]%}$USER@%{$fg[cyan]%}%m ${PROMPT}"`
