---
layout: default
title: "Home"
---

<!-- Internal links look posts up by slug so they stay correct no
     matter what permalink style the site uses. -->
{% assign p_mcp = site.posts | where: "slug", "mcp" | first %}
{% assign p_mcp_search = site.posts | where: "slug", "mcp-search" | first %}
{% assign p_coloured_print = site.posts | where: "slug", "coloured-3d-print" | first %}
{% assign p_comfyui = site.posts | where: "slug", "comfyui" | first %}
{% assign p_vr_network = site.posts | where: "slug", "vr-network" | first %}
{% assign p_pipecat = site.posts | where: "slug", "pipecat" | first %}
{% assign p_smart_clock = site.posts | where: "slug", "smart-clock" | first %}
{% assign p_pinn = site.posts | where: "slug", "pinn" | first %}
{% assign p_homescreen = site.posts | where: "slug", "phone-homescreen-lights-fan" | first %}
{% assign p_strata = site.posts | where: "slug", "strata-opencode" | first %}
{% assign p_storage = site.posts | where: "slug", "tiered-storage-pool" | first %}

## Introduction

This notebook is where I log the builds, runbooks, and fixes from my hobby projects. The common thread through everything I build is doing it locally, on my own hardware: running AI models on my own GPU instead of the cloud, streaming games around my own network, and printing the parts that hold it all together. When I work out how to make something and nobody has written it down, I write down the exact steps myself, so the next build is reproducible and anyone who hits the same wall can skip straight past it.

## Local AI

Local AI is where I spend most of my engineering time. On [Civitai](https://civitai.com/user/PixelMuseAI) I go by PixelMuseAI: I have been part of the community since late 2023, published two dozen models, and shared the images and videos they generate, picked up tens of thousands of times by other makers.

My biggest project so far is [speech2speech_ai](https://github.com/bentay85/speech2speech_ai) on GitHub: a real-time speech-to-speech avatar that runs entirely on a single consumer GPU. You talk to a lip-synced AI avatar while a voice detector gates the microphone, Gemma 4 (with audio input, via llama.cpp) forms the answers, Kokoro TTS speaks the replies, and a FlashHead model animates the avatar. It also takes webcam or desktop screenshots as input, so the agent can see what you see. It grew out of an earlier experiment, a [Pipecat + Qwen3-VL multimodal voice agent with vision]({{ p_pipecat.url | relative_url }}).

Most of what I learn ends up written up here as I go: [wiring MCP servers into LLM tools]({{ p_mcp.url | relative_url }}), [getting a language model to search the internet]({{ p_mcp_search.url | relative_url }}), [migrating to a fresh ComfyUI install]({{ p_comfyui.url | relative_url }}), and [running a fully local coding agent on an RTX 4060 Ti]({{ p_strata.url | relative_url }}). I also keep a foot in the research side, from [Physics-Informed Neural Networks]({{ p_pinn.url | relative_url }}) to the machine's plumbing: [tiered storage pools]({{ p_storage.url | relative_url }}) and the [dedicated router I run my VR headset through]({{ p_vr_network.url | relative_url }}). And not everything is AI: some projects exist purely for comfort at home, like the [smart clock that turns on my TV]({{ p_smart_clock.url | relative_url }}) and the [buttons on my phone's homescreen that run my room lights and fan]({{ p_homescreen.url | relative_url }}).

## Gaming

I game both on a flat screen and in VR, and story is what keeps me playing. On the flat screen I go for RPGs and cinematic adventures with characters worth rooting for. I also have the patience for the long ones, and my most-played games are MMO-RPGs like World of Warcraft and Elder Scrolls Online. In VR, I look for what only VR can do: roomscale presence, hands-on interaction. My all-time favourites are Eye of the Temple, Moss and the Titanfall 2 VR mod. I play on a Quest 3 with a BoboVR S3 Pro strap, over Virtual Desktop.

## 3D printing

I print things we need and things we love. Around the house I’ve designed pipe clips for our toilet, a wall holder and charging dock for our stick vacuum, and stopper blocks that keep the kitchen appliances from sliding off their shelf. My daughter finds cool prints on YouTube and I make them in the colours she picks: a full-colour Trace-e robot, a multi-part Flygon and a mechanical hand she can control with her fingers are just some of the examples. For myself I print multicolour figures from games I love, like Cloud, 2B, Mario and Gundam — some made with [turning an image into a coloured 3D print]({{ p_coloured_print.url | relative_url }}). My current printer is a Snapmaker U1.
