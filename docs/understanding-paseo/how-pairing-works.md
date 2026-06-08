# How pairing works

Scanning the QR code connects your phone to your computer's daemon and sets up an encrypted channel that not even Paseo's relay server can read. This page explains exactly what happens, and why it's safe, in plain language.

This is the one place these docs explain the cryptography in full. The [security](security.md) page focuses on the threat model and refers back here for the mechanics. Unfamiliar words are defined in [key concepts](key-concepts.md).

---

## First, the thing people get wrong

**Scanning the QR code does not read or import your agent sessions.** It does exactly one thing: it connects your phone to the daemon.

The daemon could already see your sessions — they're files sitting on the same disk it runs on. The QR is about *connecting a remote control*, not about *discovering your work*. Those are two separate mechanisms that happen to both involve the daemon, and conflating them is what makes Paseo feel like magic. How the daemon sees sessions is covered in [how Paseo finds your sessions](how-paseo-finds-your-sessions.md).

With that cleared up, here's what the QR actually carries and does.

---

## What's inside the QR code

The QR code is just a compact way to carry a link. The link looks like this:

```
https://app.paseo.sh/#offer=<the daemon's public key + how to reach it>
```

Two things to notice:

**It contains the daemon's public key.** Recall from [key concepts](key-concepts.md) that a **public key** is the safe-to-share half of a keypair — like a padlock you can hand out freely. Anyone holding it can lock a box that only the daemon's matching **private key** can open. The private key never leaves your computer. So putting the public key in a QR is safe by design.

**The key rides in the part after the `#`.** Everything after the `#` in a web link is called the **fragment**, and browsers never send the fragment to the web server. So even though the link points at `app.paseo.sh`, the Paseo website never receives the key — your phone reads it locally. The website is just a convenient place for the app to open.

> **Still treat the QR as a secret.** "The website can't see the key" is not the same as "anyone can have it." Whoever holds this link can start a connection to your daemon. Don't post it publicly. It's the trust anchor for the whole system.

---

## The handshake, step by step

When your phone uses that link to connect (through the **relay**, the public meeting-point server), the two sides run a short setup conversation called a **handshake**. The daemon refuses to act on *any* command until this finishes.

Here it is, plainly:

1. **Your phone makes a throwaway key.** For this one connection, the phone generates a fresh **ephemeral keypair** — "ephemeral" means single-use and discarded afterward. It sends its new public key to the daemon in a message called `e2ee_hello`.

2. **Both sides compute the same shared secret — without ever sending it.** The daemon has its own private key and now has the phone's public key. The phone has its own private key and (from the QR) the daemon's public key. A piece of math called **ECDH** (Elliptic-Curve Diffie–Hellman) lets each side combine *their* private key with the *other side's* public key and arrive at the **same** secret number — independently, on their own device. That secret is never transmitted. Anyone watching the wire sees only the two public keys, which are useless without a matching private key.

   The specific math uses **Curve25519**, a widely trusted, modern elliptic curve.

3. **Every later message is sealed with that shared secret.** From here on, both sides encrypt messages using **XSalsa20-Poly1305** (the well-known "NaCl box" scheme). This does two jobs at once: it **scrambles** the message so only the holder of the shared secret can read it, and it **stamps** the message so any tampering is detected and rejected. Each sealed message is sent as `[random nonce][scrambled bytes]` — the "nonce" is a random value that keeps every message's encryption unique.

That's the whole handshake. After it, your phone and daemon share a private language that only they understand.

---

## Why the relay in the middle can't hurt you

The **relay** (`relay.paseo.sh`) is the server that forwards messages between your phone and your daemon when they're on different networks. Paseo deliberately assumes the relay could be **untrusted or even compromised**, and the design still holds. Here's what a malicious relay still cannot do:

| A bad relay tries to… | Why it fails |
| --------------------- | ------------ |
| **Read your messages** | They're encrypted with a secret it never has. It sees only scrambled bytes plus the public-key "hello" frames. |
| **Pretend to be your daemon** | Without the daemon's private key, it can't compute the shared secret, so anything it sends fails the phone's tamper-check and is rejected. |
| **Send commands as you** | Same reason — the daemon only accepts messages that unlock and verify under the shared secret, which requires its own private key. |
| **Alter a message in transit** | The Poly1305 "stamp" on each message detects any change; tampered messages are thrown out. |
| **Replay an old session's messages into a new one** | Each connection derives a fresh secret, so yesterday's scrambled bytes are meaningless today. |

The relay does see some metadata it can't avoid: the IP addresses connecting, the timing and sizes of messages, the session IDs, and the plaintext hello frames (which contain only public keys). It cannot turn any of that into your code or your commands.

> **One honest limit:** protection against replaying a message *within a single live session* is not yet implemented (cross-session replay is fully prevented). In practice the encrypted, authenticated channel still blocks an outsider from crafting valid traffic — but the security model is stated plainly rather than overclaimed. See [security](security.md).

---

## The trust anchor, in one sentence

Everything hangs on the QR code: it carries the daemon's public key, which is what lets your phone build a channel only your daemon can be on the other end of. Guard the QR like a password, and the rest of the system protects itself.

---

## Where to go next

- The broader safety picture, including running Paseo on a server: [security](security.md)
- The *other* mechanism people confuse with this one: [how Paseo finds your sessions](how-paseo-finds-your-sessions.md)
- Just want it paired? [quickstart](quickstart.md)
