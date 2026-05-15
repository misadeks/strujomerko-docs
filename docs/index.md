# Strujomerko dokumentacija

Ovo je objavljiva dokumentacija za Strujomerko. Organizovana je po logičkim celinama kako bi se jasno razdvojili proizvod, hardver, firmware, backend, proizvodni procesi, validacija i reference.

Svi tekstualni dokumenti u ovoj strukturi su na srpskom. Tehnički identifikatori, API putanje, JSON polja, nazivi firmware servisa i komande ostaju u originalnom obliku jer predstavljaju ugovor sa kodom i alatima.

## Struktura dokumentacije

| Odeljak | Namena |
| --- | --- |
| [`proizvod/`](proizvod/pregled.md) | Opšti opis sistema, specifikacija i korisničko uputstvo. |
| [`hardver/`](hardver/pregled-hardvera.md) | Komponente, merni front-end, napajanje, pinout i hardverska validacija. |
| [`softver/`](softver/arhitektura-firmware-a.md) | Firmware arhitektura uređaja, lokalni API, skladištenje i NVS. |
| [`backend/`](backend/pregled-backenda.md) | Backend podsistemi, API ugovori, modeli podataka, telemetrija, tarife i plan implementacije. |
| [`frontend/`](frontend/pregled-frontenda.md) | Frontend aplikacija, rute, API integracija, UI moduli i razvojni standardi. |
| [`proizvodnja-i-validacija/`](proizvodnja-i-validacija/bring-up-i-test-plan.md) | Proizvodna konfiguracija, USB provisioning, kalibracija i bring-up validacija. |
| `reference/` | Izvorni projektni zadatak i dodatni referentni materijali. |
| `slike/` | Slike, dijagrami i screenshot-ovi koji će biti dodati kasnije. |

## Preporučeni redosled čitanja

1. [`Pregled proizvoda`](proizvod/pregled.md)
2. [`Podsistemi sistema`](proizvod/podsistemi-sistema.md)
3. [`Specifikacija`](proizvod/specifikacija.md)
4. [`Pregled hardvera`](hardver/pregled-hardvera.md)
5. [`Arhitektura firmware-a`](softver/arhitektura-firmware-a.md)
6. [`Pregled backend-a`](backend/pregled-backenda.md)
7. [`Backend podsistemi`](backend/podsistemi.md)
8. [`Pregled frontend-a`](frontend/pregled-frontenda.md)
9. [`Lokalni servisni API`](softver/lokalni-servisni-api.md)
10. [`Bring-up i test plan`](proizvodnja-i-validacija/bring-up-i-test-plan.md)

## Slike

Slike, šeme, dijagrami i screenshot-ovi treba da idu u `docs/slike/`. U dokumentima su već ostavljena mesta za dijagrame backend arhitekture, podsistema, API grupa, modela podataka, telemetrije, obračuna i plana implementacije.

Primer linkovanja slike iz Markdown dokumenta:

```markdown
![Opis slike](../slike/naziv-slike.png)
```

Ako je slika u istom folderu kao dokument, koristi odgovarajuću relativnu putanju.

## Napomena o statusu

Ova dokumentacija je napravljena iz postojećih fajlova iz foldera `old docs/`. Pre objave kao zvanične release dokumentacije treba proveriti da li su API, pinout, kalibracioni tok i nalazi verifikacije usklađeni sa najnovijim firmware repozitorijumom.
