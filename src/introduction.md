# Introduction

There is a certain kind of conversation that happens at hacker meetups every few months. Someone mentions Meshtastic. Someone else mentions Tailscale. A third person, helpfully, says "oh, like cjdns?" and a fourth person, less helpfully, says "isn't that just LoRa?" Within ninety seconds the word *mesh* has been used to refer to four mutually incompatible things and nobody has the energy to untangle which is which.

This book is the untangling.

## The premise

Mesh networking is a topic where the available literature splits into two unhelpful piles.

The first pile is academic: SIGCOMM papers, IETF drafts, dissertations on mobile ad-hoc routing. They are, on the whole, excellent — and they assume you have read every prior paper they cite, which means you have not read them, because nobody has time to read every prior paper. They are not for you.

The second pile is project marketing. The Reticulum landing page. The Meshtastic homepage. The Tailscale "How it works" diagram. They are, on the whole, well-produced — and they assume you have already decided to use the project in question. They are not surveys. They cannot tell you whether to choose this project or a different one, because every one of them is the project they're selling.

There is almost nothing in between.

This book aims to fill that gap, for one specific reader: a working engineer who has heard of half these projects, doesn't know how they compare, and wants an honest survey before committing a weekend to one of them. If you are looking for an academic treatment, the bibliography points outward. If you are looking for a tutorial for one particular tool, the project's own documentation will serve you better than this book ever could. What this book offers is the thing the existing literature does not: the comparative map.

## What we mean by 2026

The mesh networking field has been through several waves. The early 2010s wave — Hyperboria, the cjdns-curious, the radio-mesh-as-political-act communities — is largely over, and pretending otherwise would waste your time. The mid-2010s consumer-mesh wave (the Eero/Plume "buy three boxes for your house" definition of mesh) is alive but is a different category of object than what most of this book is about. The 2020s wave — Meshtastic going mainstream, Reticulum maturing, mesh VPNs becoming infrastructure — is the wave you are sitting in the middle of as you read this.

Scoping to 2026 means: the projects discussed are the ones a reader can actually engage with right now. Where a project is dormant, the book says so. Where a project is thriving, the book says so. Where a project's status is genuinely ambiguous — and a few of them are — the book says that too, and points at the most recent meaningful activity rather than pretending to certainty it doesn't have.

## What you will get out of this

By the end of the book you will be able to:

- Distinguish the four things "mesh" gets used to mean, and call out vendors using the word as marketing rather than description.
- Reason about why physical layer choice cascades through everything else — range, throughput, regulatory limits, who can join.
- Hold the routing problem in your head clearly enough to understand why flooding-based meshes (Meshtastic) degrade above ~100 nodes and what the alternatives are doing differently.
- Have an opinion about which project to install on a Raspberry Pi this weekend, indexed by what you actually want to feel.

## What you will not get out of this

You will not become an expert on any of these projects. You will not be qualified to operate a regional mesh deployment, audit Reticulum's cryptography, or contribute to Yggdrasil's routing core. Those are jobs that take longer than a weekend, and the people doing them have written better material than this book ever will once you know which doors to walk through.

What you will get is: which doors to walk through.

## How the book is organized

**Part I (chapters 1–3)** does the foundational work. What "mesh networking" actually means once you tease apart the four uses of the word. Why physical layer choice cascades through every other decision. Why routing is the hard part and how the major approaches differ.

**Part II (chapters 4–6)** is the LoRa family. Meshtastic is the entry point, MeshCore is the engineered alternative, Reticulum is the project that takes the whole stack seriously. Each gets an honest assessment of what it does well, what it does badly, and what shape of user it serves.

**Part III (chapters 7–8)** moves to IP-layer meshes. Yggdrasil is alive and worth running. cjdns and Hyperboria are largely historical, but the lessons they left are real and worth honoring.

**Part IV (chapters 9–11)** covers the adjacent shapes. Scuttlebutt and the gossip family — mesh-adjacent, useful as contrast. Briar, Bitchat, and short-range Bluetooth-mesh messaging. And mesh VPNs (Tailscale, Nebula, ZeroTier, Headscale) — which legitimately use the word *mesh* but solve a completely different problem.

**Part V (chapter 12)** is the punchline. Indexed by what you want, here is what to install this weekend, what hardware to buy, and what an evening with it looks like.

## A note on tone

This book will occasionally be blunt about projects that did not pan out. That is not disrespect — those projects were attempts at something hard, and the people who built them deserve credit for the attempt regardless of the outcome. But the book is written for a reader who is about to spend a weekend on something, and softening "this network is a ghost town" into "this network has a small but dedicated community" would be a disservice. The reader can handle "this didn't work and here is what we learned." So the book treats them that way.

Let's begin.
