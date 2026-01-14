---
title: Connection Multiplexing with SSH Control Master
date: 2025-12-09T17:00:00-05:00
genre: note
category: comp
tags:
  - devops
  - network
keywords:
  - ssh
---

Use connection multiplexing to avoid passwords, keyphrases or 2FA again and again.

<!-- more -->

Publicly owned large clusters are usually gated behind [2FA systems](https://en.wikipedia.org/wiki/Multi-factor_authentication) for safety concerns. Whenever you tried to establish an SSH connection to it, you need to authenticate it via passkey, mobile authenticator app or whatever. This makes it tedious to login, let alone to transfer files or develop remotely.

OpenSSH has a built-in feature allowing you to reuse an existing SSH connection for multiple subsequent SSH sessions to the same remote host. It creates a persistent socket to serve this purpose.

First, manually create a path to store your sockets, typically `~/.ssh/sockets`. Next, append the follow block to your SSH configuration. It must be placed after your host alias definitions.

```plaintext
Host *
# Allow persistence for every server
# You can also use Match to limit it to particular servers, for example:
# `Match host *.<domain-name>.<tld>`
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h:%p
    # Socket name format: user@host:port
    # You can use %C for just a hash
    ControlPersist 12h
    # Set it to persist for 12h
```

After the first 2FA, within the persistence time, *all* of your future SSH connections will reuse the same socket, bypassing the login prompt. You can use the following commands to control the sockets.

```shell
# Check the health
ssh -O check <user>@<host>

# Terminate it
ssh -O exit <user>@<host>
```

[Rsync](https://rsync.samba.org/) and [Rclone](https://rclone.org/) are commonly used to transfer and keep remote files in sync with local ones. Since `rsync` calls the `ssh` binary by default, it is readily available after the configuration. For `rclone`, you need to configure it to reuse your system `ssh`:

```toml
[<remote-name>]
type = sftp
ssh = /usr/bin/ssh <user>@<host>
# Do not include -s sftp, Rclone will handle it.
```
