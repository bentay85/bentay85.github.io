---
layout: default
title: "About Me"
permalink: /about/
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

This notebook is where I log the builds, runbooks, and fixes from my hobby projects. The common thread through everything I build is doing it locally, on my own hardware: running AI models on my own GPU instead of the cloud, streaming games around my own network, and printing the parts that hold it all together. When I work out how to make something and nobody has written it down, I write down the exact steps myself, so the next build is reproducible and anyone who hits the same wall can skip straight past it.

## Local AI

Local AI is where I spend most of my engineering time. On [Civitai](https://civitai.com/user/PixelMuseAI) I go by PixelMuseAI: I have been part of the community since late 2023, published two dozen models, and shared the images and videos they generate, picked up tens of thousands of times by other makers.

My biggest project so far is [speech2speech_ai](https://github.com/bentay85/speech2speech_ai) on GitHub: a real-time speech-to-speech avatar that runs entirely on a single consumer GPU. You talk to a lip-synced AI avatar while a voice detector gates the microphone, Gemma 4 (with audio input, via llama.cpp) forms the answers, Kokoro TTS speaks the replies, and a FlashHead model animates the avatar. It also takes webcam or desktop screenshots as input, so the agent can see what you see. It grew out of an earlier experiment, a [Pipecat + Qwen3-VL multimodal voice agent with vision]({{ p_pipecat.url | relative_url }}).

Most of what I learn ends up written up here as I go: [wiring MCP servers into LLM tools]({{ p_mcp.url | relative_url }}), [getting a language model to search the internet]({{ p_mcp_search.url | relative_url }}), [migrating to a fresh ComfyUI install]({{ p_comfyui.url | relative_url }}), and [running a fully local coding agent on an RTX 4060 Ti]({{ p_strata.url | relative_url }}). I also keep a foot in the research side, from [Physics-Informed Neural Networks]({{ p_pinn.url | relative_url }}) to the machine's plumbing: [tiered storage pools]({{ p_storage.url | relative_url }}) and the [dedicated router I run my VR headset through]({{ p_vr_network.url | relative_url }}). And not everything is AI: some projects exist purely for comfort at home, like the [smart clock that turns on my TV]({{ p_smart_clock.url | relative_url }}) and the [buttons on my phone's homescreen that run my room lights and fan]({{ p_homescreen.url | relative_url }}).

## Gaming

I game both with a headset on and without one. On the flat screen I gravitate toward story-driven single-player games: RPGs and cinematic adventures with characters worth rooting for, from classic JRPGs to modern western RPGs (the Final Fantasy and Dragon Age series, Mass Effect, Nier Automata, The Witcher 3). I also have the patience for the long ones: the MMO grind (World of Warcraft, The Elder Scrolls Online) and turn-based tactics played at my own pace (Divinity: Original Sin 2, XCOM 2). In VR, what I love is what VR alone can do: real roomscale presence, hands-on interaction, and atmosphere that gets under your skin.

A few all-time favourites: **Eye of the Temple** — no other VR game delivers an equivalent roomscale experience; **Ghost Town** — storytelling and atmosphere that made my hair stand on end; **Half-Life: Alyx** — very polished VR interaction, great story; **Titanfall 2** and its **VR mod** — this is just epic; **Moss** and **Moss: Book II** — kept me engaged and rooting for Quill; **Nier Automata**; **The Witcher 3: Wild Hunt**; and **Divinity: Original Sin 2**.

## 3D printing

I print with a Snapmaker U1, mostly for enclosures, brackets, and functional parts around the house and for my hobby builds. When I have time for something purely for fun, I have experimented with [turning an image into a coloured 3D print]({{ p_coloured_print.url | relative_url }}), which pairs nicely with the image-generation side of my AI hobby.
