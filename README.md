## Anycubic Kobra 4 runs Klipper — and they're progressively scrubbing the evidence

**TL;DR:** The Kobra 4 firmware contains a Klipper build (commit `739e5146b`, branch `TEST`). Anycubic removed version identifiers from the MCU binaries, doesn't provide source code, and doesn't include the GPL license. This continues a pattern of obfuscation across the K3 → S1 → K4 generations.

---

### Background

It's been known in the community for a while that the **Anycubic K3** series runs Klipper. That's not news.

What **is** news: the **Kobra 4** also runs Klipper — and Anycubic has been systematically removing the evidence across product generations.

### The Evidence

**1. Klipper commit hash in the boot log**

```
commit=739e5146b, branch=TEST, built=2026-05-12
```

This appears in the K4's `avata_main` boot log (firmware 1.2.1.3). It's a git commit hash from a Klipper repository, built on a `TEST` branch — not upstream master. A private fork, never published.

**2. `identify_data` surgically removed from MCUs**

Every standard Klipper MCU build contains an `identify_data` blob: version, build date, git hash, compiler info. It's how you check what's running on your steppers.

**Anycubic cut it out.** Binary comparison against standard Klipper builds shows the block has been removed. This isn't accidental — it's deliberate obfuscation of the software's origin.

**3. MCU firmware speaks Klipper protocol byte-for-byte**

The K4's AT32F403A microcontrollers communicate using Klipper's native serial protocol (`serialprotocol.c`). I verified this by observing a complete Klipper initialization handshake between the K4's host processor and its MCUs — 881 bytes out, 2,529 bytes back, standard Klipper message format.

**4. The obfuscation gets worse with each generation**

| Generation | Klipper visible? | identify_data? | Plaintext config? |
|---|---|---|---|
| K3V2 | Yes (known) | Partially | Yes (device.ini) |
| S1 | Reduced | Mostly removed | Reduced |
| K4 | Hidden (boot log only) | **Removed** | No |

This is a clear trend: each generation hides more of the Klipper origin.

**5. A standard Klipper host can drive the K4's MCUs**

I'm building [Rinkhals](https://github.com/elias/kobra4_rinkhals) — a translation layer that connects Mainsail/Fluidd/Moonraker directly to the K4's microcontrollers. The MCUs respond to standard Klipper commands without any firmware modification, because they **are** running Klipper.

### Why This Matters

Klipper is licensed under **GPL v3**. This means if you distribute a binary, you must:

1. **Provide source code** — Anycubic doesn't
2. **Include the license** — Anycubic doesn't
3. **Preserve copyright notices** — Anycubic doesn't
4. **Not add restrictions** — removing `identify_data` restricts the user's ability to know what software they're running

Anycubic benefits enormously from the open-source 3D printing ecosystem (Klipper, Moonraker, mainsail community) while simultaneously hiding their use of it and refusing to give back.

### What I'm Doing About It

- I've documented everything with binary comparisons, disassembly, and live protocol captures
- I've submitted findings to the relevant authorities (BSI, Germany)
- I'm building Rinkhals so owners can run **actual open-source Klipper** on their K4 hardware
- I will disclose security findings separately after responsible disclosure

### What You Can Do

If you own a K4 and want to verify yourself:

```bash
# Check your boot log for the commit hash
grep -a "commit=" /data/logs/avata_main.log

# Try connecting to the MCU with standard Klipper tools
# (details in the Rinkhals repo)

# Ask Anycubic for the source code — you're entitled to it under GPL v3
```

---

*This research was conducted on my own hardware. No other devices were accessed. Full technical documentation will be published after responsible disclosure.*

*Edit: Yes, I know Klipper on K3 is old news. The new part is K4 + the systematic removal of identifiers. Please read before commenting "K3 Klipper has been known."*
