# Debloat Any Android TV via ADB — No Root Required

A complete guide to removing bloatware, ads, and recommendation rows from Android TVs using ADB. Works on **Xiaomi, TCL, Hisense, Sony, Philips**, and any TV running Android TV or Google TV.

Tested on a **Xiaomi Mi TV 4A Horizon Edition** (1 GB RAM / 8 GB storage), but the approach, AI prompt, and most package names are universal. Xiaomi-specific packages are clearly labeled.

> **Why this guide?** Most debloat guides either target one specific brand or are too generic to be safe. This guide provides a **copy-paste AI agent prompt** that works for any Android TV (the agent discovers your TV's packages automatically), plus a manual reference with both universal and brand-specific package lists.

---

## Quick start — paste this prompt to an AI agent

If you have **Claude Code, Codex, Cursor, or any AI coding agent** running on your computer, copy the prompt below, fill in your TV's model and IP address, and paste it in. The agent will connect to your TV, sort the packages, disable them in batches, and walk you through testing — all automatically.

> **Prerequisite:** Enable Developer Options and ADB on your TV first (see [Step 1](#step-1--enable-adb-on-your-mi-tv) below), then paste this prompt:

```
I want you to clean up my Android TV over ADB. The goal: get rid of the ad and
recommendation rows, disable the factory apps I never use, and make the TV
faster. Run the commands yourself and explain what you are doing in plain
language as you go.

MY TV
- Make / model: FILL THIS IN            (example: Xiaomi Mi TV 4A Horizon Edition)
- TV's IP address: FILL THIS IN         (Settings > About > Status — or — Settings > Network & internet > Wi-Fi > IP address)
- TV has PatchWall (Xiaomi's launcher) installed, but I may be using Android TV's
  stock launcher. Confirm the active launcher with:
  `dumpsys activity activities | grep mResumedActivity`
- Enable BOTH "USB debugging" AND "MiTV ADB debugging" in Developer Options.
  The second one is Xiaomi-specific — without it, adb connect fails silently.
- adb connect to <IP>:5555 should already be authorized. Verify with `adb devices`
  and proceed straight to the cleanup.

RULES - follow these exactly
1. DO NOT UNINSTALL ANYTHING. Use only `pm disable-user --user 0`, so every
   change can be undone with `pm enable <package>`. `pm uninstall` is forbidden.
2. Do not suggest or attempt rooting, unlocking the bootloader, or flashing a
   custom ROM. That factory-resets the TV, and if the Widevine certificate
   drops from L1 to L3 Netflix falls back to SD. There is nothing to gain.
3. MEASURE FIRST. Before you start, capture `dumpsys meminfo`,
   `pm list packages -s` and `pm list packages -d` to a file. Repeat the same
   measurements at the end and give me a before/after table.
4. WORK IN SMALL BATCHES (5 packages at a time — this TV has 1GB RAM, go
   slower than the default), then STOP and have me test: does the remote's
   "Inputs / Source" button still work, can I switch to HDMI, do Netflix and
   YouTube still open, do sound and the on-screen keyboard still work? Do not
   move to the next batch until I say it is fine.
5. NEVER DISABLE THE FOLLOWING (these make the TV unusable). The package names
   below are TCL examples only — find the Xiaomi/PatchWall equivalent for each
   category before doing anything, and ask me if unsure:
      com.tcl.suspension          -> the Inputs/Source menu on the remote.
                                     Find this TV's equivalent "quick panel"
                                     or input-switching package.
      com.tcl.tv, com.tcl.tvinput -> HDMI / antenna input services
      com.google.android.tv.remote.service,
      com.tcl.tcl_bt_rcu_service,
      com.tcl.autopair            -> the remote control
      com.google.android.gms,
      com.google.android.gsf,
      com.android.vending         -> Play Services and the Play Store
      com.android.location.fused  -> disabling it sends the TV into a boot loop
      *.inputmethod.*             -> the on-screen keyboard
      com.google.android.apps.tv.launcherx
                                  -> the home screen. Disabling it before an
                                     alternative launcher is installed leaves
                                     you with a black screen.
6. KEEP A RECORD. Write every package you disable to `kapatilanlar.txt` inside
   a `tv-debloat` folder. Keep a `DEBLOAT-LOG.md` in the same folder describing
   what you did, why, and the exact command to undo each change.
7. If something breaks, re-enable the last batch first, then narrow it down one
   package at a time.

DO IT IN THIS ORDER
a) Check whether `adb` is installed and install it if not. Confirm the
   connection is live with `adb devices`.
b) Take the measurements and list the packages.
c) Sort the packages into three groups and show me a table: (1) definitely junk:
   unused streaming apps, demos, screensavers, voice assistants, setup wizards,
   telemetry; (2) depends on me: ask whether I actually use them; (3) untouchable.
   Get my approval and start with group 1.
d) Halve the animation speeds:
      settings put global window_animation_scale 0.5
      settings put global transition_animation_scale 0.5
      settings put global animator_duration_scale 0.5
e) Clear caches with `pm trim-caches`.
f) Reboot the TV and take the measurements again.

REPLACE THE HOME SCREEN
Google TV's own home screen is full of ads and "recommended for you" rows. I
want to use FLauncher instead (it is on the Play Store, free, open source, and
shows nothing but a grid of my apps).
- First ask me to install FLauncher and confirm that it is installed.
- Have me open FLauncher once and set it as the home screen.
- Only then disable the stock launcher package (confirm the actual package name
  with `pm list packages | grep launcher` first — do not assume).
- Reboot the TV and confirm the home screen is still FLauncher.

WHEN YOU ARE DONE, GIVE ME
1. The before/after RAM table.
2. The list of packages you disabled, each with a one-line "what this was".
3. A single command that undoes everything.
```

---

## Manual steps (if you prefer to do it yourself)

### What you need

- A computer (Linux/Mac/Windows) with **ADB** installed
  - Linux: `sudo apt install adb`
  - Mac: `brew install android-platform-tools`
  - Windows: download from [developer.android.com](https://developer.android.com/tools/releases/platform-tools)
- Your Mi TV and computer on the **same Wi-Fi network**
- A USB keyboard or the Mi Remote (to enter a PIN on screen)

---

## Step 1 — Enable ADB on your Mi TV

1. Go to **Settings** > **Device Preferences** > **About**
2. Scroll to **Build** and press OK on the remote **7 times** rapidly. This enables Developer Options.
3. Go back to **Settings** > **Device Preferences** > **Developer Options**
4. Enable **both** of these (the second one is Xiaomi-specific and required for ADB over Wi-Fi):
   - **USB debugging**
   - **MiTV ADB debugging** ← without this, `adb connect` will fail silently on Xiaomi TVs
5. Note your TV's IP address — either:
   - **Settings** > **Device Preferences** > **About** > **Status** > **IP address**, or
   - **Settings** > **Network & internet** > **Wi-Fi** (connected network) > **IP address**

## Step 2 — Connect from your computer

```bash
adb connect <TV_IP>:5555
# Example: adb connect 192.168.1.2:5555
```

A popup will appear on the TV asking "Allow USB debugging?". Check **Always allow** and tap OK.

Verify the connection:
```bash
adb devices
# Should show: <TV_IP>:5555   device
```

## Step 3 — Take a baseline measurement

Before disabling anything, capture the current state:
```bash
adb shell dumpsys meminfo           # RAM usage
adb shell pm list packages -s       # system packages
adb shell pm list packages -d       # already disabled packages
```

## Step 4 — Disable packages (in small batches)

**Critical rules:**
- Use `pm disable-user --user 0` — NEVER `pm uninstall`. Every change is reversible.
- Disable **5 packages at a time max**. Wait and test after each batch.
- NEVER disable: Play Services, Play Store, remote service, Bluetooth, keyboard, TV input, location fused, or the active launcher.
- Test after each batch: Does the **remote's Source/Input button** work? Can you **switch to HDMI**? Does **YouTube** open? Does **sound** work? Does the **on-screen keyboard** appear?

### Safe to disable — Telemetry & Ad Tracking (Xiaomi-specific)

| Package | What it is |
|---------|-----------|
| `co.sensara.tv.mitv` | Sensara ad-tracking analytics (~30 MB RAM) |
| `com.miui.tv.analytics` | MIUI telemetry sending data to Xiaomi |
| `com.xiaomi.statistic` | Xiaomi usage statistics collector |

### Safe to disable — Recommendation Engines (universal — all Android TVs)

| Package | What it is |
|---------|-----------|
| `com.google.android.katniss` | Google recommendation engine behind content rows (~23 MB) |
| `com.google.android.tvrecommendations` | "Recommended for you" row (~11 MB) |
| `com.google.android.leanbacklauncher.recommendations` | Old Leanback recommendations (dead code) |
| `com.google.android.leanbacklauncher` | Old Leanback launcher (unused) |

### Safe to disable — Unused Streaming Apps (universal — check what's on your TV)

| Package | What it is |
|---------|-----------|
| `com.netflix.ninja` | Netflix (~90 MB RAM!) |
| `com.amazon.amazonvideo.livingroom` | Amazon Prime Video |
| `com.google.android.videos` | Google Play Movies & TV |
| `com.sonyliv` | SonyLIV streaming |
| `in.startv.hotstar` | Hotstar / Disney+ Hotstar |
| `com.google.android.youtube.tvmusic` | YouTube Music for TV |
| `com.google.android.play.games` | Google Play Games |

> **Note:** These can be re-installed from the Play Store anytime. Only disable apps you don't use.

### Safe to disable — Screensavers, Dead Code & Bloatware (universal — all Android TVs)

| Package | What it is |
|---------|-----------|
| `com.mitv.dream` | Mi TV screensaver |
| `com.android.dreams.basic` | AOSP daydream screensaver |
| `com.google.android.backdrop` | Chromecast Ambient Mode screensaver |
| `com.google.android.tungsten.setupwraith` | First-boot setup wizard (already ran) |
| `com.google.android.partnersetup` | Partner setup (one-time, already ran) |
| `com.google.android.onetimeinitializer` | One-time init (dead weight) |
| `com.android.onetimeinitializer` | Same |
| `com.google.android.tv.frameworkpackagestubs` | Framework stubs (dead code) |
| `com.google.android.tv.bugreportsender` | Bug report sender |
| `com.google.android.feedback` | Google feedback |

### Safe to disable — Hardware Apps TV Doesn't Have (universal — all Android TVs)

| Package | What it is |
|---------|-----------|
| `com.android.camera2` | Camera app — TV has no camera |
| `com.android.printspooler` | Print spooler — TV has no printer |
| `com.android.smspush` | SMS push — TV has no SMS |

### Safe to disable — Unused Xiaomi Apps (Xiaomi/PatchWall-specific)

> **Other brands?** TCL has `com.tcl.*`, Hisense has `com.hisense.*`, Sony has `com.sony.*`. Run `adb shell pm list packages | grep <brand>` to find yours.

| Package | What it is |
|---------|-----------|
| `com.mitv.tvhome.atv` | PatchWall home screen (~19 MB, not active if using Google TV launcher) |
| `com.mitv.videoplayer` | Mi Video Player (use VLC instead) |
| `com.xiaomi.mitv.smartshare` | Smartshare / DLNA casting |
| `com.mitv.milinkservice` | MiLink casting service |
| `com.xiaomi.mimusic2` | Xiaomi Music app |
| `com.xiaomi.mitv.handbook` | Mi TV user handbook bloatware |
| `com.xiaomo.tv.milegal` | Xiaomi legal info page |
| `com.xm.webcontent` | Xiaomi web content viewer |
| `com.xiaomi.floatingframe` | Floating mini-window feature |
| `com.android.gallery3d` | Old AOSP Gallery |
| `com.mitv.gallery` | Mi Gallery |
| `com.android.wallpapercropper` | Wallpaper crop utility |
| `com.android.wallpaperbackup` | Wallpaper backup utility |
| `com.android.htmlviewer` | HTML file viewer |

### Safe to disable — Accessibility (only if unused)

| Package | What it is |
|---------|-----------|
| `com.google.android.marvin.talkback` | TalkBack screen reader (keep if you use accessibility) |
| `com.google.android.tts` | Text-to-speech (keep if you use voice features) |
| `com.google.android.speech.pumpkin` | Voice search helper (keep if you use remote mic) |

### NEVER disable these

These are **universal across all Android TV brands**. The brand-specific ones (TCL, Xiaomi, Hisense, etc.) are listed after.

**Universal (all brands):**

| Package | Why |
|---------|-----|
| `com.google.android.gms` | Play Services — everything depends on it |
| `com.google.android.gsf` | Google Services Framework |
| `com.android.vending` | Play Store |
| `com.android.location.fused` | Disabling causes boot loop |
| `com.google.android.inputmethod.latin` | On-screen keyboard |
| `com.google.android.tv.remote.service` | Remote control |
| `com.android.bluetooth` | Bluetooth (remote pairing) |
| `com.android.tv` | Android TV framework |
| `com.android.systemui` | System UI |
| `com.google.android.apps.mediashell` | Chromecast / Google Cast (keep if you cast from phone) |
| `com.google.android.youtube.tv` | YouTube (keep if you watch) |

**Xiaomi / DroidLogic:**

| Package | Why |
|---------|-----|
| `com.droidlogic.tvinput` | HDMI / antenna input |
| `com.droidlogic` | Core TV firmware |

**TCL:**

| Package | Why |
|---------|-----|
| `com.tcl.suspension` | Inputs/Source menu — disabling it kills HDMI switching |
| `com.tcl.tv` | HDMI / antenna input |
| `com.tcl.tvinput` | HDMI / antenna input |
| `com.tcl.tcl_bt_rcu_service` | Remote control |
| `com.tcl.autopair` | Remote auto-pairing |

> **Not sure what a package does?** Run `adb shell dumpsys package <package.name>` to see its permissions and activities, or ask your AI agent before disabling it.

## Step 5 — Replace the home screen (optional)

The stock Google TV launcher is full of ads and recommendation rows. Replace it with **FLauncher** (free, open source, clean app grid):

1. Install [FLauncher](https://play.google.com/store/apps/details?id=me.efesser.flauncher) from the Play Store on your TV
2. Open FLauncher once
3. Set it as default:
   ```bash
   adb shell pm set-home-activity me.efesser.flauncher/.MainActivity
   ```
4. If the old launcher keeps reclaiming focus, disable it:
   ```bash
   adb shell pm disable-user --user 0 com.google.android.tvlauncher
   ```
5. Press the Home button — FLauncher should appear

## Step 6 — Clear caches and reboot

```bash
adb shell pm trim-caches 1G
adb reboot
```

---

## Results — Mi TV 4A Horizon Edition (1 GB RAM)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Used RAM | 705 MB | 575 MB | **-130 MB freed** |
| Free RAM | 285 MB | 381 MB | **+96 MB more free** |
| ZRAM swap | 63 MB | 11 MB | **-82% less swapping** |
| Lost RAM | 105 MB | 63 MB | **-41 MB** |
| Disabled packages | 0 | 43 | |

The TV feels noticeably snappier. App switching is faster, and there's enough free RAM that the system isn't constantly swapping to ZRAM.

---

## Undo everything — single command

If something breaks, run this from your computer to revert all changes:

```bash
adb shell pm enable com.google.android.tvlauncher && adb shell pm enable co.sensara.tv.mitv && adb shell pm enable com.google.android.katniss && adb shell pm enable com.google.android.tvrecommendations && adb shell pm enable com.miui.tv.analytics && adb shell pm enable com.google.android.tungsten.setupwraith && adb shell pm enable com.amazon.amazonvideo.livingroom && adb shell pm enable com.google.android.videos && adb shell pm enable com.sonyliv && adb shell pm enable com.google.android.leanbacklauncher.recommendations && adb shell pm enable com.google.android.leanbacklauncher && adb shell pm enable in.startv.hotstar && adb shell pm enable com.google.android.youtube.tvmusic && adb shell pm enable com.google.android.play.games && adb shell pm enable com.google.android.backdrop && adb shell pm enable com.xiaomi.statistic && adb shell pm enable com.mitv.dream && adb shell pm enable com.android.dreams.basic && adb shell pm enable com.android.camera2 && adb shell pm enable com.android.printspooler && adb shell pm enable com.android.smspush && adb shell pm enable com.android.gallery3d && adb shell pm enable com.mitv.gallery && adb shell pm enable com.xiaomi.mitv.handbook && adb shell pm enable com.xiaomo.tv.milegal && adb shell pm enable com.google.android.tv.bugreportsender && adb shell pm enable com.google.android.feedback && adb shell pm enable com.google.android.onetimeinitializer && adb shell pm enable com.android.onetimeinitializer && adb shell pm enable com.android.wallpapercropper && adb shell pm enable com.android.wallpaperbackup && adb shell pm enable com.xm.webcontent && adb shell pm enable com.android.htmlviewer && adb shell pm enable com.google.android.tv.frameworkpackagestubs && adb shell pm enable com.xiaomi.floatingframe && adb shell pm enable com.google.android.partnersetup && adb shell pm enable com.netflix.ninja && adb shell pm enable com.google.android.marvin.talkback && adb shell pm enable com.mitv.tvhome.atv && adb shell pm enable com.mitv.videoplayer && adb shell pm enable com.xiaomi.mitv.smartshare && adb shell pm enable com.mitv.milinkservice && adb shell pm enable com.xiaomi.mimusic2 && adb reboot
```

---

## Re-enabling individual packages

Any single package can be re-enabled anytime:
```bash
adb shell pm enable <package.name>
```

Example: re-enable Netflix:
```bash
adb shell pm enable com.netflix.ninja
```

---

## Tested on

- **Device:** Xiaomi Mi TV 4A Horizon Edition
- **OS:** Android TV (PatchWall installed, Google TV launcher active)
- **RAM:** 1 GB
- **Storage:** 8 GB
- **Date:** September 2026

The **AI prompt** works on any Android TV brand — it discovers packages automatically. The **manual package tables** below are split into universal (all brands) and Xiaomi-specific sections. For other brands, the agent in the prompt will find the equivalents, or run `adb shell pm list packages` to list everything on your TV.

---

## License

Do whatever you want with this. If it helps you, share it forward.

---

## Credits

This guide was inspired by [tv.cobanov.dev](https://tv.cobanov.dev/) by [Mert Cobanoglu](https://x.com/mertcobanov) — the original "paste a prompt to an AI agent and let it debloat your Android TV" concept. That site provides a generic prompt with TCL-specific package names. This guide extends it with:

- Xiaomi/PatchWall-specific package names (Sensara, MiTV analytics, PatchWall home, etc.)
- The **MiTV ADB debugging** toggle requirement (Xiaomi-specific, missing from most guides)
- Smaller batch sizes (5 instead of 10) for low-RAM TVs (1 GB)
- FLauncher home screen replacement for Xiaomi TVs
- Before/after RAM benchmarks on actual hardware

The original prompt and concept belong to cobanov.dev — go star [their repo](https://github.com/cobanov/tv).
