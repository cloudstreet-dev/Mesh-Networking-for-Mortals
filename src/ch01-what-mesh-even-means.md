# What "Mesh Networking" Even Means

The word *mesh* is doing too much work. In the span of a single hallway conversation it can refer to four different things, only some of which have anything in common with each other, and no one will stop to clarify which one they meant. This chapter is the disambiguation. It is short and a little pedantic, and it earns the rest of the book.

## The four things people call "mesh"

When a working engineer says *mesh networking*, they mean one of:

1. **A mesh routing protocol.** A specification (and an implementation) for how a population of nodes that can hear some of each other should agree on which node forwards which packet, with no node designated as the gateway. Examples: OLSR, BATMAN-adv, Babel, Yggdrasil's spanning-tree, cjdns's source-routing-with-DHT.

2. **A mesh application** — a piece of user-facing software that runs *over* a mesh-shaped network and is the thing the user actually interacts with. Meshtastic the app. Briar the app. Manyverse on top of Scuttlebutt.

3. **A mesh VPN.** An overlay where every node can address every other node directly, peer-to-peer, over the existing public Internet, with NAT traversal and key exchange handled by a coordination service. Tailscale. Nebula. ZeroTier. Headscale. The traffic is mesh-shaped (point-to-point, no central relay in the data path); the substrate is the regular Internet.

4. **"Mesh" as a marketing word.** Three WiFi access points in a house. Two LoRa nodes on a desk. A Bluetooth speaker that pairs with another Bluetooth speaker. None of these are mesh networks in any structural sense — they are typically star topologies with one or two hops — but the word *mesh* tests well in marketing copy and so it gets stuck on the box.

These are not the same thing. They are not different layers of the same thing. A mesh routing protocol is concerned with *which node forwards a packet when there is no central authority to ask*. A mesh VPN is concerned with *how do two nodes that already have an Internet connection bypass the need for a central relay*. Those are nearly opposite problems. Both legitimately call themselves *mesh*. Both are useful. Conflating them is the source of about 80% of the confusion in this space.

## The structural distinction that matters

Here is a test that cuts through almost all of the marketing.

> **Does the network function when the public Internet is unavailable?**

If yes, you are looking at category 1 (a routing protocol over an alternative substrate — radio, BT, sneakernet) or category 2 (an application built on category 1).

If no — if the network requires the underlying public Internet to deliver packets between nodes — you are looking at category 3 (a mesh VPN) or category 4 (marketing).

Both are valid. They solve different problems. But the answer to that one question tells you which conversation you are in.

A second test, less binary but more revealing:

> **What is the routing problem the protocol is actually solving?**

For category 1, the routing problem is *Node A wants to send a packet to Node F. Node A can hear Node B and Node C. Node F can hear Node D and Node E. Nobody has a complete map. How does the packet get there?* This is a hard problem. There are no central authorities. The links come and go. The radio environment shifts. Algorithms like OLSR, Babel, Yggdrasil's spanning-tree, and Reticulum's transport layer exist to solve exactly this.

For category 3, the routing problem is *Node A and Node F both have public Internet connections. Node A is behind a CGNAT in São Paulo, Node F is behind a corporate firewall in Stockholm. They want a direct connection. Where does the packet go?* This is also a hard problem, but it is a *different* hard problem, mostly involving NAT traversal, STUN/TURN, hole punching, and trust establishment via a coordinator. Tailscale's WireGuard-and-DERP architecture is a beautiful solution to it.

Same word. Different problems. Different solutions. Different chapters of this book.

## What "marketing-mesh" usually means

When a vendor sells you a "mesh router" for your house, what you are buying is a small number (typically two to four) of WiFi access points that share a common SSID, hand off clients between themselves, and backhaul to a single primary unit which is connected to your ISP. There is exactly one path from your laptop to the Internet: laptop → secondary AP → primary AP → modem. There is no real routing decision being made. The "mesh" topology, such as it is, exists only in the small redundancy where two secondaries can backhaul through each other if one's backhaul to the primary is poor.

This is not nothing — it's a genuine improvement over the single-AP-with-poor-far-room-coverage situation. But calling it *mesh networking* in the same sentence as Reticulum or Yggdrasil is a category error. They are not in the same conversation. They are not solving the same problem. The vendor is using the word *mesh* to mean *we have more than one node*, which is the marketing definition.

You will sometimes see "mesh" used the same way for IoT products: a smart bulb, a smart switch, and a hub all use Zigbee or Thread, and the marketing copy calls this a mesh. Here the term is closer to honest — Zigbee and Thread *do* have multi-hop routing, and bulbs *can* legitimately forward packets for other bulbs — but the user-visible behavior is rarely affected by this routing because the network is small and the hub is always reachable. It's a mesh in the protocol sense and a star in the practice sense.

## The terminological mess in summary

| When someone says "mesh"... | They might mean... | Example |
|---|---|---|
| ...the protocol layer | A multi-hop routing protocol with no central authority | Babel, BATMAN-adv, Yggdrasil, Reticulum's transport |
| ...the application layer | An app that runs over such a network | Meshtastic the app, Briar, Manyverse |
| ...the VPN layer | A peer-to-peer overlay on the public Internet | Tailscale, Nebula, ZeroTier, Headscale |
| ...the marketing layer | "We have more than one node" | Eero, Plume, "mesh WiFi" boxes |

For the rest of this book: when we say *mesh*, we mean category 1, 2, or 3. We will be specific about which. We will also occasionally point at category 4 to be honest that it exists and that it muddies the search results when you go looking for serious material on the other three.

## What to take from this chapter

You should now be able to:

- Hear someone say *mesh networking* and ask, in a tone that does not sound rude, "do you mean the routing kind or the VPN kind?"
- Recognize when a vendor's "mesh" is doing structural work and when it is doing marketing work.
- Understand why a book that surveys Reticulum *and* Tailscale in the same volume is not a category error — both legitimately use the word, and both deserve serious treatment, but they live at different layers of the stack and solve different problems.

The next two chapters set up the foundations for the routing-protocol kind: why the physical layer cascades through every other decision (chapter 2), and why routing itself is the dominant constraint as networks grow (chapter 3). After that, the projects.
