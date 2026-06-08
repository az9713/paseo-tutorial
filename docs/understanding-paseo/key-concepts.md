# Key concepts

Every term used in these docs, defined in plain English. Read this once and the rest will make sense. Terms are grouped by topic, not alphabetized, so related ideas sit together.

---

## The core split

**Daemon** — A small program that runs quietly in the background on your computer and does the actual work: starting agents, watching them, and reading files. "Daemon" is an old computing word for a background helper process that you don't interact with directly — you talk to it through something else. In Paseo, that something else is a client. The daemon is the heart of Paseo; everything else is a remote control for it.

**Client** — Any app you use to control the daemon: the phone app, the web app at app.paseo.sh, the desktop app, or the `paseo` command in your terminal. A client never does the real work itself — it sends requests to the daemon and shows you what the daemon reports back. You can have several clients connected to one daemon at the same time.

**Client-server** — The name for this split: one program does the work (the "server," here the daemon) and other programs ask it to (the "clients"). Paseo borrows this exact model from Docker, a popular developer tool built the same way. If you've never heard of either, the everyday analogy is a TV (the daemon) and its remotes (the clients): the TV does the work; any remote in the house can drive it.

---

## Agents and providers

**Agent** — One running AI coding assistant working on one task, in one folder, with one model. When you tell Paseo "fix the login bug in my project," it launches an agent to do that. An agent has a conversation history (its "timeline"), a status (working, waiting, finished), and a folder it works in. One agent = one job.

**Provider** — The brand of AI agent behind the scenes. Paseo doesn't have its own AI; it drives the agent tools you already installed. The built-in providers are **Claude Code**, **Codex**, **GitHub Copilot**, **OpenCode**, **Pi**, and **OMP**. You pick a provider (and a model within it) per agent, so you can run a Claude agent and a Codex agent side by side.

**Model** — The specific AI brain a provider offers, like `opus-4.6` (a Claude model) or `gpt-5.4` (a Codex model). One provider usually offers several models; you choose which one each agent uses.

**CLI** — Short for "command-line interface": a tool you run by typing commands in a terminal instead of clicking buttons. Each provider ships its own CLI (the `claude` command, the `codex` command, and so on). Paseo runs these CLIs for you. Paseo also has its own CLI, the `paseo` command, which is one of the clients.

---

## Sessions and discovery

**Session** — A saved conversation with an agent. This word means two slightly different things depending on who's talking, so this guide keeps them apart:

- **Provider session** — The conversation history that an agent CLI saves on its own. When you run `claude` in a terminal, Claude Code writes your chat to a file on your disk. That file is a provider session. It exists whether or not Paseo is involved.
- **Client session** — The live connection between one client and the daemon. Internal plumbing; you rarely think about it.

In these docs, "session" means **provider session** (the saved conversation on disk) unless stated otherwise.

**Import** — The act of telling Paseo to adopt a provider session you started elsewhere. If you chatted with `claude` in a terminal yesterday, that conversation is sitting in a file. Importing it pulls that conversation into Paseo so you can continue it from the app. Import is always something *you* choose to do — Paseo never silently takes over your conversations. See [how Paseo finds your sessions](how-paseo-finds-your-sessions.md).

**Stop (interrupt)** — Halting an agent's *current turn* without ending it. The agent and its conversation stay; you can send a new message. It's a pause, not a removal. In the app it's the ⏹ button that replaces Send while the agent is running.

**Archive** — Ending an agent and removing it from your active list, while keeping its record on disk so it can be brought back later (a "soft delete"). This is the everyday "I'm done with this" action; on the phone you trigger it by closing the agent's tab. Archiving stops the agent's process. It is **not** the same as **delete**. See [ending and removing a session](how-paseo-finds-your-sessions.md#ending-and-removing-a-session).

**Delete** — Permanently removing an agent record. Unlike **archive**, it cannot be undone. Done from the command line with `paseo delete`.

**`PASEO_HOME`** — The folder where the daemon keeps its own data: the agents *it* launched, its settings, its logs, and its identity key. By default this is `~/.paseo` (a hidden `.paseo` folder in your home directory). It is separate from where the provider CLIs save their sessions (Claude uses `~/.claude`, Codex uses `~/.codex`).

---

## Connecting and pairing

**Pairing** — Connecting a client (usually your phone) to the daemon for the first time by scanning a QR code or opening a link. Pairing sets up a secure channel so the two can talk safely. See [how pairing works](how-pairing-works.md).

**QR code** — The square barcode the daemon shows you to pair. Your phone's camera reads it. The QR is just a convenient way to carry a pairing link without typing it. **Treat the QR (and the link it contains) like a password** — it holds the key that lets a client connect to your daemon.

**Relay** — A small public server, run by Paseo at `relay.paseo.sh`, that acts as a meeting point so your phone can reach your daemon even when they're on different networks (phone on cellular, daemon at home). The relay passes encrypted messages back and forth but cannot read them. Think of it as a post office that forwards sealed envelopes — it sees the addresses, never the letters inside. See [security](security.md).

**Direct connection** — The alternative to the relay: a client connects straight to the daemon's network address (for example `localhost:6767` on the same machine, or a home-network address). Used mostly on the same computer or a trusted local network.

**Port** — A numbered "door" on your computer that a program listens at, so other programs know where to reach it. The Paseo daemon listens on port `6767` by default. The address `127.0.0.1:6767` means "this same computer, door 6767." (`127.0.0.1`, also called `localhost`, always means "this machine.")

---

## Security terms

**End-to-end encryption (E2EE)** — Scrambling messages so that only the two endpoints — your phone and your daemon — can read them. Anything in the middle, including the relay, sees only scrambled bytes. "End-to-end" stresses that the protection covers the *entire* path, not just part of it.

**Public key / private key (keypair)** — A matched pair of cryptographic keys. The **public key** is safe to share — it's like a padlock you hand out; anyone can use it to lock a box for you. The **private key** never leaves the daemon — it's the only key that opens those boxes. Paseo uses a keypair so your phone can lock messages that only your daemon can unlock, and vice versa. The QR code carries the daemon's *public* key (safe to show); the private key stays on your machine.

**Handshake** — The short back-and-forth at the start of a connection where the two sides agree on a shared secret to encrypt the rest of the conversation. Until the handshake finishes, the daemon refuses to act on any command. Step-by-step details are in [how pairing works](how-pairing-works.md).

**Loopback** — Another name for `127.0.0.1` / `localhost`: the network address that always loops back to your own machine and is never reachable from outside it. By default the daemon binds to loopback, meaning only programs on your computer can reach it directly.

**Password (optional)** — A shared secret you can set so that even a client on your machine (or one reaching the daemon over a direct network connection) must prove it knows the password before the daemon obeys it. Off by default; recommended if you expose the daemon beyond your own machine. See [security](security.md).
