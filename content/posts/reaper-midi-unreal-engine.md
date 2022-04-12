---
title: "Sending Reaper MIDI to Unreal Engine"
date: "2022-04-11T12:00:00-05:00"
tags: ["music", "reaper", "unreal"]
---

Setting up Reaper to send MIDI to Unreal Engine

<!--more-->

## Setting up Reaper

* Install [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html) (This allows us to create a loopback MIDI device)

* Add a new MIDI device in loopMIDI called `unreal_port`
![loopMIDI](/img/reaper-midi-unreal-engine/loopmidi.png)

* Enable new MIDI output in Reaper via Options -> Preferences -> Audio -> MIDI Devices  
{{< note >}}
Only enable the MIDI Output
{{< /note >}}
![midi-output](/img/reaper-midi-unreal-engine/midi-output.png)

* Setup MIDI output routing per track
  * Either via the Routing Matrix (View -> Routing Matrix)
![route-matrix](/img/reaper-midi-unreal-engine/reaper-routematrix.png)
**OR**
  * Track routing settings (IO button on track)
![route-track](/img/reaper-midi-unreal-engine/reaper-routetrack.png)

Set each track to a separate MIDI channel, so each track number maps to its corresponding channel number.

## Setting up Unreal Engine

Heavily based on [this forum thread](https://forums.unrealengine.com/t/setting-up-a-blueprint-midi-manager-with-4-14-version-of-midi-device-support-plugin/91606)

* Enable MIDI Device Support Plugin
![midi-plugin](/img/reaper-midi-unreal-engine/midi-plugin.png)

* Import [Midi Blueprints](https://dev.epicgames.com/community/snippets/JKp/unreal-engine-midi-input)

* Create Blueprint Interface with `OnNoteAction` and `OffNoteAction` functions called `MidiListener`

* Set Game Instance Class via Edit -> Project Settings -> Maps & Modes -> Game Instance -> Game Instance Class
![game-instance](/img/reaper-midi-unreal-engine/game-instance.png)

* Set `MidiDeviceId` in `MidiManager` to map to the output of `unreal_port` (You can find this out from the debug logs in the console)

* Place `MidiDebug` in the scene to print note debug information to the console

* Place `TestListener` in the scene and set `MidiChannel` to whichever track # in Reaper

### Test it out

If you have added MIDI to the track, set the `MidiInput` number and `MidiChannel` number on the `TestListener`. Hit play in Unreal Engine and then Play in Reaper, the `TestListener` cube should respond to MIDI input.
