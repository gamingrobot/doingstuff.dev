---
title: "Home Assistant - Linux tablet dashboard and voice assistant"
description: "Setting up a Lenovo Tablet 10 as a Home Assistant dashboard and voice assistant"
date: 2024-05-27
slug: "home-assistant-dashboard-tablet"
taxonomies:
  tags: ["Linux", "Home Assistant"]
extra:
  cover_image: "tablet_small.webp"
---

Turning an Intel Atom based Lenovo Tablet 10 (20E4) into a Home Assistant dashboard and voice assistant. Most tutorials for dashboard displays are using Android tablets, but I had this old tablet laying around and figured out how to use it with Home Assistant.

{{ img(src="tablet_small.webp", align="center" alt="Lenovo tablet with home assistant dashboard") }}

<!-- more -->

{%info()%}January 2025: I have also set up a Home Assistant kiosk on the Lenovo ThinkSmart View using [this guide](https://github.com/pgale/lineage_15.1_Installation_TSV).{%end%}

## Home Assistant setup

In the Home Assistant UI:

- Create a `kiosk` user that can only login from the local network
- Create an Assist voice assistant, Settings -> Voice Assistants -> Add Assistant

## Picking a Linux distro

I wanted a Linux distro that ran well on the low-powered Atom processor while having a good touch interface.

- Ubuntu 24.04
  - Performance: Gnome ran poorly, this would possibly be improved by gnome-tweaks and disabling animations.
  - Keyboard: Gnome on-screen keyboard would popup but wouldn't accept input.
  - Touch: Gnome has a good tablet mode with display scaling.
- Fedora Workstation 40
  - Performance: Same Gnome performance issues as Ubuntu.
  - Keyboard: Gnome on-screen keyboard worked but had an odd quirk where the shift key acted like caps lock.
  - Touch: Same as Ubuntu
- Lubuntu/Kubuntu 24.04
  - Performance: LXQt and KDE Plasma 5 both were lighter than Gnome and ran well.
  - Keyboard: Onboard keyboard crashes. Florence isn't in the package repos. Maliit keyboard seems to only work under Wayland (which is current not the default).
  - Touch: Touch worked but the mouse pointer was visible.
- KDE Neon
  - Performance: KDE Plasma 6 ran well but had some missing icons.
  - Keyboard: I had to manually set the virtual keyboard to Maliit keyboard but then worked well.
  - Touch: KDE Plasma 6 has a tablet mode making icons larger, display scaling settings and the mouse pointer is not visible.
- Fedora Spin KDE Desktop 40
  - Performance: KDE Plasma 6 ran well and was **not** missing icons.
  - Keyboard: Defaulted to the Maliit keyboard.
  - Touch: Same as KDE Neon.

KDE Plasma 6 with the Maliit keyboard gave me the best tablet mode and on-screen keyboard. The difference between KDE Neon and Fedora Spin KDE Desktop was that the Fedora Spin version does not use the bleeding edge version of the KDE apps. (KDE Neon is also based on Ubuntu LTS and Fedora Spin KDE Desktop is based on Fedora)

### Alternative on-screen keyboards

- [Onboard](https://launchpad.net/onboard)
- [Maliit Keyboard](https://github.com/maliit/keyboard)
- [Florence](https://sourceforge.net/projects/florence)
- [GJS OSK](https://github.com/Vishram1123/gjs-osk)
- [CoreKeyboard](https://gitlab.com/cubocore/coreapps/corekeyboard)

## Installing Linux

1. Shutdown the tablet
2. Power on the tablet and hold the volume-up button until the ThinkPad Tablet Setup app starts
3. Plug-in Linux USB drive
4. Reorder USB to be the first boot device
5. Reboot

{%warning()%}When installing you should leave the Recovery Partition. I removed it and the ThinkPad Tablet Setup broke.{%end%}

## After installation

These are the changes I made to make it function as a kiosk:

- Make a `kiosk` user and login as `kiosk`
- Setup Firefox
  - Set homepage to `http://homeassistant.local:8123`
  - Add [autofullscreen](https://addons.mozilla.org/en-US/firefox/addon/autofullscreen/) Extension to auto-fullscreen on startup.
  - Disable resuming on crash `about:config` -> `browser.sessionstore.resume_from_crash` -> `false`
- Setup KDE to start Firefox on login
  - Settings -> Autostart -> Add -> Firefox
- Tweak screen sleep time
  - Settings -> Energy Saving
- Disable screen locking
  - Settings -> Screen Locking
- Disable animations
  - Quick Settings -> Animation Speed: Instant
- Set hostname
  - `sudo hostnamectl set-hostname --static <new-name>`
- Enable SSH (optional)

## Tablet voice assistant setup

### Test MIC and playback

```console
$ arecord -r 16000 -c 1 -f S16_LE -t wav -d 5 test.wav
$ aplay -r 22050 -c 1 -f S16_LE -t wav test.wav
```

### Install Python 3.11

Python 3.12 is [currently not supported](https://github.com/tensorflow/tensorflow/issues/62003) by `tflight-runtime`

```console
# yum install git python3.11
```

### Setup wyoming-openwakeword

[wyoming-openwakeword](https://github.com/rhasspy/wyoming-openwakeword) allows us to use local wake word detection rather than Home Assistant's wake word detection.

#### Clone and setup python venv

```console
$ git clone https://github.com/rhasspy/wyoming-openwakeword.git /opt/ha-assist/wyoming-openwakeword
$ cd /opt/ha-assist/wyoming-openwakeword
$ python3.11 script/setup
```

#### Setup wyoming-openwakeword systemd service

- Replace `ok_nabu` with whichever wake word model you want

See wyoming-openwakeword docs for more information.

```console
# systemctl edit --force --full wyoming-openwakeword.service
```

```ini
[Unit]
Description=Wyoming openWakeWord

[Service]
Type=simple
User=kiosk
Group=kiosk
ExecStart=python3.11 script/run --uri 'tcp://127.0.0.1:10400' --preload-model 'ok_nabu'
WorkingDirectory=/opt/ha-assist/wyoming-openwakeword
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```

#### Enable and start wyoming-openwakeword service

```console
# systemctl daemon-reload
# systemctl enable wyoming-openwakeword.service
# systemctl start wyoming-openwakeword.service
```

### Setup wyoming-satellite

[wyoming-satellite](https://github.com/rhasspy/wyoming-satellite) communicates with Home Assistant using the wyoming protocol.

#### Clone and setup python venv and install `webrtc-noise-gain`

```console
$ git clone https://github.com/rhasspy/wyoming-satellite.git /opt/ha-assist/wyoming-satellite
$ cd /opt/ha-assist/wyoming-satellite
$ python3.11 script/setup
$ .venv/bin/pip3 install 'webrtc-noise-gain==1.2.3'
```

#### Setup wyoming-satellite systemd service

- Replace `ok_nabu` with whichever wake word model you picked.
- Change `ha-kiosk` to the name you want to see in Home Assistant.
- Change the `/run/user/<uid>` uid to match your `kiosk` user, setting `XDG_RUNTIME_DIR` is required for `arecord` and `aplay` to work when run from a systemd service. (Otherwise you will get a error: `arecord: main:850: audio open error: Host is down`)

See wyoming-satellite docs for additional settings.

```console
# systemctl edit --force --full wyoming-satellite.service
```

```ini
[Unit]
Description=Wyoming Satellite
Wants=network-online.target
After=network-online.target
Requires=wyoming-openwakeword.service

[Service]
Type=simple
User=kiosk
Group=kiosk
Environment=XDG_RUNTIME_DIR=/run/user/1001
ExecStart=python3.11 script/run --name 'ha-kiosk' --wake-word-name 'ok_nabu' --awake-wav "/opt/ha-assist/sounds/awake.wav" --mic-no-mute-during-awake-wav --mic-auto-gain 5 --mic-noise-suppression 2 --uri 'tcp://0.0.0.0:10700' --mic-command 'arecord -r 16000 -c 1 -f S16_LE -t raw' --snd-command 'aplay -r 22050 -c 1 -f S16_LE -t raw' --wake-uri 'tcp://127.0.0.1:10400'
WorkingDirectory=/opt/ha-assist/wyoming-satellite
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```

#### Enable and start wyoming-satellite service

```console
# systemctl daemon-reload
# systemctl enable wyoming-satellite.service
# systemctl start wyoming-satellite.service
```

#### Additional settings I used

- `--mic-no-mute-during-awake-wav` - Keeps the mic enabled when playing the wake word detection sound.
- `--awake-wav "/opt/ha-assist/sounds/awake.wav"`- Plays the `/opt/ha-assist/sounds/awake.wav` wav file when the wake word is detected.
- `--mic-auto-gain 5 --mic-noise-suppression 2` - Settings for `webrtc-noise-gain` to reduce noise and boost gain.

### Setup wake word detection sound (optional)

Save the sound you want to play when the wake word is detected as `/opt/ha-assist/sounds/awake.wav`

### Check wyoming-satellite and wyoming-openwakeword are working

- `systemctl status wyoming-satellite.service wyoming-openwakeword.service`
- Test your wake word with "ok nabu. What is the weather?"
- `--debug` can be added to both wyoming-satellite and wyoming-openwakeword for additional logging.

## Additional Home Assistant integrations

These are some integrations that further augment the tablet:

- Dashboard modifications
  - [browser_mod 2](https://github.com/thomasloven/hass-browser_mod)
  - [kiosk-mode](https://github.com/NemesisRE/kiosk-mode)
  - [WallPanel](https://github.com/j-a-n/lovelace-wallpanel)
- Linux sensor
  - [LNXlink](https://github.com/bkbilly/lnxlink)
  - [Go Hass Agent](https://github.com/joshuar/go-hass-agent)
