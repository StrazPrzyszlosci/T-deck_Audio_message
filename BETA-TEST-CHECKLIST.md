# Beta-test checklist — T-Deck Audio Message firmware

**Wypełnij po każdym teście i wklej/załącz jako issue w tym repo** (albo wyślij jako odpowiedź w
[dyskusji](https://github.com/meshtastic/firmware/discussions/12093)). Nie musisz testować wszystkiego —
wypełnij to, co sprawdziłeś/aś, i zostaw komentarze tam, gdzie coś działa inaczej.

- Urządzenie: [ ] T-Deck [ ] T-Deck Plus | Preset: ______ | Region: ______
- Firmware: ______ (np. 2.8.2.ca10d3d zwykły / 2.8.2.04de643-log) | Data testu: ______
- Tester: ______ | Drugi węzeł w teście (stock czy też ten fork?): ______

## PL — lista kontrolna

Odpowiedzi: `[x]` = tak · `[ ]` = nie sprawdzono · **NIE/CZĘŚCIOWO** dopisz w komentarzu.

### 1. Start i podstawy
- [ ] TAK / NIE / CZĘŚCIOWO — Firmware się uruchamia (boot do ekranu głównego, bez pętli restartów)
- [ ] TAK / NIE / CZĘŚCIOWO — Urządzenie bootuje poprawnie po kilku cyklach zasilania (włącz/wyłącz z baterii)
- [ ] TAK / NIE — Ekran, klawiatura i trackball działają normalnie
- [ ] TAK / NIE — Linia `S:B:...` w logu (USB) zawiera wersję 2.8.2.x i repo `StrazPrzyszlosci/T-deck_Audio_message`

### 2. Łączność z telefonem i komputerem
- [ ] TAK / NIE / CZĘŚCIOWO — Urządzenie łączy się z telefonem przez **Bluetooth (BLE)** — tak, dotyczy też T-Decka: BLE to jedyna łączność bezprzewodowa z aplikacją (alternatywnie USB)
- [ ] TAK / NIE — Widoczne przez **USB** (konsola szeregowa 115200 działa, logi się przewijają)
- [ ] TAK / NIE — Aplikacja pokazuje węzeł poprawnie (nazwa, numer, bateria)
- [ ] TAK / NIE — Region ustawiony i radio nadaje (węzeł widzi sąsiadów / pozycje / telemetrię)

### 3. Kompatybilność ze zwykłą siecią Meshtastic
- [ ] TAK / NIE / CZĘŚCIOWO — Urządzenie **odbiera wiadomości tekstowe** od węzłów **bez** tego firmware
- [ ] TAK / NIE / CZĘŚCIOWO — Zwykłe węzły odbierają teksty od testowanego urządzenia
- [ ] TAK / NIE / NIE TESTOWANO — Pakiety przechodzą przez relay/stockowe węzły na drodze (2+ hopów)
- [ ] TAK / NIE — Żaden stockowy węzeł nie zgłasza błędów przy pakietach głosowych testera (zapytaj właścicieli)

### 4. Moduł audio w ustawieniach
- [ ] TAK / NIE — W aplikacji jest włącznik: **Module Config → Audio → Codec2 (Enabled)**
- [ ] TAK / NIE — Po włączeniu i restarcie w logu widać `Voice: self-test ok` oraz `ES7210 mic online`
- [ ] TAK / NIE / CZĘŚCIOWO — Można **wybrać rate/bitrate audio** (3200/2400/1600/1400/1300/1200/700C) i zmiana jest widoczna po restarcie w `!voice status`
- [ ] TAK / NIE — Mikrofon nagrywa cokolwiek (w `-log` buildzie: widać `chunk airtime ~...` przy nagrywaniu)

### 5. Wiadomości głosowe (najważniejsze — najlepiej dwa T-Decki z tym firmware)
- [ ] TAK / NIE / CZĘŚCIOWO — Komenda `!voice` (z klawiatury lub telefonu) **uruchamia nagrywanie** i NIE leci w eter jako tekst
- [ ] TAK / NIE — Nagrywanie pokazuje odliczanie; dowolny klawisz przerywa i wysyła
- [ ] TAK / NIE / CZĘŚCIOWO — **Wiadomość audio zostaje wysłana** (w `-log`: linie `TX chunk k/n bytes ~ airtime`)
- [ ] TAK / NIE / CZĘŚCIOWO — **Drugi T-Deck odbiera i odtwarza audio** z głośnika, automatycznie, jednym ciągiem
- [ ] TAK / NIE — Odtwarzanie da się przerwać / nie odtwarza się ponownie bez powodu
- Jakość głosu (ocena 1–5): przy 1300 bps: ___ · przy wybranym innym trybie (____): ___ · Komentarz: ______
- [ ] TAK / NIE — Tryb **PTT** działa (`!voice ptt` → trzymanie środkowego trackballa nagrywa, puszczenie wysyła)
- [ ] TAK / NIE — `!voice gain <0-38>` zmienia słyszalny poziom nagrania
- [ ] TAK / NIE — `!voice vol <0-16>` zmienia głośność odtwarzania
- [ ] TAK / NIE / NIE TESTOWANO — Zgubiony fragment = cisza w jego miejscu, reszta wiadomości odtwarza się (test na granicy zasięgu)
- [ ] TAK / NIE / NIE TESTOWANO — Budżet airtime działa (przy 3200 bps na Short Slow nagranie samo się skraca do ~9 s; w `-log`: `capped`)
- [ ] TAK / NIE — Wiadomości audio działają też **na kanale prywatnym / jako DM** (szyfrowane)
- [ ] TAK / NIE — Czas wysyłki 10 s wiadomości na Short Slow to ~10–15 s zegarowych (8–9 pakietów co 1 s)

### 6. Stabilność i raport
- [ ] TAK / NIE — Urządzenie stabilne podczas sesji głosowej (bez restartów/crashów; ~godzina użytkowania)
- [ ] TAK / NIE / NIE TESTOWANO — Zużycie baterii z włączonym modułem audio akceptowalne (subiektywnie: jak zwykle / gorzej: ___)
- [ ] TAK / NIE / NIE TESTOWANO — Limiter duty cycle nie blokował normalnego użytkowania (brak "exceeding duty cycle" w logu przy rozsądnym użyciu)
- [ ] TAK / NIE — `!voice logsend` (jako DM) dostarcza logi do czatu na telefonie
- [ ] TAK / NIE — Powrót na **oficjalny firmware** przebiegł bez problemów, ustawienia/klucze przetrwały
- Załącz proszę: fragment logu USB (szczególnie z builda `-log`, linie `Voice:`, `TX chunk`, `RX chunk`, `S:B:`) + opis warunków testu (odległość, przeszkody, preset)

---

# Beta-test checklist (English)

**Fill in after each test session and paste/attach as an issue in this repo** (or reply in the
[discussion](https://github.com/meshtastic/firmware/discussions/12093)). You don't have to test everything —
fill in what you checked and comment wherever behavior differs.

- Device: [ ] T-Deck [ ] T-Deck Plus | Preset: ______ | Region: ______
- Firmware: ______ (e.g. 2.8.2.ca10d3d normal / 2.8.2.04de643-log) | Test date: ______
- Tester: ______ | Second node in the test (stock or this fork too?): ______

## EN — checklist

Answers: `[x]` = yes · `[ ]` = not checked · write **NO/PARTIAL** in the comment.

### 1. Boot and basics
- [ ] YES / NO / PARTIAL — Firmware boots (reaches the main screen, no boot loops)
- [ ] YES / NO / PARTIAL — Device boots correctly across several power cycles (battery on/off)
- [ ] YES / NO — Screen, keyboard and trackball behave normally
- [ ] YES / NO — The `S:B:...` line in the (USB) log shows the 2.8.2.x version and the `StrazPrzyszlosci/T-deck_Audio_message` repo

### 2. Phone and computer connectivity
- [ ] YES / NO / PARTIAL — Device connects to the phone over **Bluetooth (BLE)** — yes, this applies to the T-Deck too: BLE is its only wireless link to the app (USB is the alternative)
- [ ] YES / NO — Visible over **USB** (serial console at 115200 works, logs scroll)
- [ ] YES / NO — App shows the node correctly (name, number, battery)
- [ ] YES / NO — Region set and radio transmits (sees neighbors / positions / telemetry)

### 3. Compatibility with a stock Meshtastic network
- [ ] YES / NO / PARTIAL — Device **receives text messages** from nodes **without** this firmware
- [ ] YES / NO / PARTIAL — Stock nodes receive texts from the tested device
- [ ] YES / NO / NOT TESTED — Packets traverse stock relays (2+ hops)
- [ ] YES / NO — No stock node reports problems with your voice packets (ask the owners)

### 4. Audio module in settings
- [ ] YES / NO — The toggle exists in the app: **Module Config → Audio → Codec2 (Enabled)**
- [ ] YES / NO — After enabling + reboot the log shows `Voice: self-test ok` and `ES7210 mic online`
- [ ] YES / NO / PARTIAL — The **audio rate/bitrate is selectable** (3200/2400/1600/1400/1300/1200/700C) and the change shows up in `!voice status` after reboot
- [ ] YES / NO — The mic records anything (in the `-log` build you see `chunk airtime ~...` when recording)

### 5. Voice messages (the important part — ideally two T-Decks on this firmware)
- [ ] YES / NO / PARTIAL — The `!voice` command (keyboard or phone) **starts a recording** and is NOT transmitted as text
- [ ] YES / NO — Recording shows a countdown; any key stops early and sends
- [ ] YES / NO / PARTIAL — The **audio message is sent** (in `-log`: `TX chunk k/n bytes ~ airtime` lines)
- [ ] YES / NO / PARTIAL — The **second T-Deck receives and plays the audio** from the speaker, automatically, in one continuous stream
- [ ] YES / NO — Playback can be interrupted / does not replay spuriously
- Voice quality (rate 1–5): at 1300 bps: ___ · at another mode you tried (____): ___ · Comment: ______
- [ ] YES / NO — **PTT** works (`!voice ptt` → holding the trackball center records, release sends)
- [ ] YES / NO — `!voice gain <0-38>` changes the audible recording level
- [ ] YES / NO — `!voice vol <0-16>` changes playback loudness
- [ ] YES / NO / NOT TESTED — A lost fragment becomes silence in its place, the rest still plays (range-edge test)
- [ ] YES / NO / NOT TESTED — The airtime budget works (at 3200 bps on Short Slow the recording auto-shortens to ~9 s; in `-log`: `capped`)
- [ ] YES / NO — Voice messages also work **on a private channel / as DMs** (encrypted)
- [ ] YES / NO — Sending a 10 s message on Short Slow takes ~10–15 wall-clock seconds (8–9 packets, 1 s apart)

### 6. Stability and reporting
- [ ] YES / NO — Device stays stable during voice sessions (no reboots/crashes; ~1 h of use)
- [ ] YES / NO / NOT TESTED — Battery drain with the audio module enabled is acceptable (subjectively: same / worse: ___)
- [ ] YES / NO / NOT TESTED — The duty-cycle limiter never blocked normal use (no "exceeding duty cycle" in the log under sane use)
- [ ] YES / NO — `!voice logsend` (as a DM) delivers the log to the phone chat
- [ ] YES / NO — Returning to **official firmware** went fine; settings/keys survived
- Please attach: a USB log excerpt (especially from the `-log` build: `Voice:`, `TX chunk`, `RX chunk`, `S:B:` lines) plus test conditions (distance, obstacles, preset)
