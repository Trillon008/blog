+++
title = "Headless streaming with bazzite"
date = "2025-05-22T10:32:06+01:00"
draft = false
tags = ["tech", "linux"]
translationKey = "streaming_with_bazzite"
slug = "headless-streaming-with-bazzite"
+++

Or any other distro

I recently gave in and decided to buy a 5070Ti to replace my aging RX5700XT. I hesitated for a long time given the prohibitive GPU prices, but eventually pulled the trigger.
Honestly, I would have preferred going with an AMD card, but the lack of HDMI 2.1 support in the open-source driver was too problematic and pushed me toward Nvidia. Since the PC is connected to my TV for a console-like experience, DisplayPort was not an option. The inconsistent quality of DP → HDMI adapters ultimately convinced me to go with the green team.

## HDMI 2.1 Issues

Too often overlooked, this bug (or rather, this missing functionality) is well known among Linux gamers trying to take advantage of their 4K TVs’ features such as HDR and VRR. TVs do not have DisplayPort inputs, and only HDMI 2.1 allows 4K at 120 FPS and VRR (Variable Refresh Rate).
This limitation is not due to AMD failing to provide a proper driver, but rather to the HDMI Consortium, which does not want open-source implementations of the HDMI 2.1 standard.
So much effort from rights holders trying to protect their licensing while the HDMI standard itself is an absolute mess. Thanks as well to TV manufacturers who increasingly add gaming features to TVs but still refuse to include a DisplayPort input — an interface not subject to licensing fees and costing only a few cents.

## The Idea

This type of setup is usually used in the direction from a desk PC to a TV PC, but I use it the other way around: the PC next to my TV streams to my desk PC or to my Steam Deck.
For various reasons, this setup works better for me.
Regardless, I still need a headless configuration.

## Headless Streaming — What Is It?

On Steam, when you stream games, your main display remains on. The streamed display is also active, meaning you effectively have two active screens. As a result, screen #1 cannot be used by someone else who might want to use the PC. You get a near real-time mirror of the displays.

In a headless setup, we want the source display to remain off while the receiving display is, of course, on. As if you were running a server without a physical screen.

## How To Do It
- Retrieve the EDID from the source display

We will create a virtual display. For that, we need an EDID file. You can find some online, but in my case I simply used the EDID from my own monitor, which is a 1440p 144Hz display.

```
## Find connected display ports
$ for p in /sys/class/drm/*/status; do con=${p%/status}; echo -n "${con#*/card?-}: "; echo "$p: $(cat $p)"; done

DP-1: /sys/class/drm/card1-DP-1/status: connected
DP-2: /sys/class/drm/card1-DP-2/status: connected
DP-3: /sys/class/drm/card1-DP-3/status: disconnected
HDMI-A-1: /sys/class/drm/card1-HDMI-A-1/status: disconnected

## Get the EDID file from DisplayPort 1
$ cat /sys/class/drm/card1-DP-1/edid > /tmp/vscreen

## Check that the EDID is not empty
$ hexdump -c /tmp/vscreen | head
```

- Find an available HDMI/DP output on the destination system
```
for p in /sys/class/drm/*/status; do con=${p%/status}; echo -n "${con#*/card?-}: "; echo "$p: $(cat $p)"; done
```

For me, HDMI-A-1 is occupied (the TV). I chose DP-1 as the available port for the new virtual display.

- Copy the EDID file to the destination system
Copy the EDID file (e.g., via SSH) and place it in a dedicated directory. I put mine in: **/lib/firmware/edid/**


Then we need to add the virtual display via a kernel argument.
On Bazzite, I recommend: rpm-ostree kargs --editor. In my case, I used an append command:
```
rpm-ostree kargs append-if-missing="firmware_class.path=/lib/firmware/edid drm.edid_firmware=DP-1:vscreen video=DP-1:2560x1440@144e"
```

The arguments are fairly self-explanatory.

However, it is very important to explicitly set the resolution and refresh rate — especially if, like me, your display has never actually powered on. I don’t fully understand why, but if the resolution is not specified, I am forced to physically turn the monitor on and off once, as if to initialize it.

That’s it. Everything should now work.

The remaining step is fine-tuning Sunshine and Moonlight to find the right balance.

In Sunshine, you can run arbitrary commands at the start and end of a stream. For example, you can disable one display and enable another:

```
## Disable HDMI-A-1 and enable DisplayPort 2
/usr/bin/kscreen-doctor output.HDMI-A-1.disable && /usr/bin/kscreen-doctor output.DP-2.enable
``` 

In my case this is not necessary, but it can be useful.  
Enjoy.
