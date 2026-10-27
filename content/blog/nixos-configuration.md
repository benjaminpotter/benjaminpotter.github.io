+++
date = 2026-10-19
draft = true
title = "NixOS Configuration"
description = "Documentation on how I setup NixOS."

[extra]
keywords = "Nix"
series = "Linux"
toc = true

[taxonomies]
tags = [
    "Linux",
    "Markdown",
]
+++



Here is the nixos configuration information.

I made a bootable USB stick.
Had to disable secure boot to get the BIOS to boot into the stick.
Brought up the live nixos environment.
Then went through the install steps.
I chose to install no desktop environment because I prefer to use sway.

From the basic install I did a few things to get started:
- nmcli to get connected to wifi
- install git, vim, firefox
- install sway, tmux
- enable avahi for mdns lookup

I want to get my configuration under version control as soon as possible.
I don't really understand the big picture yet, since I am just getting started.
However, it seems standard to get going with nix flakes and home-manager.

Need to enable flakes as they are currently experimental.

```
nix.settings.experimental-features = [ "nix-command" "flakes" ];
```


TODO:
- version control

### Thunderbird with Proton Bridge



