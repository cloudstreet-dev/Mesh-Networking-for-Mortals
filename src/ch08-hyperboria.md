# The Hyperboria Story

This chapter is a memorial as much as a survey. cjdns and the Hyperboria network were, in the early 2010s, the most ambitious open-source mesh-networking project in the world. The vision was beautiful: a globally-routed, end-to-end encrypted, self-organizing IPv6-shaped network with cryptographic identity at the routing layer, growing organically from neighbor-to-neighbor links into a parallel Internet. The project did not pan out the way its proponents hoped. Honoring it requires being honest about that.

For a curious reader in 2026: cjdns is not the right project to install this weekend. The network is mostly historical. Yggdrasil (chapter 7) is what you should run instead, and the lessons cjdns left behind are baked into Yggdrasil and Reticulum. But the story is worth knowing, because the failure modes are instructive and the cultural moment matters.

## What cjdns was

cjdns — the "Caleb James DeLisle Network Suite" — was a software router and protocol that started in 2011, written by Caleb James DeLisle, and rapidly attracted a community of mesh-networking enthusiasts who built **Hyperboria** on top of it. The core technical ideas:

- **Cryptographic identity at the routing layer.** Every cjdns node has a Curve25519 public key. Its IPv6 address is derived from a hash of the public key (in the `fc00::/8` deprecated-but-pragmatically-useful range). Identities are unforgeable. This was the main thing cjdns got *right*, and the design has been broadly vindicated by every project that came after.
- **End-to-end encryption by default.** Same idea Reticulum and Yggdrasil eventually adopted. cjdns was the first widely-deployed implementation of this in a routing-layer context.
- **DHT-based location lookup.** This was the new idea. To route to a destination, cjdns would do a Kademlia-style DHT lookup against the destination's public-key hash; the DHT returned a *path* through the network expressed as a sequence of physical-link choices, and the source node would then *source-route* the packet along that path. The DHT was distributed across all nodes; no single node was authoritative.
- **Source routing with switches at every node.** Each cjdns node was both a "switch" (forwarding source-routed packets) and a "router" (originating source-routed packets to destinations it knew via the DHT). The conceptual separation was clean.

The combination — cryptographic identity, encryption by default, DHT-based lookup, source-routed forwarding — was novel for its time and remains technically interesting in 2026 even though the network it built has not.

## What Hyperboria was

Hyperboria was the public network that grew on top of cjdns. From roughly 2012 to 2016 it was the most prominent example of a "parallel Internet" project — a network of volunteer-run nodes connected over the public Internet (and, in some places, over local wireless), with services hosted in-network: chat servers, wikis, code forges, the kind of services-by-and-for-the-community infrastructure that the Tildes-and-Pleroma generation now builds on Tor and the Fediverse.

At its peak, Hyperboria had perhaps low-thousands of active nodes globally, a meaningful presence in Eastern European hacker communities, in some North American cities, and online via the EFnet-shaped IRC culture of the 2010s. There were real meetups. There were physical mesh deployments — rooftop antennas, neighborhood mesh nodes, the political-project flavor of mesh networking that motivated some of the cultural moment.

And then, slowly, it faded.

## Why it didn't work

The reasons are several and they compound. None of them is a single fatal flaw; together they were enough.

**The DHT was the wrong abstraction for high-churn networks.** Kademlia and DHTs in general work well for relatively stable participants — BitTorrent's tracker-less mode is the canonical example, where peers stay around for hours. cjdns's network had nodes appearing and disappearing constantly: people's home machines, hobbyist VPSes, mesh nodes that lost power. The DHT spent a lot of its bandwidth chasing route-lookup answers that had become invalid by the time they returned. Path lookups for a never-before-contacted destination could take noticeable seconds; lookups against a destination whose path had recently changed could fail and require retry. This was livable in good conditions and frustrating in bad ones, and the bad ones were common.

**Source routing meant every packet carried its own path.** This was elegant and had real engineering virtues — no per-hop routing decision, no re-convergence overhead — but it also meant that a packet's path was decided at the source and could not adapt to mid-flight changes. If a link in the middle of the path went down, the packet was dropped, and the source had to do another DHT lookup. In high-churn environments, this happened a lot.

**Single-developer governance.** cjdns was, for much of its life, a one-developer project — DeLisle himself. The work was good and prolific, but the bus factor was one, and the architectural decisions all came through a single channel. When DeLisle's attention shifted in the mid-2010s (he moved to other projects, including some Bitcoin and PKT work), the network's development pace slowed, and there was no clear successor leadership.

**The community split over priorities.** Some Hyperboria participants wanted to build a usable public network with real services. Others wanted to research routing theory. Others wanted to fork the codebase to do specific local-mesh deployments. The project had room for all of these but no shared roadmap, and the energy diffused into multiple smaller efforts rather than coalescing into a single sustained push.

**Yggdrasil happened.** In 2018, Arceliar (a long-time cjdns contributor) and Neil Alexander forked away from cjdns to build Yggdrasil, taking the parts they thought worked (cryptographic identity, end-to-end encryption, IPv6-overlay shape) and replacing the parts they thought didn't (the DHT-based location lookup, source routing). Yggdrasil's spanning-tree approach is, in significant part, a deliberate response to what they had learned didn't work in cjdns. Many of the technically engaged cjdns users moved to Yggdrasil over 2019–2022, and the cjdns network has been smaller since.

By the early 2020s, cjdns and Hyperboria had become a residual community. The network exists in 2026; nodes are reachable; some of the original infrastructure still runs. But the active community has largely either migrated to Yggdrasil or moved on, and recommending a curious reader install cjdns to "try mesh networking" would point them at a quiet network where the response time on questions is poor and the in-network services are mostly stale.

## What's still there in 2026

The cjdns codebase is still maintained, in a maintenance sense — security fixes happen, the build still works on current systems, there are still a small number of contributors. The Hyperboria network is still up; you can join it; you can find a few in-network services, an IRC server or two, some long-running personal pages. The Reddit and IRC communities are quiet but present. None of this is nothing.

But this is the wrong place to send a reader who wants to *feel* mesh networking. The hands-on experience of joining cjdns in 2026 is, mostly, the experience of joining a network of mostly-empty rooms. The technical interest is real and the historical interest is real and the "this is what a 2014 mesh-utopia project felt like" interest is real. The "this is a fun thing I'm going to play with this weekend" interest is not really there.

If you are reading this chapter to decide whether to install cjdns: don't, unless you specifically want to study the codebase or pay respects to the cultural moment. Install Yggdrasil instead. It is what cjdns was trying to be, with five years of additional engineering and an active community.

## What it left behind

The lessons cjdns left to the field are real, and the projects that built on those lessons are alive in 2026:

- **Cryptographic identity as the routing-layer address.** This is now a commonplace design choice. Reticulum, Yggdrasil, and even mesh VPNs (in the WireGuard-key-as-identity sense) all do it. cjdns was an early adopter and the design has been vindicated.
- **End-to-end encryption as default, not opt-in.** Now table-stakes for any new mesh project being taken seriously.
- **The IPv6-overlay shape.** The pattern of "give every node an IPv6 address derived from its public key, present a TUN interface to the OS, route invisibly underneath" comes from this lineage and lives on in Yggdrasil.
- **What does not work: DHT-based location lookup in high-churn networks.** This is the negative result, and it is genuinely valuable. The field has moved on from DHT-as-routing-layer toward path-discovery-with-caching (Reticulum) or spanning-tree (Yggdrasil) precisely because cjdns demonstrated, at scale, that DHT lookups were the wrong tool for a routing decision that had to happen at packet send time.
- **What does not work: source routing in dynamic networks.** Same lesson.
- **What does not work: single-maintainer governance for a globally-scaled network.** Yggdrasil and Reticulum have learned from this, with varying degrees of success — Reticulum is still effectively single-maintainer, with the same risk; Yggdrasil has a small but real core team. The lesson is acknowledged; the solution is partial.

## How to honor a project that didn't work

There is a kind of writing about open-source projects that did not pan out where the author leans hard on euphemism — "the project explored interesting territory," "the community has a smaller but dedicated presence," "the codebase is a wonderful learning resource." This is well-meaning and it is, at the margin, dishonest. It is also unhelpful to a reader trying to decide where to spend their time.

The straight version is: cjdns was a serious, ambitious, technically interesting project that did not become the network its founders hoped it would. The reasons it did not are partly engineering, partly community dynamics, partly the timing of when other approaches matured. The work was good. The lessons survived. The network did not.

That is okay. Most ambitious projects do not become what their founders hoped. The honest accounting of why is more useful to the field than a politely-vague summary that leaves the next ambitious project's founders with no way to learn from the failure modes.

Caleb James DeLisle, Arceliar, Neil Alexander, the Hyperboria community: thank you for the work. The next generation of mesh-networking projects sits on the shoulders of what you built and what you learned.

## What to take from this chapter

You should now be able to:

- Recognize cjdns and Hyperboria as a foundational project in mesh networking's open-source history without confusing it for a project to install today.
- Explain why DHT-based location lookup, beautiful in theory, did not survive contact with high-churn networks in production.
- Trace the line from cjdns to Yggdrasil and see what was kept, what was discarded, and why.
- Hold an opinion about how the field should write about projects that didn't pan out — not with disrespect, but not with euphemism either.

The next four chapters cover networks that are *adjacent to* mesh networking rather than examples of it strictly. We start with Scuttlebutt and the gossip family.
