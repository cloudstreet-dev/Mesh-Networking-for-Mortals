# Briar, Bitchat, and the Bluetooth Family

The previous chapters were about networks designed to span distances — kilometers in Meshtastic's case, the world in Reticulum's and Yggdrasil's. This chapter is about networks designed to span *rooms*. When the problem you actually have is "two people in the same building need to communicate without infrastructure" — which is the problem at protests, in disaster zones, in places where cellular networks are down or untrusted, in subway tunnels, in dense urban environments where the next person might be 30 meters away but might also be 30 floors up — the right tool is not a LoRa mesh or an IPv6 overlay. It is short-range Bluetooth-mesh messaging.

This is a small-scope but real category. The two main projects in 2026 are Briar and Bitchat, and they make different tradeoffs that are worth seeing in contrast.

## The shape of the problem

Bluetooth-mesh messaging exists because of one observation: every smartphone has Bluetooth on already, and Bluetooth (especially BLE) can do peer-to-peer discovery and message exchange without any infrastructure. If two phones are within ~10 meters of each other, they can talk. If three phones form a chain, the message can hop from end to end. If a hundred phones are in the same building, you have a small mesh that requires no carrier, no AP, no LoRa hardware — just the phones already in everyone's pocket.

The constraints are real:

- **Range is limited.** 10–15 meters indoors through one wall, 20–50 meters outdoors line-of-sight. BLE long-range coding (Coded PHY) can extend this somewhat, but you're still in tens-of-meters territory, not kilometers. Going outside the building means leaving the network.
- **Battery cost is real.** Continuous Bluetooth scanning and advertising is not free. Apps in this category trade off discovery aggressiveness against battery; in heavy use you can expect double-digit-percent additional battery drain on a typical smartphone.
- **Throughput is modest.** BLE's effective application-layer throughput is in the tens-to-low-hundreds of kilobits per second under good conditions, much less in mesh-shaped traffic. Plenty for messaging and small attachments; useless for video.
- **iOS Bluetooth APIs are restrictive.** Background BLE on iOS is tightly limited and apps in this space have to make compromises (foreground operation, manual triggering, paired-vs-unpaired tradeoffs) that the Android side doesn't have. Cross-platform Bluetooth-mesh apps generally work better on Android.

The good news is that within the constraints, the experience can be quite usable. Sending a text message to someone across a crowded venue, with no carrier, no Wi-Fi, no LoRa hardware — that works, and it works in a way that no other category of project in this book can match because no other category of project has the user already carrying the radio in their pocket.

## Briar

**Briar** is the older and more carefully-engineered of the two. It is an Android (and recently desktop, beta) messaging application that supports communication over Bluetooth, local Wi-Fi, and Tor — choosing whichever transport is available, transparent to the user. The project has been quietly ongoing since around 2014, with steady releases, a small but consistent contributor base, and a security model designed for users with serious threat models.

Key design choices:

- **End-to-end encryption with forward secrecy.** Briar uses BTP (the Briar Transport Protocol), a custom protocol with strong security properties: messages are end-to-end encrypted, the protocol provides forward secrecy, and metadata leakage is minimized.
- **No central server.** Briar contacts are exchanged directly between users (in person via QR code, or remotely via a trusted introducer). There is no Briar.com server running an account directory.
- **Tor as an Internet transport.** When Bluetooth and local Wi-Fi aren't available, Briar can route messages over Tor to other Briar users who are also reachable via Tor. This gives the project its core value proposition: a single app where messages travel over Bluetooth when you're nearby, local Wi-Fi when you're on the same network, and Tor when you're far apart — all without the user having to choose.
- **Forum and blog primitives.** Beyond direct messaging, Briar has group conversations and a forum-shaped feature where members can post threaded discussions. This is unusual for a Bluetooth-mesh app and reflects the project's broader vision: not just "messaging when infrastructure is down," but "a self-contained social-and-coordination platform that works without central infrastructure."

Briar's threat model is genuinely serious. The project documentation explicitly addresses adversaries with the ability to monitor network traffic, run network infrastructure, and so on. It has been used in real adversarial contexts — journalists working under regimes hostile to a free press, activists in countries where messaging apps are surveilled. The cryptographic design has been informed by academic work and the project has had reviews (though, like Reticulum, no large-scale formal audit at the time of writing).

What Briar gets wrong, or is honestly limited about:

- **iOS support.** As of 2026, Briar's iOS support remains limited compared to Android. The Bluetooth-mesh limitations on iOS make it hard to provide the same experience.
- **Setup friction.** The "exchange contacts via QR code in person" step is the right answer for the threat model, but it's not what most users expect from a messaging app, and it limits adoption.
- **Smaller community.** Real users measure in tens of thousands globally, not millions. Most messaging apps in your contact list will be Signal, WhatsApp, or iMessage; almost none will be Briar.
- **Throughput is modest and the UX reflects that.** Messages can take seconds to deliver even between adjacent nodes. This is the protocol working correctly, not a bug, but it is not the always-instant feel of carrier-mediated messaging.

License: GPLv3. Repository: `code.briarproject.org/briar/briar`. The project is alive in 2026, with regular releases, though development pace is steady-not-fast.

## Bitchat

**Bitchat** is the more recent (early 2020s) entrant in this space, focused specifically on Bluetooth-mesh messaging without the Tor and broader-platform ambitions of Briar. It is, in essence, a more focused take on the same problem: an open-source app that turns smartphones into a Bluetooth-mesh, prioritizing simplicity and the immediate "send a message at a protest" use case.

Compared to Briar:

- **Narrower scope.** Bluetooth-mesh only; no Tor, no Wi-Fi-LAN transport, no forum primitives. Just messages over BT-mesh.
- **Easier to onboard.** Without the in-person contact-exchange model, a user can install and start using Bitchat with less friction. The tradeoff is a less rigorous trust model.
- **Cross-platform from the start.** iOS and Android both supported, with the iOS-side limitations the platform forces on the project.
- **Smaller team and project than Briar.** Real but small.

The honest positioning: **Bitchat is the easier door for a curious user to walk through; Briar is the right door for a serious user with a specific threat model.** Both are alive in 2026. Both have specific use cases where they're the right answer.

## Bluetooth Mesh (the standard) and the broader BT-mesh space

For completeness: there is an actual *Bluetooth Mesh* specification, ratified by the Bluetooth SIG, designed primarily for IoT use cases (lighting, smart-building, sensor networks). It is a real protocol with real deployments, mostly invisible to consumers and unrelated to the messaging-app category this chapter is about. Apps like Briar and Bitchat use Bluetooth (specifically BLE) but generally implement their own meshing on top of it rather than using the Bluetooth Mesh specification, because the spec's design doesn't fit the messaging use case very well.

If you encounter a smart bulb, a door lock, or an industrial sensor that talks "Bluetooth Mesh," that is the spec. The mesh-messaging apps are doing something different over the same radio, and the two should not be confused.

## When to actually pick one of these

The honest cases:

1. **You expect to need messaging in environments without infrastructure.** Protests, festivals, disaster scenarios, dense urban environments where cellular is unreliable. Briar or Bitchat installed in advance is real preparation; neither is useful as a thing you install in the moment of crisis.
2. **You have a serious threat model.** Briar, specifically, with the in-person contact-exchange ritual, is the right tool. Most readers of this book do not have this threat model, but the ones who do should know about Briar.
3. **You're curious about Bluetooth-mesh and want to feel it.** Install Bitchat on two phones in the same room. Send messages. Walk apart until the signal drops. The phenomenon is interesting and the install cost is small.

If none of those describe you, this category is probably not load-bearing for your use case. It is worth knowing exists.

## What this category teaches

The structural lesson here, for a book that has been mostly about long-distance mesh networking, is this: **physical layer choice is not just about "how far can the network reach." It is about who can join.**

Meshtastic requires LoRa hardware. Reticulum requires deliberate setup, often hardware. Yggdrasil requires an Internet connection. *Bluetooth-mesh requires only a phone that is already in everyone's pocket.* That difference — the absolute zero-marginal-cost participation barrier — is what makes Bluetooth-mesh applications viable for use cases like protests where every additional hour of setup is an hour the participants don't have.

The cost is range. The benefit is reach measured in *people who can join right now*, which is a different and sometimes more important metric than range measured in kilometers.

A complete mesh-networking stack for serious use ought to have something in this category and something in the LoRa-mesh category, used for different purposes: Bluetooth-mesh for the rooms-full-of-people scenario, LoRa for the people-on-different-hilltops scenario, with bridges between them where it makes sense (Reticulum is again the project that takes this seriously).

## Where to go next

- **Briar** at `briarproject.org` — the project page, install link, threat model documentation.
- **Bitchat** — the project repository at `github.com/permissionlesstech/bitchat` (verify before relying on for a long-term plan; sub-projects in this space have moved repos historically).
- **Bluetooth Mesh specification** — Bluetooth SIG, mostly relevant if you're working on IoT rather than messaging. Linked from `bluetooth.com`.

## What to take from this chapter

You should now be able to:

- Recognize Bluetooth-mesh messaging as a different category of project from LoRa-mesh and IP-overlay-mesh, with different strengths and weaknesses.
- Decide between Briar (serious threat model, broader platform integration via Tor) and Bitchat (lighter-weight, BT-only, easier onboarding).
- Articulate the structural reason this category exists: when the radio in everyone's pocket is the substrate, the network can have orders of magnitude more participants than any LoRa or VPN-based alternative.
- Name when this category is the right tool (rooms-full-of-people scenarios, infrastructure-down events) and when it isn't (regional networks, real-time low-latency comms across distances).

The next chapter is the last category: mesh VPNs. Tailscale, Nebula, ZeroTier, Headscale. The tools you might already use at work, and what makes them legitimately mesh-shaped without being mesh networking in the same sense as the rest of this book.
