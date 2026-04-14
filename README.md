# AdBot — Rewarded Ad Automation for Mobile Games

[![Version](https://img.shields.io/github/v/release/s4production/adbot)](https://github.com/s4production/adbot/releases/latest)
[![License](https://img.shields.io/badge/license-Commercial-blue)](LICENSE.md)

**AdBot** is a professional automation tool for watching rewarded ads in mobile games. It works with Android emulators (LDPlayer recommended) and physical phones connected via ADB.

## What does AdBot do?

The bot performs a complete rewarded-ad cycle, fully hands-free:

1. **Detects the ad trigger** on-screen (template matching)
2. **Taps the ad button** → starts the ad sequence
3. **Closes each ad** automatically — finds the close button (X, arrows, skip)
4. **Clicks OK** on any reward overlay that appears
5. **Repeats the cycle** — hundreds of ads per day, no manual interaction

## Key features v12.14

### 3-layer close button detection
- **UI dump** (uiautomator) — searches for close/skip/dismiss keywords across 5 languages (EN/PL/DE/FR/ES)
- **Template matching** — 13 external X/arrow templates + synthetic X in 3 sizes, multi-scale, 5-pass with blur/CLAHE/equalizeHist normalization
- **OCR** (Tesseract) — recognizes "skip"/"close"/"dismiss" text on WebView-rendered ads

### Full coverage of major ad SDKs
- IronSource (ControllerActivity, playable ads, install endcards)
- Google AdMob (AdActivity, TikTok/Google Play interstitials)
- Unity Ads (FullScreenWebViewDisplay)
- AppLovin, Vungle, AdColony, Chartboost, InMobi, MoPub
- **Fyber Inneractive** (with IACloseButton trap handling)
- Pangle, Facebook Ads, StartApp, DigitalTurbine, Tapjoy, Kidoz

### Anti-trap recovery (battle-tested on 60+ cycle autonomous runs)
- **Poison filter** — positions that opened Google Play / Chrome are blacklisted for the rest of the cycle
- **NO-TAP-ZONE** — bot will NEVER tap the bottom 8% of the screen (protects navigation bars)
- **Deceptive browser detection** — InternalBrowserActivity (install trap) → instant BACK
- **Force-bring-to-foreground** — when Chrome ignores BACK keyevent, bot uses `monkey LAUNCHER` to return to the game
- **Race-safe BACK** (v12.14) — focus checked BEFORE every BACK; bot never accidentally exits the current screen during long ads
- **Cascading recovery** — 8s single BACK → 15s triple BACK → 25s rapid 6× BACK → 35s kill WebView → 45s engine recover

### Smart reward overlay handling
- Template matching (`ok_template.png`) with 11 scales + CLAHE normalization
- Fallback yellow-button detection (HSV color mask)
- **Sacred Loop Rule:** never presses BACK on the game's main activity (a BACK would exit the target screen)

### Auto-update via GitHub Releases
Every new release published to https://github.com/s4production/adbot/releases is automatically detected by the bot on startup. Users get a clean update dialog — one click to download and relaunch.

## Quick start

### Requirements
- Windows 10/11 (64-bit)
- LDPlayer 9 (recommended) or any emulator with ADB
- The target game installed in the emulator
- AdBot license (purchase on Discord — see [`SUPPORT.md`](SUPPORT.md))

### Installation
1. Download the latest EXE from [Releases](https://github.com/s4production/adbot/releases/latest)
2. Run `AdBot_v12.14.exe` (no installation required)
3. Enter your license key
4. Start the emulator, launch the target game, navigate to the ad trigger screen
5. Click **Start** in the bot

Detailed setup: [`INSTALLATION.md`](INSTALLATION.md)

## Documentation

| File | Contents |
|---|---|
| [`INSTALLATION.md`](INSTALLATION.md) | LDPlayer setup, ADB configuration, first run |
| [`CHANGELOG.md`](CHANGELOG.md) | Full version history and bug fixes |
| [`SUPPORT.md`](SUPPORT.md) | Discord, FAQ, bug reporting |
| [`LICENSE.md`](LICENSE.md) | End-User License Agreement (EULA) |

## Support

- **Discord:** https://discord.gg/gn6DFnYF93
- **Issues:** [github.com/s4production/adbot/issues](https://github.com/s4production/adbot/issues)

## Project status

- ✅ Actively maintained
- ✅ Auto-update built-in (bot checks GitHub Releases on startup)
- ✅ In production since 2022
- 🛡️ Anti-piracy protection: HMAC license files, signed binaries, self-check

## Privacy

AdBot sends minimal data:
- **Hardware ID** — for license validation (sent to license server)
- **IP address** — visible to GitHub when checking for updates (normal HTTP)

AdBot does **NOT** send:
- Game credentials or passwords
- Screen contents or game data
- Any personal information beyond hardware ID

---

© 2022-2026 s4production. All rights reserved.
