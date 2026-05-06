# Reticulum

If this book has a thesis, it is the one in this chapter. Reticulum is the most important mesh-networking project for a 2026 reader to understand. Not because it has the largest user base — it doesn't — but because it is the only project surveyed in this book that takes seriously, simultaneously, the things every project ought to: cryptographic identity, transport-agnostic routing, end-to-end encryption by default, and operation on substrates that are slow, lossy, and unreliable.

The recommendation that has emerged in the mesh-networking community over 2024–2026 — *start with Meshtastic, adopt MeshCore for the upper end of network size, invest in Reticulum for serious work* — has Reticulum at the end of that arrow for a reason. This chapter is why.

## What it is

Reticulum is a networking stack. Not a router, not an application, not a protocol bound to a particular substrate — a stack, in the same sense that TCP/IP is a stack. It sits at roughly the layer that IP sits at, but with very different design choices, motivated by very different constraints.

The core design commitments:

- **Cryptographic identity at the routing layer.** Every Reticulum endpoint has a cryptographic identity — a Curve25519 + Ed25519 keypair. Addressing is by hash of the public key. Identities are unforgeable, packets are signed, and the system as a whole does not depend on any external trust authority for naming.
- **End-to-end encryption by default.** Every Reticulum link between endpoints is encrypted with a forward-secret key derived via X25519. Intermediate nodes route encrypted packets without seeing their contents, and without knowing — for opaque destinations — who the original sender was.
- **Transport-agnostic.** A Reticulum network can run over LoRa, WiFi, serial, I2P, TCP, or anything else that can carry bytes, with a single routing layer that bridges across substrate boundaries transparently. The application sees one network. The packets travel over whichever combination of links the routing layer thinks is best.
- **Designed for slow, lossy substrates.** The protocol is not retrofitted from datacenter assumptions. It assumes packet loss, high latency, and limited bandwidth as ordinary operating conditions, not edge cases.
- **MIT-licensed.** Permissive, no CLA, the source is the source.

These commitments compound in ways that the project's marketing copy can't quite communicate without sounding overheated. The combination is what matters: cryptographic identity *plus* transport-agnostic routing *plus* end-to-end encryption *plus* designed-for-slow-substrates. Each one alone is interesting; together, they describe a stack that is genuinely different from what came before.

## The killer property

Here is the property that, more than any other, justifies the time investment in Reticulum.

> **A single Reticulum network can simultaneously run over LoRa + WiFi + I2P + serial + Ethernet, transparently bridged.**

Consider what that means in practice. You have a Reticulum daemon on your laptop. It has interfaces configured for: a LoRa radio attached over USB, the local WiFi network, an I2P tunnel to the public Reticulum testnet, and a TCP socket to a friend's home server. Your laptop's daemon sees all of these as carriers. When an application asks Reticulum to send a packet to a destination identity, the routing layer decides — based on which paths it has discovered — which interface or interfaces to send it over. The application doesn't know. The application doesn't need to know.

Two consequences:

1. **The network is robust to the loss of any one substrate.** If your ISP goes down, the I2P and TCP interfaces stop working, but the LoRa and WiFi interfaces keep going. If the LoRa antenna falls off, the rest keeps working. You don't have to migrate the network; the network is already on multiple substrates.

2. **The network can bridge between substrates that no single user could individually reach.** A hiker in the woods with a LoRa-only node can reach a node in a city via a chain of intermediate nodes — some LoRa, some WiFi, some Internet — without any of those nodes having to do anything other than be themselves. The bridging is the routing layer's job.

This is what TCP/IP almost was, before pragmatic specialization down the stack made every interface require its own routing layer. Reticulum is not trying to replace TCP/IP — it runs *on top of* TCP/IP as one substrate among many — but it does something TCP/IP gave up on, which is presenting a unified routing layer across radically heterogeneous links.

## Cryptographic primitives

Reticulum uses a small, deliberately-chosen set of cryptographic primitives:

- **Curve25519** for X25519 ECDH key exchange and Ed25519 signatures. Identities are 32-byte public keys; the project uses a hash of the public key as the routing-layer address.
- **AES-128 in CBC mode with HMAC-SHA-256** for symmetric authenticated encryption (the choice predates AES-GCM's wide availability in low-resource environments, and the project has resisted swapping it without good reason).
- **SHA-256** for hashing throughout.
- **Forward-secret session keys** for every link, with periodic rekeying.

The primitive choices are conservative. The implementation has been reviewed by community cryptographers but has not had a formal academic audit at the time of writing. For users whose threat model demands one, that is a genuine limitation. For users whose threat model is "I'd rather not have my mesh traffic readable by the next ham operator with a recording," Reticulum's cryptography is by far the best of the projects in this book.

## RNS: the implementation

The reference implementation of Reticulum is **RNS** (Reticulum Network Stack), written in Python. Yes, Python — on a project that runs on microcontroller-class hardware. The choice is deliberate: Python's reach into Linux, microcontrollers (via MicroPython), Android (via Termux), and embedded systems makes it possible for one codebase to run nearly everywhere a Reticulum node might want to run, at the cost of some performance ceiling that isn't load-bearing on slow substrates anyway.

For genuinely embedded targets — ESP32-class microcontrollers running native firmware — the Reticulum project also ships a C/C++ implementation and a microcontroller-friendly subset, used in the RNode firmware described below.

## RNode: the hardware

**RNode** is the canonical Reticulum hardware platform. It is, physically, a LoRa radio board (typically the LilyGo T-Beam, T-Echo, or similar) running RNode firmware, which presents a Reticulum-aware interface to a host computer over USB or Bluetooth. You can buy pre-built RNodes from the project store for around $100, build your own from off-the-shelf boards and the published firmware for less, or convert an existing Meshtastic-compatible board.

An RNode is not strictly required to use Reticulum — the project's RNS daemon will happily run over WiFi, I2P, or TCP without any radio at all — but it is the canonical way to add a long-range radio interface to your Reticulum stack, and it is the entry point for most users who want the *off-grid* part of off-grid mesh.

## Sideband and MeshChat: the application layer

Reticulum is a stack, not an application. The applications that run on top of it include:

- **Sideband** — the official Reticulum messaging app, available for Linux, macOS, Windows, and Android. The closest thing to "the user-facing UI of Reticulum." Lets you message individual identities, join broadcast channels, and run as a routing node.
- **MeshChat** — a community-built web-UI messaging client that runs on top of RNS and gives you a browser-based mesh chat experience. Particularly nice for shared deployments (e.g., a Raspberry Pi running RNS and MeshChat together as a community node).
- **Nomad Network** — a terminal-UI messaging and content-sharing application. Old-school, beautifully understated, and a delight to use over a slow link.
- **Various smaller projects** — file transfer, voice (yes, low-bitrate voice over LoRa is real and works, with caveats), distributed dashboards, telemetry uplinks.

The application layer is smaller and less polished than Meshtastic's. This is a real cost. Reticulum users routinely have to choose between "use the existing applications, which are good but few" and "build the application I actually want, using the Python API." Most readers of this book will, on first contact, find Sideband sufficient.

## A worked example: a Reticulum cross-substrate bridge

Here is a concrete configuration that demonstrates the killer property. The file is `~/.reticulum/config` after `rnsd` has been run once to generate a default. (Format edited slightly for readability.)

```ini
[reticulum]
  enable_transport = True
  share_instance = True

[interfaces]
  # An RNode on USB serial, talking to nearby LoRa nodes.
  [[RNode LoRa Interface]]
    type = RNodeInterface
    enabled = True
    port = /dev/ttyUSB0
    frequency = 915000000
    bandwidth = 125000
    txpower = 17
    spreadingfactor = 11
    codingrate = 5

  # A UDP broadcast on the local WiFi, talking to other Reticulum nodes
  # in the same building.
  [[Local WiFi Interface]]
    type = AutoInterface
    enabled = True
    devices = wlan0

  # A TCP client connecting to the public Reticulum testnet for
  # bridging to the wider network.
  [[RNS Testnet]]
    type = TCPClientInterface
    enabled = True
    target_host = amsterdam.connect.reticulum.network
    target_port = 4965

  # An I2P tunnel for anonymous-bridging to identities reachable that way.
  [[I2P Bridge]]
    type = I2PInterface
    enabled = True
    peers = anonymouspeer.b32.i2p
```

That single configuration block is a node with four substrates. Once `rnsd` is started, your applications — Sideband, Nomad Network, the Python API — talk to a unified network that spans LoRa, WiFi, I2P, and TCP. A path request to a remote identity will explore all four. A packet sent will be routed over whichever substrate the path discovery says to use. The application code is unchanged regardless of which substrate it ends up traveling over.

This is what people mean when they say Reticulum is taken seriously.

## A worked example: the Python API

Reticulum's Python API is the build-your-own-tooling story. The minimal "send a message between two identities" looks like this:

```python
import RNS
import time

# Initialize the Reticulum stack (uses ~/.reticulum/config)
RNS.Reticulum()

# Generate or load this node's identity
identity = RNS.Identity.recall_app_data() or RNS.Identity()
identity.store_app_data()

# Register a destination — an addressable endpoint on this identity
destination = RNS.Destination(
    identity,
    RNS.Destination.IN,
    RNS.Destination.SINGLE,
    "example_app",
    "messaging",
)

# Set up a callback for incoming packets
def packet_callback(data, packet):
    print(f"Received: {data.decode('utf-8')}")
destination.set_packet_callback(packet_callback)

# Send a packet to a known peer's destination hash
peer_hash = bytes.fromhex("a1b2c3d4...")  # the peer's destination hash
peer_dest = RNS.Destination(
    RNS.Identity.recall(peer_hash),
    RNS.Destination.OUT,
    RNS.Destination.SINGLE,
    "example_app",
    "messaging",
)
RNS.Packet(peer_dest, b"hello, mesh").send()

# Stay alive to receive
while True:
    time.sleep(1)
```

This is roughly thirty lines for a working two-way Reticulum messaging client. There is no socket setup. There is no key exchange to manage manually. There is no NAT traversal logic. The cryptographic identity is a single object you generate and store; the routing is the stack's job; the encryption is automatic. *That* is the level of abstraction Reticulum is operating at.

## What it gets right

- **The stack as a whole.** No part of Reticulum feels bolted on. Cryptographic identity, routing, encryption, transport-agnosticism — these were design commitments from day one and the architecture reflects that.
- **The substrate range.** No other project in this book runs on as many substrates with as little ceremony.
- **The license and governance.** MIT-licensed, no CLA, single-maintainer-driven (which is both a feature and a risk; see below).
- **The documentation.** The Reticulum manual is genuinely excellent. It is one of the better-written technical documents in the open-source mesh-networking world. A reader who has finished this chapter can productively read the manual cover-to-cover in a long evening and come out of it with working knowledge.

## What it gets wrong (or, more honestly, what it doesn't yet do well)

- **Smaller community than Meshtastic.** Real numbers as of 2026 are low five figures of users globally, vs. Meshtastic's six. The community is more technical and more engaged per capita, but the absolute size matters for "someone has hit my problem before."
- **Application layer is thin.** Sideband and MeshChat are good. They are not as polished as the Meshtastic Android app. If your bar is "non-technical family member can use this," Reticulum is closer than it was in 2022 but not yet there in the way Meshtastic is.
- **No formal cryptographic audit.** Mentioned above. The primitives are conservative and the design is sound, but a formal audit has not been published. For audit-level threat models, this is a real gap.
- **Single-maintainer governance risk.** The project has primarily been driven by one person (Mark Qvist, "markqvist" on GitHub). The work is excellent. The bus factor is one. The project does have a small number of regular contributors and the codebase is in good enough shape that it would survive a maintainer transition, but the risk is real and worth naming.
- **Path-discovery latency.** The first packet to a never-before-contacted destination has to do path discovery, which takes seconds-to-tens-of-seconds depending on the substrate. Subsequent packets to the same destination are cached and fast, but the first-contact cost is real and noticeable.

## The recommendation, plainly

If you are going to invest *one* weekend in mesh networking, spend it on Meshtastic. If you are going to invest *one month*, spend the first weekend on Meshtastic to feel the phenomenon, and the rest of the month on Reticulum, with the goal of getting to "I have a Reticulum node that bridges three substrates and I have written a small custom application against the Python API." Reticulum will reward the additional investment in a way that Meshtastic, MeshCore, and Yggdrasil — for different reasons each — will not.

## Where to go next

- **The Reticulum manual** at `markqvist.github.io/Reticulum/manual/`. Cover to cover; it is that good.
- **The reticulum.network site** for the project overview and links to applications.
- **The community** at the Discord linked from the project page. Smaller, technical, generally helpful.
- **RNode hardware** from the project store or built from off-the-shelf boards.
- **Sideband** as the first application to install. MeshChat as the second. Nomad Network as the gift to your future self.

## What to take from this chapter

You should now be able to:

- Explain why Reticulum is structurally different from Meshtastic and MeshCore: cryptographic identity, transport-agnostic routing, end-to-end encryption, all by default, all from the start.
- Configure (in principle) a Reticulum node that bridges multiple substrates simultaneously.
- Write a minimal Python application against RNS that sends and receives encrypted packets over a mesh.
- Hold an opinion on which project is the long-term investment for a serious user, and why the consensus has converged on this one.

The next chapter shifts category: from LoRa-shaped networks running on hardware in your hand to IPv6-shaped networks running on the public Internet but routing differently. Yggdrasil.
