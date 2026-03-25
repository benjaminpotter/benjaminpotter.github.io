---
title: "DisplayLink on Linux: Say Goodbye to Hardware Acceleration"
date: 2026-03-25
tags: [linux, arch, thinkpad, displaylink, sway, wayland, docking-station]
draft: false
description: "Running external displays over my ThinkPad Hybrid USB-C Dock pegged my CPU at 80%. What I tried and where I am now."
---

I got a **Lenovo ThinkPad Hybrid USB-C with USB-A Dock** and plugged it into my ThinkPad X1 running Arch Linux and Sway. 
Two external monitors, keyboard, mouse, and fast charging.
More importantly, the ability to (dis)connect my laptop from my desk setup with one cable.
In practice I had fans screaming, `DisplayLinkManager` pinned at 80% CPU, and Sway running entirely on software rendering.

## The Initial Problem: No Display Output At All

Out of the box, nothing worked. 
The displays connected to the dock showed no signal. 
This is a common first-hurdle for DisplayLink docks on Linux since the driver doesn't ship with the kernel and has to be installed manually.
In reality, this is a problem with many peripheral devices on Linux.
It's a good thing I like to get my hands dirty and play around with these things.

### ArchWiki to the Rescue

The ArchWiki is a great resource for addressing these problems.
They have a [page dedicated to setting up DisplayLink drivers on Linux machines](https://wiki.archlinux.org/title/DisplayLink#USB_3.0_DL-6xxx,_DL-5xxx,_DL-41xx,_DL-3xxx_Devices).
What follows is a basic summary of that page with some of my experience added in.

### Installing the DisplayLink Driver

The fix is to install the `displaylink` package from the AUR, which pulls in the `evdi` kernel module and a systemd service:

```bash
yay -S displaylink
sudo systemctl enable --now displaylink.service
```

`evdi` is a kernel module that creates virtual DRM (Direct Rendering Manager) devices. 
The DisplayLink daemon (`DisplayLinkManager`) takes those virtual devices, encodes the framebuffer data, and ships it over USB to the dock's DisplayLink chip, which then drives the physical monitors.

I tried rebooting my system.
After loading the module and starting the service, Sway still wouldn't launch normally.
I am running [the Ly login manager](https://github.com/fairyglade/ly) which detected Sway as an environment to boot into automatically.
After entering valid credentials, I was getting sent right back to the login manager.
I suspected that Sway was crashing on boot, so I opened a TTY to view the output.
it complained about unsupported GPU devices. 
The fix was to start it with:

```bash
sway --unsupported-gpu
```

With that flag, both external monitors came up and were fully usable.
If you want to run with this you have to update the session configuration in Ly.
I would likely opt to create a new `.desktop` file in `/etc/ly/custom-sessions/` and update the `Exec` field to include the `--unsupported-gpu` flag.

## The Real Problem: Software Rendering Everywhere

The displays worked, but the system felt wrong. 
The laptop fans were running hard constantly, and checking `htop` revealed `DisplayLinkManager` sitting at around 80% CPU usage. 

When `evdi` creates virtual DRM devices, those devices have no real GPU backing them. 
Sway detects this and refuses to use hardware acceleration. 
The `--unsupported-gpu` flag forces it to proceed anyway, but the fallback is **llvmpipe**, which is a software renderer that runs entirely on the CPU. 

This means:
- Every frame on *every* display (including the laptop screen) is rendered in software
- `DisplayLinkManager` additionally encodes each frame and sends it over USB
- The CPU is doing the work of a GPU for three monitors simultaneously

The `swaymsg -t get_outputs` confirmed the setup was working but gave away the software rendering story.
The DisplayLink outputs appeared as `DVI-I-1` and `DVI-I-2`, which are presumably the virtual device names created by `evdi`.

## The Hope: Maybe the Dock Supports Native DP Alt Mode?

The dock is called "Hybrid" for a reason. 
It's designed to work with both USB-A laptops (via a dongle) and USB-C laptops. 
For USB-C hosts, the spec sheet lists support for USB-C, USB4, Thunderbolt 3, and Thunderbolt 4. 
This raised an obvious question: could the dock negotiate a native DisplayPort Alt Mode connection and bypass DisplayLink entirely?
If it could, the displays would be driven directly through the ThinkPad's Intel GPU over the Thunderbolt controller.
So, no `evdi`, no software rendering, no CPU overhead.

Since this didn't work on first plug-in, I was skeptical.
I am also not sure how the single USB-C cable between the laptop would work with DP Alt Mode and still support the peripherals.
My only consideration was that I had not installed the `bolt` package available from `pacman`.
This provides access to the `boltctl` command which lets me enumerate the thunderbolt devices connected over USB.

I don't really know how DP Alt Mode works.
The last time I tried to make it work was on an M1 Macbook running Asahi Linux.
They had not yet introduced DP Alt Mode support on the Asahi side, so that fizzled quickly.
In fact, I am not sure if DP Alt Mode has support in Asahi, even now.


### What Native TB4 Would Look Like

On a proper Thunderbolt 4 dock, you'd see:

- The dock enumerated in `boltctl list` as a Thunderbolt device
- Display outputs named `DP-1`, `DP-2` etc. (not `DVI-I-*`)
- No `evdi` or DisplayLink involvement at all
- Full hardware acceleration in Sway without `--unsupported-gpu`

Or at least, that's what I was hoping for.

### Testing It

The test was straightforward: unload `evdi`, unplug and replug the dock, and see what happens.

```bash
sudo systemctl stop displaylink.service
sudo modprobe -r evdi
# replug dock
boltctl list
lsusb
```

The results were unambiguous.
`boltctl list` returned **nothing**.
The dock was not enumerated as a Thunderbolt device at all.
`lsusb` showed:

```
Bus 002 Device 004: ID 17e9:6015 DisplayLink ThinkPad Hybrid USB-C with USB-A Dock
```

The vendor ID `17e9` is DisplayLink's USB vendor ID. 
Even with the DisplayLink driver completely unloaded, the dock presented itself to the OS as a pure DisplayLink USB device. 
There was also a `Lenovo Billboard Device` visible.
From my understanding, this is a USB descriptor that the dock broadcasts when Alt Mode negotiation fails, advertising its capabilities as a fallback signal.

The kernel did show a live TB4/USB4 controller in `dmesg`:

```
usb4_port1 (ops connector_ops [thunderbolt])
```

So the hardware on the laptop side is capable. 
The dock simply isn't using it.

## Why the Dock Can't Do Native Alt Mode

Reading the spec sheet more carefully may explain this. 
The ThinkPad Hybrid USB-C with USB-A Dock has:

- 2× DisplayPort outputs
- 2× HDMI outputs
- A USB hub with multiple USB-A and USB-C ports
- Ethernet
- Audio jack

From what I can gather about the behaviour of the dock.
All of those video outputs are fed by the **DisplayLink chip inside the dock**. 
The dock doesn't do passive Thunderbolt passthrough to its video ports.
The DisplayLink chip *is* the video controller. 
The "Hybrid" in the name refers to supporting both USB-A and USB-C *host connections*, not to having two separate display subsystems.
This is fundamentally different from a Thunderbolt 4 dock, where the laptop's GPU drives the displays directly over the TB4 link and the dock is largely passive on the video path.

## Current State and What's Next

The setup works, but at a cost. 
Three displays running through DisplayLink and llvmpipe is a significant CPU overhead (enough to keep the fans audible).
I didn't notice any lag in the display output, but I only tested basic terminal actions and static webpages.
It may be the case that dynamic outputs like YouTube videos cause a more significant burden on the CPU.
The dock was expensive, so it lives to fight another day, but my search for a better solution is ongoing.
