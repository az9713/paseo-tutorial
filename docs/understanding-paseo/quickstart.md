# Quickstart

Get Paseo installed and your phone connected to your computer's agents in about 10 minutes. Every command is copy-pasteable. Terms in **bold** are defined in [key concepts](key-concepts.md).

This works the same on macOS, Linux, and Windows (use Git Bash on Windows so the commands match).

---

## Before you start

You need two things installed. Check each with the verify command:

**1. Node.js 22 or newer** — the runtime Paseo is built on.

```bash
node --version
```

Expected: `v22.x.x` or higher. If it's missing or older, install from [nodejs.org](https://nodejs.org/download).

**2. At least one agent CLI** — the actual AI tool Paseo will drive. You need *one* of these, installed and logged in:

```bash
claude --version      # Claude Code
codex --version       # Codex
opencode --version    # OpenCode
```

Expected: any one of them prints a version. If none are installed, set one up first — Paseo cannot run an agent you don't have. Paseo does **not** handle these tools' sign-in; log into your chosen one by running it once on its own.

> **Why this matters:** Paseo has no AI of its own. It orchestrates the agent CLIs you already have, using your existing accounts. No agent CLI means nothing for Paseo to run.

---

## Step 1 — Install the Paseo CLI

```bash
npm install -g @getpaseo/cli
```

The `-g` means "install globally" so the `paseo` command works from any folder. This adds about 290 small packages and takes under a minute.

Verify it:

```bash
paseo --version
```

Expected:

```
0.1.90
```

---

## Step 2 — Start the daemon and show the pairing code

```bash
paseo onboard
```

This is the all-in-one first-run command. It starts the **daemon** (the background worker), waits until it's ready, then prints a **QR code** and a pairing link. It asks once whether to enable voice features — answer **No** unless you want it to download local speech models.

Expected (shortened):

```
Welcome to Paseo
Daemon started (PID 12345)
Daemon ready on 127.0.0.1:6767

Scan to pair:
█▀▀▀▀▀█ ▀▄ █▀ █▀▀▀▀▀█
█ ███ █ ▀█▄▀▀ █ ███ █
█ ▀▀▀ █ █ ▄▀█ █ ▀▀▀ █
▀▀▀▀▀▀▀ █▄▀▄█ ▀▀▀▀▀▀▀
...
https://app.paseo.sh/#offer=...
```

Leave this terminal window open. The daemon keeps running here.

> **Tip:** Already past the welcome screen and need the code again later? Run `paseo daemon pair` in any terminal to reprint it.

---

## Step 3 — Connect your phone

1. Install the **Paseo** app on your phone — from the App Store / Play Store, or open [app.paseo.sh](https://app.paseo.sh) in your phone's browser and add it to your home screen.
2. In the app, choose to add a host / scan a code.
3. Point your phone's camera at the QR code in your terminal.

Your phone connects to your computer's daemon through the encrypted **relay**, so this works even on cellular — the two don't need to be on the same network. What actually happens in this step is explained in [how pairing works](how-pairing-works.md).

> **The QR code is a secret.** It carries the key that lets a device connect to your daemon. Don't post it publicly or paste it into a chat.

---

## Step 4 — Run your first agent

From your phone (or the same terminal), start an agent. From the terminal it looks like this:

```bash
paseo run --provider claude/opus-4.6 "list the files in this folder and summarize what this project does"
```

Expected:

```
Agent started: agt_a1b2c3
Provider: claude   Status: running
```

The agent runs on *your computer*, inside whatever folder you're in. Watch it live:

```bash
paseo attach agt_a1b2c3
```

Or just watch it on your phone — same agent, both screens update together.

---

## What just happened

You installed a background worker (the daemon) and connected a remote control (your phone) to it over an encrypted channel. When you launched an agent, the daemon started a real Claude Code process on your machine and streamed its output to every connected client at once. Close the app and the agent keeps going — the daemon doesn't stop.

You now have the whole Paseo model running: one worker on your machine, controllable from anywhere.

---

## Useful commands

These work from any terminal — each one quietly connects to the daemon:

```bash
paseo ls                 # list your agents
paseo daemon status      # health check (daemon up? providers detected?)
paseo attach <id>        # stream an agent's live output
paseo send <id> "..."    # send a follow-up instruction
paseo daemon pair        # reprint the pairing QR + link
```

A healthy `paseo daemon status` looks like this:

```
KEY               VALUE
Local Daemon      running
Listen            127.0.0.1:6767
CLI               0.1.90
Providers
  Claude          .../claude  (detected)
  Codex           .../codex   (detected)
  OpenCode        .../opencode (1.15.13)
```

---

## Next steps

- Understand what you just connected to: [what is Paseo?](what-is-paseo.md)
- Understand the QR scan you just did: [how pairing works](how-pairing-works.md)
- Pull in chats you started by hand: [how Paseo finds your sessions](how-paseo-finds-your-sessions.md)
- Lock it down before exposing it anywhere: [security](security.md)

---

## If something goes wrong

**`paseo: command not found`** — npm's global folder isn't on your PATH. Close and reopen your terminal. If it persists, run `npm config get prefix` and make sure that folder's `bin` (Windows: the folder itself) is on your PATH.

**`Cannot connect to daemon`** — the daemon isn't running. Start it with `paseo daemon start`, then retry.

**A provider shows `--version failed` in `paseo daemon status`** — Paseo found the agent CLI but couldn't read its version (common on Windows with `.EXE`/`.CMD` wrappers). It will usually still run. Confirm by running that CLI directly (e.g. `claude --version`); if that works on its own, the agent will work in Paseo too.

**An imported session doesn't update when I type in my laptop's `claude`/`codex` terminal** — expected, not a bug. Import takes a one-time snapshot and then runs Paseo's *own* agent process; your separate terminal is a different process that Paseo doesn't watch. Either send follow-ups **from Paseo** (so everything stays in sync), or, if you keep typing in the terminal, manually pull the latest disk state with `paseo agent reload <agent-id>` (one-shot, not live). Full explanation: [how Paseo finds your sessions → importing is not live mirroring](how-paseo-finds-your-sessions.md#importing-is-not-live-mirroring). There is no setting for automatic live sync from an outside terminal.
