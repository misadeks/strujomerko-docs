# Lokalni servisni API

Lokalni servisni API je HTTP API dostupan u servisnom režimu uređaja. Namenjen je servisnom portalu, fabričkim alatima, dijagnostici i lokalnoj integraciji tokom razvoja.

API putanje, JSON ključevi i status kodovi u ovom dokumentu ostaju zapisani kao tehnički identifikatori i ne prevode se.

## Dostupnost

API je dostupan kada je uređaj u servisnom režimu i kada je aktivan lokalni Wi-Fi AP ili captive portal. Tipična bazna putanja je:

```text
/api/v1
```

## Autentifikacija

Status autentifikacije:

```http
GET /api/v1/auth/status
```

Servisne akcije koje menjaju stanje uređaja treba da budu zaštićene servisnom autentifikacijom. Posebno treba zaštititi komande koje menjaju konfiguraciju, kalibraciju, tamper latch, logove ili fabričko stanje.

## Konfiguracija uređaja

Konfiguracija obuhvata identitet uređaja, mrežne parametre, servisni AP, backend podešavanja, vremensku zonu i operativne intervale.

Tipični endpoint-i:

```http
GET /api/v1/device/config
POST /api/v1/device/config
```

## Živi merni podaci

Najnoviji snapshot:

```http
GET /api/v1/meter/latest
```

Stream mernih podataka:

```http
GET /api/v1/meter/stream
```

Snapshot treba da sadrži napon, struju, frekvenciju, aktivnu snagu, energiju, aktivnu tarifu, kvalitet merenja, fault code i tamper stanje.

## Status endpoint-i

Statusne putanje služe za servisni pregled sistema:

| Endpoint | Namena |
| --- | --- |
| `GET /api/v1/system/status` | Opšti status sistema. |
| `GET /api/v1/service/status` | Servisni režim i lokalni portal. |
| `GET /api/v1/network/status` | Wi-Fi i lokalna mreža. |
| `GET /api/v1/uplink/status` | Backend uplink i modem. |
| `GET /api/v1/matter/status` | Matter commissioning i stanje. |
| `GET /api/v1/storage/status` | NVS, SPIFFS, checkpoint i emergency stanje. |
| `GET /api/v1/measurement-log/info` | Informacije o mernom logu. |
| `GET /api/v1/tamper/status` | Tamper ulaz i latch. |
| `GET /api/v1/power/status` | Napajanje i power-fail stanje. |
| `GET /api/v1/time/status` | RTC, Unix vreme i sinhronizacija. |
| `GET /api/v1/display/status` | LCD i lokalni UI. |

## Merni log

Merni log se koristi za kratkoročnu dijagnostiku:

```http
GET /api/v1/measurement-log/info
GET /api/v1/measurement-log/read
POST /api/v1/measurement-log/clear
```

Čitanje treba da bude paginirano da bi portal ostao odzivan i da se ne preoptereti uređaj.

## Kalibracija

Kalibracioni endpoint-i služe za očitavanje trenutnih koeficijenata, primenu referentnih tačaka i upis novih vrednosti.

Primeri:

```http
GET /api/v1/calibration
POST /api/v1/calibration/apply-point
POST /api/v1/calibration/write
```

Kalibracija mora jasno razlikovati privremeno izračunavanje od perzistentnog upisa u NVS.

## Captive portal

Captive portal endpoint-i služe za otkrivanje portala i osnovno preusmeravanje korisnika ili servisera ka lokalnom UI-ju.

## Model grešaka

Greške treba vraćati kao strukturisan JSON sa stabilnim kodom greške, tekstualnom porukom i opcionalnim detaljima. HTTP status mora odražavati tip greške: neautorizovan pristup, loš zahtev, konflikt stanja, interna greška ili nedostupan servis.

## Integracione napomene

- Aplikacija ne treba da pretpostavi da su svi servisi uvek spremni.
- Merni snapshot treba tretirati kao trenutno stanje, ne kao obračunski zapis.
- Za istoriju koristiti measurement log ili backend telemetriju.
- Promene konfiguracije moraju vratiti novu reviziju ili potvrdu primene.
