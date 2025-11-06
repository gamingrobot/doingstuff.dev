---
title: Blender - Milling Machine Breakdown
description: "Breakdown for the stylized milling machine render"
date: 2025-11-02
slug: blender-milling-machine-breakdown
taxonomies:
  tags:
    - Blender
    - Breakdown
extra:
  jszoom: true
  cover_image: "render.webp"
---

A breakdown for the stylized milling machine render. 
{{ img(src="render.webp", alt="3d render of a milling machine") }}

<!-- more -->

{{ video(src="render_animation.mp4", autoplay="true" ) }}

{{ img(src="mill_clay.webp", alt="clay render for milling machine") }}

## Software
- Rendered in Blender 4.5 using EEVEE
- Line art is rendered using [Malt](https://github.com/bnpr/Malt)/[Cider](https://github.com/gamingrobot/Cider)
- Grain was generated using [Grainy](https://superhivemarket.com/products/grainy)

## Light and Shadow
To get sharp shadows, I set the **Shadow Step Count** to 1 in **Render -> Shadows**. Then set these settings on lights in **Light Settings -> Shadows**:
- Enable Jitter with 10% Overblur 
- Filter to 0.0
- Resolution Limit to 0.0001m

To disable background light emission, I disconnected the **Background** from the output in the **World** shader (or by setting the **Strength** to 0.0). This allows control of the exact color of the objects and shadows in the material shader.

### Spot Lamp
There is a spot lamp inside the mill's lamp illuminating the mill and the volumetric.
{{ img(src="light_spot.webp", alt="spot lamp settings") }}

### Sun Lamp
The low-power sun lamp has a copy rotation constraint targeting the camera and has shadows disabled. This light also uses light linking to exclude some items from the light due to weird shading.
{{ img(src="light_sun.webp", alt="sun lamp settings") }}

Since it is a low-powered light, it only gives highlights to areas outside the main spot lamp.
{{ img(src="light_sun_split.webp", alt="showing highlights added by the sun lamp") }}

## Materials
I set the **Color Management** to **Standard** and added a **Shader AOV** for **Bloom (Color)** in the **View Layer** settings. The bloom is a separate AOV because everything in the scene is emissive, and I only want bloom on specific objects. A Cryptomatte Object pass would also work for this.
{{ img(src="shader_nodes.webp", alt="shader node graph") }}

These are the nodes for the **DiffuseColor** node group. The **ColorOffset** value changes how dark the shadows/diffuse shading are (faking the background illumination without changing the main shader color). 
{{ img(src="custom_diffuse_nodes.webp", alt="node group graph for DiffuseColor") }}

## Volumetrics
The lamp volumetrics are set up on a separate view layer with the spot lamp so I could tweak it in compositing if I needed to. The mill was set to **Holdout**, so it didn't render the volumetric where the mill was.
{{ img(src="volume_node.webp", alt="volumetrics shader settings") }}

## Particles
These are the settings I used for the [swarf](https://en.wikipedia.org/wiki/Metal_swarf) particle system. The swarf models were in a collection that was randomly selected. I had difficulty with getting the physics to be stable, so I set the scene gravity to half, set collision on the metal block, and baked the simulation before rendering.

{{ img(src="particle.webp", alt="particle settings") }}

## Line Art
I tried a couple of different approaches for the line art. 

{{ img(src="render_lineart.webp", alt="grease pencil line art render", text="[LineArt Modifier](https://docs.blender.org/manual/en/4.5/grease_pencil/modifiers/generate/line_art.html)") }}
<br/>
{{ img(src="render_freestyle.webp", alt="freestyle line art render", text="[FreeStyle](https://docs.blender.org/manual/en/4.5/render/freestyle/introduction.html)") }}
<br/>
{{ img(src="render.webp", alt="malt line art render", text="[Malt](https://github.com/bnpr/Malt)") }}

Since I really liked how Malt's line art looked, I decided to go with that one. Malt is a custom render engine, so I had to use a linked scene since each scene can only have one render engine selected. 

I used the default line width settings and set the **Color** to transparent. In the main **Default Render** node tree, I also set the background to transparent.
{{ img(src="malt_lineart_nodes.webp", alt="malt line art node settings") }}

After this project I created [Cider](https://github.com/gamingrobot/Cider), which uses Malt but renders the line art alongside the main render so it can be used in compositing. It also makes it easier to set line art styles directly in the material settings and has support for a viewport preview. Cider is meant for just line art on top of an EEVEE or Cycles render, so Malt should still be used for more complex line art.

## Compositing

{{ video(src="composite_layers.mp4") }}

For compositing, I enabled transparency in **Render -> Film -> Transparent** since I had disabled the background in the World shader earlier. Then I can then composite in any color as the background. 

I enabled additional passes in the View Layer settings:
- Z/Depth (for Vector Blur)
- Vector (for Vector Blur)
- Cryptomatte Object (to mask out specific objects for Vector Blur)

I had a weird issue with one frame in the middle of the render having a large vector change to the metal block in the vector pass but not in the combined pass. So I used the Cryptomatte Object pass to mask out the cutter to apply the Vector Blur to.
{{ img(src="compositor_nodes.webp", alt="compositor nodes") }}

The settings I used for generating the grain with the Grainy addon:
{{ img(src="grainy_settings.webp", alt="grainy settings") }}

If you liked this, you can find more of my art at [gamingrobot.art](https://gamingrobot.art).