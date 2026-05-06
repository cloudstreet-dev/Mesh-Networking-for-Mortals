# The Routing Problem

You have a population of nodes. Each node can hear some subset of the others. There is no central authority. There is no DHCP server, no BGP table, no operator at the top of the building dictating routes. A packet originates at Node A and needs to reach Node F. Nobody — including A and F — has the complete map. How does the packet get there?

That is the routing problem. It is the hard problem in mesh networking. It is harder than people who have only ever worked in datacenters expect, and once you understand why, the rest of the field's design decisions stop looking arbitrary.

## Why it's hard

In a traditional IP network, routing is solved by infrastructure that does not exist in a mesh. Your laptop has an ARP table for the local LAN, a default gateway for everything else, and a DNS server it trusts. The default gateway has a routing table provisioned by an operator. The operator's router runs BGP with their upstream's router. There is a hierarchy. Each layer trusts the layer above to know more than it does, and at the top of the hierarchy is a small number of organizations (registries, root DNS operators, transit providers) that we have collectively decided to trust.

A mesh has none of that. There is no hierarchy. Any node can leave at any time. New nodes appear unannounced. Links come up, links go down — especially over radio, where a node's reachability depends on weather, time of day, and whether someone moved a couch in front of an antenna. The set of nodes you can hear changes minute to minute.

So the routing protocol has to:

1. **Discover the topology** as it changes, with no central authority publishing it.
2. **Decide who forwards what** without consuming so much bandwidth in coordination that there is no bandwidth left for actual data.
3. **Scale to the network's growth** without every node having to remember a routing entry for every other node.
4. **Tolerate packet loss, link loss, and node loss** as ordinary, expected events.

These constraints are in tension with each other. A protocol that discovers the topology aggressively burns bandwidth in coordination overhead. A protocol that conserves bandwidth misses topology changes and routes packets badly. A protocol that scales to thousands of nodes can't afford to have every node know about every other node, so it must pick *some* nodes to know and *some* to forget — but that means some packets won't have routes and the question becomes what to do then.

The history of mesh routing protocols is the history of different positions in this tradeoff space.

## Strategy 1: Just flood it

The simplest possible routing protocol is no routing at all: every node rebroadcasts every packet it hears, with a TTL to prevent infinite loops and a cache to avoid rebroadcasting the same packet twice. This is *flooding*.

Flooding has the enormous virtue of being trivially simple to implement and trivially robust to topology change. There is no routing table. There is no protocol negotiation. A new node joins, it starts hearing packets, it starts forwarding them. A node dies, the others don't notice or care.

Flooding has the enormous vice of consuming bandwidth proportional to the network size — every packet uses the airtime of every node within transitive earshot, regardless of whether the packet was destined for any of them. On a 10-node mesh, this is fine; the airtime cost of a flood is small. On a 100-node mesh, flooding starts to cost. On a 1000-node mesh, flooding is essentially a self-DDoS. The medium fills up with rebroadcasts, the actual messages drown in the noise, and the network's effective capacity collapses.

This is not theoretical. **Meshtastic uses flooding (with rebroadcast suppression and a small TTL), and the practical scaling ceiling for a single-channel Meshtastic mesh in a region is roughly 100 nodes** before the flood-traffic begins crowding out the messages. Some communities have pushed it higher with careful channel allocation and node-density management; many have hit the wall at much lower numbers. The wall is structural. It is not a software bug that the next release will fix. Chapter 4 will return to this in detail.

Flooding is what most mesh-networking projects start with, because it's the simplest thing that works, and most projects don't get big enough to feel the pain. The ones that do — and that survive past the pain — replace it with something else.

## Strategy 2: Distance vector and link state

The classical alternatives to flooding come from the wired-network world. Distance-vector protocols (RIP, BGP) have each node tell its neighbors what destinations it can reach and at what cost; routes propagate hop by hop, each node updates based on what it hears. Link-state protocols (OSPF, IS-IS) have each node flood a description of *its links* (not destinations); every node accumulates the floods and computes the topology locally.

Both have been adapted to mesh contexts. **OLSR** (Optimized Link State Routing) was the canonical link-state protocol for mobile ad-hoc networks; it is still around, used in Freifunk and other community wireless deployments, but it is no longer the bleeding edge. **Babel** is a more modern distance-vector protocol designed for both wired and wireless mesh, with good loop-avoidance properties; it ships in OpenWrt and is genuinely good at what it does. **BATMAN-adv** (Better Approach To Mobile Ad-hoc Networking) is a distance-vector-ish protocol that operates at layer 2 — it makes a multi-hop wireless mesh look like a single Ethernet segment to everything above it, which is delightful for some use cases and limiting for others.

These protocols work well at *medium scale* — tens to low thousands of nodes — and they have the major advantage of producing real routing tables that look like the routing tables you already understand. They have the disadvantages of (a) requiring nodes to maintain state proportional to the network size (every node needs to know about every reachable destination, which is what the routing table is), and (b) having to expend bandwidth re-converging when the topology changes.

The state cost is the killer. **Routing-table size becomes the dominant constraint as networks grow.** A mesh of a million nodes cannot have every node carrying a routing table with a million entries — not because the memory cost is impossible (a million IPv6 routes is a few hundred MB, large but not catastrophic) but because the bandwidth to keep that table converged on every node, in every node's view of the network, on a substrate that may be 250 bit/s of LoRa, is utterly infeasible. The protocol overhead alone exceeds the link capacity.

This is the wall that more ambitious projects are trying to climb.

## Strategy 3: Spanning-tree routing (Yggdrasil)

Yggdrasil takes a different approach. It builds a *spanning tree* over the network — a single, agreed-upon tree where every node has exactly one parent (except the root) — and then assigns each node a coordinate based on its position in the tree. Routing a packet to a destination becomes: look at the destination's coordinate, look at your own, and forward the packet toward whichever neighbor's coordinate is closer to the destination's.

The brilliance here is that **the routing decision is made locally, with no global state.** Each node knows only its own coordinate and the coordinates of its immediate neighbors. The routing table size is bounded by the number of neighbors, not by the network size. A node can join a network of a billion others and still have a small routing table.

The cost is that spanning-tree routing produces *suboptimal paths*. The tree may route a packet between two nodes that are physically close but on different branches of the tree through the root, which is a long detour. Yggdrasil mitigates this with link-local shortcuts and other engineering, but the fundamental property remains: spanning-tree routes are not always shortest-path.

For a network where shortest path matters less than scalability — and for the use case Yggdrasil is built for, the *IPv6 overlay over the public Internet*, this is mostly true — the tradeoff is acceptable. Chapter 7 takes Yggdrasil seriously.

## Strategy 4: DHT-based location lookup (cjdns)

cjdns took a different swing at the same problem. The idea was to use a Distributed Hash Table (think Kademlia, the algorithm that made BitTorrent's tracker-less mode work) to look up the *location* of a destination — which neighbor to forward to in order to make progress — based on the destination's cryptographic identity. Cryptography wasn't an add-on; it was load-bearing. Your address *was* the hash of your public key, which made spoofing impossible and made the network self-validating in a way that earlier protocols were not.

This was beautiful. It did not work, at least not at the scale its proponents hoped. Hyperboria, the cjdns network of the early 2010s, hit a peak of perhaps a few thousand nodes and a long, slow decline thereafter. Chapter 8 covers what happened and why; for now, the relevant lesson for *this* chapter is: **the DHT-based location-lookup approach taught the field that cryptographic identity at the routing layer is a real idea worth pursuing, but that DHT lookups in a high-churn environment have failure modes that the original designers underestimated.** Reticulum learned from this. Yggdrasil learned from this. The lessons stuck even when the network didn't.

## Strategy 5: The Reticulum approach

Reticulum (chapter 6) takes a position that is, in retrospect, obvious but took a long time for the field to articulate. The core insight is: **routing is path discovery, not route maintenance.** You don't try to keep a converged routing table. You don't flood announcements. When a node wants to talk to a destination it has not talked to before, it sends a *path request* into the network. Nodes that know how to reach the destination respond with *path announcements* describing the route. The originating node caches the answer for as long as it remains valid.

This is, broadly, the architecture used in opportunistic and delay-tolerant networking research. Reticulum makes it production-grade by:

- Pairing every node with a cryptographic identity, so path announcements are signed and unforgeable.
- Building everything on a small set of carefully chosen primitives (Curve25519, Ed25519, SHA-256, AES-128) so the cryptographic costs are predictable.
- Treating every interface — LoRa, WiFi, serial, I2P, TCP — as just another carrier for the same packet format, so a single network can span all of them.

The result is a routing protocol that scales to networks where many strategies above wouldn't, on substrates where many strategies above wouldn't run at all. Reticulum is not magic — it has its own tradeoffs (path setup latency is non-trivial; the network is not optimized for high-bandwidth bulk transfer) — but it is a serious engineering response to the routing problem that takes the constraints of low-bandwidth, high-churn substrates seriously from the start.

## Where the projects sit in the design space

| Project | Routing strategy | Scaling regime | Notes |
|---|---|---|---|
| Meshtastic | Flooding with rebroadcast suppression | Up to ~100 nodes | Simple, robust, doesn't scale |
| MeshCore | Multi-hop with explicit forwarding | Hundreds to low thousands | Better than flood; embedded |
| Reticulum | Path-discovery, signed announcements | Designed for arbitrary scale | Cryptographic identity at the core |
| Yggdrasil | Spanning-tree with coordinate routing | Designed for global scale | Suboptimal paths; small tables |
| cjdns | DHT-based location lookup | Theoretically unlimited; in practice did not scale | Lessons survived the network |
| Babel / OLSR / BATMAN-adv | DV / LS adapted for wireless | Tens to low thousands | The classical answer |
| Tailscale et al. | Coordinator-mediated peer discovery | Limited by coordinator | Different problem entirely |

The remaining chapters of part II and III work through the column on the right. The point of *this* chapter is that the column on the right is the consequence of the column on the left, not an arbitrary choice. Each project's strengths and weaknesses are downstream of the routing strategy it picked, which was downstream of the substrate it picked, which was downstream of the user it was trying to serve.

## What to take from this chapter

You should now be able to:

- Explain why flooding is the simplest mesh routing strategy and why it doesn't scale past ~100 nodes.
- Explain why distance-vector and link-state protocols hit a wall in routing-table size as networks grow.
- Recognize spanning-tree routing (Yggdrasil) and DHT-based location lookup (cjdns) as two distinct attempts to escape that wall, with different tradeoffs.
- Recognize Reticulum's path-discovery model as a third attempt, currently looking like the most promising of the three.
- Read a project's marketing copy and ask, "what's the routing strategy, and does it actually scale to the network size you're pitching?"

We are now done with the foundations. The next eight chapters are the projects themselves, starting with the one most readers should install first: Meshtastic.
