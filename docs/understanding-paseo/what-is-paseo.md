# What is Paseo?

Paseo is one program on your computer that runs AI coding agents, plus a set of remote controls (phone, web, desktop, terminal) that let you drive those agents from anywhere.

That's the whole idea. The rest of this page builds the mental model so the pieces stop feeling like magic. No commands here — just the picture. If a term is unfamiliar, it's defined in [key concepts](key-concepts.md).

---

## The problem it solves

AI coding agents like Claude Code and Codex normally run in a terminal on one computer, and you have to sit at that computer to use them. That's limiting: you can't check on a long-running task from your phone, you can't easily run a Claude agent and a Codex agent at once, and each tool has its own separate interface.

Paseo puts a single, consistent control surface in front of all of them, reachable from any device, while keeping the agents — and all your code — on your own machine.

---

## The mental model: a worker and its remote controls

Picture two halves.

**The worker (the daemon).** A small background program called the **daemon** runs on your computer. It is the only part that does real work: it starts agents, watches them, feeds them your instructions, and reads the files they produce. It keeps running even when you close every app. "Daemon" just means a quiet background helper — you never click on it directly.

**The remote controls (the clients).** The phone app, the web app, the desktop app, and the `paseo` terminal command are all **clients**. A client does no real work. It sends your requests to the daemon and displays whatever the daemon reports back. Several clients can be connected at once — start a task on your desktop, watch it finish from your phone, all driving the same daemon.

This split is the single most important thing to understand. Once you see it, two things that look like magic become obvious:

- **"How does my phone run agents on my laptop?"** It doesn't. Your laptop's daemon runs the agents. Your phone is a remote control sending instructions to that daemon.
- **"How does Paseo know about coding chats I started myself?"** Because those chats are saved as files on the same disk the daemon runs on. The daemon can simply read them. (Details: [how Paseo finds your sessions](how-paseo-finds-your-sessions.md).)

---

## Architecture at a glance

```
   YOUR DEVICES (clients)              THE RELAY                YOUR COMPUTER
  ┌─────────────────────┐         ┌──────────────┐        ┌──────────────────────┐
  │  📱 phone app        │         │              │        │                      │
  │  💻 desktop app      │◄───────►│ relay.paseo  │◄──────►│   Paseo daemon       │
  │  🌐 web app          │  (when  │  .sh         │        │   (the worker)       │
  │  ⌨️  paseo CLI       │  remote)│              │        │      │               │
  └─────────────────────┘         │ forwards     │        │      ├─ starts/watches│
            ▲                      │ sealed       │        │      │  AI agents     │
            │                      │ messages,    │        │      │   (claude,     │
            │ (when on the same    │ can't read   │        │      │    codex, …)   │
            └─ machine/network)────┤ them         │        │      │               │
               connects directly   └──────────────┘        │      └─ reads agent  │
               to the daemon                               │         files on disk│
                                                           └──────────────────────┘
```

Two connection paths, same daemon:

- **Direct** — When a client is on the same computer or the same trusted network, it talks to the daemon straight (for example `localhost:6767`). Nothing else involved.
- **Relay** — When your phone is somewhere else (cellular, a coffee shop), it can't reach your home daemon directly. So the daemon connects *out* to the public relay server, and your phone meets it there. The relay forwards messages but cannot read them, because they're sealed with [end-to-end encryption](how-pairing-works.md).

The relay matters because it means **no firewall changes, no open ports, no networking setup**. The daemon reaches out; you never have to let the outside world in.

---

## Two kinds of agents Paseo shows you

Paseo's agent list mixes two sources. Telling them apart removes most of the remaining confusion.

| Kind | Where it came from | Where its record lives | How Paseo sees it |
| ---- | ------------------ | ---------------------- | ----------------- |
| **Paseo-launched** | You started it from a Paseo client | `~/.paseo/agents/` | The daemon launched it, so it tracks it live |
| **External session** | You ran `claude` or `codex` yourself in a terminal | The provider's own folder (`~/.claude`, `~/.codex`, …) | The daemon reads the files the CLI saved, and offers them for you to import |

Only the second kind feels like "magic," and it isn't: the daemon is reading files that already sit on your disk. See [how Paseo finds your sessions](how-paseo-finds-your-sessions.md) for exactly how.

---

## How it all fits together: a typical day

You open the Paseo app on your laptop and start a Claude agent: "add tests to the checkout module." The daemon launches a real Claude Code process, hands it your instruction, and streams the output back to your screen.

You head out. On the train, you open Paseo on your phone — already paired — and it reconnects to your laptop's daemon through the relay. The agent has been working the whole time (the daemon never stopped). You read its progress and send a follow-up: "also handle the empty-cart case."

Later you remember a Codex chat you ran by hand last week in a terminal. In Paseo you open the list of recent sessions, spot that Codex conversation (the daemon found it by reading Codex's own files), and import it. Now you can continue it from the same app, alongside everything else.

Three different devices, two different providers, one daemon doing all the work on your machine.

---

## What Paseo is *not*

- **Not a cloud AI service.** Paseo runs no models and sells no inference. It drives the agent tools you already installed, using your existing accounts and keys. Your code never leaves your machine.
- **Not a replacement for your agent CLIs.** It wraps them. You still need `claude`, `codex`, or another provider installed and logged in. Paseo does not handle their sign-in.
- **Not automatically watching all your chats.** The daemon can *read recent session files when asked*, and you *import* the ones you want. It does not silently absorb every conversation. (This distinction matters — see [how Paseo finds your sessions](how-paseo-finds-your-sessions.md).)
- **Not exposing your machine to the internet by default.** The daemon binds to your own computer only. Remote access goes through the encrypted relay, which you opt into by pairing. See [security](security.md).

---

## Where to go next

- Get it running: [quickstart](quickstart.md)
- The QR code, demystified: [how pairing works](how-pairing-works.md)
- The session "magic," demystified: [how Paseo finds your sessions](how-paseo-finds-your-sessions.md)
- The safety story in full: [security](security.md)
