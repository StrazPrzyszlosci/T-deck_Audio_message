# T-Deck Audio Message — wiadomości głosowe Codec2 dla LilyGO T-Deck (fork Meshtastic)

> ⚠️ **BUILD EKSPERYMENTALNY — NIE TESTOWANY NA FIZYCZNYM URZĄDZENIU.**
> Kod się kompiluje (pełny build `t-deck` przechodzi), ale tor audio nie był
> jeszcze potwierdzony na sprzęcie. Wgranie na własną odpowiedzialność.
> **Dobra wiadomość: T-Deck (ESP32-S3) praktycznie nie da się "uwalić" —
> pełne odzyskanie opisane jest niżej i wymaga tylko kabla USB.**

---

## Po polsku

### Co to jest
Firmware Meshtastic z nagrywanymi **wiadomościami głosowymi** (nie streamingiem)
dla zwykłego T-Decka: nagrywasz do 10 s, audio jest kompresowane kodekiem
**Codec2** i wysyłane zwykłymi pakietami Meshtastic; drugi T-Deck z tym
firmware automatycznie odtwarza wiadomość z głośnika. Bez zmian w protobufach
— standardowe węzły Meshtastic przekierowują te pakiety jak każde inne.

**Baza:** oryginalny [meshtastic/firmware](https://github.com/meshtastic/firmware)
**v2.8.2** — sieciowo kompatybilny z każdą wersją 2.8.x.

### Urządzenia
- ✅ **LilyGO T-Deck** (zwykły, SX1262 868/915 MHz) — mikrofon ES7210 i głośnik są na pokładzie także w wersji standardowej
- ✅ **LilyGO T-Deck Plus** (to samo + GPS i dotyk — identyczny tor audio)
- ❌ **T-Deck Pro / Pro v1.1** — inny układ, brak toru audio w variancie
- ❌ **T-Deck Max** — brak wsparcia w Meshtastic
- ❌ **T-Deck z SX1280 (2,4 GHz)** — ten build zakłada SX1262 sub-GHz

### 💾 Krok 0 — backup konfiguracji (protobufy urządzenia). ZRÓB TO NAJPIERW.
Zanim cokolwiek wgrasz, wyeksportuj i zapisz konfigurację swojego węzła
(protobufy: `config`, `module_config`, kanały + klucze):
- **aplikacja Meshtastic:** ustawienia urządzenia → eksport / udostępnij
  konfigurację (Share/Export configuration) → zapisz plik,
- **CLI:** `meshtastic --export-config config.yaml`.

Ten plik odbuduje Ci węzeł (klucze, kanały, region, nazwę) w minutę — nawet po
pełnym restarcie fabrycznym. Flashowanie nie kasuje ustawień, ale w alfa-buildzie
bezpieczeństwo ponad wszystko.

### Który plik wgrać
**To build ALFA — zawsze wgraj plik `full`** (zawiera bootloader + partycje + aplikację, offset `0x0`).

| Plik | Offset |
|---|---|
| `t-deck-audio-full-2.8.2.ca10d3d.bin` | `0x0` |

"full" to ta sama zawartość, którą PlatformIO nazywa `factory.bin` — **to NIE
znaczy "ustawienia fabryczne"**: plik nie rusza NVS (klucze, region, kanały,
nazwa węzła zostają nietknięte).

### Instalacja — web flasher
1. Podłącz T-Deck kablem USB-C **do transmisji danych**.
2. Wejdź na https://flasher.meshtastic.org
3. Wybierz **LILYGO T-Deck**, opcję wgrywania własnego firmware
   ("Advanced / specify firmware URL" lub wybór pliku z dysku).
4. Wskaż `t-deck-audio-full-2.8.2.ca10d3d.bin` i kliknij **Flash**.
5. Alternatywa terminalowa:
   `esptool --chip esp32s3 --port /dev/ttyACM0 write_flash 0x0 t-deck-audio-full-2.8.2.ca10d3d.bin`

### Odzyskiwanie (gdyby coś poszło nie tak)
ESP32-S3 ma bootloader w ROM — nie da się go nadpisać zwykłym flashowaniem:
1. Przytrzymaj **środkowy przycisk trackballa (BOOT/GPIO0)** i podłącz USB.
2. Uruchom flasher ponownie (lub `esptool … write_flash 0x0 …factory…`) —
   urządzenie zawsze wejdzie w tryb wgrywania.
3. Chcesz wrócić na oficjalny Meshtastic? Wgraj stock przez ten sam flasher —
   partycje są identyczne, więc **klucze i konfiguracja zostają**.

### Konfiguracja (z aplikacji Meshtastic)
> ⚠️ **Moduł głosowy jest domyślnie WYŁĄCZONY.** Po wgraniu firmware'u trzeba
> go **włączyć w ustawieniach aplikacji** (patrz krok 3 poniżej) — dopóki
> tego nie zrobisz, `!voice` poleci w sieć jako zwykły tekst i nic się nie
> nagra.

1. Ustaw **region** (np. EU_868) — bez regionu radio milczy.
2. Radio → preset: **Short Slow — jedyny rozsądny preset dla głosu.**
   10 s nagrania = ~8 pakietów ≈ **2,8–3 s airtime**. **LongFast się NIE
   nadaje**: jeden pakiet głosowy to tam ~2,1 s czasu antenowego (~17 s na
   10 s wiadomości) i fork ścięłby nagranie do ~2 s. **LongFast = tylko tekst,
   nie głos.**
3. **Module Config → Audio → Codec2: Enabled.**
4. **Bitrate**: wybierz z natywnych trybów Codec2 — **3200 / 2400 / 1600 /
   1400 / 1300 (domyślny) / 1200 / 700C**. Zmiana działa "na żywo" (bez restartu).
   Odbiornik rozpoznaje tryb z nagłówka wiadomości — można mixować tryby.
   **Budżet duty cycle:** fork sam pilnuje, żeby cała wiadomość nie zajęła
   więcej niż **~6 s airtime** — przy wysokich bitrate lub wolnych presetach
   nagranie automatycznie się skróci (przy 3200 bps na Short Slow: ~9 s,
   na LongFast: ~2 s).
5. Restart węzła. W logach (USB) szukaj `Voice: self-test ok` i `ES7210 mic online`.

### Użycie — komendy (wysyłane jako zwykły tekst z T-Decka lub z telefonu; komenda nigdy nie leci w eter)
| Komenda | Działanie |
|---|---|
| `!voice` lub `!voice 5` | nagranie 10 s (lub 1–10 s), dowolny klawisz = wcześniejszy stop i wysyłka |
| `!voice ptt` | tryb PTT: **przytrzymaj środkowy trackball** i mów, puść = wyślij; nagranie tnie samo przy **10 s** (do adresata komendy — DM lub kanał; sesja 60 s, dowolny klawisz = wyjście) |
| `!voice gain 30` | wzmocnienie mikrofonu 0–37,5 dB (skoki ES7210; domyślnie max) |
| `!voice vol 12` | głośność odtwarzania 0–16 (domyślnie 16; cyfrowa, w MAX98357A nie da się ustawić sprzętowo) |
| `!voice log on` / `off` | włącz/wyłącz log modułu (domyślnie **on**) |
| `!voice logsend` | **wyślij log jako wiadomości tekstowe** — wyślij tę komendę jako **DM do siebie/drugiego węzła**, a log przyjdzie na telefon przez Bluetooth |
| `!voice status` | aktualne ustawienia (tryb, gain, głośność, log, stan mikrofonu) |

Wiadomość głosowa idzie **do adresata komendy**: wyślesz `!voice` jako DM do
węzła X — głos poleci jako DM do X; na kanale — broadcast na ten kanał.

Odtwarzanie: **jeden ciągły strumień** (dekodowanie ramka po ramce do I²S z
buforem DMA 0,5 s — bez przerw). Brakujący fragment = cisza w jego miejscu,
reszta wiadomości odtwarza się normalnie.

### Logi dla testerów — jak to działa
- **USB (od razu, bez żadnej konfiguracji):** podłącz do komputera i otwórz
  konsolę 115200 (`pio device monitor`, PuTTY, `meshtastic --seriallog info`) —
  wszystkie zdarzenia modułu głosowego widać na żywo w logu firmware.
- **Telefon przez Bluetooth:** log ostatnich zdarzeń trzymany jest w urządzeniu
  (bufor kołowy w RAM, ostatnie ~3 KB). `!voice logsend` wysłane jako DM
  odsyła ten log **do czatu w aplikacji** — tester nie potrzebuje komputera.
  `!voice log off` wyłącza zbieranie logu.
- To celowo **nie** jest zapis na flash (LittleFS): log w RAM nie zużywa
  pamięci flash i nie wymaga żadnych zmian w protobufach.

### Na bazie jakiego kodu powstał ten port
- **[meshtastic/firmware](https://github.com/meshtastic/firmware)** v2.8.2 —
  baza całego buildu; z niej pochodzą: struktura modułów (wzorzec: `AudioModule`
  = PTT dla SX1280), wariant `t-deck` z już zdefiniowanymi pinami ES7210/I²S,
  router, kolejka TX, limiter duty cycle.
- **[meshtastic/codec2](https://github.com/meshtastic/codec2)** — vocoder
  Codec2 w "vocoder only" pakiecie (LGPL-2.1) — vendored do
  `src/modules/esp32/voice/codec2/`.
- **[varna9000/reticulum-tdeck](https://github.com/varna9000/reticulum-tdeck)**
  — **najważniejsze źródło sprzętowe**: zweryfikowana na T-Decku sekwencja
  rejestrów ES7210 (I²C 0x40, wzmocnienie PGA, DLL, format I²S) oraz
  **poprawki precyzji podwójnej dla enkodera codec2 na ESP32** (double w
  autocorrelate/Levinson-Durbin/Chebyshev w `lpc.c`/`lsp.c` i pipeline
  `speech_to_uq_lsps` w `quantise.c`) — sportowane do vendored kopii.
  Ich README opisuje też warm-up ADC (stąd ciągły drain mikrofonu od startu).
- **[deulis/ESP32_Codec2](https://github.com/deulis/ESP32_Codec2)** (fork meshtastic) —
  biblioteka, z której korzysta oryginalny `AudioModule`; punkt odniesienia
  dla I²S i formatu ramek.
- **[Xinyuan-LilyGO/T-Deck](https://github.com/Xinyuan-LilyGO/T-Deck)** —
  `UnitTest.ino`: konfiguracja I²S mikrofonu (piny 47/21/14/48, MCLK 256×, 16 kHz),
  potwierdzenie, że zwykły T-Deck ma mikrofon i głośnik.
- **[Xinyuan-LilyGO/documentation](https://github.com/Xinyuan-LilyGO/documentation)** —
  dokumentacja T-Deck/Plus (piny audio, różnice między wersjami).
- **[dudmuck/lora_codec2](https://github.com/dudmuck/lora_codec2)** —
  koncepcja Codec2→fragmentacja→SX1262→reasemblacja na sub-GHz.
- **[NadeeshaNJ/Ranger](https://github.com/NadeeshaNJ/Ranger)** — dowód
  wykonalności Codec2 1300 + LoRa na ESP (nagrywanie → kodowanie → wysyłka
  pakietowa zamiast streamingu).

### Budowanie ze źródeł
Wymagany **PlatformIO 6.1.x** (np. 6.1.19). PlatformIO 6.2.0 pobiera
tool-scons 4.11, który psuje build pioarduino 55.03.311
(`ModuleNotFoundError: SCons.Tool.FortranCommon`).
Ścieżka projektu **nie może zawierać spacji**.

```bash
git clone https://github.com/meshtastic/firmware firmware && cd firmware
git checkout 9e4d301   # v2.8.2 - baza tego forka
git apply ../voice-module.patch
python3 -m venv .venv && . .venv/bin/activate
pip install "platformio==6.1.19"
pio run -e t-deck
```

### Licencje
Fork dziedziczy licencję [meshtastic/firmware]. Vendored Codec2: **LGPL-2.1**
(patrz `src/modules/esp32/voice/codec2/LICENSE`). Ten fork dodaje kod tylko w
`src/modules/esp32/voice/` + 2 małe hooki (`MeshService`, `Modules`) i wpis w
`variants/esp32s3/t-deck/platformio.ini`.

---

## English

### What this is
A Meshtastic firmware fork adding **recorded voice messages** (not streaming)
for the plain LilyGO T-Deck: record up to 10 s, compress with **Codec2**, send
as regular Meshtastic packets; another T-Deck on this fork reassembles and
plays the message through its speaker. No protobuf changes — stock Meshtastic
nodes route these packets like any other.

**Base:** upstream [meshtastic/firmware](https://github.com/meshtastic/firmware)
**v2.8.2** — network-compatible with any 2.8.x node.

### Supported devices
- ✅ **LilyGO T-Deck** (standard, SX1262 sub-GHz) — the ES7210 mic and speaker
  are onboard even in the base version
- ✅ **LilyGO T-Deck Plus** (same audio path, adds GPS/touch)
- ❌ **T-Deck Pro / Pro v1.1** (different board, no audio path defined)
- ❌ **T-Deck Max** (not supported by Meshtastic)
- ❌ **T-Deck with SX1280 (2.4 GHz)** — this build targets the SX1262 variant

### 💾 Step 0 — back up your configuration (device protobufs). DO THIS FIRST.
Before flashing anything, export and save your node configuration
(the protobufs: `config`, `module_config`, channels + keys):
- **Meshtastic app:** device settings → export / share configuration,
- **CLI:** `meshtastic --export-config config.yaml`.

That file rebuilds your node (keys, channels, region, name) in a minute —
even after a full factory reset. Flashing does not erase settings, but with
an alpha build, better safe than sorry.

### Which file to flash
**This is an ALPHA build — always flash the `full` file** (bootloader + partitions + app, offset `0x0`).

| File | Offset |
|---|---|
| `t-deck-audio-full-2.8.2.ca10d3d.bin` | `0x0` |

"full" is what PlatformIO calls `factory.bin` — it does **not** mean "factory
settings": it never touches NVS (keys, region, channels, node name survive).

### Install — web flasher
1. Connect the T-Deck with a **data** USB-C cable.
2. Open https://flasher.meshtastic.org
3. Pick **LILYGO T-Deck**, choose the custom firmware option and select
   `t-deck-audio-full-2.8.2.ca10d3d.bin`, then **Flash**.
4. Terminal alternative:
   `esptool --chip esp32s3 --port /dev/ttyACM0 write_flash 0x0 t-deck-audio-full-2.8.2.ca10d3d.bin`

### Recovery
The ESP32-S3 boot ROM cannot be overwritten by normal flashing:
1. Hold the **trackball center button (BOOT/GPIO0)** while plugging in USB.
2. Re-run the flasher (or esptool) — the device always enters download mode.
3. To return to stock Meshtastic, flash the official build the same way;
   partitions are identical, so **your keys and settings are kept**.

### Configuration (Meshtastic app)
> ⚠️ **The voice module is disabled by default.** After flashing you must
> **enable it in the app's settings** (step 3 below) — until then, `!voice`
> goes out as a plain text message and nothing gets recorded.

1. Set your **region** first.
2. Preset: **Short Slow — the only sensible preset for voice.** 10 s of
   recording = ~8 packets ≈ **2.8–3 s of airtime**. **LongFast is NOT
   suitable**: one voice packet costs ~2.1 s of airtime there (~17 s per 10 s
   message) and the fork would trim the recording to ~2 s. **LongFast = text
   only, no voice.**
3. **Module Config → Audio → Codec2: Enabled.**
4. **Bitrate**: choose from the native Codec2 modes — **3200 / 2400 / 1600 /
   1400 / 1300 (default) / 1200 / 700C**; changes apply live (no reboot).
   The receiver reads the mode from the message header, so mixed modes work.
   **Duty-cycle budget:** the fork keeps the whole message under **~6 s of
   airtime** — on high bitrates or slow presets the recording shortens
   automatically (3200 bps on Short Slow: ~9 s, on LongFast: ~2 s).
5. Reboot and check the serial log for `Voice: self-test ok` and
   `ES7210 mic online`.

### Commands (sent as plain text from the T-Deck keyboard or the phone; never transmitted)
| Command | Action |
|---|---|
| `!voice` or `!voice 5` | record 10 s (or 1–10 s); any key stops early and sends |
| `!voice ptt` | PTT mode: **hold the trackball center** to talk, release to send; recording self-caps at **10 s** (to the addressee of the command; 60 s session, any key exits) |
| `!voice gain 30` | mic gain 0–37.5 dB (ES7210 steps; default max) |
| `!voice vol 12` | playback volume 0–16 (default 16; the MAX98357A has no hardware volume) |
| `!voice log on` / `off` | enable/disable the voice log (default **on**) |
| `!voice logsend` | **send the log as text messages** — send this as a **DM** to receive it in the phone chat over Bluetooth |
| `!voice status` | current settings (mode, gain, volume, logging, mic state) |

Voice goes to the addressee of the command: DM the node → voice DM; channel →
broadcast on that channel.

Playback is **one continuous stream** (frame-by-frame decode into I²S with a
0.5 s DMA buffer). A lost chunk becomes silence in its place; the rest of the
message still plays.

### Tester logging
- **USB (works out of the box):** open a serial console at 115200 and all
  voice-module events are streamed live.
- **Phone over Bluetooth:** the device keeps a small RAM ring buffer of recent
  voice events; `!voice logsend` (as a DM) delivers it to the app chat — no
  computer needed. `!voice log off` stops collection.
- Deliberately **no LittleFS/flash logging** — the RAM ring costs no flash wear
  and required zero protobuf changes.

### Code provenance
See the Polish section above for the full list of upstream repositories this
port was built from (meshtastic/firmware, meshtastic/codec2, varna9000/
reticulum-tdeck, deulis/ESP32_Codec2, Xinyuan-LilyGO/T-Deck and documentation,
dudmuck/lora_codec2, NadeeshaNJ/Ranger).

### Building from source
Requires **PlatformIO 6.1.x** (6.2.0 pulls tool-scons 4.11, which breaks the
pioarduino hybrid build: `ModuleNotFoundError: SCons.Tool.FortranCommon`).
The project path must not contain spaces.

### Licenses
Inherits the [meshtastic/firmware] license. Vendored Codec2 is **LGPL-2.1**
(`src/modules/esp32/voice/codec2/LICENSE`).
