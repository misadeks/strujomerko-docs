# Podsistemi sistema

Strujomerko treba posmatrati kao sistem sastavljen od više podsistema. Svaki podsistem ima jasnu odgovornost, ulaze, izlaze i mesto u procesu izrade.

## 1. Merni hardverski podsistem

Namena:

- merenje napona,
- merenje struje,
- računanje aktivne snage,
- akumulacija aktivne energije,
- obezbeđivanje sirovih mernih podataka firmware-u.

Glavne komponente:

- ATM90E26,
- naponski delitelj,
- strujni transformator,
- SPI veza ka ESP32-C6,
- kalibracioni elementi.

Kako treba da se odradi:

1. Definisati šemu mernog front-end-a.
2. Potvrditi ulazne opsege za napon i struju.
3. Implementirati realni ATM90E26 driver.
4. Uvesti kalibracioni postupak.
5. Validirati očitavanja referentnim instrumentom.

## 2. Firmware podsistem

Namena:

- čitanje mernog hardvera,
- formiranje mernog snapshot-a,
- lokalna obrada tarifa,
- prikaz na LCD-u,
- lokalni servisni režim,
- skladištenje kritičnog stanja,
- priprema telemetrije za backend.

Glavni servisi:

- `svc_metering`,
- `svc_storage`,
- `svc_display`,
- `svc_uplink`,
- `svc_wifi`,
- `svc_matter`,
- `svc_tamper`,
- `power_guard_task`.

Kako treba da se odradi:

1. Stabilizovati komunikaciju između servisa.
2. Završiti realno očitavanje mernog IC-a.
3. Završiti perzistentno čuvanje konfiguracije.
4. Povezati power-fail i tamper tokove.
5. Izložiti dijagnostiku kroz servisni API.
6. Dodati testove za kritične tokove.

## 3. Lokalni korisnički podsistem

Namena:

- prikaz trenutne potrošnje,
- prikaz ukupne energije,
- prikaz aktivne tarife,
- signalizacija grešaka i upozorenja,
- osnovna interakcija preko tastera.

Elementi:

- LCD 2x16,
- UI taster,
- statusne ikone,
- servisni režim.

Kako treba da se odradi:

1. Definisati osnovne ekrane.
2. Definisati ponašanje tastera.
3. Definisati poruke grešaka.
4. Povezati UI sa mernim snapshot-om i statusnim događajima.
5. Proveriti čitljivost i ponašanje pri greškama.

## 4. Servisni podsistem uređaja

Namena:

- lokalna dijagnostika,
- kalibracija,
- pregled statusa,
- očitavanje mernog loga,
- servisne akcije.

Elementi:

- Wi-Fi AP servisni režim,
- lokalni HTTP API,
- captive portal,
- servisna autentifikacija.

Kako treba da se odradi:

1. Definisati servisne endpoint-e.
2. Zaštititi akcije koje menjaju stanje.
3. Omogućiti pregled merenja i statusa.
4. Omogućiti kalibraciju i proveru NVS-a.
5. Obezbediti jasan model grešaka.

## 5. Komunikacioni podsistem

Namena:

- GSM/GPRS komunikacija sa backend-om,
- slanje telemetrije,
- heartbeat,
- preuzimanje konfiguracije,
- rad u offline režimu.

Elementi:

- Quectel M65,
- UART komunikacija,
- AT komande,
- uplink queue,
- retry i buffering logika.

Kako treba da se odradi:

1. Stabilizovati modem init.
2. Definisati HTTP payload limite.
3. Implementirati retry politiku.
4. Implementirati offline buffer.
5. Potvrditi slanje telemetrije u realnim mrežnim uslovima.

## 6. Backend podsistem

Namena:

- identitet uređaja,
- provisioning,
- prijem telemetrije,
- čuvanje merenja,
- konfiguracija uređaja,
- tarife,
- obračun,
- admin i korisnički portali.

Detaljna backend dokumentacija je u [`../backend/pregled-backenda.md`](../backend/pregled-backenda.md).

Kako treba da se odradi:

1. Napraviti osnovu backend projekta.
2. Implementirati registar uređaja i provisioning.
3. Implementirati telemetrijski ingestion.
4. Implementirati konfiguracione revizije.
5. Implementirati tarife i verifikaciju.
6. Implementirati obračun.
7. Dodati admin i korisnički portal.

## 7. Proizvodni podsistem

Namena:

- flash firmware-a,
- upis fleet i unit konfiguracije,
- kalibracija,
- verifikacija fabričkog stanja,
- zaključavanje uređaja za isporuku.

Elementi:

- USB Serial/JTAG,
- proizvodni protokol,
- PC CLI alat,
- fabrički web UI,
- fabrički izveštaj.

Kako treba da se odradi:

1. Definisati USB protokol.
2. Implementirati PC fabrički alat.
3. Implementirati kalibracioni tok.
4. Generisati fabrički izveštaj.
5. Definisati politiku proizvodnog zaključavanja.

## 8. Obračunski podsistem

Namena:

- provera potrošnje,
- razdvajanje potrošnje po tarifama,
- generisanje računa,
- audit obračuna.

Ovaj podsistem se implementira na backend-u, ali zavisi od ispravne telemetrije sa uređaja.

Kako treba da se odradi:

1. Definisati tarifne planove.
2. Definisati ugovore i obračunske periode.
3. Verifikovati uređajem prijavljene tarife.
4. Izračunati potrošnju po tarifnim opsezima.
5. Generisati račun i stavke računa.
