---
layout: post
title: How to use your SSH keys in a VSCode devcontainer
date: 2022-12-05 20:37 +0000
---
I absolutely love VSCode's devcontainers feature, but I've never gotten them to work with my SSH keys. I've been writing my code in a container and then hopping out to WSL to run my fetches, pushes, and pulls. How bizarre! Well earlier today, I finally cracked the case, and here's how you do it.

1. Obviously, create an SSH key. Here's how I generated an `ed25519` key, which was then stored in `~/.ssh/id_ed25519`.
```sh
> ssh-keygen -t ed25519 -C "EMAIL_ADDRESS"
```
2. Add your public key to the remote.
3. Add (or create) the following to a file at `~/.ssh/config`. This tells your agent to forward your keys. You may want to restrict the host if you could be SSH-ing to multiple hosts.
```
Host *
    ForwardAgent yes
```
4. Install `socat` and `keychain` for managing your agent.
```sh
> sudo apt install -y socat keychain
```
5. Add the following to your `~/.config/fish/config.fish` file. If you use a shell that isn't fish, you'll need to put it into your shell's startup file and make some slight adjustments.
```sh
eval (keychain --eval --agents ssh id_ed25519)
```

When you start the WSL distro, the agent will start and VSCode will forward your keys to the container!
