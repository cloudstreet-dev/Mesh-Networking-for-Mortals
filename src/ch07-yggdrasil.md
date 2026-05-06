# Yggdrasil

The previous three chapters were about networks where the substrate is radio and the goal is to operate independent of the public Internet. This chapter is about a network where the substrate *is* the public Internet (and any other reachable transport) and the goal is to route differently than the public Internet's BGP-and-tier-1-providers arrangement does. Yggdrasil is an IPv6 overlay mesh, and it is the project for the reader who wants a real network of computers that doesn't depend on the public Internet's routing being fair.

## What it is

Yggdrasil is an end-to-end encrypted IPv6 overlay network with a self-organizing routing protocol based on a spanning tree. You install the Yggdrasil daemon on a machine; it generates a public/private keypair; the IPv6 address of the machine in the Yggdrasil network is derived deterministically from the public key (specifically, it lives in the `200::/7` prefix, with the specific address derived from a hash of the public key). The daemon connects to peers — initially via a small set of public peers from the Yggdrasil community, then directly to other Yggdrasil nodes as it discovers them — and exchanges spanning-tree-routing information with them.

Once running, every Yggdrasil node has an IPv6 address and can reach every other Yggdrasil node by that address, with end-to-end encrypted traffic, with no central authority involved. From the operating system's perspective, Yggdrasil presents a TUN interface; from an application's perspective, the Yggdrasil network is just IPv6, and any IPv6-aware application can use it without modification.

## Why it exists

The pitch for Yggdrasil — and for the broader category of self-routing IPv6 overlays — comes from a specific dissatisfaction with how the public Internet routes. The dissatisfaction has multiple flavors, and different users come to Yggdrasil for different reasons:

- **Routing politics.** BGP routes are decided by tier-1 providers based on commercial agreements. Some routes go through countries you'd rather they didn't. Some don't exist at all because no provider has commercial interest in establishing them. A self-organizing overlay routes through whichever peers are available, irrespective of who's selling transit to whom.
- **Censorship resistance.** A network where there is no central authority and traffic is end-to-end encrypted is harder to selectively block than one where it is centralized (Tailscale's coordinator, ZeroTier's controller).
- **Just being interesting.** Some users run Yggdrasil because they like running Yggdrasil. This is a legitimate reason in the open-source ecosystem and the project's docs are honest about it being part of the appeal.

## The routing model, briefly

Chapter 3 covered spanning-tree routing as one strategy in the routing-protocol design space. Yggdrasil is the canonical implementation of this approach in production today.

The mechanics in summary:

- Every node has a cryptographic identity (Ed25519). Its IPv6 address is derived from a hash of its public key.
- The network self-organizes into a spanning tree. There is one logical root at any given time, chosen by an algorithm that prefers stable, well-connected nodes; the root's identity is announced through the network and the tree is rebuilt when it changes.
- Each node's *coordinate* in the tree is a sequence of port numbers describing its path from the root.
- To route a packet, a node looks at the destination's coordinate (which it can ask for via a DHT lookup against the destination's public-key hash, but it usually has cached) and forwards to the neighbor whose coordinate is on the shortest path toward the destination.

The size of the routing decision a node makes is bounded by the number of its direct peers, not the size of the network. A node with 20 peers has a 20-entry forwarding table, regardless of whether the network has a hundred or a hundred thousand nodes. This is the property that lets Yggdrasil claim, plausibly, to scale to global-network sizes with manageable per-node state.

The cost is that spanning-tree paths are not always shortest. Two nodes in the same city might route through a peer in another country if that's where the spanning tree happens to go. The project mitigates this with link-local shortcuts and other engineering, but the fundamental tradeoff stands: bounded routing state in exchange for sometimes-suboptimal paths.

## What you get when you install it

A worked example is in order, because Yggdrasil is one of the easier projects in this book to actually run.

```bash
# Linux, Debian-derived
sudo apt install yggdrasil

# Or build from source (Go required)
git clone https://github.com/yggdrasil-network/yggdrasil-go
cd yggdrasil-go
./build

# Generate a default config
yggdrasil -genconf > /etc/yggdrasil.conf

# Start the daemon
sudo systemctl start yggdrasil
```

After starting, `ip addr show ygg0` shows your Yggdrasil IPv6 address. It will be in `200::/7`, looking something like `200:abcd:1234:5678:9abc:def0:1234:5678`.

By default, the daemon needs to find peers. The Yggdrasil community maintains a list of public peers; out of the box, the config will be empty and the daemon will only connect to nodes you explicitly configure. Add some:

```yaml
# /etc/yggdrasil.conf, partial
Peers:
  - tls://example-peer-1.example.org:443
  - tcp://example-peer-2.example.org:8443
  - tls://[2001:db8::1]:443

# Optionally listen for incoming peers, if you have a public IP
Listen:
  - tls://0.0.0.0:443
```

The current public peer list is at `github.com/yggdrasil-network/public-peers`; pick three or four geographically diverse ones and you're connected to the Yggdrasil network. Restart the daemon and within a few seconds your node has joined the spanning tree.

A worked test:

```bash
# Find a known service on the Yggdrasil network
ping6 200:6e7c:5f9c:50bb:d8b9:0a5e:7c13:abcd

# Or browse to one of the in-network sites
curl -6 'http://[200:1234:5678::1]/'
```

There are real services running on Yggdrasil — Git mirrors, IRC servers, status pages, the project's own infrastructure. The network is small but it is alive. As of 2026, public node counts on the Yggdrasil network sit in the low five figures depending on how you measure. The number is meaningful — it's a real network with real people running real services on it — without being mass-market.

## What it gets right

**Bounded routing state.** This is the central engineering claim and Yggdrasil delivers on it. A node in a 10,000-node network does not have a 10,000-entry routing table. It has a peers-list-sized routing table. The network can grow without exploding state per node. This is a property the alternatives in this design space have not all achieved.

**End-to-end encryption that is genuinely end-to-end.** Yggdrasil traffic between two nodes is encrypted at the source and decrypted at the destination. Intermediate nodes carry encrypted traffic without the keys to read it. This is what end-to-end encryption is supposed to mean and Yggdrasil's implementation lives up to it.

**IPv6-native, transparent to applications.** From an application's perspective, Yggdrasil is a TUN interface that lets you reach IPv6 addresses in `200::/7`. No application changes are needed. SSH, HTTPS, IRC, your own custom protocol — they all work, they're all encrypted by Yggdrasil's link-layer crypto, and the application doesn't need to know.

**Cross-platform.** Linux, macOS, Windows, FreeBSD, OpenBSD, Android (via the Yggdrasil Android app) all have working clients. The Go-based reference implementation is the same code on all of them.

**Active in 2026.** The project ships releases, the community maintains the public peers list, the development is steady. Not booming, not dying. Alive.

## What it gets wrong (or is honestly limited about)

**Path suboptimality.** Discussed above. For most uses (latency-tolerant traffic, file transfers, IRC, web browsing) the suboptimality is invisible. For latency-sensitive uses (real-time voice, gaming) the spanning-tree path is sometimes a problem.

**Bootstrap requires public peers.** A node has to know how to find at least one peer to join the network. The community-maintained public peers list is the practical solution; it works, but it is a centralization point in a design that wants not to have any. The project has discussed alternatives (multicast peer discovery on the local network is supported and works) but for reaching the *global* Yggdrasil network you do start by connecting to known peers.

**Not for off-grid.** Yggdrasil is not a LoRa-mesh competitor. It runs over the public Internet (and other reachable transports). When the Internet is down, your Yggdrasil network is down too, modulo whatever local-network paths you've configured. This is fine — it is not what Yggdrasil is for — but a reader who has just finished the Reticulum chapter should not confuse the two.

**Smaller community than mesh-VPN alternatives.** A reader looking for "give my homelab a flat IPv6 network" is more likely to use Tailscale (chapter 11) than Yggdrasil. Yggdrasil's appeal is the *self-organizing, no-central-authority* property; if you don't need that, Tailscale is more polished and faster to set up. Yggdrasil knows this. The user base reflects it.

**Routing decisions are not user-controllable.** You don't get to say "route my traffic through these specific nodes." The protocol decides. This is by design but occasionally frustrating.

## When to actually run a Yggdrasil node

The honest cases:

1. **You want to understand mesh routing by running a node.** This is the chapter-12 recommendation for the curious reader. Spin up a Yggdrasil node on a VPS, connect it to public peers, browse the in-network services, watch the routing table evolve. It will give you a tangible feel for how a self-organizing overlay actually behaves that the LoRa-shaped projects in this book cannot, because they're either too small to feel the routing properties or too constrained to expose them.

2. **You want a no-central-authority IPv6 network for your own use.** Connect a half-dozen of your own machines, peer them with each other and with a few public peers, and you have a flat IPv6 network among your machines that doesn't depend on any single coordinator service.

3. **You believe in the political project.** Some Yggdrasil users are there because they want a network that is structurally outside the BGP-and-tier-1-providers arrangement. The project does not require you to share that belief, but if you do, this is the implementation.

4. **You are building something that needs the routing property.** Distributed services where any-to-any reachability matters more than peak throughput. Some IPFS-shaped, some federated-services-shaped, some research deployments.

If none of those describe you, Yggdrasil is interesting but probably not load-bearing for your use case. Tailscale (homelab) or Reticulum (off-grid) is the better pick.

## License and project status

Yggdrasil is under the LGPLv3. The reference implementation is Go, in `github.com/yggdrasil-network/yggdrasil-go`. The Android client is `github.com/yggdrasil-network/yggdrasil-android`. The project is governed by a small core team plus a broader contributor base; commits and releases are regular as of 2026.

## What to take from this chapter

You should now be able to:

- Install Yggdrasil on a Linux machine, peer it with the public network, and reach an in-network service.
- Explain how spanning-tree routing keeps per-node state bounded as the network grows, and what that costs (path suboptimality).
- Position Yggdrasil correctly in the design space: it is not a LoRa-mesh competitor, it is not a Tailscale competitor, it is its own thing — a self-organizing IPv6 overlay where the appeal is the no-central-authority routing.
- Decide whether you actually want to run a node, or whether you are just curious enough to read about it.

The next chapter is the project Yggdrasil's lineage came out of, and which is no longer the right place to send a curious reader for hands-on experience: cjdns, Hyperboria, and what the early-2010s mesh-utopia movement left behind.
