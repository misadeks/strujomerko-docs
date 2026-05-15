# Plan implementacije backend-a

Ovaj plan predlaže redosled izrade backend sistema. Cilj je da se prvo napravi stabilan vertikalni tok od uređaja do baze, zatim administracija, pa obračun i korisnički prikaz.

## Faza 1: Osnova projekta

Uraditi:

- izabrati backend framework,
- podesiti bazu podataka,
- definisati migracije ili schema modele,
- postaviti konfiguraciju okruženja,
- dodati logging,
- dodati osnovni health endpoint,
- dodati testnu infrastrukturu.

Izlaz:

- backend se pokreće lokalno,
- postoji health check,
- baza je dostupna,
- testovi mogu da se pokrenu.

## Faza 2: Registar uređaja i provisioning

Uraditi:

- model `devices`,
- kreiranje uređaja iz admin strane ili seed skripte,
- claim code,
- bootstrap endpoint,
- izdavanje uređajskog tokena,
- čuvanje token hash-a,
- osnovna konfiguracija uređaja.

Izlaz:

- uređaj može prvi put da se registruje,
- dobija token,
- dobija početnu konfiguraciju.

## Faza 3: Telemetrija

Uraditi:

- endpoint za telemetriju,
- validaciju tokena,
- schema validaciju,
- deduplikaciju po `device_id + seq`,
- čuvanje sirovog payload-a,
- normalizovana merenja,
- status odgovora uređaju.

Izlaz:

- uređaj šalje merenja,
- backend ih čuva,
- duplikati se bezbedno obrađuju.

## Faza 4: Konfiguracija uređaja

Uraditi:

- model `device_configs`,
- revizije konfiguracije,
- endpoint za preuzimanje konfiguracije,
- endpoint za potvrdu primene,
- audit promene konfiguracije.

Izlaz:

- backend može promeniti konfiguraciju,
- uređaj može preuzeti i potvrditi novu reviziju.

## Faza 5: Tarife

Uraditi:

- model `tariff_plans`,
- admin CRUD za tarife,
- generisanje uređajske tarifne konfiguracije,
- verifikaciju `active_tariff_code`,
- čuvanje rezultata verifikacije.

Izlaz:

- uređaj dobija tarifne opsege,
- backend proverava prijavljene tarife.

## Faza 6: Događaji i alarmi

Uraditi:

- model `device_events`,
- pravila za tamper, power-fail i greške,
- pregled događaja u admin API-ju,
- status uređaja na osnovu poslednjeg javljanja.

Izlaz:

- servis vidi probleme uređaja,
- alarmi se mogu filtrirati i zatvarati.

## Faza 7: Obračun

Uraditi:

- modele `billing_periods` i `bills`,
- generisanje potrošnje po periodu,
- stavke računa,
- status računa,
- verifikacionu dijagnostiku.

Izlaz:

- backend može generisati draft račun iz telemetrije.

## Faza 8: Portali

Uraditi:

- admin portal,
- korisnički portal,
- grafove potrošnje,
- pregled računa,
- pregled statusa uređaja.

Izlaz:

- operator može upravljati sistemom,
- korisnik može videti potrošnju i račune.

## Faza 9: Produkciono očvršćavanje

Uraditi:

- rate limiting,
- audit log,
- backup baze,
- monitoring,
- alerting,
- obradu offline batch-eva,
- migracione procedure,
- sigurnosnu proveru tokena i lozinki.
