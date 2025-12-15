---
title: Streaming games to the Steam Deck
description: Setting up Sunshine and Moonlight for streaming to the Steam Deck
date: 2025-12-12
slug: steamdeck-linux-streaming
taxonomies:
  tags:
    - Linux
---

My notes for setting up Steam Deck remote game streaming using [Sunshine](https://docs.lizardbyte.dev/projects/sunshine/) and [Moonlight](https://moonlight-stream.org/) and a custom [EDID](https://en.wikipedia.org/wiki/Extended_Display_Identification_Data). 

<!-- more -->

Sunshine is a game stream host running on my desktop (Fedora 43 KDE Desktop with an NVIDIA graphics card), and Moonlight is the client running on the Steam Deck. I decided to use Sunshine/Moonlight due to latency issues with Steam Remote Play.

Fedora KDE Desktop uses Wayland, which adds some complexity when trying to match the Steam Deck resolution and refresh rate. Since the Steam Deck is using a 16:10 aspect ratio (1280x800), when streaming from a 16:9 display, there will be black bars on the top and bottom of the screen.

With X11 you can use `xrandr` (`wlroots` based Wayland compositors can use `wlr-randr`) to specify a custom resolution and refresh rate (display mode). Under KDE with Wayland, `kscreen-doctor` only supports setting the display mode to one that is supported by the display. We can get around this by using a custom [EDID](https://en.wikipedia.org/wiki/Extended_Display_Identification_Data), either with a dummy display dongle or using `drm.edid_firmware` to specify a custom EDID.


## Setup Sunshine and Moonlight

I set up Sunshine using the [Fedora COPR](https://docs.lizardbyte.dev/projects/sunshine/latest/md_docs_2getting__started.html). Then I set up Moonlight on my Steam Deck following [this guide](https://www.xda-developers.com/how-install-use-moonlight-steam-deck/).


## Testing Normal Streaming

Confirm everything is working normally by streaming to the Steam Deck and seeing the 16:9 aspect ratio.


## Custom EDID

{%warning()%}Custom EDIDs should be safe, but I'm not responsible for your hardware.{%end%}

1. Plug the monitor into a Windows machine and use [Custom Resolution Utility](https://www.monitortests.com/forum/Thread-Custom-Resolution-Utility-CRU). On Linux, [wxEDID](https://flathub.org/en/apps/net.sourceforge.wxEDID) might work to edit the EDID, but CRU was easier to use.
2. Add a new `Standard Resolution` of `1920x1200` (the native resolution of `1280x800` also works, but I got better results with `1920x1200`) and a refresh rate of `90` for the OLED Steam Deck or `60` for the LCD version.
3. Export the modified EDID from Custom Resolution Utility and transfer it to Linux.
4. Save that modified EDID to `/usr/lib/firmware/edid/custom-edid.bin`
5. Use `kscreen-doctor -o` to get the primary display port name and current display mode.
6. Add the kernel argument `sudo grubby --update-kernel=ALL --args="drm.edid_firmware=<port-name>:edid/custom-edid.bin"`, replacing `<port-name>` with the port from the previous step (it should be `DP-X` or `HDMI-A-X`)


## Update Sunshine Application Settings
1. Set up a new application
    1. Set the Do Command to `kscreen-doctor output.<port-name>.mode.<resolution>@<refresh-rate>`, replacing `<resolution>` and `<refresh-rate>` from the above Step 2.
    2. Set the Undo Command to `kscreen-doctor output.<port-name>.mode.<current-mode>`, replacing `<port-name>` and `<current-mode>` from the above Step 5.
    3. (Optional) If you want Steam Big Picture to close when you stop streaming, add a new Undo Command `setsid steam steam://close/bigpicture`
{{ img(src="sunshine-settings.webp", alt="" link="") }}


## Reboot and Test

Reboot and test streaming; it should now be in the 16:10 aspect ratio (your monitor will turn off due to an unsupported mode).