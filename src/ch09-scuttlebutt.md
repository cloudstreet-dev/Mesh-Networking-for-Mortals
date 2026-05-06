# Scuttlebutt and the Gossip Family

This chapter is about a different shape of network. Scuttlebutt is not a mesh in the routing-protocol sense the rest of this book has been working with. It is not trying to deliver a packet from A to F through a sequence of forwarders, in real time, with no central authority. It is trying to do something else entirely — give every participant an *eventually consistent* view of a distributed log of messages, by gossiping with whoever they can reach when they can reach them.

It is mesh-adjacent rather than mesh-proper. We include it because (a) the project is alive, (b) the design choices are instructive as a contrast to the routing-protocol family, and (c) for a non-trivial set of use cases, gossip is actually the right answer and the routing-protocol designs are the wrong answer.

## What it is

Scuttlebutt — formally Secure Scuttlebutt or SSB — is a peer-to-peer protocol for replicating append-only logs of signed messages across a network of participants. Each participant maintains their own log; that log is replicated to friends, who replicate it to *their* friends, and over time everyone with a path to you in the social graph ends up with a copy of your log. The data flows asynchronously, opportunistically, and is happy to flow over sneakernet (a USB drive carried between two Pi nodes), local WiFi, the Internet, or any combination. The system makes no attempt to deliver any specific message to any specific recipient at any specific time. It just ensures that *eventually*, if there is a path, the message arrives.

The properties this gives you:

- **Local-first.** Every node has a local copy of the data it cares about. Reading is local. Writing is local. Syncing happens in the background when connectivity exists.
- **Offline-tolerant.** A node that goes offline for a week reconnects and catches up; messages from the past week sync at reconnection. There is no notion of "online users" in the way there is in real-time messaging.
- **Cryptographically signed.** Every message is signed by its author; logs are append-only and tamper-evident.
- **Social-graph-bounded.** You only see messages from people in your social graph, transitively. There is no global feed.

This shape of system is profoundly different from a routing-protocol mesh. Reticulum tries to deliver a packet *now* by finding a path *now*. Scuttlebutt doesn't care whether the packet is delivered now; it cares whether, in aggregate, the social-graph-relevant messages eventually reach you. Different problems, different solutions.

## Why this design exists

Scuttlebutt came out of a specific community — initially the Sailing Scuttlebutt liveaboard community, where literal sailors with intermittent Internet on boats wanted a way to keep up with friends without depending on the cloud — and the design reflects the use case. The protocol was designed for environments where the network is *not always there*. It assumes intermittence as the default operating condition, not an edge case.

This makes it useful for cases that real-time mesh routing protocols are bad at:

- **Distributed social applications** where users come and go and "is this person online" is not a load-bearing question.
- **Asynchronous coordination** where the relevant unit of communication is a post, not a packet.
- **Sneakernet-tolerant deployments** where you sync logs by carrying USB drives between locations.
- **Long-term archival distribution** where the goal is "everyone in this community ends up with a copy of these documents."

It is *not* useful for:

- **Real-time messaging.** SSB messages can take seconds, minutes, or longer to propagate; latency depends on whether your peers happen to be online, and the protocol does not try to optimize for this.
- **Targeted delivery to a specific recipient.** SSB is gossip-shaped: messages flow to everyone in the relevant subgraph, not to a specific address. Private messages exist (SSB has encrypted private messages between identities) but they are still gossiped — encrypted-blob-shaped, but in everyone's logs who is in the propagation path.
- **High-throughput data movement.** The protocol is designed for human-scale write rates. It will happily carry your social posts. It will not happily replace your file server.

## The state of Scuttlebutt in 2026

Honest accounting:

The Scuttlebutt protocol's peak adoption was probably around 2019–2021, when the Manyverse client made the network accessible from a phone and the broader "we're tired of centralized social media" wave of the late 2010s gave it cultural energy. The network was not large in absolute terms — peak active-user counts in the tens of thousands range — but it was unusually engaged, with real communities forming around shared interests and a culture that took the local-first social vision seriously.

Since around 2022, the network has been smaller. Some of the user attrition was natural: people who liked the idea but didn't sustain the habit. Some was ecosystem fragmentation: experiments with different SSB-protocol variants (the original "ssb-classic" log format, an evolved "ssb-bendy" format, and a divergent "p2panda" project that took some of the same ideas in a different direction) splintered the community. Some was the maturation of the broader fediverse — Mastodon and ActivityPub absorbed a lot of the "decentralized social media" energy, and ActivityPub's *almost*-local-first, *almost*-peer-to-peer compromise was easier to onboard onto.

Where things stand in 2026: **Manyverse keeps the client side alive.** The Manyverse Android and desktop applications continue to be maintained and remain a usable way to access the network. The network is functional. New posts arrive, friends still gossip, the basic operations work. But it is a smaller community than its 2020 peak, and a reader showing up in 2026 will find a network where the response time on a new post is measured in minutes-to-hours rather than seconds, and where some of the once-active feeds are now mostly silent.

This is not the cjdns-and-Hyperboria pattern, where the network is essentially historical. Scuttlebutt in 2026 is genuinely active, just smaller than it once was. A reader who is interested in the protocol shape, in distributed social applications, in offline-tolerant gossip — there is real value in installing Manyverse, joining a few pubs (the gateway nodes that bootstrap new users into the network), following the active feeds, and seeing how the system behaves.

## A worked example: setting up an SSB node

The most accessible way to see Scuttlebutt in 2026:

1. Install Manyverse on your phone (Android or iOS) or desktop (the project ships a desktop version too).
2. Generate an identity. The app does this for you — it generates an Ed25519 keypair and your SSB ID is the public key, base64-encoded with an `@` prefix.
3. Connect to a *pub*. A pub is a long-running server that participates in the SSB network and helps new users bootstrap by gossiping with them and onboarding them into the broader social graph. The Manyverse client comes with a list of public pubs; pick one and request an invite (most have automated systems for this).
4. Follow some people. Manyverse will discover what other identities the people you follow follow, and gradually your local copy of the network grows.
5. Post something. The post is signed, appended to your local log, and gossiped to your peers next time they sync.

At this point you have a working SSB node, and the basic phenomena of the network — eventual consistency, social-graph-scoped visibility, asynchronous sync — are visible to you in everyday use.

## What gossip protocols teach about routing protocols

The structural lesson here, the one this chapter exists to deliver: **gossip and routing are different tools for different problems, and conflating them produces bad designs in both directions.**

A routing protocol's job is to deliver this packet to that destination, ideally now, ideally over a path that is short or cheap or both. The protocol must do work to discover the path and keep it valid. The cost is overhead and the benefit is targeted, low-latency delivery.

A gossip protocol's job is to ensure that, eventually, every participant in the relevant scope sees every relevant message. The protocol does not care which specific pair of nodes a message travels between, in what order, or with what latency, as long as it eventually reaches everyone. The cost is high redundancy and unbounded latency; the benefit is robustness to topology change and tolerance for arbitrary connectivity patterns.

A network that wants to deliver "alice messages bob in real time, low latency, addressed delivery" should use a routing protocol. A network that wants to deliver "everyone in this community sees all the posts in this community, eventually" should use gossip. Trying to use a routing protocol for the gossip-shaped problem produces a system that is brittle in poor connectivity. Trying to use a gossip protocol for the routing-shaped problem produces a system that is wasteful and slow.

Some applications need both, and the best designs in this space recognize that and use each tool for the job it is good at. Reticulum, for example, has *channels* (routing-protocol-shaped, real-time messaging) *and* an explicit gossip mode for slow-substrate broadcast. The two are not in tension; they are complementary.

## Where to go next

- **Manyverse** at `manyver.se` is the entry-point client.
- **The Scuttlebutt protocol guide** at `scuttlebot.io/more/protocols/secure-scuttlebutt.html` is dated but still the canonical protocol description.
- **Manyverse community pubs.** The Manyverse documentation maintains a list; pick one and request an invite.
- **p2panda** at `p2panda.org` is the spiritual successor experiment that took some of SSB's ideas in different directions; if you are interested in where this lineage is going next, that is one place to look.

## What to take from this chapter

You should now be able to:

- Explain why Scuttlebutt is not a mesh in the routing-protocol sense, and why it is mesh-adjacent in a way worth understanding.
- Recognize gossip as a distinct tool from routing — useful for different problems, with different cost structures, and not interchangeable.
- Decide whether the SSB shape is right for your use case (asynchronous, social-graph-scoped, eventually-consistent) or wrong for it (real-time, addressed, latency-sensitive).
- Have an opinion about whether SSB is alive enough in 2026 to be worth your time. The honest answer: yes for the protocol-curious, possibly not for the user looking for a daily-driver social network, but worth a weekend either way.

The next chapter shifts again, to the messaging-only mesh family — Briar, Bitchat, and Bluetooth-mesh applications.
