---
title: "Sending MIDI from Reaper to Unreal Engine"
description: "How-to setup MIDI routing from Reaper to Unreal Engine"
date: 2022-04-11
slug: "reaper-midi-unreal-engine"
taxonomies:
  tags: ["Music"]
extra:
  hide_toc: true
---

How to setup Reaper to send MIDI to Unreal Engine. 

<!-- more -->

{%info()%}If you are looking to use OSC instead, take a look at [daw-out](https://github.com/gamingrobot/daw-out).{%end%}

## Setting up Reaper

- Install [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html) (This allows us to create a loopback MIDI device)

- Add a new MIDI device in loopMIDI called `unreal_port`
{{ img(src="loopmidi.png", alt="loopMIDI with unreal_port added") }}

- Enable new MIDI output in Reaper via Options -> Preferences -> Audio -> MIDI Devices  

{%info()%}Only enable the MIDI Output{%end%}

{{ img(src="midi-output.png", alt="Reaper MIDI devices with unreal_port enabled") }}

- Setup MIDI output routing per track via:
{{ img(src="reaper-routematrix.png", text="Routing Matrix (View -> Routing Matrix)") }}
**OR**
{{ img(src="reaper-routetrack.png", text="Track routing settings (IO button on track)") }}

Set each track to a separate MIDI channel, so each track number maps to its corresponding channel number.

## Setting up Unreal Engine

Based on [this forum thread](https://forums.unrealengine.com/t/setting-up-a-blueprint-midi-manager-with-4-14-version-of-midi-device-support-plugin/91606)

- Enable MIDI Device Support Plugin
{{ img(src="midi-plugin.png", alt="Unreal Engine's MIDI Device Support plugin enabled") }}

- Import [Midi Blueprints](https://dev.epicgames.com/community/snippets/JKp/unreal-engine-midi-input)

- Create Blueprint Interface with `OnNoteAction` and `OffNoteAction` functions called `MidiListener`

- Set Game Instance Class via Edit -> Project Settings -> Maps & Modes -> Game Instance -> Game Instance Class
{{ img(src="game-instance.png", alt="Unreal Engine's Game Instance Class set to MidiManager") }}

- Set `MidiDeviceId` in `MidiManager` to map to the output of `unreal_port` (You can find this out from the debug logs in the console)

- Place `MidiDebug` in the scene to print note debug information to the console

- Place `TestListener` in the scene and set `MidiChannel` to whichever track # in Reaper

### Test it out

If you have added MIDI to the track, set the `MidiInput` number and `MidiChannel` number on the `TestListener`. Hit play in Unreal Engine and then Play in Reaper, the `TestListener` cube should respond to MIDI input.
