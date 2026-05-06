# Pick One This Weekend

Eleven chapters of survey. One chapter of recommendations. This is the one where you decide what to actually install, and the goal is that you walk away knowing exactly what you're going to do, what hardware you need, what an evening looks like, and what you expect to feel when it's running.

The recommendations are indexed by goal, because the right project is genuinely different for different goals. Pick the row that matches yours.

## "I want to send messages off-grid with friends"

**Install: Meshtastic.**

This is what Meshtastic is for. It is the entry door for a reason.

**Hardware to buy:**

- Two Heltec LoRa V3 boards. ~$20–25 each. Make sure to pick the right frequency variant for your region (US-915 for North America/Australia/NZ, EU-868 for Europe, AS-923 for Japan/Korea, and so on; chapter 2 has the full table).
- Two matching antennas. ~$5–10 each. The antenna matters more than people realize. A bad antenna will halve your range.
- Two USB-C cables (probably already in your drawer).
- Optional: two 2000 mAh LiPo batteries (~$8–12 each) and 3D-printed cases.

Total: $80–150 for a complete two-node setup.

**What an evening looks like:**

1. Order the hardware Monday. It arrives Wednesday or Thursday.
2. Friday evening: open the boxes, plug board #1 into your computer.
3. Visit `flasher.meshtastic.org` in Chrome or Edge. Follow the flashing flow. Five minutes.
4. Install the Meshtastic app on your phone. Pair to board #1 over Bluetooth. Set the region. Pick a channel.
5. Repeat for board #2 with a friend's phone.
6. Send your first message. Feel the small, irrational delight of "this is going over a radio with no Internet involved."
7. Take one board outside, walk down the street with it, watch the signal degrade with distance and walls. Get a sense of what 250 bit/s of LoRa actually feels like.
8. The next morning: drive ten minutes apart with the two boards. Test whether you can still talk. The answer will probably be yes.

**What you'll learn:**

- That LoRa range numbers in the marketing copy are real but conditional. Line of sight matters. Antennas matter. Trees matter.
- That a two-node mesh is small but not nothing. Three nodes start to feel like a network.
- That the OLED screen on a Heltec V3 is, surprisingly, a satisfying thing to glance at when a packet arrives.

**What you won't get out of it:**

- A real understanding of routing. Meshtastic floods. You won't see a routing decision until you have many nodes. Chapter 3 is what you read for that.
- Production-grade encryption. The threat model is "casual eavesdropper," not "nation-state adversary."
- A scalable network. The 100-node-ceiling is real. For small-group use, this doesn't matter.

## "I want to understand mesh routing by running a node"

**Install: Yggdrasil on a VPS.**

Yggdrasil is the project that lets you *see routing happening* in a network with enough scale to make the property visible, with enough engineering to make the experience pleasant, and with low enough barrier to entry that you can do it from a cheap VPS.

**Hardware to buy:** None. You need a $5/month VPS — Hetzner, DigitalOcean, OVH, Vultr, your favorite.

**What an evening looks like:**

1. Spin up a small Linux VPS. Ubuntu 22.04 or 24.04 is fine.
2. `apt install yggdrasil` (it's in the standard repos in 2026).
3. `yggdrasil -genconf > /etc/yggdrasil.conf`. Edit the config to add three or four public peers from `github.com/yggdrasil-network/public-peers`. Pick geographically diverse ones.
4. `systemctl start yggdrasil`.
5. `ip addr show ygg0` — your Yggdrasil IPv6 address.
6. `ping6 200:6e7c:5f9c:...` against an in-network service from the public peers list. Watch it work.
7. While the network is bootstrapping, read [the Yggdrasil whitepaper](https://yggdrasil-network.github.io/whitepaper.html) and the routing-protocol section of chapter 3 of this book.
8. Run `yggdrasilctl getPeers` periodically. Watch new peers appear as the network discovers you.
9. The next day: try running `traceroute6` from your node to a few in-network destinations. See what the spanning-tree paths actually look like.

**What you'll learn:**

- What spanning-tree routing looks like in production. The fact that two nodes in the same city sometimes route through a third country, and why that's the protocol working as designed.
- What it feels like to be a node in a self-organizing IPv6 overlay. The network exists. You are now part of it. There is no central authority. This is a different feeling than running a Tailscale node.
- The difference between path-discovery latency on first contact (slow) and steady-state forwarding (fast).

**What you won't get out of it:**

- Off-grid mesh. Yggdrasil runs over the public Internet. When the Internet is down, your Yggdrasil network is down.
- Hardware-in-your-hand mesh. There is no LoRa, no radio, no physical thing on your desk. The mesh is virtual.

## "I want to build something that bridges substrates"

**Install: Reticulum (RNS) plus an RNode.**

Reticulum is the project that rewards investment, and "I want to build something" is the use case it's best suited for.

**Hardware to buy:**

- One RNode. ~$100 from the project store, or build your own from a LilyGo T-Echo or similar (~$40–60) using the published RNode firmware. The pre-built option is recommended unless you specifically enjoy building hardware.
- A Linux machine to run the RNS daemon. A Raspberry Pi 4 or 5 is ideal; any Linux laptop or VPS works too.

Total: $100–250.

**What an evening looks like:**

1. Get RNS running on your machine. `pip install rns`. Run `rnsd` once to generate the default config.
2. Plug in the RNode. Edit `~/.reticulum/config` to add an `RNodeInterface` for it. Restart `rnsd`.
3. Add a second interface — a `TCPClientInterface` to one of the public Reticulum testnet nodes (the project page has the current list).
4. Add a third interface — `AutoInterface` for your local WiFi.
5. Install Sideband on your laptop. Pair it to RNS. Watch your node appear on the network.
6. Send a message to a known public destination. Watch path discovery happen.
7. Open `~/.reticulum/storage` and look at the path table that's accumulating.
8. The next evening: write a 30-line Python script that uses the RNS API to send a packet. Use the worked example in chapter 6 as a template. Make it do something specific to your use case — broadcast a message every minute, listen for a particular destination, whatever.
9. The weekend after that: design and build the actual application you wanted to build. You will have everything you need.

**What you'll learn:**

- What it feels like to write code against a network stack where cryptographic identity, encryption, and routing are all the stack's job, not yours.
- What "transport-agnostic" means in practice when you can take an interface offline and watch the network keep working over the others.
- Why people who know this space recommend Reticulum for serious work.

**What you won't get out of it:**

- An immediate first-evening payoff. The bootstrap is longer than Meshtastic's. Plan for a weekend, not an evening.
- A polished consumer-app experience. Sideband and MeshChat are good, but they're closer to "good open-source app" than to "Apple-grade product."

## "I want to mesh-VPN my homelab"

**Install: Tailscale (or Headscale if you want self-hosting).**

This is the chapter-11 category. Mesh VPN. Different problem from the LoRa-shaped projects above.

**Hardware to buy:** None, beyond the machines you already want to network.

**What an evening looks like (Tailscale, hosted):**

1. `curl -fsSL https://tailscale.com/install.sh | sh` on each machine.
2. `sudo tailscale up` on each. OAuth-login the first time.
3. The machines are now on a flat, encrypted IPv4-and-IPv6 network. They can reach each other by tailnet name.
4. Install the phone app. Same OAuth login. Your phone is now on the network.
5. SSH from your phone to your home server. Without port forwarding. Without DDNS. Without an IP. This will, the first time, feel slightly like magic.

Five minutes, no exaggeration.

**What an evening looks like (Headscale, self-hosted):**

1. Install Headscale on a server with a public IP. (`apt install headscale` in 2026, or run from the Go binary.)
2. Configure with a hostname and TLS.
3. Set up your first user and pre-auth key.
4. On each client machine: `tailscale up --login-server=https://your-headscale.example.org`.
5. Same mesh, same experience, your control plane.

**What you'll learn:**

- What zero-trust networking actually feels like, after years of reading about it.
- That NAT traversal, in 2026, is mostly a solved problem if the right people have written the code for you.
- How much of the "VPN is annoying" experience was specifically a hub-and-spoke-VPN problem rather than an inherent VPN problem.

**What you won't get out of it:**

- Off-grid anything. Tailscale is a public-Internet overlay; it cannot operate without the Internet.
- Insight into the mesh-routing-protocol design space. Tailscale is structurally a different category from Reticulum and Yggdrasil; do not expect to learn about routing protocols from running it.

## "I want to send a message at a protest"

**Install: Briar (Android) or Bitchat (Android/iOS).**

Bluetooth-mesh messaging. Chapter 10 is the long version. The short version: install in advance, exchange contacts in advance, test in advance. The thing you do not want to do is install one of these for the first time during the event you need it for.

**Hardware to buy:** None. Just phones.

**What an evening looks like:**

1. Install the app on two or three phones (yours and a friend's, ideally).
2. Exchange contacts via QR code, in person.
3. Test sending messages between phones in the same room.
4. Walk apart. Watch range degrade. Get a sense of where the limits are.
5. Try with three or four phones in a chain — does B forward A's message to C when A and C are out of direct range?
6. Now you know what the tool actually does, and you have it ready for whenever it might matter.

**What you'll learn:**

- How short Bluetooth range really is.
- That a tool you already practiced with works dramatically better than a tool you're learning under stress.

## A summary table

| Goal | Project | Hardware | Cost | Time |
|---|---|---|---|---|
| Off-grid messaging with friends | Meshtastic | 2 × Heltec LoRa V3 + antennas | $80–150 | An evening |
| Understand mesh routing | Yggdrasil | $5/mo VPS | $5 | An evening |
| Build something serious | Reticulum + RNode | RNode + Pi or laptop | $100–250 | A weekend |
| Mesh-VPN a homelab | Tailscale | (existing machines) | $0 | 5 minutes |
| Self-hosted mesh-VPN | Headscale | A small VPS | $5/mo | An hour |
| Messaging at protests | Briar / Bitchat | (your phone) | $0 | An evening |

## Pick one. Then pick another.

The most useful framing for the working engineer reading this book: pick the goal that matches what you actually want, do the relevant install, *then* — when the immediate curiosity has been satisfied — pick a second one from the list and do it. The goals are not mutually exclusive. They are tools. Most people who get serious about this space end up running Tailscale for their homelab *and* a Meshtastic node on their desk *and* eventually a Reticulum stack for the project they decided to build.

The point is not which one you pick first. The point is to stop reading about mesh networking and start running some of it. The hardware is cheap, the software is free, the time investment is bounded, and at the end of the evening you will have done something that, ten years ago, would have been a research project.

That is the offer this book has been making. Take it.

## What to take from this chapter — and from the book

You should now be able to:

- Pick a project that matches your actual goal, with a clear sense of what hardware to buy, what an evening looks like, and what you'll learn.
- Distinguish the four uses of *mesh* and not be confused by vendor copy.
- Reason about routing protocols, physical-layer constraints, and the tradeoffs each project is making, well enough to read the project's own documentation productively.
- Have an opinion about which projects are alive and worth your time in 2026, which are dormant and should be honored without being installed, and why the difference matters.

Go install one. The hardware is on your desk by next Wednesday if you order tonight. The evening is yours.
