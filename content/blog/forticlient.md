+++
date = 2026-10-28
draft = false
title = "FortiClient VPN on NixOS"
description = "Documentation on how I setup FortiClient VPN."

[extra]
keywords = ""
series = "Linux"
toc = true

[taxonomies]
tags = [
    "Linux",
    "Markdown",
    "NixOS",
]
+++

Fortinet is a proprietary VPN solution used by Queen's University.
Some publishers require that you are connected to the campus network in order to access full text publications.
It's annoying, but it doesn't seem to work when I just try the institutional login route.
So, here, I document the approach for connecting to the VPN from a NixOS machine.

Install the openfortivpn package in the system configuration.

```nix
environment.systemPackages = with pkgs; [
  # preceeding packages...
  
  # for campus vpn connection
  openfortivpn
];
```

The Queen's VPN requires SAML/SSO login to connect.
Openfortivpn does not support SSO by default, so we need to include the openfortivpn-webview package.
All this package does is spawn a browser window, where you complete the SSO, and then returns a token that can be used by openfortivpn.
I am using home-manager, so I setup a basic script that handles the full workflow.
Just run `quvpn` and everything happens automagically.

```nix
home.packages = with pkgs; [
  openfortivpn-webview

  # shell script for starting the campus vpn
  (writeShellScriptBin "quvpn" ''
    GATEWAY="campusvpn.queensu.ca:443"
    openfortivpn-webview "$GATEWAY" | sudo openfortivpn "$GATEWAY" --cookie-on-stdin "$@"
  '')
];
```

### Other Resouces

- [Blog 1](https://leo.leung.xyz/wiki/FortiClient)
- [Blog 2](https://l-lin.github.io/nix/install-Forticlient-VPN-with-SAML-in-NixOS)
