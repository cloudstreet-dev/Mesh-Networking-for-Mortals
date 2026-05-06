# Mesh Networking for Mortals

**A 2026 survey of mesh networking, scoped for the working engineer.**

**[Read online at cloudstreet-dev.github.io/Mesh-Networking-for-Mortals](https://cloudstreet-dev.github.io/Mesh-Networking-for-Mortals/)**

## About This Book

The available literature on mesh networking splits into two unhelpful piles: academic papers that assume you've read every prior academic paper, and project-website pitches that assume you're already sold. There is almost nothing in between for the engineer who has heard of half these projects, doesn't know how they compare, and wants an honest survey before they commit a weekend to playing with one.

This book is that survey, scoped to 2026.

The reader is a competent developer who has shipped real products. They know what TCP/IP is. They've maybe heard of Meshtastic from a maker friend, vaguely remember that cjdns was a thing in the 2010s, use Tailscale at work, and aren't sure whether any of those are actually the same kind of thing. They want to know which projects are alive in 2026, what each one is structurally good at, what it gets wrong, and which one to install on a Raspberry Pi this weekend if they want to actually feel mesh networking in their hands.

## About CloudStreet

CloudStreet is a catalog of short, opinionated technical books on topics that working engineers ought to understand and mostly don't. Each book is written to be read in a weekend and to leave the reader more dangerous than they were on Friday.

## AI Authorship

This book is written by Claude Opus 4.7 (Anthropic), with editorial direction and project management from the CloudStreet team. The byline is the model's own. We disclose this plainly because the alternative is dishonest, and because the work is good enough to defend on its merits.

## Building Locally

```sh
cargo install mdbook
mdbook serve --open
```

## Deploying

Pushes to `main` build the book and deploy to GitHub Pages via the workflow at `.github/workflows/deploy.yml` (peaceiris/actions-mdbook + actions/deploy-pages).

## License

CC0 1.0 Universal — public domain dedication. Take it, fork it, ship it, claim it as your own. See [LICENSE](LICENSE).
