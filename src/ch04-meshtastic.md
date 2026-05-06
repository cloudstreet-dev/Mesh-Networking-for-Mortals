# Meshtastic

If you have a weekend, $50, and a vague sense that you should at some point feel mesh networking in your hands, this is the chapter that gets you there. Meshtastic is the entry point. It is the project that, for most readers of this book, will be the first time the abstract idea of "off-grid radio mesh" turns into two LED-blinking devices on a desk that send each other text messages with no Internet involved.

It is also a project that gets several things wrong, has a contentious recent history with parts of its own community, and will not scale to the regional network its marketing copy implies. This chapter will be honest about all of that. The honesty is not a reason not to start with Meshtastic; it is a reason to start with Meshtastic *with eyes open*.

## What it is

Meshtastic is an open-source project that turns inexpensive LoRa radio boards into a multi-hop messaging mesh. You buy a small piece of hardware (typically $25–60 for the bare board, $30–150 for a starter kit including antenna, USB cable, and case), flash it with Meshtastic firmware, pair it to a phone via Bluetooth, and you have a node. Two nodes within radio range can exchange text messages, location pings, and small structured packets without any infrastructure. Three or more nodes form a mesh: messages between non-adjacent nodes are forwarded by the nodes in between.

The project began in 2019, hit broad awareness in 2022–2023 as the maker and prepper communities discovered it, and as of 2026 has by far the largest user base of any open-source mesh-networking project — well into six figures of nodes deployed worldwide, regional communities in dozens of countries, and a real (not just theoretical) presence in disaster-response scenarios. The Portugal-Spain blackout of April 2025 was the most prominent recent example: when the power grid went down across most of the Iberian peninsula, Meshtastic networks stayed up because they didn't depend on the grid in the first place, and reports of people coordinating supplies and welfare checks over the local mesh circulated for days.

## What you'll buy

The shortest path to a working two-node Meshtastic setup in 2026:

| Item | Approximate price (USD) | Notes |
|---|---|---|
| Heltec LoRa V3 (× 2) | $20–25 each | The default starter board. ESP32-S3, OLED screen, USB-C, integrated battery connector. |
| 868/915 MHz antenna (× 2) | $5–10 each | Match the frequency to your region. Don't power on without an antenna. |
| 2000 mAh LiPo battery (× 2) | $8–12 each | Optional but worth it; otherwise tethered to USB. |
| USB-C cable (× 2) | Already in your drawer | For flashing and charging. |
| 3D-printed case (× 2) | Free if you have a printer; $5–15 if you don't | The bare board is fragile. |

You'll spend $80–150 for the pair. You can absolutely spend more — RAK Wireless's WisBlock platform is more polished, the LilyGo T-Beam adds GPS and is a nicer all-in-one, and various small companies sell production-finished nodes for $100–250 — but two Heltec V3s will get you the experience.

**Pick the right frequency for your region** (chapter 2 has the table). A US-region radio (915 MHz) will not work in Europe, and vice versa. The vendors mostly ship region-specific SKUs; check before you buy.

## What an evening looks like

Setting up Meshtastic in 2026 has gotten genuinely friendly. The current sequence:

1. **Plug board into computer.** Visit the Meshtastic web flasher at `flasher.meshtastic.org` in a recent Chrome or Edge (it uses Web Serial). Select your board model. Select the latest stable firmware. Click flash. The web flasher does the rest.
2. **Install the phone app.** Meshtastic has Android and iOS apps. Pair to the board over Bluetooth. The app walks you through naming the node, picking a region, and joining or creating a channel.
3. **Repeat for the second board.** Both boards on the same channel will discover each other within seconds.
4. **Send a message.** From the phone connected to board A, send a text. Board B's phone receives it. The OLED screens on both boards light up. The LEDs blink. You will, if you are like most people who do this for the first time, grin involuntarily.

That's it. You have a mesh network. Two nodes is technically the smallest mesh, and it does not yet feel meaningful. Where it gets interesting is when you add a third board, separate the three by enough distance that A cannot directly hear C, and watch B forward messages between them. Or you take one board outside, put a friend's board on a hill across town, and discover that two nodes 4 km apart with line of sight are still in radio contact when nothing else is.

## A worked example: a config snippet

Meshtastic's configuration is via the phone app, the web client, or the `meshtastic` CLI. Here's the CLI version of setting up a node — useful because it shows what's actually being configured under the hood.

```bash
# Install the CLI
pip install --upgrade meshtastic

# Connect to a node over USB and read its config
meshtastic --info

# Set the region (this is required before transmit)
meshtastic --set lora.region US   # or EU_868, CN, JP, ANZ, KR, IN, etc.

# Set a long-range, slow channel preset
meshtastic --set lora.modem_preset LONG_FAST

# Name the node
meshtastic --set-owner "AlpineRelay" --set-owner-short "ALPN"

# Send a test message on the primary channel
meshtastic --sendtext "first packet over the mesh"
```

The `LONG_FAST` preset is Meshtastic's default in 2026: spreading factor 11, 250 kHz bandwidth, achieving roughly 1 kbit/s of payload throughput at a range that comfortably exceeds 5 km in suburban terrain. There are slower presets (`LONG_SLOW`, `VERY_LONG_SLOW`) for genuinely far-apart nodes, and faster ones (`MEDIUM_FAST`, `SHORT_FAST`) for dense urban deployments where range is less of an issue.

## What it gets right

**Immediate usability.** Most projects in this book require you to read documentation for a couple of hours before you have anything to play with. Meshtastic gets you to "I sent a message over a radio with no Internet involved" in under thirty minutes from opening the box. That matters. Most readers of this book are not going to commit to a project that takes a weekend just to bootstrap.

**Hardware ecosystem.** A dozen vendors ship Meshtastic-compatible boards. Antennas, cases, batteries, GPS modules, solar add-ons — there is a real supply chain. You will not be soldering things from scratch.

**Community.** Discord, Reddit, regional Telegram groups. Real numbers of real users. When something goes wrong with your node, someone has hit the same problem before. This is unusual in this space.

**Disaster-deployment track record.** The Portugal-Spain blackout of April 2025 is the headline example, but Meshtastic networks have come up during hurricanes (US gulf coast, multiple events), earthquakes (Türkiye, 2023, smaller scale), and protests where cellular networks were either down or assumed-untrustworthy. The track record is real, not aspirational.

**Phone integration.** Bluetooth pairing to an Android or iOS app means a non-technical user can interact with the mesh through a familiar messaging UI. This is a bigger deal than it sounds. The mesh is only useful if regular humans can use it.

## What it gets wrong

**Flooding-based routing.** As discussed in chapter 3, Meshtastic's routing is essentially flood-and-suppress: every node rebroadcasts every message it hears (deduplicated by message ID, with a small TTL). This is robust at small scale and degrades hard at large scale. The community-reported wall in 2026 is around 100 nodes per channel in a region; some deployments push higher with careful channel allocation, others hit the wall earlier when nodes are densely packed. The project has experimented with smarter routing (the "next hop" routing introduced in firmware ~2.5 was a step toward less wasteful forwarding) but the fundamental architecture remains flood-shaped.

**Encryption was retrofitted, not designed in.** Meshtastic encrypts channel traffic with AES-256, but the original design did not have encryption at the core, and this shows. Channel keys are pre-shared symmetric keys with weak primary-channel defaults; direct messages between users use a derived key with limitations that have been documented and partially addressed across firmware versions. Compared to Reticulum, which has cryptographic identity at the core of the protocol, Meshtastic's encryption story is "good enough for the casual threat model, not designed for adversaries who care."

For most uses — local hobby networks, prepper coordination, hiking groups — this is fine. For uses where the threat model is serious (journalists, activists, anyone who would be in trouble if traffic were intercepted) Meshtastic is not the right tool, and the project documentation has gotten more honest about saying so.

**The CLA controversy.** In 2024–2025 the Meshtastic project introduced a Contributor License Agreement that requires contributors to grant a broad license to their contributions to the project's umbrella organization. A non-trivial portion of the contributor community pushed back, with concerns ranging from "this enables future relicensing" to "this is governance creep." Some long-time contributors stopped upstreaming work. Some have migrated to MeshCore (chapter 5), which has explicitly positioned itself as not requiring a CLA. The Meshtastic project remains GPLv3-licensed in 2026, but the governance dispute is real and the contributor base is more fragmented than it was in 2023. Readers evaluating which project to invest *long-term* effort in should be aware.

**Regional fragmentation.** This is not Meshtastic's fault — it is a consequence of the LoRa physical layer, as chapter 2 covered — but it bears repeating: a Meshtastic mesh in Asia cannot directly interoperate with a Meshtastic mesh in Europe or the Americas. The regional bands are different. The "global Meshtastic mesh" exists only as a federation of regional meshes connected over Internet bridges (typically MQTT). This is fine, but it is not "the network is global."

**Routing-table size as the network grows.** Meshtastic's flood-with-suppression has each node maintain at least a brief cache of recently-seen message IDs to suppress rebroadcasts. As node density increases, this cache and the duty-cycle pressure compete: the cache wants to be larger, the airtime wants to be smaller, and the network's effective capacity decreases.

## License and governance

GPLv3, with the CLA discussed above. The project is governed by the Meshtastic Solutions LLC umbrella, which holds the trademark and runs the official infrastructure (web flasher, MQTT bridges, default channel definitions). The firmware repository is at `github.com/meshtastic/firmware`; the Android app is `github.com/meshtastic/Meshtastic-Android`; the protobuf definitions (which are the actual interoperability surface) are `github.com/meshtastic/protobufs`.

Forks exist. Some are technical (alternative firmware variants targeting specific hardware), some are governance-driven (forks of the firmware that pre-date the CLA). MeshCore, discussed in chapter 5, is not a Meshtastic fork in the strict sense, but it occupies adjacent ground and has absorbed contributors from both the technical-disagreement camp and the governance-disagreement camp.

## Why it's still the right first recommendation

After all of the above, the recommendation in chapter 12 will still be: start with Meshtastic. Here's why.

The thing you want to feel, the first time you do this, is the basic phenomenon: that two boards $30 apart can talk to each other over kilometers of distance with no Internet involved. Meshtastic gets you to that feeling in an evening. Reticulum is a more serious project but the bootstrap cost is higher. MeshCore is more carefully engineered but the user experience is less polished. Yggdrasil is a different category entirely. For the *first* contact, Meshtastic is right.

Once you have felt the phenomenon, you can be more thoughtful about where to invest from there. Chapter 12 walks through that decision tree explicitly. But the entry door is Meshtastic, and the entry door has been thoughtfully designed even if some of the rooms beyond it are crowded and some of the foundation is the wrong shape for the building they're now trying to build.

## Where to go next

- **Official docs.** `meshtastic.org` is the canonical reference. The "Getting Started" path is genuinely good.
- **Web flasher.** `flasher.meshtastic.org` — Web Serial flashing in the browser, the easiest way to get firmware on a board.
- **Hardware index.** The "Supported Hardware" page on meshtastic.org covers compatible boards and their region restrictions; check it before buying.
- **Erethon's comparison post.** Greek engineer Erethon wrote a widely-circulated 2024 piece comparing Meshtastic, MeshCore, and Reticulum that captured the prevailing community recommendation: *start with Meshtastic, adopt MeshCore for the upper end of network size, invest in Reticulum for serious work.* That framing has held up well into 2026 and the rest of part II of this book is, in effect, an expansion of it.
- **Privacy Guides.** The Privacy Guides community discussion on Meshtastic versus alternatives, particularly around encryption and the CLA, is worth reading if those concerns are load-bearing for you.

## What to take from this chapter

You should now be able to:

- Buy two boards, flash them, pair them to phones, and exchange messages over a mesh you built yourself, in an evening.
- Explain why Meshtastic's flooding-based routing limits practical scale to roughly 100 nodes per region.
- Name the major weaknesses (encryption design, CLA governance, regional fragmentation) without being dissuaded from starting here anyway.
- Hold an opinion about when to graduate from Meshtastic to MeshCore or Reticulum, and why someone would.

The next chapter is MeshCore, the engineered alternative for when Meshtastic's wall is the wall you've hit.
