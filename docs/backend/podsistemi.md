# Backend podsistemi

Ovaj dokument opisuje backend podsisteme i šta svaki od njih treba da radi. Cilj je da implementacija bude podeljena na jasne celine, bez mešanja uređajskog API-ja, administracije, obračuna i skladištenja telemetrije.

## 1. Identitet i pristup

Ovaj podsistem upravlja korisnicima i pravima pristupa.

Treba da podrži:

- administratorske naloge,
- servisne naloge,
- korisničke naloge,
- uloge i dozvole,
- login i refresh token,
- audit log za kritične akcije.

Predložene uloge:

| Uloga | Prava |
| --- | --- |
| `admin` | Puna administracija sistema. |
| `operator` | Upravljanje korisnicima, ugovorima, tarifama i računima. |
| `service` | Servisni pregled uređaja, dijagnostika i reset određenih stanja. |
| `viewer` | Pregled podataka bez izmena. |
| `customer` | Pregled sopstvene potrošnje i računa. |

## 2. Registar uređaja

Registar uređaja čuva identitet svakog Strujomerko uređaja.

Treba da sadrži:

- serijski broj,
- model i hardversku reviziju,
- firmware verziju,
- status uređaja,
- claim code ili provisioning status,
- vezu ka korisniku, lokaciji ili ugovoru,
- vreme poslednjeg javljanja,
- trenutno poznatu konfiguracionu reviziju.

Statusi uređaja mogu biti:

- `manufactured`,
- `claimed`,
- `active`,
- `suspended`,
- `retired`,
- `service_required`.

## 3. Prvo povezivanje uređaja

Provisioning povezuje fizički uređaj sa backend zapisom i izdaje uređajski token.

Tipičan tok:

1. Fabrika kreira uređaj u backend-u ili generiše claim code.
2. Uređaj šalje bootstrap zahtev sa serijskim brojem i claim code-om.
3. Backend proverava da li je uređaj poznat i dozvoljen.
4. Backend vraća `device_id`, `meter_id`, uređajski token i početnu konfiguraciju.
5. Uređaj čuva token u NVS-u i koristi ga za dalju komunikaciju.

## 4. Telemetrija

Telemetrijski podsistem prima merenja i statusne informacije.

Treba da radi:

- validaciju uređajskog tokena,
- proveru schema verzije,
- proveru redosleda sekvenci,
- deduplikaciju ponovljenih paketa,
- čuvanje sirovog payload-a,
- normalizaciju merenja,
- izračunavanje agregiranih podataka,
- upis događaja i alarma.

## 5. Konfiguracija uređaja

Backend mora uređaju slati konfiguraciju kroz verzionisani ugovor.

Konfiguracija može obuhvatiti:

- interval slanja telemetrije,
- heartbeat interval,
- backend endpoint-e,
- tarifnu konfiguraciju,
- vremensku zonu,
- servisna ograničenja,
- feature flag-ove,
- komande koje uređaj treba da izvrši.

Svaka konfiguracija mora imati reviziju. Uređaj treba da prijavi koju reviziju koristi.

## 6. Tarife

Tarifni podsistem čuva planove, vremenske opsege, cene i pravila.

Uređaju se šalje samo operativni deo koji je potreban za lokalni izbor aktivne tarife. Cene i obračunska pravila ostaju na backend-u.

## 7. Obračun

Obračunski podsistem koristi verifikovanu telemetriju i tarifne planove za izradu računa.

Treba da podrži:

- obračunske periode,
- ručno i automatsko generisanje računa,
- stavke računa,
- poreze i naknade,
- korekcije,
- status računa,
- audit verifikaciju potrošnje.

## 8. Događaji i alarmi

Događaji služe za servisni i operativni nadzor.

Tipični događaji:

- tamper otvoren,
- tamper latch obrisan,
- power-fail,
- emergency save,
- uređaj offline,
- modem greška,
- merenje u grešci,
- reset brojača,
- zastarela konfiguracija,
- drift sata.

## 9. Administratorski portal

Admin portal treba da omogući:

- pregled uređaja,
- pregled korisnika i ugovora,
- pregled telemetrije,
- servisne akcije,
- upravljanje tarifama,
- generisanje računa,
- pregled alarma,
- audit log.

## 10. Korisnički portal

Korisnički portal treba da prikaže:

- trenutnu i istorijsku potrošnju,
- potrošnju po tarifama,
- statuse uređaja relevantne korisniku,
- račune,
- obaveštenja i upozorenja.

## Mesto za slike

Ovde dodati dijagram podsistema:

```markdown
![Backend podsistemi](../slike/backend-podsistemi.png)
```
