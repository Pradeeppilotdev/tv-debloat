# Debloat Xiaomi Mi TV via ADB — No Root Required

A complete guide to removing bloatware, ads, and recommendation rows from Xiaomi Android TVs using ADB. Tested on a **Mi TV 4A Horizon Edition** (1 GB RAM / 8 GB storage), but the package names and approach work on any Xiaomi/PatchWall TV running Android TV.

> **Why this guide?** Most debloat guides online target TCL or generic Android TVs. Xiaomi TVs ship with PatchWall, Sensara ad-tracking, and a stack of pre-installed Indian streaming apps — all eating RAM on already resource-constrained hardware. This guide identifies the actual Xiaomi package names and disables them safely in small batches.

---

## What you need

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
4. Turn on **USB debugging** (sometimes labeled "ADB debugging")
5. Note your TV's IP address: **Settings** > **Device Preferences** > **About** > **Status** > **IP address**

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

### Safe to disable — Telemetry & Ad Tracking

| Package | What it is |
|---------|-----------|
| `co.sensara.tv.mitv` | Sensara ad-tracking analytics (~30 MB RAM) |
| `com.miui.tv.analytics` | MIUI telemetry sending data to Xiaomi |
| `com.xiaomi.statistic` | Xiaomi usage statistics collector |

### Safe to disable — Recommendation Engines (the ad rows)

| Package | What it is |
|---------|-----------|
| `com.google.android.katniss` | Google recommendation engine behind content rows (~23 MB) |
| `com.google.android.tvrecommendations` | "Recommended for you" row (~11 MB) |
| `com.google.android.leanbacklauncher.recommendations` | Old Leanback recommendations (dead code) |
| `com.google.android.leanbacklauncher` | Old Leanback launcher (unused) |

### Safe to disable — Unused Streaming Apps

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

### Safe to disable — Screensavers, Dead Code & Bloatware

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

### Safe to disable — Hardware Apps TV Doesn't Have

| Package | What it is |
|---------|-----------|
| `com.android.camera2` | Camera app — TV has no camera |
| `com.android.printspooler` | Print spooler — TV has no printer |
| `com.android.smspush` | SMS push — TV has no SMS |

### Safe to disable — Unused Xiaomi Apps

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

| Package | Why |
|---------|-----|
| `com.google.android.gms` | Play Services — everything depends on it |
| `com.google.android.gsf` | Google Services Framework |
| `com.android.vending` | Play Store |
| `com.android.location.fused` | Disabling causes boot loop |
| `com.google.android.inputmethod.latin` | On-screen keyboard |
| `com.google.android.tv.remote.service` | Remote control |
| `com.android.bluetooth` | Bluetooth (remote pairing) |
| `com.droidlogic.tvinput` | HDMI / antenna input |
| `com.droidlogic` | Core TV firmware |
| `com.android.tv` | Android TV framework |
| `com.android.systemui` | System UI |
| `com.google.android.apps.mediashell` | Chromecast / Google Cast (keep if you cast from phone) |
| `com.google.android.youtube.tv` | YouTube (keep if you watch) |
| `org.videolan.vlc` | VLC (keep if you use it) |

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

The package names in the "Telemetry", "Recommendation Engines", "Unused Streaming", and "Xiaomi Apps" sections are Xiaomi-specific. Other brands (TCL, Hisense, Sony) will have different package names — run `adb shell pm list packages` on your TV to find yours.

---

## License

Do whatever you want with this. If it helps you, share it forward.
