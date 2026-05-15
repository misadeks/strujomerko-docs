# Pregled proizvoda

Strujomerko je pametni jednofazni sistem za merenje potrošnje električne energije u domaćinstvu. Uređaj meri napon, struju, frekvenciju, aktivnu snagu i aktivnu energiju, vodi lokalne brojače po tarifama, prikazuje očitavanja na LCD ekranu i šalje telemetriju ka backend sistemu.

## Glavne funkcije

- Merenje RMS napona, RMS struje, frekvencije, aktivne snage i aktivne energije.
- Lokalno akumuliranje ukupne energije i energije po tarifama.
- Prikaz osnovnih očitavanja, statusa i alarma na LCD 2x16 ekranu.
- Lokalni servisni režim preko Wi-Fi AP mreže i servisnog portala.
- Matter integracija za lokalni ekosistem pametne kuće.
- GSM/GPRS uplink ka backend sistemu.
- Fabrička konfiguracija, provisioning i kalibracija preko USB serijskog protokola.
- Čuvanje kritičnih podataka u NVS memoriji i kratkoročnog mernog loga u SPIFFS particiji.
- Detekcija otvaranja kućišta i nestanka napajanja.

## Sistemske ravni

| Ravan | Opis |
| --- | --- |
| Korisnička | LCD ekran, taster, servisni portal i Matter prikaz. |
| Provajderska | GSM/GPRS modem i uplink telemetrija. |
| Otpornost na otkaze | Power-fail detekcija, smanjenje potrošnje i hitno čuvanje stanja. |
| Bezbednosna | Tamper prekidač, latched stanje i alarmiranje. |
| Fabrička | USB protokol, fabrički alat, provisioning i kalibracija. |

## Glavni dokumenti

- [`Specifikacija`](specifikacija.md) daje najširi opis sistema.
- [`Podsistemi sistema`](podsistemi-sistema.md) razdvaja hardver, firmware, servisni režim, komunikaciju, backend, proizvodnju i obračun.
- [`Korisničko uputstvo`](korisnicko-uputstvo.md) namenjeno je krajnjim korisnicima.
- [`Pregled hardvera`](../hardver/pregled-hardvera.md) objašnjava komponente i električne blokove.
- [`Arhitektura firmware-a`](../softver/arhitektura-firmware-a.md) opisuje servise, događaje i tokove podataka.
- [`Pregled backend-a`](../backend/pregled-backenda.md) opisuje server, telemetriju, tarife, obračun i portale.
