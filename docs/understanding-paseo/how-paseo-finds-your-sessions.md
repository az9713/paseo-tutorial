# How Paseo finds your sessions

Paseo can show you coding chats you started yourself — like a `claude` or `codex` conversation you ran in a terminal — because those chats are saved as files on your disk, and the daemon reads them. There is nothing magic about it once you see where the files live.

This is the mechanism people most often confuse with the QR code. They are unrelated. This page demystifies it precisely. Terms in **bold** are defined in [key concepts](key-concepts.md).

---

## The misconception, corrected

It's tempting to think: *"I scanned one QR code and suddenly Paseo knew about all my agent sessions — it must be tracking everything."*

That's not what happens, in two ways:

**The QR didn't do it.** Scanning the QR connected your *phone* to the *daemon*. That's all. See [how pairing works](how-pairing-works.md).

**Paseo isn't tracking everything.** The daemon doesn't watch your every chat. When you *ask* (by opening the "recent sessions" / import view), it reads your **most recent** sessions off disk — about 20 of them — and offers them to you. Adopting one is a separate, deliberate step called **import**. Nothing is absorbed silently.

The honest one-liner: **the daemon can read the session files your agent CLIs already write, and on request it lists your recent ones so you can import the ones you want.**

---

## Why the files are already there

Here's the key fact that makes everything click: **your agent CLIs save their conversations to disk on their own, with or without Paseo.**

When you run `claude` in a terminal and chat with it, Claude Code writes that conversation to a file under a hidden folder in your home directory. Codex does the same in its own folder. This is just how those tools work — they keep a history so you can resume later.

The Paseo daemon runs on that **same computer**, as **you**, so it can read those same files. It isn't reaching into anything private or remote — it's opening files that sit right next to it on the disk, that your own user account already owns.

| Provider | Where it saves sessions | What a session file is |
| -------- | ----------------------- | ---------------------- |
| Claude Code | `~/.claude/projects/<your-folder-as-dashes>/<session-id>.jsonl` | One conversation, one file |
| Codex | `~/.codex/sessions/<year>/<month>/<day>/rollout-<timestamp>-<session-id>.jsonl` | One conversation, one file |

(`~` means your home folder. A `.jsonl` file is just a text file with one record per line.)

So "Paseo knows about my Claude chat" really means "the daemon read `~/.claude/projects/.../<that-chat>.jsonl`."

---

## What the daemon actually does when you ask

When you open the recent-sessions view, the daemon does this — no background magic, just file reading on demand:

1. **Asks each provider it supports to list its recent sessions.** For Claude, that means looking in `~/.claude/projects`, finding the most recently changed session files, and reading enough of each to get a title and timestamp. Codex and others do the equivalent in their own folders.

2. **Merges and sorts them by most-recent-first**, then keeps roughly the top 20. It does **not** load your entire history — only a recent slice, so the view is fast.

3. **Filters out noise**, so the list is useful:
   - Sessions with no actual question from you (empty or system-only) are skipped.
   - Internal housekeeping sessions (Paseo generating a title, for example) are hidden.
   - Sessions you've **already imported** are removed, so you don't see duplicates.

4. **Shows you the result.** You see a short, recent, de-duplicated list of real conversations, each labeled by provider.

That's the entire "awareness." It's a directory listing with good manners, run when you ask for it.

---

## Importing: the deliberate step

Seeing a session in the list does nothing to it. To actually continue a conversation in Paseo, you **import** it — a separate action you take.

When you import, the daemon:

1. Finds that exact session file again by its ID.
2. **Resumes** it as a live Paseo agent — it reopens the conversation with the right provider so you can send the next message.
3. **Loads the past messages** into Paseo's timeline so you see the full history.

From that point the conversation behaves like any other Paseo agent. The original file keeps working too; importing continues it rather than copying it away.

> **Nothing is taken without you.** Listing reads file metadata to show you options. Importing — resuming a real conversation — only happens when you choose it.

---

## Importing is not live mirroring

This is the trap that catches almost everyone, so it's worth stating bluntly: **importing a session does not turn Paseo into a live window onto your terminal.**

Import takes a **one-time snapshot** of the conversation as it stood on disk, then **resumes it as a fresh agent that Paseo itself runs.** From that moment, the imported agent is a *separate* live conversation, driven by a process the daemon controls.

The consequence surprises people:

- **Messages you send from Paseo** flow through the daemon's process and sync to every connected device instantly. That part feels magic and works.
- **Messages you type into a separate `claude` or `codex` terminal on the same laptop** go to a *different* process writing to disk. Paseo is **not watching that file**, so those messages do **not** appear in the imported agent — not on your phone, not anywhere.

There is no background file-watcher, and there is **no setting to enable automatic live sync from an outside terminal.** It's a deliberate design choice: the daemon *reads* session files on demand (cheap, no locks), it doesn't *subscribe* to them (which would mean two processes fighting over one conversation).

### What to do instead

Pick one of two clear paths — don't run both at once on the same conversation:

| You want to… | Do this |
| ------------ | ------- |
| Continue the conversation everywhere, in sync | After importing, send follow-ups **from a Paseo client** (phone, web, or CLI). Treat import as "this conversation now lives in Paseo." |
| Keep using the laptop terminal, but peek from Paseo occasionally | Type in the terminal, then **manually pull the latest disk state** into Paseo with the **Reload agent** action — or from the CLI: |

```bash
paseo agent reload <agent-id>
```

**Reload** wipes the imported agent's timeline and re-reads the current session file from scratch. It's **manual and one-shot** — run it again each time you want to catch up. It is not a live feed.

> **Don't drive one conversation from two places.** Running the live terminal session *and* the imported Paseo agent at the same time means two processes editing the same conversation; they can diverge into separate branches, and **Reload** only ever shows whatever is on disk for that session right now. Pick one driver per conversation.

---

## The two buckets, one more time

This is the whole model in a table. Most confusion disappears once you separate these:

| | **Paseo-launched agents** | **External sessions** |
| --- | --- | --- |
| Who started it | You, from a Paseo client | You, running `claude`/`codex` by hand |
| Where the record lives | `~/.paseo/agents/` (the daemon's own folder) | The provider's folder (`~/.claude`, `~/.codex`, …) |
| How Paseo knows about it | The daemon launched it, so it tracks it live | The daemon reads the file the CLI saved |
| Shows up automatically? | Yes — it's Paseo's own | Only when you open the recent-sessions view |
| Becomes a full Paseo agent | Already is | After you **import** it |

The first bucket never felt mysterious. The second is the one that looks like magic — and it's just file reading plus an explicit import.

---

## Why it's built this way

Paseo deliberately reads the providers' *own* session files instead of inventing its own storage. Two payoffs:

- **No lock-in.** Your conversations stay in Claude's and Codex's native formats. If you stop using Paseo tomorrow, every chat is still right where those tools put it.
- **It meets you where you already are.** Work you did in a plain terminal isn't stranded — Paseo can pick it up, and work you do in Paseo is readable by the underlying tools.

This mirrors Paseo's whole philosophy: your code, your keys, your files, on your machine. Paseo is a control surface over what's already yours, not a vault that captures it.

---

## Ending and removing a session

When you're done with an agent — imported or not — Paseo gives you three distinct actions. They're easy to mix up, so match the gesture to what you actually want.

| Action | Where to do it | What it does | Reversible? |
| ------ | -------------- | ------------ | ----------- |
| **Stop** (interrupt) | The ⏹ button in the composer while the agent is running | Halts the *current turn* only. The agent and its conversation stay; you can send another message. | n/a — it's a pause, not a removal |
| **Archive** | Close the agent's tab (the ✕ on its tab) | Ends the session: stops the agent's process **and** removes it from your active list. This is the everyday "I'm done with this" gesture. | Yes — it's a soft delete; the record stays on disk and can be brought back |
| **Delete** | The `paseo delete` command | Permanently removes the agent record. | No |

**Stop** is a pause. **Archive** is the real "terminate." **Delete** is permanent.

### Archiving, step by step

On the phone, close the agent's tab. If the agent is still working, Paseo asks first:

> **Archive running agent?** This agent is still running. Archiving it will stop the agent and close the tab.

Confirm, and the agent's process stops and it leaves your active list. Because archive is a **soft delete**, the conversation isn't destroyed — it's hidden. You can list archived agents and revive one later.

### The one thing to remember for imported sessions

Archiving stops **the process Paseo runs** for that agent. It does **not** touch a separate `claude` or `codex` you may still have open in a laptop terminal — that's a different process Paseo never controlled. If you imported a session *and* kept the terminal one running, ending it on your phone closes the Paseo side only; close the terminal one yourself if you want both gone.

### From the command line

```bash
paseo stop <agent-id>      # interrupt a running agent (a pause)
paseo archive <agent-id>   # end it and remove from active list (recoverable)
paseo delete <agent-id>    # permanent removal (interrupts first if running)
paseo ls -a                # list everything, including archived, to find an id
```

---

## Where to go next

- The other half people confuse this with: [how pairing works](how-pairing-works.md)
- The big picture: [what is Paseo?](what-is-paseo.md)
- What's protected and what's on you: [security](security.md)
