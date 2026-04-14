# CHANGELOG

Pełna historia wersji AdBota z opisem naprawionych bugów i dodanych funkcji. Najnowsze wersje na górze.

---

## v12.14 (2026-04-14) — RACE-SAFE BACK (wheel exit fix)

### 🐛 Fix: Bot wychodził z koła na długich reklamach

**Problem:** Na długich reklamach bot wchodził w cascading recovery (po 8s/15s/25s z zacięcia). Recovery odpalało 1×/3×/6× BACK w pętli. Jeśli reklama **naturalnie kończyła się podczas tej pętli BACK**, jeden z BACK-ów trafiał w **Unity (wheel)** zamiast w reklamę → wyjście z koła do miasta.

Race condition:
```
1. Ad stuck 15s → cascade fires level 2 (3x BACK)
2. BACK #1 → ad closes naturally (Unity focus)
3. BACK #2 → lands on Unity wheel → EXIT to city!
```

**Fix:** Wszystkie BACK-e sprawdzają focus **PRZED** wysłaniem (nie po):

```python
# STARE (bug):                   # NOWE (fix):
adb.back()                       # if "UnityPlayerActivity" in focus:
time.sleep(gap)                  #     return  # ad ended, SKIP BACK
if unity: return  # za późno     # adb.back()
```

Zastosowane w 4 miejscach:
- `_ad_recovery_cascade` — cascade levels 1/2/3
- `_back_from_foreign` — recovery z Google Play/Chrome
- `close_ad_v15` timeout path — 4× BACK fallback
- Deceptive browser second BACK

Bot nigdy już nie BACK-nie na Unity. Długie reklamy działają bez wyjścia z koła.

---

## v12.13 (2026-04-14) — CATEGORICAL TAP BAN + GITHUB RELEASES

### 🛡️ Fix: Kategoryczny zakaz tapów gdy TV nie jest widoczne

**Problem:** Gdy TV było zasłonięte (animacja, overlay), bot po 20s próbował auto-navigate. `detect_screen` defaultował do `SCREEN_CITY` → bot tapował wheel pin → pin matchował dolny navbar wheel-tab icon → **bot wychodził z koła losu**.

**User request:** "ON MA KATEGORYCZNY ZAKAZ KLIKANIA CZEGOKOLWIEK JAK NIE MA TV"

**Fix — 3 zmiany:**

1. **`auto_navigate` DEFAULT = False** (było True). Bot nie będzie nawigować bez zgody użytkownika. Użytkownik musi ręcznie doprowadzić grę do koła przed startem.

2. **`detect_screen` fallback = SCREEN_UNKNOWN** (było SCREEN_CITY). Jeśli bot nie jest w 100% pewny co widzi → nie zgaduje.

3. **SCREEN_UNKNOWN w navigate_to_wheel = NO TAP, NO BACK**. Wcześniej na unknown screen bot dawał BACK (który mógł wyjść z koła). Teraz czeka.

### 🔄 Nowy URL auto-updatera — GitHub Releases

Bot przeszedł z custom servera na GitHub Releases:
- **Stare:** `http://dew-games.media.pl/version.txt`
- **Nowe:** `https://api.github.com/repos/s4production/adbot/releases/latest`

Przy starcie bot pobiera najnowszy release z GitHub API i porównuje tag (np. `v12.13`) z aktualną wersją. Jeśli dostępny update → download EXE z asset browser_download_url.

Download URL jest dynamicznie pobierany z GitHub (nie hardcoded) — każdy release automatycznie staje się dostępny.

### 🔗 Repo publikacyjne

- Repository: https://github.com/s4production/adbot
- Releases: https://github.com/s4production/adbot/releases
- Issues: https://github.com/s4production/adbot/issues

Repo zawiera TYLKO dokumentację — kod źródłowy pozostaje prywatny (ochrona IP i antypiracka).

---

## v12.12 (2026-04-14) — TIGHTER VISUAL REGIONS

### 🐛 Fix: Fałszywe matche na endcardach instalacji

**Problem:** Na endcardzie "Tasty Travels: Merge" (IronSource promo) bot zwracał `(932, 232)` ze score 0.95 zamiast prawdziwego X w `(1020, 135)`. Template `x11` po CLAHE-normalizacji matchował się do ikony aplikacji na score wyższy niż prawdziwy X.

**Przyczyna:** Regiony visual search były zbyt wysokie (`y < 18% = 345px`). Pod prawdziwym X na pozycji y=135 były app icon, title, rating stars — wszystko w tym samym regionie.

**Fix:**
```
top-right ROI:  y < 18% (345px)  →  y < 10% (192px)
top-left ROI:   y < 18% (345px)  →  y < 10% (192px)
top-center ROI: y < 10% (192px)  →  y < 7%  (134px)
```

Fałszywy match `y=232` teraz automatycznie odrzucony (232 > 192). Prawdziwy X `y=135` zostaje (135 < 192).

Zweryfikowane na 3 zaciętych reklamach: Tasty endcard, Tasty Tops arrows, Whiteout Survival arrows — wszystkie 3 idealnie trafione.

---

## v12.11 (2026-04-14) — FOOD ICON FALSE MATCH FIX

### 🐛 Fix: Bot tapował ikony jedzenia w dolnej nawigacji reklam gier

**Problem:** Reklamy gier typu "Tasty Tops" mają kolorowy pasek z ikonami jedzenia na dole ekranu. Template `x11` (mała kolorowa ikonka close button) matchował się do tych ikon ze score 0.95, bijąc prawdziwe strzałki w prawym-górnym (score 0.91). Bot zwracał `(211, 1827)` zamiast `(1003, 78)`.

**Fix:** Usunięto `bottom-right` i `bottom-left` z regionów visual search. Prawdziwe close buttons ZAWSZE w TOP corners/top-center. Bottom corners były źródłem false positives (ikony gry, paski jedzenia).

**Bonus:** Inneractive `IACloseButton` na dole (pułapka otwierająca przeglądarkę) jest teraz doubly zablokowany — visual nie matchuje bottom, a jeśli UI dump zwróci tę pozycję, `_in_no_tap_zone` ją odrzuci.

---

## v12.10 (2026-04-14) — CHROME REFUSES BACK

### 🐛 Fix: Reklama otwiera Chrome, BACK nie zamyka

**Problem:** Reklamy otwierały Chrome z zewnętrznym URL (np. `kasho.pl/?gad_source=...`). Bot wykrywał Chrome jako foreign app, ale **BACK keyevent nie działał** — Chrome konsumuje BACK dla swojej historii przeglądania zamiast zamykać aktywność. Bot tapował BACK 4 razy, nic się nie działo, bot utykał na zewnętrznej stronie.

**Test ręczny:**
```
adb shell input keyevent BACK  → Chrome IGNORUJE (focus dalej Chrome)
adb shell monkey -p com.example.yourgame -c LAUNCHER 1  → ✅ Unity (gra)!
```

**Fix:** Nowa funkcja `_force_game_to_foreground()`:
```python
adb.shell("monkey", "-p", "com.example.yourgame",
          "-c", "android.intent.category.LAUNCHER", "1")
```

Używa Android `monkey` do wysłania LAUNCHER intent — natychmiast przywraca grę na foreground. Chrome zostaje w tle (system go zwolni przy niskiej pamięci). Zintegrowane w `_back_from_foreign`: najpierw 3x BACK, jeśli nie działa → force foreground.

---

## v12.9 (2026-04-14) — SPEED UP RECOVERY

### ⚡ Usprawnienie: 2× szybszy recovery

User feedback: "Ratuje się ale za wolno". Optymalizacje:

| Sytuacja | v12.8 | v12.9 |
|---|---|---|
| Foreign app BACK gap | 1.3s × 4 = **5.2s** | 0.6s × 4 = **2.4s** |
| First BACK przy zacięciu | 15s | **8s** |
| Aggressive 3xBACK | 25s | **15s** |
| Rapid 6xBACK | 40s | **25s** |
| Kill WebView | 55s | **35s** |
| Hard abort | 70s | **45s** |

### 🐛 Fix: Inneractive `InternalBrowserActivity` — nowy typ pułapki

Reklama Inneractive ma "IACloseButton" na dole który zamiast zamknąć reklamę otwiera `InneractiveInternalBrowserActivity` z tracking URL który daje 404 ("Webpage not available"). Bot wcześniej czekał 8-15s na cascading recovery żeby wyjść.

**Fix:** Nowa lista `_DECEPTIVE_BROWSER_MARKERS`:
- `internalbrowser` (Inneractive)
- `inappbrowser` (generic)
- `fullscreenwebview` (Unity Ads)

Wykrycie → natychmiastowy BACK + poison tej pozycji. Recovery w 0.6s zamiast 8-15s.

---

## v12.8 (2026-04-14) — VISUAL MATCHING PRIORITY FIX

### 🐛 Fix: Bot nie trafiał w strzałki >> (Tasty Tops, Whiteout Survival)

**Problem:** Reklama "Whiteout Survival" (playable Cocos ad) wymagała tapnięcia strzałek `>>` w prawym-górnym. UI dump pokazywał strzałki jako `clickable=False` android.view.View — niewidzialne dla bota. Visual matching miał je znaleźć, ale zwracał `(484, 74)` ze score 0.58 (fałszywy match syntetycznego X w top-center).

**Root cause:** Flow bota:
```
1. Syntetyczne X templates → score 0.58 w (484, 74) → MATCH (> próg 0.55)
2. External templates (arrowx2, strzalki) → NIGDY nie odpalały (bo synthetic "wygrał")
```

Mimo że `arrowx2` template dawał score **0.93** w prawdziwej pozycji `(1003, 78)` — nigdy nie były testowane.

**Fix — dwie zmiany:**

1. **Próg syntetycznych podniesiony:** `0.55 → 0.70`
2. **External templates ZAWSZE odpalają** (nie jako fallback) — bot porównuje marginy nad progiem obu metod i wybiera lepszy

### ➕ Dodano nowy template `strzalki2.jpg`

Customer-provided template dla konkretnego typu strzałek. Dodany do loadera + bundlowania w EXE. 13 external templates total.

---

## v12.7 (2026-04-14) — SIMPLE POST-AD FLOW

### 🐛 Fix: Bot zbyt skomplikowanie szukał koła po reklamie

**User request:** "Bez kombinowania. OK → TV → tap TV. Nie ma TV = czekam."

**Przed (v12.6):**
```
Po reklamie → dismiss_reward_overlay → _recover_to_wheel()
_wait_for_tv po 20s próbuje re-navigate (tapuje pin w mieście)
Pin w mieście czasem matchował do bottom-navbar wheel-tab icon → bot wychodził z koła
```

**Po (v12.7):**
```python
# Auto-nav TYLKO przy pierwszym cyklu (cycle_idx == 1)
# Po cyklu: NIGDY _recover_to_wheel — ufamy wait_for_tv następnego cyklu
# _wait_for_tv: BEZ re-navigate, BEZ tapów. Tylko czeka na TV.
```

Flow teraz prosty jak obrazek:
1. `close_ad_v15` → SUCCESS
2. `dismiss_reward_overlay` → tap OK
3. `wait_for_tv` → czeka (zero tapów)
4. `tap TV` → cykl
5. Powtórz

---

## v12.6 (2026-04-14) — NO-TAP-ZONE + CASCADING RECOVERY

### 🐛 Fix: Bot tapował ikony w dolnym navbarze koła losu

**Problem:** Gdy TV było ukryte (animacja ptaka), bot po 20s próbował re-navigate. `detect_screen` nie wykrywał SPIN (teal) → default CITY → `get_wheel_pin_pos` szukał kolo.png → matchował małą ikonę koła w **dolnym navbarze koła losu** → bot tapował → wychodził z koła.

**Fix A:** `kolo.png` i `tpl_wheel_pin` template matching ograniczone do **górnych 85% ekranu** (dolne 15% = navbar, zakazany region).

**Fix B:** Nowa funkcja `_in_no_tap_zone(pos, sw, sh)` — kategoryczny zakaz tapowania dolnych 8% (`y > sh * 0.92`) w close_ad_v15. Jeśli UI dump lub visual zwróci bottom pozycję → odrzuć + wymuś visual fallback w górnych regionach.

### 🐛 Fix: UI dump tapował phantom TikTok icon

**Problem:** Reklama "TikTok" Google Ads miała TikTok app icon w top-left (`[72,72] 144x144 click=False`). Heurystyka bota "30-200px element w rogu = skip button" wybierała icon jako close, tapowała → nic się nie działo → STUCK.

**Fix:** W heurystyce wymóg `clickable=True`. Non-clickable phantom elementy (icons, text labels) są teraz pomijane.

### ➕ Cascading Recovery

Bot nigdy więcej nie zacina się na reklamie dłużej niż kilkadziesiąt sekund. Track stuck duration (since last screen hash change):

- 15s stuck → Level 1: 1× BACK
- 25s → Level 2: 3× BACK rapid (0.7s gap)
- 40s → Level 3: 6× BACK rapid (0.4s gap)
- 55s → Level 4: Kill WebView sandbox processes (nie zabija gry)
- 70s → HARD ABORT → engine `_recover_to_wheel` przejmuje

---

## v12.5 (2026-04-14) — POISON FILTER

### 🐛 Fix: Fyber Inneractive `IACloseButton` pułapka

**Problem:** Reklama Fyber Inneractive ma UI dump z elementem:
```
[810,1800][1080,1896] desc=IACloseButton clickable=true
```

Nazwa sugeruje close button, ale tapnięcie go otwiera Google Play (deep link do install). Bot tapował, Chrome się otwierał, `_back_from_foreign` wracał do reklamy, bot znowu widział IACloseButton, znowu tapował — **nieskończona pętla**.

**Fix:** `poisoned_positions` — per-cycle lista pozycji które doprowadziły do foreign app. Po każdym tapie, jeśli focus zmieni się na foreign w <4s → pozycja jest "zatruwana" (50px radius). W kolejnych iteracjach bot odmawia tapowania w tym miejscu — wymusza fallback na visual matching.

Dziala dla KAŻDEGO przyszłego deceptive close button, niezależnie od SDK.

---

## v12.4 (2026-04-14) — 60-CYCLE DATA LEARNINGS

### ➕ Based on 60-cycle autonomous run

Uruchomiony nocny test 60 cykli z pełnym debug logowaniem do `D:\BOT DANE\cycle_NNN\`. Wynik: 60/60 SUCCESS, ale zauważone patterny:

### 🚀 Early OK probe

Niektóre reklamy kończą się natychmiast po tapie TV (overlay reward pojawia się bez reklamy). Bot czekał 180s w `close_ad_v15` zamiast od razu tapnąć OK.

**Fix:** Po każdym `tap TV + 1.5s` bot sprawdza czy OK template już widoczny. Jeśli tak → tap OK → Unity focus → INSTANT-CLOSE w <3s.

### 🚀 Dynamic post-tap delay

Dane z 60 cykli: 184 tapy w top-left, 29 stuck eventów (15% failure rate). Animacja close button po tapnięciu top-corner trwa dłużej niż typowa.

**Fix:** Post-tap sleep zależny od pozycji:
- Top-corner (y<15%, x<20% lub x>80%): **1.2s**
- Inne: 0.7s

### 📊 Per-cycle close time logging

Logging: `close_ad finished in X.Xs success=True/False` — pozwala wykrywać outliers i tuning progowych.

---

## v12.3 (2026-04-14) — OK TEMPLATE + DISMISS OVERLAY

### 🐛 Fix: Bot wychodził z koła losu po reklamie

**Regression:** Wcześniejsza wersja robiła BACK na overlay rewardu. BACK na Unity = wyjście z koła losu. Klient tracił postęp.

**Fix:** Przywrócono logikę z v12.0:
1. Template matching `ok_template.png` (422x142, 11 skal, multi-blur)
2. Fallback yellow button detection (HSV mask)
3. Last resort tap (sw/2, sh*0.825)
4. **NIGDY BACK** jeśli focus = Unity

Sacred Loop Rule: `NEVER BACK on Unity overlay`.

### ➕ Tesseract OCR bundling

Tesseract OCR (16MB) wbudowany w EXE:
- `tesseract.exe` + 12 DLL
- `tessdata/eng.traineddata`
- Bot używa OCR jako 3rd layer detection (po UI dump + template matching)

Wykrywa słowa: skip, close, dismiss, exit, zamknij, pomiń (multi-language).

---

## v12.2 (2026-04-13) — LICENSE SECURITY AUDIT

### 🛡️ 15 luk bezpieczeństwa zidentyfikowanych i naprawionych

Audyt licensing system znalazł krytyczne luki:

1. **Public key w configu** (klient mógł podmienić) → hardcoded w frozen EXE
2. **Brak HMAC na state files** → dodano HMAC-SHA256
3. **Time-skew attack** (klient cofał zegar) → sprawdzanie monotonic clock
4. **Rollback attack na memory file** → counter w HMAC
5. **Fail-open on security check crash** → fail-closed (blokuje uruchomienie)
6. **Brak signature verification na updates** → dodano RSA signature check
7. (i 9 innych)

Security gate w `simplebot_pro_main.py`: jeśli `startup_security_gate()` rzuci exception → aplikacja **nie startuje**.

---

## v12.1 (2026-04-13) — NAVIGATOR BETA

### ➕ Auto-navigate do koła losu

Nowy moduł `navigator.py`:
- `detect_screen()` — rozpoznaje charselect / city / wheel na podstawie koloru i template matching
- `navigate_to_wheel()` — automatycznie przechodzi z charselect → city → wheel
- Template `kolo.png` — pin koła losu w mieście

Przy uruchomieniu bot sam trafia na koło losu, nawet jeśli gra startuje na innym ekranie.

---

## v12.0 (2026-04-12) — MAJOR CLOSE_AD REFACTOR

### 🎯 close_ad_v15 — universal hybrid ad closer

Przepisany od nowa (poprzednia wersja `close_ad_v14` zachowana jako fallback — Sacred Loop Rule):

**3-warstwowa detekcja close button:**
1. **UI dump** (highest priority) — 25+ keywordów w 5 językach
2. **Template matching** (X-shape + 12 external templates + strzalki)
3. **OCR** (Tesseract fallback)

**Safety features:**
- Screen change detection (perceptual hash watchdog)
- Click debounce (<1.5s same target = skip)
- Same-tap guard (6× w to samo miejsce = STUCK, break)
- Large-CTA rejection (nie tapuj wielkiego "Install" button w środku)
- Unity-only escape (focus=Unity + no real widgets = SUCCESS)

**Per-cycle debug data:** wszystko (screenshots, UI dump, focus, decisions) zapisywane do `D:\BOT DANE\cycle_NNN\`.

---

## v11.x (2026-04-10 → 2026-04-12) — SimpleBot Pro rebrand + TV detector

### ➕ Professional GUI (CustomTkinter)
- Welcome dialog (wybór języka PL/EN/DE/FR/ES)
- Splash screen (loading templates, license, devices)
- Main window: config panel, stats, log viewer, preview
- Startup guide dialog (krok-po-kroku dla nowych klientów)
- Update dialog (auto-update notification)

### ➕ TV detector — resolution-agnostic template matching
- Multi-scale (0.35x - 1.2x) żeby działać na każdym telefonie
- ROI-based search (TV zawsze w lewym-górnym)
- Stability check — 2 kolejne detekcje w 60px radius przed akceptacją (eliminuje noise matches)
- Osobne thresholdy dla emulatora (0.35) vs telefonu (0.50)

### ➕ All-in-one EXE bundling (PyInstaller)
- ADB + DLLs embedded (AdbWinApi.dll, AdbWinUsbApi.dll)
- 30+ template PNG/JPG
- Tesseract OCR + 12 DLL + eng.traineddata
- Zero external dependencies

---

## v10.x (2022-2026) — Production core

Pierwsza wersja produkcyjna. Jednoekranowy GUI, podstawowy close_ad_v14 (sekwencyjny UI dump → template fallback), basic TV detection.

Notable milestones:
- v10.1 (2023): License system (RSA-2048 keys, HMAC integrity)
- v10.1.5 (2024): LDPlayer emulator support + ratio-based tap positions
- v10.2 (2025): Per-device config profiles
- v10.3 (Q1 2026): External X template expansion (x1-x11)

---

## Sacred Loop Rule

⚠️ **Nigdy nie modyfikuj `close_ad_v14` core logic bez ekstremalnej ostrożności.** Jest zachowany jako fallback safety net — nawet jeśli `close_ad_v15` ma bug, v14 uratuje klienta. Każda zmiana v14 wymaga testów na 20+ reklamach.

## Feedback & bug reports

Zgłaszaj problemy przez [GitHub Issues](https://github.com/s4production/adbot/issues) z:
- Wersja bota (widoczna w title barze)
- Screenshot reklamy
- Log z debug data (`D:\BOT DANE\cycle_NNN\`)
- Focus string (`adb shell dumpsys window windows | grep mCurrentFocus`)

---

© 2022-2026 s4production
