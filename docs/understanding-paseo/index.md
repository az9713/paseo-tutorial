# Understanding Paseo

Paseo lets you run AI coding agents (Claude Code, Codex, and others) on your own computer and control them from anywhere — your phone, another laptop, the terminal. This guide demystifies how that works, in plain language, assuming no prior experience with this kind of tool.

These docs are written for the curious newcomer who wants to *understand the magic*, not just copy commands. If you only want it running, the quickstart alone is enough.

---

## Two ways to read this

**Just want it running?**

| Start here | What you get |
| ---------- | ------------ |
| [Quickstart](quickstart.md) | Paseo installed and your phone connected in about 10 minutes |

**Want to understand the magic?** Read these three, in order:

| Read | What it explains |
| ---- | ---------------- |
| [What is Paseo?](what-is-paseo.md) | The mental model: one program on your machine, many remote controls |
| [How pairing works](how-pairing-works.md) | What actually happens when you scan that QR code — and why it's safe |
| [How Paseo finds your sessions](how-paseo-finds-your-sessions.md) | Why Paseo "knows about" the Claude/Codex chats you already started |

---

## Everything else

| Doc | What's inside |
| --- | ------------- |
| [Key concepts](key-concepts.md) | Every term used in these docs, defined in plain English |
| [Security](security.md) | The full safety story: what's protected, what's on you, and the honest limits |

---

## The one-paragraph version

Paseo is split into two halves. A **daemon** — a small background program — runs on your computer and does the real work: it starts coding agents, watches them, and reads the files those agents leave on your disk. Everything else (the phone app, the web app, the desktop app, the command line) is just a **remote control** that talks to the daemon. Scanning the QR code connects a remote control to the daemon over an encrypted channel. The daemon already knew about your agents — they live on the same machine it runs on. Once you see that split, nothing about Paseo is magic anymore.

> **New to the vocabulary?** Skim [key concepts](key-concepts.md) first. Every term below — daemon, agent, provider, pairing — is defined there before it's used elsewhere.
