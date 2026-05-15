# Strujomerko - tehnički priručnik

## 1. Namena

Ovaj priručnik je namenjen developerima, servisnim inženjerima i fabričkom operateru koji rade sa firmware-om Strujomerko uređaja. Pokriva podešavanje okruženja, build, upload, servisni režim, fabrički provisioning, kalibraciju, dijagnostiku i osnovne procedure za rešavanje problema.

## 2. Radno okruženje

Repozitorijum je ESP-IDF projekat koji se gradi kroz PlatformIO.

Potrebno:

- Windows računar ili razvojno okruženje sa PlatformIO Core.
- USB kabl za ESP32-C6.
- ESP32-C6-DevKitC-1 ili hardver kompatibilan sa trenutnim pin mapping-om.
- Za realna merenja: ATM90E26 merni front-end, izolovan AC izvor ili bezbedna merna postavka, referentni instrument i poznata opterećenja.
- Za GSM: SIM kartica, antena i pokrivenost mrežom.
- Za fabrički web UI: Python dependencies iz `tools/requirements-factory-web.txt`.

Instalacija PlatformIO Core:

```powershell
python -m pip install -U platformio
pio --version
```

Ako se koristi PlatformIO virtual environment iz korisničkog profila:

```powershell
& "$env:USERPROFILE\.platformio\penv\Scripts\pio.exe" --version
```

## 3. Struktura projekta

```text
main/                  ESP-IDF app entry point
components/            Projektni servisi, driver-i i lokalni vendor override-i
components/third_party Lokalno override-ovan esp_matter
docs/                  Dokumentacija
factory/               Demo fleet/calibration fajlovi i fabrički izveštaji
tools/                 PC alati za NVS, factory CLI i factory web
assets/                Projektni asset-i
include/               Shared include placeholder
lib/                   PlatformIO library placeholder
test/                  Test placeholder-i
platformio.ini         PlatformIO okruženja
partitions.csv         Flash particiona tabela
sdkconfig.defaults     Default ESP-IDF konfiguracija
```

Ne commit-ovati generisane direktorijume kao što su `.pio/`, `managed_components/`, `node_modules/` i build output.

## 4. Okruženja za izgradnju firmware-a

Osnovni build:

```powershell
pio run -e esp32-c6-devkitc-1
```

Demo build sa emulacijom ATM90E26:

```powershell
pio run -e demo-strujomerko
```

`demo-strujomerko` dodaje:

```text
APP_CFG_EMULATE_MEASUREMENT_CHIP=1
BOARD_LCD_DRIVER_TYPE=BOARD_LCD_DRIVER_PCF8574
BOARD_LCD_I2C_ADDRESS=0x27
```

Ne pokretati clean build osim kada je to stvarno potrebno za rešavanje konkretnog build/config problema.

## 5. Flash i serijski monitor

Flash:

```powershell
pio run -e esp32-c6-devkitc-1 -t upload
```

Ako port nije automatski detektovan:

```powershell
pio run -e esp32-c6-devkitc-1 -t upload --upload-port COM5
```

Serial monitor:

```powershell
pio device monitor --baud 115200
```

Build, upload i monitor:

```powershell
pio run -e esp32-c6-devkitc-1 -t upload -t monitor
```

## 6. Statička analiza i testovi

PlatformIO check:

```powershell
pio check
```

Kompajliranje testova bez upload-a:

```powershell
pio test --without-uploading -vvv
```

Ako Windows konzola ima problem sa encoding-om:

```powershell
$env:PYTHONIOENCODING='utf-8'
$env:PYTHONUTF8='1'
pio pkg list
```

## 7. Prvi boot smoke test

1. Flash-ovati firmware.
2. Otvoriti serial monitor na 115200.
3. Proveriti da nema reset loop-a ili brownout-a.
4. LCD treba da prikaže boot splash, zatim energy ekran ili jasan metering fault ekran.
5. Kratak pritisak na taster treba da menja ekrane.
6. Ako je ATM90E26 emulacija uključena, vrednosti treba da se menjaju deterministički.
7. Ako je emulacija isključena, a hardver nije povezan, očekivan je metering fault.

Minimalni uspešan smoke test: uređaj radi bar 2 minuta, ne resetuje se i UI taster reaguje.

## 8. Servisni režim

Ulazak:

1. Držati taster 4 s.
2. Pustiti kada LCD prikaže `Release Button`.
3. Ponovo držati 4 s u okviru potvrđenog prozora.

Uređaj pokreće AP:

```text
Strujomerko-Service-XXXX
```

Default AP lozinka:

```text
strujomerko1234
```

Otvoriti:

```text
http://192.168.4.1
```

Default login lozinka:

```text
admin1234
```

Izlazak:

- držati taster 3 s u servisnom režimu,
- ili pozvati portal akciju za gašenje service mode-a.

## 9. Lokalni service API brze komande

Login:

```powershell
curl -i -X POST http://192.168.4.1/api/v1/auth/login `
  -H "Content-Type: application/json" `
  -d "{\"password\":\"admin1234\"}"
```

Najnovije merenje:

```http
GET /api/v1/meter/latest
```

Sistem:

```http
GET /api/v1/system/overview
```

Storage:

```http
GET /api/v1/storage/status
```

Measurement log:

```http
GET /api/v1/measurement-log/info
GET /api/v1/measurement-log/read?offset=0&max_records=128
POST /api/v1/measurement-log/clear
```

Tamper:

```http
GET /api/v1/tamper/status
POST /api/v1/tamper/clear
```

Power:

```http
GET /api/v1/power/status
```

Matter:

```http
GET /api/v1/matter/status
POST /api/v1/matter/action
```

Detaljan opis je u `docs/LOCAL_SERVICE_API.md`.

## 10. Proizvodni CLI alat

USB fabrički alat koristi isti USB port kao firmware logovi, ali filtrira protokol po prefiksu `@SMF:`.

Flash:

```powershell
python tools/factory_tool.py flash --port COM3
```

Provisioning:

```powershell
python tools/factory_tool.py provision --port COM3 --fleet factory/fleet.<batch>.json --batch BATCH-001
```

Kalibracija:

```powershell
python tools/factory_tool.py calibrate --port COM3 --points factory/cal_points.<batch>.json
```

Verifikacija:

```powershell
python tools/factory_tool.py verify --port COM3 --out factory/reports/verify.json
```

Proizvodno zaključavanje:

```powershell
python tools/factory_tool.py lock --port COM3
```

## 11. Proizvodni web interfejs

Instalacija zavisnosti:

```powershell
python -m pip install -r tools\requirements-factory-web.txt
```

Pokretanje:

```powershell
python -m uvicorn tools.factory_web:app --host 127.0.0.1 --port 8088
```

Otvoriti:

```text
http://127.0.0.1:8088
```

Proizvodni web interfejs omogućava:

- izbor COM porta,
- živa USB merenja,
- fleet i unit provisioning,
- pregled stanja povezanog uređaja,
- reset backend provisioning-a i forsirano ponovno povezivanje,
- konzolu runtime logova,
- preuzimanje mernog loga i CSV izvoz,
- vođeni kalibracioni tok,
- proizvodnu proveru i izveštaj.

Frontend se nalazi u `tools/factory_web_ui`. Posle izmena korisničkog interfejsa:

```powershell
cd tools\factory_web_ui
$env:Path = "C:\Program Files\nodejs;$env:Path"
& "C:\Program Files\nodejs\npm.cmd" install
& "C:\Program Files\nodejs\npm.cmd" run build
cd ..\..
```

## 12. Kalibracioni postupak

### 12.1 Preduslovi

- Emulacija mernog čipa mora biti isključena.
- Uređaj mora biti na bezbednoj test postavci.
- Potrebni su referentni voltmetar/power analyzer i stabilna opterećenja.
- Pre kalibracije pustiti uređaj i opterećenje da se termički stabilizuju 5-10 minuta.

### 12.2 Preporučeni tok

1. Povezati uređaj preko USB-a.
2. Pokrenuti factory web UI.
3. Otvoriti Calibration Suite.
4. Učitati trenutne registre.
5. Uhvatiti no-load tačku sa naponom prisutnim i bez opterećenja.
6. Uhvatiti nominalnu resistivnu tačku.
7. Uneti referentni napon, struju i aktivnu snagu.
8. Po potrebi uhvatiti tačku poznatog power factor-a.
9. Pokrenuti Preview Solve.
10. Proveriti predložene promene registara.
11. Upisati u uređaj.
12. Reboot.
13. Ponovo izmeriti nominalnu i drugu tačku.

### 12.3 Ciljne validacione tolerancije

Za internu validaciju:

- napon unutar 1%,
- struja unutar 2%,
- aktivna snaga unutar 3%,
- frekvencija unutar 0,2 Hz ako se validira referentnim izvorom.

Za produkciju se moraju definisati formalni metrološki kriterijumi i postupak verifikacije.

## 13. ATM90E26 hardverska provera

Ako merni čip nije dostupan:

1. Proveriti `APP_CFG_EMULATE_MEASUREMENT_CHIP`.
2. Ako se radi provera bez hardvera, koristiti emulaciono okruženje.
3. Ako se radi realna hardverska provera, emulacija mora biti 0.

Ako se javlja metering fault:

1. Pogledati LCD fault ekran.
2. Otvoriti serial log.
3. Proveriti stage iz `drv_atm90e26` dijagnostike: SPI bus, SPI device, CAL_START, PLCONSTH/L, MMODE, CS1, ADJ_START, SYS_STATUS ili sample register.
4. Proveriti GPIO pinove za SPI.
5. Proveriti da je ATM90E26 napajan i da je CS linija korektna.
6. Proveriti SPI mode 3 i frekvenciju 1 MHz.
7. Proveriti `SYS_STATUS` za checksum/calibration greške.

## 14. Power-fail test

Kontrolisano testirati samo na bezbednoj postavci.

1. Pustiti uređaj da radi bar jedan checkpoint interval.
2. U portalu proveriti `checkpoint_valid=true`.
3. Aktivirati power-fail signal.
4. Proveriti da LCD prikazuje `POWER LOSS`.
5. Očekivani tok: save started, emergency flush, record saved ili save failed/timeout.
6. Reboot-ovati uređaj.
7. Proveriti `recovery_valid=true` u storage statusu.
8. Po potrebi preuzeti measurement log oko događaja.

## 15. Tamper test

1. Aktivirati tamper prekidač.
2. LCD/portal treba da prikažu alarm.
3. Reboot-ovati uređaj.
4. Proveriti da je latched stanje opstalo.
5. Ulogovati se u servisni portal.
6. Pozvati clear tamper.
7. Reboot-ovati i potvrditi da je latch obrisan.

## 16. GSM/uplink test

1. Umetnuti SIM karticu i antenu.
2. Proveriti APN u fleet/device konfiguraciji.
3. Pratiti logove za `MODEM_READY`.
4. Proveriti `AT+CREG?` status kroz cellular servis.
5. Proveriti `AT+CSQ` RSSI.
6. U portalu otvoriti uplink status.
7. Ako uređaj nije provisioned, proveriti serial i claim code.
8. Pokrenuti force uplink send.
9. Na backend-u proveriti bootstrap, heartbeat i telemetry.

Ako DNS preko standardnog resolver-a zakaže, `svc_uplink` ima fallback UDP DNS resolver prema dostupnim DNS serverima i javnim fallback serverima.

## 17. Matter test

1. Ući u servisni portal ili otići na LCD Matter ekran.
2. Pokrenuti commissioning.
3. Preuzeti manual code ili QR payload iz portala.
4. Upariti uređaj sa Matter controller-om.
5. Proveriti da se vide electrical measurement vrednosti.
6. Fabrički reset koristiti samo kada je prihvatljivo ponovno uparivanje.

## 18. NVS dump i dekodiranje

Čitanje NVS-a:

```powershell
python tools\read_nvs.py --port COM3
```

Prikaz raw blob-ova:

```powershell
python tools\read_nvs.py --port COM3 --dump blobs
```

Čuvanje dekodiranih projektnih zapisa:

```powershell
python tools\read_nvs.py --port COM3 --project-json-out nvs_project.json
```

Uplink token se ne ispisuje u celosti, već samo prisustvo, dužina i kratak preview.

## 19. Najčešći problemi

### Uređaj se resetuje ili javlja brownout

- Proveriti 5 V i 4 V grane pod opterećenjem.
- Proveriti GSM burst potrošnju.
- Privremeno isključiti modem ili koristiti emulacioni build.
- Proveriti ulaznu kapacitivnost i regulator.

### Nema LCD prikaza

- Proveriti I2C SDA/SCL pinove.
- Proveriti LCD adresu `0x3E`, `0x6B` za DFR ili `0x27` za PCF8574 demo.
- Proveriti build flag `BOARD_LCD_DRIVER_TYPE`.
- Pratiti log `drv_lcd_i2c`.

### Servisni AP se ne vidi

- Proveriti da je service mode stvarno aktivan.
- Proveriti sekvencu tastera.
- Proveriti da Matter commissioning ne koristi Wi-Fi u tom trenutku.
- Reboot-ovati i ponoviti.

### Portal login ne radi

- Proveriti lozinku iz NVS konfiguracije.
- Default je `admin1234`.
- Ako je unit provisioned sa drugom lozinkom, koristiti fabrički alat za read_records.

### Merenje pokazuje negativnu snagu

- Proveriti orijentaciju strujnog transformatora.
- Proveriti diferencijalni ulaz struje.
- Proveriti `METER_QUALITY_REVERSE_ENERGY_SEEN`.
- Proveriti `MMode` i kalibracione registre.

### Uplink ne šalje telemetriju

- Proveriti `CELL_READY` ili `WIFI_CONNECTED`.
- Proveriti da nije `POWER_FAIL`.
- Proveriti da Matter commissioning nije pauzirao uplink.
- Proveriti bootstrap/provisioned stanje.
- Proveriti backend host, APN, DNS i HTTP status.

## 20. Operativna pravila

- Ne raditi clean build bez konkretnog razloga.
- Ne menjati pin mapping bez usklađivanja sa šemom.
- Ne koristiti emulacioni build za tvrdnje o tačnosti.
- Ne brisati measurement log ako je potreban za analizu kvara.
- Ne raditi Matter factory reset tokom demo-a bez plana za ponovno uparivanje.
- Ne testirati mrežni napon bez izolovane i nadzirane postavke.
