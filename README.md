# Strujomerko dokumentacija

Repozitorijum objavljive dokumentacije za Strujomerko, pametni jednofazni merač električne energije.

Dokumentacija je podeljena po logičkim celinama:

- [`docs/proizvod/`](docs/proizvod/pregled.md) - pregled proizvoda, specifikacija i korisničko uputstvo.
- [`docs/hardver/`](docs/hardver/pregled-hardvera.md) - hardverske komponente, blokovi, pinout i validacija.
- [`docs/softver/`](docs/softver/arhitektura-firmware-a.md) - firmware arhitektura uređaja, lokalni servisni API, skladištenje i NVS.
- [`docs/backend/`](docs/backend/pregled-backenda.md) - backend podsistemi, API ugovori, modeli podataka, telemetrija, tarife, obračun i plan implementacije.
- [`docs/proizvodnja-i-validacija/`](docs/proizvodnja-i-validacija/bring-up-i-test-plan.md) - proizvodna konfiguracija, USB provisioning, kalibracija i validacija.
- [`docs/reference/`](docs/assets/ohmsprint-2026-projektni-zadatak.pdf) - izvorni projektni zadatak i dodatni referentni materijali.
- [`docs/slike/`](docs/slike/) - folder za slike, dijagrame i screenshot-ove.

Svi tekstualni dokumenti u `docs/` su na srpskom. Tehnički identifikatori kao što su API putanje, JSON polja, nazivi firmware servisa i komande ostaju u originalnom obliku da bi dokumentacija ostala tačna u odnosu na kod.

## Počni ovde

| Čitalac | Dokument |
| --- | --- |
| Pregled projekta | [`docs/proizvod/pregled.md`](docs/proizvod/pregled.md) |
| Podsistemi | [`docs/proizvod/podsistemi-sistema.md`](docs/proizvod/podsistemi-sistema.md) |
| Krajnji korisnici | [`docs/proizvod/korisnicko-uputstvo.md`](docs/proizvod/korisnicko-uputstvo.md) |
| Hardver | [`docs/hardver/pregled-hardvera.md`](docs/hardver/pregled-hardvera.md) |
| Firmware uređaja | [`docs/softver/arhitektura-firmware-a.md`](docs/softver/arhitektura-firmware-a.md) |
| Backend | [`docs/backend/pregled-backenda.md`](docs/backend/pregled-backenda.md) |
| Servisni API | [`docs/softver/lokalni-servisni-api.md`](docs/softver/lokalni-servisni-api.md) |
| Proizvodnja i validacija | [`docs/proizvodnja-i-validacija/bring-up-i-test-plan.md`](docs/proizvodnja-i-validacija/bring-up-i-test-plan.md) |

## Slike

Slike koje naknadno dodaš stavljamo u:

```text
docs/slike/
```

U dokumentima su ostavljena mesta za slike i dijagrame, posebno u backend odeljku.

Primer linkovanja iz dokumenta:

```markdown
![Opis slike](../slike/naziv-slike.png)
```

## GitHub Pages

Repozitorijum sadrži `mkdocs.yml` i GitHub Actions workflow u [`.github/workflows/pages.yml`](.github/workflows/pages.yml).

Za lokalni pregled:

```powershell
python -m pip install mkdocs
python -m mkdocs serve
```

Zatim otvori `http://127.0.0.1:8000`.
