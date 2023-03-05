---
title: Sending MIDI from Reaper to Unreal Engine
date: 2022-04-11T12:00:00-05:00
tags:
  - music
  - reaper
  - unreal
include_toc: false
slug: reaper-midi-unreal-engine
---

Setting up Reaper to send MIDI to Unreal Engine

<!--more-->

## Setting up Reaper

- Install [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html) (This allows us to create a loopback MIDI device)

- Add a new MIDI device in loopMIDI called `unreal_port`
{{< fig src="/img/reaper-midi-unreal-engine/loopmidi.png" >}}

- Enable new MIDI output in Reaper via Options -> Preferences -> Audio -> MIDI Devices  
{{< info >}}
Only enable the MIDI Output
{{< /info >}}
{{< fig src="/img/reaper-midi-unreal-engine/midi-output.png" >}}

- Setup MIDI output routing per track via:
{{< fig src="/img/reaper-midi-unreal-engine/reaper-routematrix.png" caption="Routing Matrix (View -> Routing Matrix)">}}
**OR**
{{< fig src="/img/reaper-midi-unreal-engine/reaper-routetrack.png" caption="Track routing settings (IO button on track)">}}

Set each track to a separate MIDI channel, so each track number maps to its corresponding channel number.

## Setting up Unreal Engine

Based on [this forum thread](https://forums.unrealengine.com/t/setting-up-a-blueprint-midi-manager-with-4-14-version-of-midi-device-support-plugin/91606)

- Enable MIDI Device Support Plugin
{{< fig src="/img/reaper-midi-unreal-engine/midi-plugin.png" >}}

- Import [Midi Blueprints](https://dev.epicgames.com/community/snippets/JKp/unreal-engine-midi-input)

- Create Blueprint Interface with `OnNoteAction` and `OffNoteAction` functions called `MidiListener`

- Set Game Instance Class via Edit -> Project Settings -> Maps & Modes -> Game Instance -> Game Instance Class
{{< fig src="/img/reaper-midi-unreal-engine/game-instance.png" >}}

- Set `MidiDeviceId` in `MidiManager` to map to the output of `unreal_port` (You can find this out from the debug logs in the console)

- Place `MidiDebug` in the scene to print note debug information to the console

- Place `TestListener` in the scene and set `MidiChannel` to whichever track # in Reaper

### Test it out

If you have added MIDI to the track, set the `MidiInput` number and `MidiChannel` number on the `TestListener`. Hit play in Unreal Engine and then Play in Reaper, the `TestListener` cube should respond to MIDI input.
