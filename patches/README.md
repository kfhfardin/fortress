# Patches — the open surface layer

These are Fortress's **surface-coherence patches**: the in-tree Chromium/Blink modifications that read a
per-launch persona and present it consistently across the JS-observable fingerprint surfaces (user-agent,
platform, WebGL, timezone, languages, screen, keyboard, media, and so on), including inside worker and
iframe realms.

> **These patches are a reference layer — not the whole engine, and not the current build.**
>
> The part that *mints* a coherent, per-launch real-device persona and *delivers* it into the process —
> the device-model corpus, the joint-distribution generator, and the process-level delivery seam — ships
> **compiled inside the released binary** and is intentionally **not** in this repository. Building from
> these patches alone will *not* reproduce the shipped stealth (you'll get surfaces that read a persona
> that nothing is providing); it demonstrates the technique, not the product.

For the real, current engine:

```bash
pip install -U tilion-fortress
# or
docker run --rm -p 9222:9222 tilion/fortress:latest
```

See the [latest release](https://github.com/tiliondev/fortress/releases/latest) for what actually ships.
