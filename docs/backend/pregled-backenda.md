# Pregled backend sistema

Backend za Strujomerko je centralni sistem koji prima telemetriju sa uređaja, upravlja uređajima, čuva merenja, šalje konfiguraciju, vodi tarife, generiše obračune i obezbeđuje administratorski i korisnički pristup podacima.

Backend ne meri potrošnju direktno. Uređaj meri i šalje podatke, a backend ih proverava, skladišti, prikazuje i koristi za obračun.

## Ciljevi backend-a

- Evidencija uređaja, korisnika, lokacija i ugovora.
- Siguran provisioning uređaja i izdavanje uređajskog tokena.
- Prijem i validacija telemetrije.
- Čuvanje mernih podataka i statusnih događaja.
- Slanje konfiguracije uređaju.
- Upravljanje tarifnim planovima.
- Nezavisna verifikacija tarifne klasifikacije.
- Generisanje računa i obračunskih perioda.
- Administratorski portal za servis, operatere i podršku.
- Korisnički portal ili API za pregled potrošnje.

## Glavni podsistemi

| Podsistem | Namena |
| --- | --- |
| Identitet i pristup | Korisnici, uloge, sesije, API tokeni i prava pristupa. |
| Registar uređaja | Evidencija uređaja, serijskih brojeva, claim kodova, modela i firmware verzija. |
| Provisioning | Prvo povezivanje uređaja sa backend-om i izdavanje uređajskog tokena. |
| Telemetrija | Prijem, validacija i skladištenje mernih podataka. |
| Konfiguracija uređaja | Slanje runtime, tarifne i uplink konfiguracije uređaju. |
| Tarife | Tarifni planovi, vremenski opsezi, cene i pravila. |
| Obračun | Obračunski periodi, računi, stavke računa i verifikacija potrošnje. |
| Događaji i alarmi | Tamper, power-fail, greške, offline stanje i servisni događaji. |
| Administracija | Interni panel za operatere, servis i podršku. |
| Korisnički pristup | Pregled potrošnje, istorije, računa i statusa uređaja. |

## Granica između uređaja i backend-a

Uređaj je odgovoran za:

- merenje,
- lokalnu akumulaciju energije,
- izbor aktivne tarife na osnovu konfiguracije i vremena,
- čuvanje kritičnog stanja,
- slanje telemetrije,
- prijavljivanje grešaka i statusa.

Backend je odgovoran za:

- identitet uređaja,
- autorizaciju,
- skladištenje podataka,
- proveru tarifne ispravnosti,
- obračun,
- administraciju,
- dugoročnu istoriju i prikaz.

## Predložena arhitektura

Minimalna MVP arhitektura može biti jedan backend servis sa jasno odvojenim modulima:

- HTTP API za uređaje,
- HTTP API za admin/krajnje korisnike,
- baza podataka,
- background worker za obradu telemetrije i obračun,
- scheduler za periodične poslove,
- object storage za izveštaje ako bude potreban.

Kasnije se podsistemi mogu izdvojiti u odvojene servise ako opterećenje ili timska organizacija to zahtevaju.
