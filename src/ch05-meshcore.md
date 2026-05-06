# MeshCore

If chapter 4 introduced Meshtastic as the entry door, this chapter introduces the door past the entry door. MeshCore is what serious community deployments adopt when they've outgrown Meshtastic's flood-based routing and they're not yet ready to switch to a different category of network entirely. It is a more carefully engineered project, less polished as an end-user product, and lives at a slightly different point in the stack.

It is also a smaller project, with a smaller community, and that is part of the honest tradeoff.

## What it is

MeshCore is a C++ embedded library for LoRa-based mesh networking. The phrase "embedded library" is doing real work in that sentence. Where Meshtastic ships as a complete firmware-plus-app product — you flash a board, you pair a phone, you have a working messaging system out of the box — MeshCore ships as a *building block*. It provides a multi-hop packet routing layer, a configurable security model, and the primitives to build mesh-shaped applications on top. The applications themselves are mostly the user's responsibility, though several reference applications and companion projects exist.

This positioning is important. MeshCore is not trying to be Meshtastic-but-better in the consumer-product sense. It is trying to be the *substrate* on which someone might build a Meshtastic-better in the consumer-product sense, or might build something else entirely — a sensor network, a community alerting system, a custom-application mesh — without inheriting Meshtastic's architectural choices.

## What it gets right

**Multi-hop packet routing that isn't flooding.** This is the headline feature. MeshCore implements an explicit forwarding model: nodes maintain neighbor information and forward packets along discovered paths, with deterministic loop avoidance and per-packet routing decisions. It is closer in spirit to the distance-vector and link-state protocols of chapter 3 than to Meshtastic's flood-and-suppress. The practical consequence is that the airtime cost of a packet does not scale linearly with the network's node count the way Meshtastic's does. A 200-node MeshCore mesh remains usable in a way a 200-node Meshtastic mesh, on the same hardware, in the same density, generally does not.

The cost is that MeshCore must do real route discovery and maintenance — the routing-table and protocol-overhead concerns from chapter 3 apply — but the project has been designed around making that overhead bounded and predictable rather than letting it grow unbounded.

**Configurable security model.** MeshCore lets the application decide how strong the security model needs to be. End-to-end encryption is supported (X25519 key exchange, AES-GCM payloads in current implementations), and the routing layer doesn't require all packets to be encrypted with the same key, which means you can build applications with per-conversation keys, group keys, broadcast channels, and signed-but-not-encrypted public announcements all on the same mesh. Compared to Meshtastic's "channels have a shared key, direct messages have a derived key" model, MeshCore gives the application more rope and assumes the application knows what it's doing with it.

For most users this is a tradeoff in the wrong direction — most users do not want to make security decisions and do not benefit from the additional flexibility. For users *building* something that ships to other users, it is the right tradeoff: the security model is the application's, not a one-size-fits-all default.

**Embedded-friendly footprint.** The library is C++, runs on the same ESP32 / nRF52 microcontroller class of hardware as Meshtastic, and is designed to be linked into a custom firmware rather than requiring you to use the project's reference firmware. A MeshCore-based product can be a single-purpose appliance — a moisture sensor, a panic button, a livestock tracker — with the mesh layer embedded and no other Meshtastic-shaped baggage.

**No CLA.** This is governance, not engineering, but it has driven real adoption in 2025–2026. Contributors who left Meshtastic over the CLA have, in non-trivial numbers, landed on MeshCore. The license terms are conventional permissive (MIT-shaped, depending on which subcomponent), the contribution process is git-shaped, and the project's stated stance is that it intends to remain that way. Whether that stance holds in five years is a question the reader will have to evaluate when five years have passed; in 2026 it is one of the project's draws.

## What it gets wrong (or, more honestly, hasn't gotten right yet)

**The user experience is rough.** "Roll your own application on top of an embedded library" is a developer's product, not an end user's. There is no equivalent of Meshtastic's pair-a-phone-and-message-your-friend onboarding. Reference applications exist but are not as polished. If you are reading this book to play with mesh networking, MeshCore should not be your first stop. If you are reading it to *build* something, MeshCore is on the shortlist.

**Smaller community.** Meshtastic has six figures of nodes deployed and a Discord with thousands of active members. MeshCore has hundreds-to-low-thousands of users and a much smaller forum and Discord presence. Both are real numbers, but the difference matters when you have a problem at 2 AM and need someone to have hit it before. The hours of community help available are dramatically different.

**Fewer pre-built integrations.** Phone apps, MQTT bridges, web dashboards — Meshtastic has a real ecosystem of these. MeshCore has fewer, and they are less polished. If your project depends on integrating with the existing world of Meshtastic-shaped tools, switching to MeshCore costs you those integrations.

**Documentation is the documentation of an engineering project, not a consumer product.** This is fine if you came in expecting it. It is jarring if you are arriving from Meshtastic's user-facing docs and expecting the same level of polish.

## A worked example: a minimal MeshCore packet send

The MeshCore API in C++ at the time of writing is roughly shaped as follows. This is intentionally a sketch — the project's APIs evolve and the canonical reference is the project's own documentation — but the shape will give a feel for what working with the library looks like.

```cpp
#include <MeshCore.h>

MeshCore mesh;

void setup() {
  Serial.begin(115200);

  // Initialize the LoRa radio (board-specific pins)
  mesh.begin(LORA_CS, LORA_IRQ, LORA_RST, LORA_BUSY);

  // Set the region and modulation parameters
  mesh.setRegion(MeshCore::REGION_US_915);
  mesh.setSpreadingFactor(11);
  mesh.setBandwidth(250e3);

  // Generate or load this node's identity (X25519 keypair)
  mesh.identity().load_or_generate("/identity.key");

  // Register a handler for received packets
  mesh.onPacket([](const Packet &p) {
    Serial.printf("From %s: %s\n",
                  p.sender_id().to_hex().c_str(),
                  p.payload_as_string().c_str());
  });
}

void loop() {
  static uint32_t last_send = 0;
  if (millis() - last_send > 60000) {
    // Send a broadcast announcement once a minute
    Packet p;
    p.set_destination(MeshCore::BROADCAST);
    p.set_payload("hello from node 0x" + mesh.identity().short_hex());
    mesh.send(p);
    last_send = millis();
  }

  mesh.loop();
}
```

The texture is recognizable to anyone who has done embedded development: there's a `setup()`, there's a `loop()`, there's a callback for received packets, the radio is configured up front, the identity is persisted to flash. What is *not* there — and what makes MeshCore worth caring about — is the flooding logic, the duty-cycle bookkeeping, the route discovery, and the cryptographic key management. Those are inside the library. The application code is the application code.

## Where MeshCore fits in the stack

Think of the LoRa-mesh ecosystem in 2026 as roughly:

- **Reticulum** — the most ambitious project, transport-agnostic, cryptographic identity at the core, designed for arbitrary scale. Chapter 6.
- **MeshCore** — engineered embedded library, multi-hop routing, configurable security, smaller community.
- **Meshtastic** — polished consumer-product experience, flooding-based routing, large community, the entry door.

These are not strictly competitive — a serious deployment might run Meshtastic for the consumer-facing nodes, MeshCore for the custom-firmware appliances, and Reticulum for the gateway nodes that bridge to the rest of the world. But for a single weekend project, the choice between them is real, and chapter 12 has the decision tree.

## When to actually pick MeshCore

The honest cases are narrow but real:

1. **You are building a product**, not running a hobby network. You have a specific application in mind — sensor telemetry, asset tracking, custom messaging — and you want a mesh substrate without inheriting an entire opinionated firmware stack.
2. **You are running a community deployment that has outgrown Meshtastic.** The 100-node wall is a real wall, you've hit it, you need a more disciplined routing protocol, and you are willing to give up some of Meshtastic's UX polish to get there.
3. **You care about CLA-free governance** and want your contributions to land in a project whose contribution model you're comfortable with for the long term.

If none of those describe you, Meshtastic for play and Reticulum for serious work are likely the better picks, with MeshCore as a project to watch rather than one to invest in this weekend.

## License and project status

MeshCore is permissively licensed (MIT for the core; component licenses vary). As of 2026 the project is active — regular commits, recent releases, an engaged maintainer team — but it is genuinely smaller than Meshtastic. "Smaller" is not "dormant." It is "fewer hands and slower releases." Plan accordingly.

Repository: `github.com/meshcore-dev/MeshCore` (canonical at time of writing; verify before relying on it for a long-term plan).

## What to take from this chapter

You should now be able to:

- Position MeshCore in the LoRa-mesh stack relative to Meshtastic and Reticulum.
- Explain why MeshCore's explicit-forwarding routing is structurally different from Meshtastic's flooding and what that buys you.
- Recognize the cases where MeshCore is the right pick and the cases where it is not.
- Have an opinion about which way you'd lean if a community deployment you cared about was hitting Meshtastic's scaling wall.

The next chapter is the project that, more than any other in this book, is the one to invest in if you are going to invest seriously: Reticulum.
