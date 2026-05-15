# Strujomerko - objedinjena hardverska i softverska specifikacija

## 1. Svrha dokumenta

Ovaj dokument opisuje sistem Strujomerko kao celinu: hardversku platformu, firmware, komunikacione tokove, skladištenje podataka, servisni portal, fabričko podešavanje, kalibraciju i poznata ograničenja trenutne implementacije.

Dokument je pisan prema trenutnom stanju repozitorijuma `strujomerko-embedded` i objedinjuje hardversku specifikaciju, firmware arhitekturu i operativne zahteve sistema.

## 2. Kratak opis sistema

Strujomerko je pametni jednofazni sistem za merenje potrošnje električne energije u domaćinstvu. Sistem meri napon, struju, frekvenciju, aktivnu snagu i aktivnu energiju, vodi lokalne brojače po tarifama, prikazuje očitavanja na LCD-u, čuva kritične podatke u NVS memoriji, vodi kratkoročni merni log u SPIFFS particiji i šalje telemetriju ka backend serveru preko dostupnog komunikacionog kanala.

Glavne ravni sistema su:

- Lokalna korisnička ravan: LCD ekran, taster, Wi-Fi servisni portal i Matter integracija.
- Provajderska ravan: GSM/GPRS modem i uplink telemetrija ka backend sistemu.
- Ravan otpornosti na otkaze: detekcija nestanka napajanja, smanjenje potrošnje, hitno upisivanje stanja u NVS.
- Bezbednosna ravan: tamper prekidač, latched stanje, perzistencija i prikaz/alarmiranje.
- Fabrička ravan: USB NDJSON protokol, fabrički alat, fabrički web UI, provisioning i kalibracija.

## 3. Ciljevi projekta

Funkcionalni ciljevi:

- Merenje RMS napona, RMS struje, frekvencije, aktivne snage i uvezene aktivne energije.
- Akumulacija ukupne aktivne energije u Wh.
- Lokalno određivanje aktivne tarife na osnovu vremena i konfiguracije dobijene od backend-a.
- Akumulacija energije po tarifnim opsezima.
- Prikaz osnovnih vrednosti i statusa na LCD 2x16.
- Lokalni servisni portal preko Wi-Fi AP režima.
- Matter prikaz električnih merenja kroz standardne Matter klastere.
- GSM/GPRS uplink ka backend serveru.
- Fabrička konfiguracija i kalibracija preko USB serijskog protokola.
- Perzistentno čuvanje konfiguracije, kalibracije, checkpoint-a, emergency record-a, tamper latch-a i uplink stanja.

Nefunkcionalni ciljevi:

- Stabilan rad bez reset petlji.
- Jasno degradirano stanje kada merni čip, modem ili mreža nisu dostupni.
- Kontrolisano ponašanje pri nestanku napajanja.
- Razdvajanje fleet konfiguracije, unit konfiguracije i kalibracije.
- Jedna firmware slika za više uređaja, uz naknadno provisioning podešavanje.

## 4. Hardverska specifikacija

### 4.1 Glavne komponente

| Komponenta | Uloga |
| --- | --- |
| ESP32-C6 | Glavni mikrokontroler, RTOS aplikacija, Wi-Fi, Matter, servisni portal, NVS/SPIFFS, USB fabrički interfejs. |
| ATM90E26 | Specijalizovani merni IC za napon, struju, frekvenciju, snagu i energiju. Komunikacija preko SPI. |
| Quectel M65 | GSM/GPRS modem za komunikaciju sa backend serverom. Komunikacija preko UART-a i AT komandi. |
| LCD 2x16 preko I2C | Lokalni prikaz merenja, statusa, alarma i servisnog režima. |
| RTC DS1307, I2C | Lokalni real-time clock za vreme kada mrežna sinhronizacija nije dostupna. |
| Tamper prekidač | Detekcija otvaranja kućišta ili neovlašćenog pristupa. |
| Power-fail komparator/signal | Detekcija gubitka glavnog napajanja. |
| Superkondenzator | Kratko rezervno napajanje za hitno čuvanje podataka. |

LCD ekran, RTC kolo i strujni transformator predstavljaju glavne eksterne komponente kroz koje uređaj prikazuje podatke, održava lokalno vreme i dobija strujni merni signal.

### 4.2 Sistemski hardverski blokovi

Hardverski sistem je podeljen na sledeće funkcionalne blokove:

| Blok | Namena | Glavne komponente/signali |
| --- | --- | --- |
| Merni front-end | Pretvaranje napona i struje u signale pogodne za ATM90E26. | Naponski delitelj, strujni transformator, diferencijalni ulazi mernog čipa. |
| Merni procesor | Digitalna obrada električnih veličina. | ATM90E26, SPI komunikacija, kalibracioni registri. |
| Glavna logika | Izvršavanje aplikacije i lokalna obrada podataka. | ESP32-C6, FreeRTOS, NVS, SPIFFS, Wi-Fi, Matter, USB. |
| Provajderska veza | Slanje telemetrije i statusa van lokala. | Quectel M65, UART, PWRKEY kontrola, SIM/APN profil. |
| Lokalni korisnički interfejs | Prikaz stanja i osnovna interakcija. | LCD 2x16 preko I2C, UI taster, statusne ikone. |
| Vreme i tarife | Lokalno vreme za tarifnu klasifikaciju. | RTC DS1307, NTP preko Wi-Fi/GSM, timezone konfiguracija. |
| Zaštita i događaji | Detekcija otvaranja i gubitka napajanja. | Tamper ulaz, power-fail ulaz, superkondenzator. |
| Napajanje | Stabilne naponske grane za logiku i modem. | AC ulaz, ispravljanje, filtracija, regulatori, rezervna energija. |

### 4.3 ATM90E26

ATM90E26 je merni čip namenjen elektronskim brojilima. U projektu se koristi za očitavanje:

- RMS napona,
- RMS struje,
- frekvencije mreže,
- aktivne snage,
- pozitivne aktivne energije (`APenergy`),
- negativne/reverzne aktivne energije (`ANenergy`).

U firmware-u se čip inicijalizuje kroz `drv_atm90e26`. Driver:

- konfiguriše SPI bus u mode 3,
- upisuje `PLconst`, `MMode` i kalibracione registre,
- proverava `CAL_START`, `ADJ_START`, `CS1`, `CS2` i `SYS_STATUS`,
- periodično čita registre `IRMS`, `URMS`, `FREQ`, `PMEAN`, `APENERGY` i `ANENERGY`,
- ima emulacioni režim za rad i proveru softverskih tokova bez fizičkog ATM90E26 čipa.

Napomena za metrološku tačnost: hardverski tekst navodi da ATM90E26 zadovoljava standarde IEC62052-11, IEC62053-21 i IEC62053-23 i ima deklarisanu tačnost aktivne energije do 0,1% u širokom opsegu. To je karakteristika čipa i referentnog dizajna, ali gotov Strujomerko mora posebno da se validira sa finalnim analognim front-end-om, naponskim deliteljem, strujnim transformatorom, PCB layout-om i kalibracijom.

### 4.4 ESP32-C6

ESP32-C6 je centralni kontroler sistema. U trenutnom projektu ciljni razvojni board je `ESP32-C6-DevKitC-1`, framework je ESP-IDF 5.4.1 kroz PlatformIO.

ESP32-C6 obavlja:

- inicijalizaciju servisa,
- FreeRTOS task orkestraciju,
- NVS i SPIFFS skladištenje,
- lokalno čuvanje do približno 30000 mernih tačaka u mernom logu,
- korišćenje dostupne flash memorije ciljane ploče za aplikaciju, konfiguraciju i merni log,
- Wi-Fi AP/STA funkcije,
- Matter stack,
- lokalni HTTP servisni portal,
- USB fabrički protokol,
- SPI komunikaciju sa ATM90E26,
- I2C komunikaciju sa LCD-om i RTC-om,
- UART komunikaciju sa Quectel M65 modemom,
- obradu tamper i power-fail ulaza,
- podršku za kalibraciju mernog čipa,
- kriptografske i sigurnosne funkcije dostupne na ESP32-C6 platformi.

### 4.5 Quectel M65

Quectel M65 se koristi za GSM/GPRS povezivanje. Firmware ima `drv_modem` i `svc_cellular` slojeve:

- `drv_modem` upravlja UART-om, PWRKEY impulsima, AT komandama, HTTP zahtevima i NTP sinhronizacijom preko modema.
- `svc_cellular` periodično proverava registraciju na mreži preko `AT+CREG?`, kvalitet signala preko `AT+CSQ` i sinhronizuje vreme kada Wi-Fi nije dostupan.

M65 se u ovom projektu tretira kao provajderski kanal. Backend komunikacija ide kroz konfigurisane HTTP endpoint-e.

### 4.6 Napajanje

Sistem se napaja sa približno 7 V AC RMS eksternog napajanja. Ulaz se ispravlja jednom diodom, filtrira ulaznim kondenzatorima i zatim preko regulatora obezbeđuje:

- 5 V DC za ESP32-C6 sistem,
- 3,3 V za logiku i ATM90E26 preko lokalne regulacije,
- približno 4 V DC za Quectel M65 modem.

Ulazno ispravljanje je polutalasno: ne koristi se Grecov spoj, zbog potencijalnih problema u konkretnoj topologiji i zbog niskog AC napona na ulazu. Posle diode se koristi ulazna filtracija i LDO regulacija. ESP32-C6 dobija 5 V, a daljom lokalnom regulacijom se obezbeđuje 3,3 V za logiku i merni deo. Quectel M65 ima zasebnu granu približno 4 V, pošto modem ima značajne kratkotrajne strujne zahteve.

Zbog polutalasnog ispravljanja, niskog AC napona i impulsne potrošnje M65 modema, ulazna kapacitivnost mora da bude velika. Hardverske beleške navode procenu oko 10 mF, uz oko 20% rezerve zbog tolerancija komponenata, greške proračuna, nepredviđenih gubitaka i ograničenih informacija o konkretnom napajanju. Najveći potrošač je M65, koji po datasheet napomenama može kratkotrajno vući strujne impulse do reda 2 A; u hardverskim beleškama je uzet scenario impulsa na svakih približno 4,615 ms u trajanju oko 577 us.

Projektni zahtev: napajanje mora da se proveri osciloskopom u najgorem slučaju - slab ulaz, aktivan modem, maksimalna GSM emisija, uključen LCD, aktivan Wi-Fi i aktivno merenje. Ne sme doći do brownout reset-a ESP32-C6 niti do pada ispod minimalnog napona M65 modema.

### 4.7 Rezerva za gubitak napajanja

Strujomerko ima rezervni energetski put preko superkondenzatora od 1 F. Pri nestanku glavnog napajanja:

1. Power-fail komparator postavlja GPIO signal.
2. ISR budi `power_guard_task`.
3. Firmware postavlja `POWER_FAIL` i `DEGRADED_MODE`.
4. Prikazuje poruku na LCD-u.
5. Gasi/suspenduje modem radi smanjenja potrošnje.
6. Zamrzava poslednji merni snapshot.
7. Formira `emergency_record_t`.
8. Upisuje emergency record u NVS.
9. Isključuje LCD backlight i ostaje u degradiranom režimu.

Konfigurisani timeout za emergency flush je `APP_CFG_EMERGENCY_FLUSH_TIMEOUT_MS = 300 ms`, uz završno čekanje od 50 ms.

Hardverska realizacija predviđa da ESP32-C6 kratko nastavi rad nakon nestanka glavnog napajanja, napajan energijom iz superkondenzatora. Spoljašnji signal za detekciju nestanka napajanja dovodi se na power-fail ulaz mikrokontrolera. Strujomerko time dobija vreme da prepozna događaj, sačuva poslednje merne podatke, upiše zapis o nestanku napajanja i kasnije taj događaj prijavi backend sistemu kada komunikacija ponovo bude dostupna.

Detekcija nestanka napajanja oslanja se na posebnu komparatorsku logiku. Komparator razdvaja normalno napajanje od power-loss stanja i daje digitalni signal firmware-u, čime se izbegava oslanjanje na posledice pada napona u samoj logici.

### 4.8 Čačkodugmić, tamper ulaz i zaštita kućišta

Kako bi se obezbedilo da korisnici ne utiču na podatke ili ne remete adekvatan rad Strujomerka, uveden je Čačkodugmić. Čačkodugmić služi da Strujomerko primeti ako neko neovlašćeno pokušava da otvori kutiju strujomera i time potencijalno načini zlodelo protiv krupnog kapitala.

U tehničkom smislu, Čačkodugmić je tamper prekidač povezan na `BOARD_PIN_TAMPER_SWITCH`. Firmware detektuje obe ivice signala, debounce-uje ulaz i vodi dva stanja:

- `open_now`: trenutno stanje fizičkog ulaza,
- `latched`: trajno zapamćeno stanje da je tamper ikada aktiviran.

Latched stanje se čuva u NVS i preživljava reboot. Brisanje tamper latch-a zahteva eksplicitnu autorizovanu servisnu komandu.

### 4.9 LCD ekran

Ekran je jedna od eksternih komponenti Strujomerka namenjenih direktnoj interakciji sa korisnikom i servisom. LCD prikazuje potrebne podatke korisniku i proizvođaču/serviseru kako bi se potrošnja i stanje uređaja mogli jasno registrovati.

LCD je izveden u 2x16 karaktera formatu, ukupno do 32 karaktera raspoređena u dva reda. Sa ekranom se komunicira preko I2C magistrale. Projekat podržava:

- DFRobot/AiP31068 varijantu,
- PCF8574 varijantu preko odgovarajuće build konfiguracije.

LCD prikazuje:

- merenje napona,
- merenje struje,
- ukupnu aktivnu energiju,
- energiju po tarifama,
- trenutnu aktivnu snagu,
- frekvenciju,
- aktivnu tarifu,
- vreme,
- datum,
- pristup mreži,
- kvalitet merenja i fault kod,
- Matter status,
- grešku na Strujomerku,
- ikone za tok energije, GSM, Wi-Fi, Matter i alarm,
- dodatne servisne i dijagnostičke poruke.

Kratak pritisak na UI taster menja ekran. Duži pritisak prikazuje pomoć ili pokreće Matter akciju na Matter ekranu.

### 4.10 RTC kolo

RTC kolo je eksterna komponenta koja omogućava da Strujomerko zadrži korisno lokalno vreme i kada mrežna sinhronizacija nije dostupna. Njegova uloga je posebno bitna za tarife, jer se promena tarife određuje prema lokalnom vremenu.

Firmware očekuje DS1307 na I2C adresi `0x68`. RTC se koristi kao fallback izvor vremena pri boot-u. Kada se vreme dobije preko NTP-a, firmware pokušava da upiše novo vreme nazad u DS1307.

Ako RTC nije dostupan ili vreme nije validno, `svc_time_now_ms()` vraća 0 dok se ne dobije validna mrežna sinhronizacija. Takva očitavanja se tretiraju kao očitavanja bez realnog Unix vremena.

### 4.11 Merenje struje

Za merenje utrošene struje koristi se eksterni strujni transformator. Za potrebe Strujomerka izabran je strujni transformator opsega do 20 A. Izabrani transformator u sebi ima integrisanu logiku koja daje naponski izlaz, pa se na ulazu mernog čipa obrađuje napon proporcionalan struji.

Izlaz strujnog transformatora prolazi kroz ulazni filter i otpornu mrežu, zatim se dovodi kao diferencijalni signal na ATM90E26.

Otpornici u ulaznom delu imaju tri uloge:

- ograničavanje signala,
- eventualno skaliranje kao naponski delitelj,
- mogućnost ukrštanja diferencijalnog signala radi korekcije greške polariteta pri žičenju.

U hardverskoj realizaciji otpornici R23, R22 i R21 služe za limitiranje i po potrebi mogu učestvovati kao naponski razdelnik strujnog transformatora. R23 i R22 se koriste i za potrebe "ukrštanja signala", odnosno za korekciju greške koja može nastati pogrešnim žičenjem strujnog transformatora. Krajnji izlaz filtera za merenje struje je diferencijalni signal koji ATM90E26 obrađuje.

Firmware dodatno detektuje reverznu energiju: ako `ANenergy` raste, snapshot dobija `METER_QUALITY_REVERSE_ENERGY_SEEN`. Ako nema pozitivne energije, a snaga je negativna, firmware može da uračuna reverzni registar kao potrošnju i time olakša otkrivanje pogrešne orijentacije strujnog transformatora.

### 4.12 Merenje napona

Da bi se izračunala aktivna utrošena snaga, potrebno je meriti i napon koji dobija domaćinstvo. U projektovanoj mernoj postavci ulazni napon za merenje je ograničen na približno 7 V AC RMS, pa se pre ulaska u ATM90E26 dodatno skalira.

Ulazni napon se skalira naponskim deliteljem. Hardverske beleške navode odnos približno 1:22 za projektovani merni ulaz od 7 V AC RMS. Time se na ulaze ATM90E26 dovodi bezbedan diferencijalni napon u opsegu koji merni čip može da obradi.

Projektni zahtev: odnos delitelja, referentna vrednost i ATM90E26 `UGAIN` moraju biti usklađeni sa stvarnim ulaznim opsegom i referentnim mernim instrumentom.

### 4.13 Hardverski zahtevi za validaciju

Pre završnog prihvatanja uređaja potrebno je proveriti:

- stabilnost svih naponskih grana pri maksimalnoj impulsnoj potrošnji modema,
- napon na superkondenzatoru i vreme dostupno za emergency save,
- nivoe i polaritet power-fail i tamper ulaza,
- I2C komunikaciju sa LCD-om i RTC-om,
- SPI komunikaciju sa ATM90E26,
- UART komunikaciju sa M65 modemom,
- orijentaciju strujnog transformatora,
- odnos naponskog delitelja i bezbednost ulaza ATM90E26,
- grešku merenja napona, struje, snage i energije u više radnih tačaka,
- ponašanje pri nestanku i povratku napajanja.

## 5. Pinout trenutnog firmware-a

Pinovi su definisani u `components/app_config/include/board_pins.h`.

| Funkcija | Pin |
| --- | --- |
| ATM90E26 SPI MOSI | GPIO10 |
| ATM90E26 SPI MISO | GPIO11 |
| ATM90E26 SPI SCLK | GPIO2 |
| ATM90E26 SPI CS | GPIO3 |
| LCD I2C SDA | GPIO6 |
| LCD I2C SCL | GPIO7 |
| Modem UART TX | GPIO1 |
| Modem UART RX | GPIO0 |
| Modem PWRKEY | GPIO23 |
| Tamper prekidač | GPIO22 |
| Power-fail detect | GPIO21 |
| UI taster | GPIO5 |

Napomena: mapiranje pinova mora biti usklađeno sa važećim PCB netlist-om i šemom uređaja.

## 6. Firmware arhitektura

### 6.1 Sistem za izgradnju firmware-a

Projekat koristi:

- PlatformIO,
- platformu `espressif32 @ 6.11.0`,
- ESP-IDF framework,
- cilj `esp32-c6-devkitc-1`,
- monitor brzinu 115200,
- upload brzinu 921600.

Postoje dva PlatformIO okruženja:

- `esp32-c6-devkitc-1`: osnovni hardverski build.
- pomoćno emulaciono okruženje: emulira merni čip i koristi PCF8574 LCD adresu `0x27`.

### 6.2 Ulazna tačka

Ulaz je `main/app_main.c`. Funkcija `app_main()` poziva:

1. `app_core_init()`,
2. `app_core_start()`.

`app_core` zatim inicijalizuje board, event loop, event group, konfiguraciju, skladištenje i sve servise.

### 6.3 Glavni servisi

| Komponenta | Uloga |
| --- | --- |
| `app_core` | Orkestracija servisa, supervisor task, periodični checkpoint, log memorije. |
| `app_config` | Default vrednosti, validacija i primena runtime/fleet/unit/device konfiguracije. |
| `bsp_board` | GPIO inicijalizacija, ISR registracija, taster, tamper, power-fail, modem PWRKEY. |
| `drv_atm90e26` | Niskonivojski SPI driver za ATM90E26 i emulacija mernog čipa. |
| `drv_lcd_i2c` | Niskonivojski LCD driver. |
| `drv_modem` | UART/AT/HTTP/NTP kontrola Quectel M65 modema. |
| `svc_metering` | Periodično očitavanje, skaliranje, energija, snapshot fan-out. |
| `svc_measurement_log` | SPIFFS ring buffer za visoko-frekventni merni log. |
| `svc_storage` | NVS konfiguracija, checkpoint, recovery, tamper latch. |
| `svc_power` | Power-fail ISR, emergency save i load shedding. |
| `svc_tamper` | Tamper debounce, latch, NVS perzistencija i događaji. |
| `svc_display` | LCD UI, ikone, ekran merenja, statusi i taster. |
| `svc_wifi` | Wi-Fi STA, servisni AP, captive portal DHCP opcije. |
| `svc_portal` | Lokalni HTTP servisni portal i JSON API. |
| `svc_service_mode` | Sekvenca tastera za ulazak/izlazak iz servisnog režima. |
| `svc_matter` | Matter electrical sensor endpoint i commissioning. |
| `svc_cellular` | GSM registracija, RSSI, CREG/CSQ, cellular NTP. |
| `svc_uplink` | Backend bootstrap, telemetry, heartbeat, config polling, event outbox. |
| `svc_factory_usb` | USB fabrički NDJSON protokol sa prefiksom `@SMF:`. |
| `svc_calibration` | Kalibracioni profil i ATM90E26 kalibracioni registri. |
| `svc_tariff` | Lokalna tarifa, per-tariff brojači i OBIS mapiranje. |
| `svc_time` | Vreme, timezone, DS1307, SNTP/cellular NTP. |
| `svc_diag` | Periodična dijagnostika. |

### 6.4 Komunikacija između servisa

Firmware koristi tri glavna obrasca:

1. Visokofrekventni snapshot fan-out:
   - `svc_metering` proizvodi `meter_snapshot_t` na svakih 200 ms.
   - Snapshot se čuva u double-buffer strukturi.
   - Consumer-i se bude direktnom FreeRTOS task notifikacijom.

2. Niskofrekventni state događaji:
   - Koristi se custom `esp_event` loop `APP_EVENT_BASE`.
   - Događaji uključuju metering ready/fault, Wi-Fi connect/disconnect, Matter commissioning, tamper, power fail, service mode i uplink status.

3. Vlasnički queue po servisu:
   - Storage, modem, display, uplink, Wi-Fi, Matter, supervisor i tamper imaju sopstvene komande i queue.

### 6.5 Event group bitovi

Sistemski bitovi uključuju:

- `SYS_BOOT_DONE`
- `STORAGE_READY`
- `METER_READY`
- `LCD_READY`
- `MODEM_READY`
- `CELL_READY`
- `CELL_CONNECTING`
- `UPLINK_READY`
- `WIFI_READY`
- `WIFI_CONNECTED`
- `MATTER_READY`
- `POWER_FAIL`
- `EMERGENCY_SAVE_DONE`
- `EMERGENCY_SAVE_FAILED`
- `TAMPER_LATCHED`
- `DEGRADED_MODE`
- `SERVICE_MODE_ACTIVE`
- `SERVICE_BUTTON_CAPTURE`

## 7. Merni tok

### 7.1 Periodično očitavanje

`svc_metering` radi na intervalu `APP_CFG_METER_POLL_INTERVAL_MS = 200 ms`.

Za svaki ciklus:

1. Formira osnovni snapshot sa monotonic timestamp-om i Unix timestamp-om ako je vreme validno.
2. Ako ATM90E26 nije spreman, pokušava inicijalizaciju.
3. Čita raw vrednosti iz driver-a.
4. Skalira raw vrednosti preko aktivnog kalibracionog profila.
5. Primeni no-load deadband za malu struju/snagu.
6. Preračuna inkrement energije iz ATM90E26 energy registara.
7. Pozove `svc_tariff_update()` da dodeli energiju aktivnoj tarifi.
8. Objavi snapshot potrošačima.
9. Upisuje kompaktan record u measurement log.

### 7.2 Model mernog snapshot-a

`meter_snapshot_t` sadrži:

- schema version,
- sekvencu,
- monotonic timestamp,
- Unix timestamp,
- RMS napon,
- RMS struju,
- frekvenciju,
- aktivnu snagu,
- ukupnu aktivnu energiju,
- aktivnu tarifu,
- brojače po tarifama,
- quality flags,
- fault code,
- tamper latch.

### 7.3 Konverzija energije

Firmware koristi `atm90e26_cf1_pulses_per_kwh` za konverziju ATM90E26 energy registara u Wh. Default vrednost je 1000 impulsa/kWh.

Formula u firmware-u za `energy_decicf` je:

```text
Wh = energy_decicf * 100 / pulses_per_kwh
```

`PLconst` default je `0x0045D174`, ali dokumentacija u kodu upozorava da ga treba preračunati za finalni analogni front-end.

## 8. Tarife

Firmware ne računa novac, račune, poreze ili popuste. On računa samo raspodelu energije po tarifnim opsezima.

`svc_tariff`:

- prima tariff config iz backend konfiguracije,
- validira JSON,
- čuva aktivnu konfiguraciju i brojače u NVS,
- određuje aktivnu tarifu prema lokalnom vremenu,
- podržava opsege koji prelaze ponoć,
- čuva `tariff_config_revision`,
- kopira tarifne podatke u `meter_snapshot_t`.

OBIS mapiranje koje koristi UI i fabrički alat:

- `1.8.0`: ukupna uvezena aktivna energija,
- `1.8.1`, `1.8.2`, ...: tarife redom iz konfiguracije.

Ako vreme nije sinhronizovano ili nema validne tarifne konfiguracije, snapshot dobija odgovarajuće quality flags.

## 9. Skladištenje

### 9.1 Particije

Trenutna particiona tabela:

| Particija | Tip | Offset | Veličina | Namena |
| --- | --- | --- | --- | --- |
| `nvs` | data/nvs | `0x9000` | `0x6000` | ESP-IDF standardni NVS. |
| `phy_init` | data/phy | `0xF000` | `0x1000` | PHY inicijalizacija. |
| `factory` | app/factory | `0x10000` | `0x400000` | Jedan firmware slot. |
| `storage` | data/spiffs | `0x410000` | `0x3E0000` | Measurement log ring buffer. |
| `app_nvs` | data/nvs | `0x7F0000` | `0x10000` | Aplikacioni NVS za Strujomerko podatke. |

OTA slotovi trenutno nisu aktivni. Trenutni raspored memorije daje prednost većoj SPIFFS `storage` particiji za merni log.

### 9.2 NVS podaci

`svc_storage` čuva:

- runtime konfiguraciju,
- legacy device konfiguraciju,
- `fleet_cfg`,
- `unit_cfg`,
- kalibraciju,
- checkpoint snapshot,
- emergency recovery record,
- tamper latch,
- tariff config/counters,
- uplink identitet/token/config/outbox u `uplink` namespace-u.

Zapisi imaju verzije, veličine i CRC polja.

### 9.3 Checkpoint

Supervisor task periodično, default na 60 s, traži poslednji snapshot i šalje ga storage servisu kao checkpoint. Na boot-u se checkpoint učitava i koristi za obnovu kumulativne energije i tarifnih brojača.

### 9.4 Emergency record

Power-fail tok formira `emergency_record_t` koji sadrži:

- verziju i veličinu,
- sekvencu,
- monotonic i Unix timestamp,
- napon, struju, frekvenciju, aktivnu snagu,
- aktivnu energiju u mWh,
- quality flags,
- tamper latch,
- CRC.

### 9.5 Merni log

`svc_measurement_log` koristi SPIFFS ring buffer. Default kapacitet je 30000 record-a. Na intervalu od 200 ms to je približno 100 minuta istorije.

Record sadrži:

- sekvencu,
- monotonic i Unix timestamp,
- skalirane fixed-point vrednosti,
- raw ATM90E26 vrednosti,
- forward/reverse energy delte,
- quality flags,
- fault code,
- tamper latch,
- CRC.

Log je namenjen hardverskoj proveri, analizi kvarova i kalibraciji. Nije zamišljen kao billing-grade arhiva.

## 10. Lokalni UI

### 10.1 LCD ekrani

LCD UI ima sledeće ekrane:

- ukupna energija,
- tarifa 1 do tarifa 4, ako postoje,
- aktivna snaga,
- napon,
- struja,
- frekvencija,
- quality/fault status,
- vreme,
- datum,
- Matter status.

Gornja linija prikazuje statusne ikone:

- aktivna tarifa,
- tok energije,
- GSM,
- Wi-Fi,
- Matter,
- alarm.

### 10.2 UI taster

Kratak pritisak:

- pali backlight ako je ugašen,
- menja aktivni ekran.

Dug pritisak na većini ekrana:

- prikazuje pomoć/OBIS opis aktivnog ekrana.

Na Matter ekranu:

- ako uređaj nije uparen, dug pritisak pokreće commissioning window,
- ako je uparen, dug pritisak ulazi u reset meni,
- dodatna potvrda reset menija pokreće Matter factory reset.

### 10.3 Servisni režim

Ulazak u servisni režim:

1. Držati taster 4 s.
2. Pustiti kada LCD prikaže zahtev.
3. Ponovo držati taster 4 s u okviru 10 s.

Izlazak iz servisnog režima:

- držati taster 3 s dok je servisni režim aktivan,
- ili isključiti servisni režim iz servisnog portala.

## 11. Servisni portal

Servisni portal radi kada je servisni režim aktivan.

Default vrednosti:

- SSID: `Strujomerko-Service-XXXX`,
- IP adresa: `http://192.168.4.1`,
- AP lozinka: `strujomerko1234`,
- login lozinka: `admin1234`.

Autentifikacija koristi cookie `sp_session`, sa timeout-om oko 30 minuta neaktivnosti.

Glavne grupe API-ja:

- auth,
- device config,
- latest meter snapshot,
- SSE meter stream,
- system overview,
- service mode,
- Wi-Fi,
- cellular,
- uplink,
- Matter,
- storage,
- measurement log,
- tamper,
- power,
- time,
- display,
- device actions,
- calibration.

Detaljan API opis je u `docs/LOCAL_SERVICE_API.md`.

## 12. GSM i backend uplink

### 12.1 Prvo povezivanje uređaja

Uplink stanje uključuje:

- `device_id`,
- `meter_id`,
- device token,
- claim/provisioned flag,
- backend config revision,
- backend config JSON,
- upload sekvencu,
- event outbox.

Ako uređaj nije provisioned, `svc_uplink` pokušava bootstrap na endpoint:

```text
/api/v1/device/bootstrap
```

### 12.2 Telemetrija

Telemetrija ide na:

```text
POST /api/v1/device/telemetry
```

Payload schema version je `1.1` i uključuje:

- device/meter ID,
- message ID,
- timestamp,
- sekvencu,
- config revision,
- napon,
- struju,
- frekvenciju,
- aktivnu snagu,
- ukupnu aktivnu energiju,
- aktivnu tarifu,
- energiju po tarifama,
- delta energiju po tarifama,
- tamper,
- quality flags.

### 12.3 Periodično javljanje uređaja

Heartbeat ide na:

```text
POST /api/v1/device/heartbeat
```

Sadrži status, uptime, RSSI, modem state, network registration, firmware verzije, tamper stanje i error flags.

### 12.4 Periodično preuzimanje konfiguracije

Firmware periodično proverava backend konfiguraciju:

```text
GET /api/v1/device/config?known_revision=<rev>&include_commands=true
```

Ako dobije novu konfiguraciju, validira je, primeni lokalno i šalje ack:

```text
POST /api/v1/device/config/ack
```

## 13. Matter integracija

Matter servis kreira electrical sensor endpoint. Implementacija koristi:

- `ElectricalPowerMeasurement` cluster za napon, struju, frekvenciju i aktivnu snagu,
- `ElectricalEnergyMeasurement` cluster za kumulativnu uvezenu energiju.

Matter commissioning:

- Window timeout je 300 s.
- Manual pairing code i QR payload su dostupni kroz portal i logove.
- Ako uređaj nije u fabric-u, firmware pokušava da otvori commissioning window.
- Nakon uspešnog pairing-a postavlja se commissioned status.
- Matter factory reset je dostupan preko LCD Matter ekrana i portala.

Tokovi Wi-Fi i uplink servisa se privremeno utišavaju tokom Matter commissioning-a da ne ometaju mrežno povezivanje.

## 14. Fabrička konfiguracija i kalibracija

### 14.1 Jedan firmware image

Projektni cilj je jedan produkcioni firmware image za sve uređaje. Razlike između uređaja se upisuju u NVS preko USB fabričkog protokola.

### 14.2 NVS podela

- `fleet_cfg`: zajedničko za batch/fleet, na primer backend host, APN, timezone, servisni AP policy, metering mode.
- `unit_cfg`: jedinstveno po uređaju, na primer serial, claim code, factory batch/station metadata, per-unit PL constant ako se koristi.
- `calib`: kalibracioni registri i legacy scale polja.

### 14.3 USB fabrički protokol

Firmware koristi line-oriented JSON preko USB serial/JTAG porta. Svaka protokolska linija počinje prefiksom:

```text
@SMF:
```

Primer:

```text
@SMF:{"id":1,"cmd":"hello"}
@SMF:{"id":1,"ok":true,"chip_mac":"..."}
```

Implementirane komande uključuju:

- `hello`,
- `job_status`,
- `read_records`,
- `verify`,
- `write_fleet_cfg`,
- `write_unit_cfg`,
- `read_meter`,
- `log_info`,
- `log_read`,
- `log_clear`,
- `clear_meter_energy` / `clear_energy`,
- `reset_backend_provisioning`,
- `force_backend_reprovision`,
- `cal_write`,
- `lock_factory`,
- `reboot`.

### 14.4 Fabrički alati

CLI alat:

```powershell
python tools/factory_tool.py flash --port COM3
python tools/factory_tool.py provision --port COM3 --fleet factory/fleet.<batch>.json --batch BATCH-001
python tools/factory_tool.py calibrate --port COM3 --points factory/cal_points.<batch>.json
python tools/factory_tool.py verify --port COM3 --out factory/reports/verify.json
```

Fabrički web interfejs:

```powershell
python -m uvicorn tools.factory_web:app --host 127.0.0.1 --port 8088
```

Browser se otvara na:

```text
http://127.0.0.1:8088
```

Web UI poseduje USB COM port, čita live merenja, vodi provisioning i kalibracioni workflow i čuva fabričke izveštaje.

### 14.5 Kalibracija

Trenutni preporučeni tok kalibracije piše ATM90E26 registre, ne softverske scale faktore.

Podržani kalibracioni registri:

- `UGAIN`,
- `IGAINL`,
- `LGAIN`,
- `LPHI`,
- `IOffsetL`,
- `POffsetL`,
- pragovi i dodatni offset registri.

Tipičan tok:

1. Zagrejati uređaj i referentno opterećenje 5-10 minuta.
2. Proveriti da emulacija mernog čipa nije uključena.
3. Uhvatiti no-load tačku.
4. Uhvatiti nominalnu resistivnu tačku.
5. Po potrebi uhvatiti tačku poznatog power factor-a.
6. Preview solve.
7. Upisati registre u NVS.
8. Reboot.
9. Proveriti očitavanja na drugoj tački.

## 15. Bezbednost

Trenutno implementirano:

- service portal login lozinka,
- HTTP-only session cookie,
- tamper latch i perzistencija,
- factory lock flag u `unit_cfg`,
- skrivanje punog uplink token-a u statusnim izveštajima.

Potrebno za produkciju:

- HTTPS/TLS za backend,
- individualne servisne lozinke ili sertifikati,
- jača autorizacija za fabričke komande,
- fizički unlock tok za menjanje zaključanog unit-a,
- NVS encryption/flash encryption po potrebi,
- sigurnosni audit portala i fabričkog protokola,
- kontrola pristupa za tamper clear i backend reset.

## 16. Kvalitet i status verifikacije

Postojeća dokumentacija navodi da su korišćeni:

```powershell
pio run
pio check
```

Plan verifikacije dodatno pominje:

```powershell
pio test --without-uploading -vvv
```

U ovom zahvatu dokumentacije nije pokretan clean build, u skladu sa lokalnim `AGENTS.md` pravilom.

## 17. Poznata ograničenja

- ATM90E26 očitavanja zahtevaju PCB validaciju i kalibraciju.
- Default `PLconst` i analogni odnosi moraju biti provereni za ciljnu hardversku reviziju.
- OTA trenutno nije omogućena u particionoj tabeli.
- Measurement log nije billing-grade skladište.
- Fabričko zaključavanje postoji, ali fizička politika otključavanja nije potpuno nametnuta.
- Service portal nema opisanu produkcionu CORS/TLS zaštitu.
- GSM/GPRS HTTP je MVP transport; HTTPS je produkciona potreba.
- Automatizovani testovi nisu kompletan sistemski test.
- Hardverske slike iz dostavljenih beleški imaju lokalne putanje sa druge mašine i nisu dostupne u repozitorijumu; u ovom dokumentu su zamenjene tekstualnim opisom.

## 18. Preporučeni sledeći koraci

1. Uskladiti `board_pins.h` sa finalnim PCB netlist-om.
2. Dodati šematske slike u `docs/assets/` i referencirati ih relativnim putanjama.
3. Završiti metrološku verifikaciju sa referentnim instrumentom.
4. Definisati produkcionu particionu tabelu sa OTA slotovima.
5. Očvrstiti fabrički lock i servisne autorizacije.
6. Uvesti backend TLS i credential lifecycle.
7. Dodati sistemske testove za NVS migracije, tariff boundary slučajeve, power-fail i uplink retry.
8. Izraditi produkcionu hardware safety dokumentaciju za napajanje i merni front-end.
