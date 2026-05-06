# The Physical Layer Decides A Lot

If you remember one thing from this book, remember this: in mesh networking, the physical layer dominates. Whatever you choose to send bits over — radio, copper, fiber, sneakernet — cascades through every other decision you will make. Range, throughput, regulatory limits, hardware cost, power draw, who can join your network at all — all of these are downstream of what you picked at the bottom of the stack. You can write the most beautiful routing protocol the world has ever seen and it will not save you from a 250 bit/s link with a hard 1% duty cycle.

This chapter does the physical-layer survey. It is not exhaustive — there is no point trying to compete with the ARRL handbook — but it is enough that when chapter 4 starts talking about Meshtastic, you already know why "868 MHz vs. 915 MHz" matters, why the duty-cycle limit is a real constraint, and why "we run over LoRa *and* WiFi *and* serial" is a non-trivial claim.

## Radio: LoRa

LoRa (the modulation; LoRaWAN is something different and we'll come back to that) is the substrate that the entry-level mesh-networking conversation in 2026 is built on. It is a chirp-spread-spectrum modulation operating in the unlicensed sub-GHz ISM bands, designed for *long range, low data rate, low power* — the inverse of WiFi.

The numbers you should know:

- **Range.** 2–5 km in suburban terrain, 10+ km with line of sight, occasionally much more with high antennas and favorable conditions. Hobbyists routinely report 20–40 km hilltop-to-hilltop links; the record at the time of writing involves balloon-borne nodes well over 700 km apart. Don't plan on those numbers. Plan on the suburban 2–5 km figure.
- **Data rate.** 250 bit/s to ~22 kbit/s depending on spreading factor. Read that again. *Bits.* Not kilobits. At the long-range settings you are sending text messages, not streaming video, not even browsing the web. The very fastest LoRa configuration is slower than a 1992 dial-up modem.
- **Power.** Receive draws ~10 mA. Transmit draws ~120 mA at 100 mW (+20 dBm) output. A node spends most of its time receiving, which means a Meshtastic node on a 2000 mAh battery can run for several days; a solar-supplemented node can run effectively forever.
- **Hardware cost.** $20–40 for a development board (Heltec, LilyGo, RAK), $100–250 for a polished production node (an RNode, a station-grade build), $30–150 all-in for a Meshtastic starter kit including USB cable and antenna.

LoRa's power-frequency-range envelope is what makes it interesting for off-grid mesh. You give up data rate, you get a link that will reach over a city.

### The frequency-fragmentation problem

LoRa operates in regional sub-GHz ISM bands. The bands are different in different parts of the world, and the regulations attached to them are different too. The major regions:

| Region | Band | Notes |
|---|---|---|
| Europe (EU868) | 863–870 MHz | Hard 1% duty cycle on most sub-bands. Strictly enforced. |
| North America (US915) | 902–928 MHz | No duty cycle, but 400 ms dwell-time limit per channel. |
| China (CN470 / CN779) | 470–510 MHz / 779–787 MHz | Lower-band variants, regional. |
| Japan/Korea (AS923) | 920–923 MHz | Listen-before-talk plus duty cycle in most sub-bands. |
| Australia/NZ (AU915) | 915–928 MHz | Similar to US. |
| India (IN865) | 865–867 MHz | Narrow allocation, duty cycle. |

A device manufactured for EU868 will not legally transmit in US915. A device manufactured for US915 will not legally transmit in EU868. This is not a software flag you toggle — different bands often need different hardware (the radios are physically tuned). And the antennas are wavelength-tuned: a 915 MHz antenna on an 868 MHz radio will radiate poorly and in some cases damage the radio.

The cascading consequence: **LoRa-mesh networks fragment along regulatory borders.** A Meshtastic mesh you build in Berlin and a Meshtastic mesh your friend builds in San Francisco are not the same network and never will be. They cannot interoperate over the air. They can, with effort, be bridged via the public Internet (an MQTT bridge, in Meshtastic's case), but at that point you are no longer doing the off-grid thing — you are using mesh-shaped messaging on top of the regular Internet, which is fine, but it is not what the marketing copy implied.

This is the most important consequence of the physical layer that this book wants you to internalize: **when someone says "the global Meshtastic mesh," they mean a federation of regional meshes connected by Internet bridges, not a single radio network.** The radio physics forbid the latter.

### Duty cycle: the slow killer

In Europe, LoRa is restricted to a 1% duty cycle on most sub-bands. That means if you transmitted for 1 second, you must remain silent on that sub-band for 99 seconds. A Meshtastic node sending a single text message at SF12 (the long-range setting) can take 1.5–2 seconds in the air. That single message has consumed your duty-cycle budget for the next two and a half minutes.

Multiply that by every node in earshot rebroadcasting (because Meshtastic's flood routing rebroadcasts everything), and you start to understand why mesh networks at this layer don't scale linearly. The medium itself is rate-limited by regulation, the rebroadcasts compound the airtime usage, and the network is silent during recovery.

This is why chapter 4 will say flatly that Meshtastic begins to degrade above ~100 nodes in a region. It's not a software failing. It's the physics of unlicensed sub-GHz radio meeting flooding-based routing.

### LoRa vs. LoRaWAN

LoRa is the modulation. LoRaWAN is the network protocol that the Things Network and similar deployments use over LoRa, with a star topology, gateways, and a back-end network server. **LoRaWAN is not a mesh.** A LoRaWAN sensor talks to a LoRaWAN gateway, which talks to a LoRaWAN network server, which talks to your application. The sensors do not talk to each other. The mesh-networking projects in this book — Meshtastic, MeshCore, Reticulum-over-LoRa — use LoRa the modulation but ignore LoRaWAN the protocol entirely. They speak their own protocols over the same radio.

## Radio: WiFi

WiFi is the other end of the radio envelope. Short range (typically ≤ 100 m outdoors, much less indoors), high data rate (tens to thousands of Mbit/s), high power (hundreds of mW typical, up to several W for outdoor units in some regions). It is what you use when the substrate is *the same building*, not *the same valley*.

For mesh purposes, WiFi shows up in two places:

- **WiFi mesh routing protocols** — BATMAN-adv and similar — let a population of WiFi devices in ad-hoc or mesh mode (IBSS, or 802.11s) forward packets for each other without an access point. This is real mesh networking in the routing-protocol sense, used in deployments like Freifunk and other community wireless networks. It is not the focus of this book, but it is alive and worth knowing exists.
- **WiFi as a Reticulum interface.** Reticulum (chapter 6) treats WiFi the way it treats any other carrier: as a substrate for its own packets. Two laptops on the same WiFi network can exchange Reticulum traffic over a UDP interface; this is one of the easiest ways to play with Reticulum without buying hardware.

## Radio: Bluetooth

Bluetooth is interesting in mesh contexts not because it is fast (it is not) or long-range (it is decidedly not, ~10 m for classic, sometimes a bit more for BLE long-range modes) but because *every smartphone has it on already*. That last property is what makes Bluetooth-mesh applications like Briar and Bitchat (chapter 10) viable: they don't require the user to plug anything in or buy any hardware, and "the network is free if you're in the same building" is a real value proposition for some use cases.

The honest range numbers for Bluetooth in the wild: 10–15 m indoors through one wall, 20–50 m outdoors line-of-sight, occasionally further with directional antennas or BLE long-range coding. Don't expect more than that. Bluetooth-mesh works well for "people in the same protest" or "phones in the same building"; it does not work for "people in the same neighborhood."

## Wired: Ethernet, fiber, serial

Wired links are not glamorous in a mesh-networking context, but they show up more than you'd expect, especially at the gateway nodes and in serious community deployments. Ethernet between two Yggdrasil nodes in the same rack. Fiber between two roof-mounted radios where the cable run is long. Serial between an RNode and a Raspberry Pi. The wires are boring; what matters for mesh purposes is that *good projects let you use them transparently*.

Reticulum is the project that takes this most seriously. A single Reticulum node can have a serial interface to one radio, a UDP interface to another node over WiFi, an I2P interface to a node halfway around the world, and a TCP interface to the public Reticulum testnet — all simultaneously, all bridged at the routing layer, with the application code unaware of which interface a packet went over. Chapter 6 has the worked example.

## Optical: free-space optical, IR, LiFi

For the sake of completeness: free-space optical mesh (FSO) is real, niche, and used mostly in fixed point-to-point inter-building links where licensed spectrum is unavailable or expensive. Infrared and LiFi exist as research curiosities. None of these are part of the practical mesh-networking conversation in 2026 and you will not see them again in this book except in passing. They are mentioned here because if someone tries to sell you a "next-generation optical mesh" startup, you should know what category they are claiming and that the category is real but small.

## Sneakernet

The most reliable, highest-bandwidth, highest-latency mesh substrate in the world is a person walking with a USB drive. It scales beautifully ("how many terabytes can fit on a station wagon" is the canonical formulation). It is not a joke in mesh-networking contexts: store-and-forward, eventually-consistent, gossip-style protocols (Scuttlebutt is the canonical example, chapter 9) work well over sneakernet. Reticulum has a transport mode designed for it. So does USENET, in case you needed reminding that this isn't a new idea.

The substrate matters. The protocol layer can compensate for it, but it cannot transcend it.

## What cascades from the choice

Here is the thing to internalize. The physical layer does not just constrain bandwidth and range. It constrains:

- **Who can join your network.** A LoRa network requires LoRa hardware. A Bluetooth-mesh network is free for anyone with a phone. A Tailscale network requires an Internet connection.
- **What regulations apply.** Sub-GHz ISM bands have duty cycles. WiFi is unlicensed but power-limited. 5 GHz outdoor use is regulated differently than indoor in most countries. Amateur radio packet links require a license.
- **What the threat model can be.** If your physical layer is local radio, an adversary needs to be in radio range to interfere. If your physical layer is the public Internet, the threat model is the entire Internet and you must encrypt everything.
- **What scale is achievable.** Flooding works on a 30-node LoRa mesh. It does not work on a 30,000-node WiFi mesh. The same routing protocol behaves differently at different scales because the physical layer determines how many nodes are in earshot at once.
- **What the network can do during a disaster.** A LoRa mesh can come up when the ISP is down. A Tailscale network cannot. This is a feature of *both* — they are different tools — but it is downstream of the physical layer.

## What to take from this chapter

The next time you read a mesh-networking project's pitch, ask:

1. What's the substrate?
2. What's the regulatory regime there?
3. What's the data rate at the long-range setting?
4. What's the duty cycle, if any?
5. Who can join — does the user need to buy hardware, or do they already have what they need?

You will learn more from those five questions than from most landing pages. With those answered, the rest of the project's design decisions stop looking arbitrary and start looking like the consequence of a substrate the engineers chose at the beginning and have been paying for ever since.

The next chapter is the routing problem itself: given that you have decided what to send bits over, how do you decide which node forwards which packet, when nobody has the whole map?
