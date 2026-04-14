# AdBot — Shakes & Fidget Fortune Wheel Automation

[![Version](https://img.shields.io/github/v/release/s4production/adbot)](https://github.com/s4production/adbot/releases/latest)
[![License](https://img.shields.io/badge/license-Commercial-blue)](LICENSE.md)

**AdBot** to profesjonalny bot automatyzujący oglądanie reklam w kole losu w grze mobilnej Shakes & Fidget. Bot działa na emulatorach Android (LDPlayer) i fizycznych telefonach podłączonych przez ADB.

## Co robi AdBot?

Bot wykonuje pełen cykl oglądania reklamy w kole losu:

1. **Wykrywa TV** w lewym-górnym rogu koła losu (template matching)
2. **Tapuje TV** → rozpoczyna sekwencję reklamową (3 reklamy pod rząd)
3. **Zamyka każdą reklamę** automatycznie wykrywając przycisk close (X, strzałki, skip)
4. **Po reklamach klika OK** na overlay'u z nagrodą
5. **Powtarza cykl** — setki reklam w ciągu dnia, bez twojej interwencji

## Kluczowe features v12.13

### Detekcja close button — 3-warstwowa
- **UI dump** (uiautomator) — szuka przycisków z keywordami w 5 językach (EN/PL/DE/FR/ES)
- **Template matching** — 13 templatów X/strzałek + syntetyczne X w 3 rozmiarach, multi-scale, 5 pass z blur/CLAHE
- **OCR** (Tesseract) — rozpoznaje teksty "skip", "close", "dismiss" na reklamach WebView

### Obsługa wszystkich głównych SDK reklamowych
- IronSource (ControllerActivity, playable ads, endcards)
- Google AdMob (AdActivity, TikTok/Google Play interstitials)
- Unity Ads (FullScreenWebViewDisplay)
- AppLovin, Vungle, AdColony, Chartboost, InMobi, MoPub
- **Fyber Inneractive** (z obsługą pułapek IACloseButton)
- Pangle, Facebook Ads, StartApp, DigitalTurbine, Tapjoy, Kidoz

### Anti-trap recovery (z realnych testów 60+ cykli)
- **Poison filter** — pozycje które otworzyły Google Play / Chrome zostają zablokowane na resztę cyklu
- **NO-TAP-ZONE** — bot nigdy nie tapnie dolnych 8% ekranu (ochrona przed wyjściem z koła losu)
- **Deceptive browser detection** — InternalBrowserActivity (Inneractive pułapka) → natychmiastowy BACK
- **Force-bring-to-foreground** — gdy Chrome ignoruje BACK, bot używa `monkey LAUNCHER` żeby wrócić do gry
- **Cascading recovery** — 8s BACK → 15s 3xBACK → 25s 6xBACK → 35s kill WebView → 45s engine recover

### Auto-navigation (pierwszy uruchom)
- Charselect screen → wybór postaci → miasto → koło losu (template matching kolo.png)
- Po pierwszym cyklu bot zostaje na kole losu (zero zbędnych tapów)

### OK reward overlay
- Template matching (`ok_template.png`) z 11 skalami + CLAHE
- Fallback yellow-button detection (HSV mask)
- **Nigdy BACK** na Unity focus (Sacred Loop Rule — BACK wyszedłby z koła)

## Szybki start

### Wymagania
- Windows 10/11
- LDPlayer 9 (zalecany) lub inny emulator z ADB
- Gra Shakes & Fidget zainstalowana w emulatorze
- Licencja AdBot (kup na Discord — patrz `SUPPORT.md`)

### Instalacja
1. Pobierz najnowszy EXE z [Releases](https://github.com/s4production/adbot/releases/latest)
2. Uruchom `AdBot_v12.13.exe` (nie wymaga instalacji)
3. Wprowadź licencję
4. Włącz emulator, uruchom grę, przejdź do koła losu
5. Kliknij **Start** w bocie

Szczegóły: [`INSTALLATION.md`](INSTALLATION.md)

## Dokumentacja

| Plik | Zawartość |
|---|---|
| [`INSTALLATION.md`](INSTALLATION.md) | Instalacja LDPlayer, konfiguracja ADB, pierwszy uruchom |
| [`CHANGELOG.md`](CHANGELOG.md) | Historia wersji i naprawionych bugów |
| [`SUPPORT.md`](SUPPORT.md) | Kontakt, FAQ, Discord, zgłaszanie błędów |
| [`LICENSE.md`](LICENSE.md) | Warunki licencji (EULA) |

## Wsparcie

- **Discord:** https://discord.gg/gn6DFnYF93
- **Issues:** [github.com/s4production/adbot/issues](https://github.com/s4production/adbot/issues)

## Status projektu

- ✅ Aktywnie rozwijany
- ✅ Auto-update wbudowany (bot sam sprawdza czy jest nowa wersja)
- ✅ Produkcja od 2022 roku (wcześniejsze wersje pod marką "SimpleBot Pro")
- 🛡️ Ochrona antypiracka: HMAC license files, signed binaries, self-check

---

© 2022-2026 s4production. Wszystkie prawa zastrzeżone. Shakes & Fidget jest znakiem towarowym Playa Games — AdBot nie jest oficjalnie powiązany z twórcami gry.
