# Installation Guide

## Wymagania systemowe

- **Windows 10/11** (64-bit)
- **4 GB RAM** (8 GB zalecane)
- **500 MB wolnego miejsca** (bot + emulator)
- **Połączenie z internetem** (sprawdzanie licencji + auto-update)

## Krok 1: Pobierz AdBota

1. Przejdź do [Releases](https://github.com/s4production/adbot/releases/latest)
2. Pobierz `AdBot_v12.13.exe` (~88 MB)
3. Umieść w dowolnym folderze (np. `C:\Gry\AdBot\`)

> ⚠️ Windows SmartScreen może zablokować uruchomienie. Kliknij **Więcej informacji → Uruchom mimo to**. EXE jest podpisany cyfrowo; niekiedy pierwszy raz wymaga ręcznego zaufania.

## Krok 2: Zainstaluj LDPlayer (zalecany emulator)

1. Pobierz LDPlayer 9 z [ldplayer.net](https://www.ldplayer.net/)
2. Zainstaluj wybierając ustawienia domyślne
3. Uruchom LDPlayer

### Konfiguracja LDPlayer

W LDPlayer Settings:
- **Performance → CPU**: 2 rdzenie
- **Performance → RAM**: 2048 MB
- **Resolution**: 540x960 (lub 1080x1920) — bot działa na każdym
- **Network → Device Mode**: `Phone`

## Krok 3: Zainstaluj docelową grę

1. W LDPlayer otwórz Google Play
2. Znajdź i zainstaluj grę mobilną z reklamami nagrodowymi
3. Zaloguj się na swoje konto
4. Przejdź do ekranu w którym pojawiają się reklamy nagrodowe

## Krok 4: Uruchom AdBota

1. Uruchom `AdBot_v12.13.exe`
2. Przy pierwszym uruchomieniu:
   - Wybierz język (PL/EN/DE/FR/ES)
   - Przeczytaj "Startup Guide"
   - Wprowadź licencję (z maila zakupu)

## Krok 5: Pierwszy start bota

**WAŻNE — pozycjonowanie:**

Bot ma **kategoryczny zakaz tapania czegokolwiek jeśli nie widzi przycisku TV** w kole losu. Musisz ręcznie doprowadzić grę do koła losu PRZED kliknięciem Start.

1. W grze: zaloguj się i wybierz postać
2. Przejdź do **Koła Fortuny** (ikona koła w dolnym pasku)
3. W bocie: kliknij **Refresh Devices** → wybierz swój emulator
4. Kliknij **Start**

Bot zacznie:
- Wykrywać TV w lewym-górnym rogu koła (template matching)
- Jeśli TV widoczne → tapnie → oglądanie reklam
- Jeśli TV niewidoczne → **CZEKA** (nie tapuje nic) aż się pojawi

## Krok 6: Co zrobić gdy TV nie jest widoczne

TV nie jest widoczne w kole losu kilka razy dziennie (limit Google) lub gdy:
- Animacja ptaka zasłania ikonę
- Pojawia się overlay nagrody
- Trwa animacja kolejnej obrotu

**Bot czeka cierpliwie do 120s** aż TV się pojawi. Jeśli nie pojawi się w tym czasie, bot zwraca `TV not visible`.

W takich przypadkach:
- Sprawdź czy jesteś na kole losu
- Jeśli osiągnąłeś dzienny limit reklam → wróć za kilka godzin
- Upewnij się że gra nie jest wyciszona (Google weryfikuje dźwięk w niektórych reklamach)

## Krok 7: Auto-update

Bot sprawdza automatycznie przy starcie:
```
https://api.github.com/repos/s4production/adbot/releases/latest
```

Jeśli jest nowsza wersja:
- Wyświetli dialog z changelogiem
- **Download** → pobiera EXE do `%TEMP%\AdBot_v12_update\`
- **Launch** → uruchamia nowy EXE, zamyka stary

Auto-update jest OPCJONALNY — zawsze można kliknąć **Skip**.

## Konfiguracja zaawansowana

### Plik config (`adbot_v11_config.json`)

Tworzony automatycznie w `%TEMP%\AdBot_v11\`:
```json
{
    "language": "pl",
    "first_run": false,
    "auto_navigate": false,
    "character_slot": 1,
    "device_serial": "emulator-5554"
}
```

- `auto_navigate`: **false** by default (v12.13). Jeśli ustawisz `true`, bot przy pierwszym cyklu spróbuje automatycznie przejść z ekranu wyboru postaci → miasta → koła. **Ryzykowne** — może tapnąć w nieprzewidziane miejsca.
- `character_slot`: 1-4. Którą postać wybrać na charselect screen.
- `device_serial`: ID emulatora (`emulator-5554` dla LDPlayer default).

### Debug mode

Bot zapisuje dane debug do `D:\BOT DANE\cycle_NNN\`:
- `iNNNN.png` — screenshoty każdej iteracji
- `iNNNN_ui.txt` — sparsowane UI dump elementy
- `iNNNN_focus.txt` — aktualny focus
- `decisions.log` — log decyzji bota

Przydatne do zgłaszania bugów (patrz `SUPPORT.md`).

## Problemy

### "TV not visible after 120s, skipping"

- Sprawdź że jesteś na kole losu (nie w mieście / charselect)
- Sprawdź że ekran koła jest widoczny (nie zasłonięty overlay'em / powiadomieniem)
- Zwiększ głośność gry (niektóre reklamy wymagają dźwięku)

### Bot nie widzi emulatora

- W bocie kliknij **Refresh Devices**
- W LDPlayer upewnij się że ADB jest włączone (Settings → Advanced → ADB debugging = ON)
- Restart LDPlayer jeśli nadal nie widzi

### Chrome się otwiera po reklamie

- To feature nie bug — v12.10 bot sam zamyka Chrome przez `monkey LAUNCHER`
- Jeśli Chrome zostaje otwarty >5s to zgłoś w Issues (załącz screenshot + focus)

### Bot wyszedł z koła losu

- To **nie powinno** się zdarzyć w v12.13 (kategoryczny zakaz tapów gdy brak TV)
- Jeśli się zdarzy: zgłoś Issue z `D:\BOT DANE\cycle_NNN\` — chcemy to naprawić natychmiast

## Support

Szczegóły w [`SUPPORT.md`](SUPPORT.md).
