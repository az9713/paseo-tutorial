# Security

Paseo is built so your code never leaves your machine, and so your phone can reach your daemon safely over the internet without you opening any ports. This page explains what's protected automatically, what becomes your responsibility if you go beyond the defaults, and the honest limits of the current design.

It's written for someone who wants to *deploy* Paseo confidently — on a laptop, a home server, or a rented machine — not just run it once. Terms in **bold** are in [key concepts](key-concepts.md). The cryptographic mechanics are explained once, in [how pairing works](how-pairing-works.md); here we focus on the threat model and the practical decisions.

This page is a plain-language companion to the project's canonical [`SECURITY.md`](../../SECURITY.md). Where they differ, the canonical file wins.

---

## The foundation: local-first

Paseo runs the **daemon** — the program that does the work — on your own computer. Agents execute there, in your folders, with your accounts. Paseo operates no cloud AI and stores none of your code. The only thing that ever travels off your machine is the encrypted control traffic between a client and the daemon, and even that is unreadable in transit.

---

## Two ways a client connects, and what protects each

| Connection | When it's used | What protects it |
| ---------- | -------------- | ---------------- |
| **Direct** | Client and daemon on the same computer or same trusted network | Network reachability (and an optional password) |
| **Relay** | Phone elsewhere — cellular, another network | Full **end-to-end encryption**; the relay can't read anything |

### The relay is assumed to be untrusted — and that's fine

When your phone reaches your daemon through the public **relay** (`relay.paseo.sh`), every message is sealed with end-to-end encryption. The relay forwards sealed envelopes; it cannot read them, alter them undetected, or impersonate either side. Paseo's design treats the relay as if it could be fully compromised and still holds up.

The step-by-step reason — the public/private keys, the handshake, exactly what a malicious relay can and can't do — is in [how pairing works](how-pairing-works.md). The short version: the daemon's **private key** never leaves your machine, so no one in the middle can build the shared secret that the encryption depends on.

> **The pairing QR / link is a secret.** It carries the daemon's public key, the anchor of that whole scheme. Anyone who gets it can start a connection to your daemon. Treat it like a password — don't post it or paste it into a chat.

---

## The local trust boundary (read this before exposing anything)

**By default, the daemon listens only on your own computer** — the **loopback** address `127.0.0.1`, reachable only from that same machine.

With no password set, the security boundary is simply *who can reach the daemon's address.* On your own machine, that's whatever runs on your machine. **Anything that can reach the daemon can control it.** This is the exact model the Docker tool uses for its own daemon, and it's safe as long as the daemon stays on loopback.

The moment you make the daemon reachable from elsewhere, that boundary changes and the responsibility becomes yours.

### If you expose the daemon beyond your own machine, it's on you

You "expose" the daemon when you do any of these:

- Bind it to `0.0.0.0` (all network interfaces) instead of loopback
- Forward it through a tunnel or reverse proxy
- Publish its port from a Docker container

In any of those cases, **set a password.** Without one, you've put an unauthenticated remote control on the network.

```bash
# Set a shared-secret password (stored hashed, never in plaintext)
paseo daemon set-password
```

Or via the `PASEO_PASSWORD` environment variable. Once set, every client must present the password before the daemon obeys it.

Two important caveats about the password:

- It is for **direct network exposure**. It is **not** a replacement for the relay's end-to-end encryption when crossing untrusted networks — for off-machine access, prefer the relay.
- Health checks (`GET /api/health`) are intentionally exempt so monitoring still works.

> **Simplest safe rule:** for remote access, use the **relay** (pair your phone) and leave the daemon on loopback. Only reach for direct exposure + password if you have a specific reason, and then always set the password.

---

## DNS rebinding protection (defense in depth)

Even on loopback, a clever malicious website could try a trick called **DNS rebinding** — pointing its own domain name at `127.0.0.1` to get your browser to talk to your local daemon on its behalf.

Paseo blocks this by checking the **Host header** — the "who did you mean to reach?" label on every request — against an allowlist. By default it accepts only `localhost`, `*.localhost`, and literal IP addresses; anything else is rejected with `403 Host not allowed`. You can add trusted hostnames via `hostnames` in `config.json` or the `PASEO_HOSTNAMES` environment variable.

This is a backup layer, not the main wall. It helps protect localhost from browser tricks; it does not replace keeping the daemon off the open network.

---

## Your agent credentials are never Paseo's business

Paseo drives agent CLIs (Claude Code, Codex, OpenCode, …) but **does not manage their logins.** Each tool handles its own authentication. **Paseo never stores or transmits your provider API keys.** Agents run in your user account with the credentials you already set up.

The practical consequence: log into each agent CLI yourself (run it once on its own), and your keys stay entirely between you and that provider.

---

## The honest limits

Good security docs state what *isn't* covered. For Paseo today:

- **In-session replay protection isn't implemented yet.** Across different sessions, replaying old encrypted messages is fully prevented (each session uses a fresh secret). *Within* one live session, the protocol uses random values but doesn't yet track message ordering to reject a replayed message. The channel is still encrypted and tamper-checked, so an outsider can't forge valid traffic — but don't market it as "fully replay-proof."
- **A connected client is a trusted operator.** Once a client is connected and (if set) authenticated, it acts with the daemon's authority — including reading files the daemon's user can read. Folder-relative paths in the UI are a convenience, not a security wall. Only pair devices you trust.
- **Exposing beyond loopback is entirely your responsibility.** Paseo gives you the tools (password, host allowlist, relay); it can't secure a network you opened.

---

## Deploying safely: pick your scenario

| You want to… | Do this |
| ------------ | ------- |
| **Use Paseo only on one laptop** | Nothing extra. Loopback default is the boundary. |
| **Control your laptop's agents from your phone** | Pair via the relay (the [quickstart](quickstart.md) flow). Leave the daemon on loopback. Guard the QR. |
| **Run the daemon on a home server / Mac Mini and reach it from your phone** | Same as above — pair via relay. The daemon reaching *out* to the relay needs no inbound ports opened. |
| **Run the daemon on a rented VPS and expose its port directly** | Set a password (`paseo daemon set-password`), put it behind TLS, and consider the host allowlist. Treat the machine as internet-facing. |

---

## Reporting a vulnerability

Found a security problem? Report it privately by emailing **hello@moboudra.com**. Don't open a public issue.

---

## Where to go next

- The cryptography behind the relay, step by step: [how pairing works](how-pairing-works.md)
- The full mental model: [what is Paseo?](what-is-paseo.md)
- The canonical, more technical source: [`SECURITY.md`](../../SECURITY.md)
