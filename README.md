# tv-debloat

ADB debloat guide for Android TV — **no root, no uninstalls, fully reversible.**

Works on **Xiaomi, TCL, Hisense, Sony, Philips**, and any Android TV / Google TV device.

## What this does

- Removes ad-tracking analytics, recommendation rows, and pre-installed bloatware
- Replaces the stock launcher (full of ads) with [FLauncher](https://play.google.com/store/apps/details?id=me.efesser.flauncher) (clean app grid)
- Frees RAM and reduces swap pressure on low-memory TVs

## Quick start

1. Enable **Developer Options** on your TV (tap Build number 7 times)
2. Enable **USB debugging** (+ **MiTV ADB debugging** on Xiaomi TVs)
3. Open **Claude Code, Codex, Cursor**, or any AI agent on your computer
4. Copy the prompt from [DEBLOAT-LOG.md](DEBLOAT-LOG.md#quick-start--paste-this-prompt-to-an-ai-agent), fill in your TV model and IP, paste it in
5. The agent does everything — connects, sorts packages, disables in batches, walks you through testing

## Results (Xiaomi Mi TV 4A, 1 GB RAM)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Used RAM | 705 MB | 575 MB | **-130 MB** |
| Free RAM | 285 MB | 381 MB | **+96 MB** |
| ZRAM swap | 63 MB | 11 MB | **-82%** |
| Packages disabled | 0 | 43 | |

## Files

| File | What it is |
|------|-----------|
| [DEBLOAT-LOG.md](DEBLOAT-LOG.md) | **Full guide** — setup instructions, AI prompt, package reference, undo commands |
| [kapatilanlar.txt](kapatilanlar.txt) | List of packages disabled on the test device |

## Safety

- **No uninstalls** — only `pm disable-user --user 0`, every change is reversible
- **No root** — bootloader stays locked, Widevine stays L1, Netflix stays HD
- **Batch testing** — 5 packages at a time, test remote/HDMI/sound/keyboard between each batch
- **Single undo command** re-enables everything and reboots

## Credits

Inspired by [tv.cobanov.dev](https://tv.cobanov.dev/) — the original "paste a prompt to an AI agent" Android TV debloat concept. This repo extends it with Xiaomi/PatchWall-specific packages, multi-brand coverage, and a universal AI prompt.

## License

Do whatever you want with this. If it helps you, share it forward.
