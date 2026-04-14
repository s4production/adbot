# Support

## Jak uzyskać pomoc

### 1. Discord (najszybciej)

Dołącz do serwera Discord: **https://discord.gg/gn6DFnYF93**

Kanały:
- `#faq` — częste pytania i odpowiedzi
- `#pomoc` — zgłoszenia problemów (polski)
- `#support` — support (English)
- `#zakup` — zakup licencji
- `#aktualizacje` — ogłoszenia nowych wersji

### 2. GitHub Issues

Bugi i sugestie: https://github.com/s4production/adbot/issues

**Szablon zgłoszenia:**

```markdown
**Wersja bota:** v12.13 (widoczna w title barze)
**System:** Windows 11
**Emulator:** LDPlayer 9.0.x (lub "Physical phone: Samsung Galaxy S21")

**Co robiłem:**
Krok 1. Uruchomiłem bota
Krok 2. Kliknąłem Start
Krok 3. Po 3 cyklu bot zaciął się na reklamie

**Oczekiwane zachowanie:**
Bot powinien zamknąć reklamę i przejść do następnego cyklu.

**Rzeczywiste zachowanie:**
Bot tapał 6x w tym samym miejscu, potem STUCK, potem BACK — wyszedł z koła losu.

**Logi / screenshots:**
- Screenshot: [załącz]
- Debug folder: C:\path\to\D:\BOT DANE\cycle_003.zip [załącz ZIP]
- Focus: mCurrentFocus=Window{...} [kopiuj z `adb shell dumpsys window windows`]
```

### 3. Email

Dla spraw licencyjnych i zakupowych: **[email kontaktowy]**

## FAQ

### Gdzie mogę kupić licencję?

Licencje są dostępne na Discord w kanale `#zakup`. Zapłać przez PayPal / BLIK / Przelewy24 → otrzymasz plik `adbot_license.json` na maila.

### Licencja jest na ile urządzeń?

1 licencja = 1 urządzenie (identyfikowane przez hardware ID). Chcesz uruchomić bota na 2 emulatorach? Kup 2 licencje.

### Można odnowić / przenieść licencję?

- **Odnowienie:** Tak, kontakt na Discord / email
- **Transfer na nowy PC:** Tak (raz w roku za darmo), kontakt na Discord

### Bot jest legalny?

AdBot **automatyzuje** oglądanie reklam — robi to co ty robiłbyś ręcznie, tylko szybciej. Nie modyfikuje gry, nie czyta pamięci, nie exploituje żadnych luk.

**JEDNAK:** Wydawcy gier mogą zabronić botów w swoich Terms of Service. Używaj na własne ryzyko. Używanie wielu kont przez bota może skutkować banem.

### Bot zawiesił się — co robić?

1. Kliknij **Stop** w bocie
2. Sprawdź czy jesteś w kole losu
3. Kliknij **Start** — bot podejmie pracę

Jeśli bot nadal nie działa:
1. Uruchom z zapisem debug data (domyślnie włączone, `D:\BOT DANE\`)
2. Reprodukuj problem
3. Zgłoś Issue z debug folderem

### Czy bot wygląda na oczywisty pirat/cheat?

Nie. Bot tapuje ekran tak jak człowiek — losowe małe delaye, fizyczne taps przez ADB, nie modyfikuje procesu gry.

### Czy bot jest bezpieczny dla mojego konta?

Nie ma gwarancji. Wydawcy gier mogą teoretycznie wykryć wzorce automatyzacji. Z doświadczenia ~4 lat produkcji: zero znanych banów wśród klientów.

**Zalecenia:**
- Nie uruchamiaj na głównym koncie (używaj altów)
- Nie uruchamiaj 24/7 (weź przerwy w ciągu dnia)
- Nie uruchamiaj przez VPN/proxy

## Wymagane informacje przy zgłoszeniu

Zanim napiszesz Issue, zbierz:

### 1. Wersja bota

Widoczna w title barze głównego okna, np. "AdBot v12.13".

### 2. Screenshot zaciętej reklamy

```cmd
adb shell screencap -p /sdcard/debug.png
adb pull /sdcard/debug.png
```

### 3. Focus activity

```cmd
adb shell dumpsys window windows | findstr mCurrentFocus
```

Wynik np.:
```
mCurrentFocus=Window{... com.example.yourgame/com.google.android.gms.ads.AdActivity}
```

### 4. UI dump

```cmd
adb exec-out uiautomator dump /dev/tty > ui.xml
```

### 5. Debug folder

Z `D:\BOT DANE\cycle_NNN\` (ostatni cykl w którym wystąpił problem):
- `iNNNN.png` — screenshoty
- `iNNNN_ui.txt` — UI dumpy
- `decisions.log` — decyzje bota

Zzipuj folder i załącz do Issue.

## Refund policy

30 dni gwarancji zwrotu pieniędzy bez pytań. Kontakt przez Discord / email.

## Kontakt

- **Discord:** https://discord.gg/gn6DFnYF93
- **Email:** [email]
- **GitHub:** [@s4production](https://github.com/s4production)

---

© 2022-2026 s4production
