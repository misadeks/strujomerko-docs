# Tarife i obračun

Tarife i obračun su backend odgovornost. Uređaj bira aktivnu tarifu u trenutku merenja, ali backend mora nezavisno proveriti tu klasifikaciju i tek onda koristiti podatke za obračun.

## Tarifni plan

Tarifni plan treba da sadrži:

- naziv plana,
- vremensku zonu,
- vremenske opsege,
- cene po opsezima,
- potrošačke tier-e ako postoje,
- fiksne naknade,
- poreze,
- popuste,
- datum početka i kraja važenja,
- status plana.

## Deo konfiguracije koji ide na uređaj

Uređaju se šalje samo operativni deo:

- `tariff_config_revision`,
- `timezone`,
- `time_bands`,
- `effective_from`,
- pravila za specijalne dane ako postoje.

Cene i obračunska pravila ostaju na backend-u.

## Verifikacija tarifa

Backend za svako očitavanje treba da:

1. Odredi lokalno vreme iz timestamp-a.
2. Pronađe očekivani tarifni opseg.
3. Uporedi očekivani opseg sa `active_tariff_code`.
4. Uporedi ukupnu energetsku deltu sa zbirom tarifnih delti.
5. Označi odstupanja za audit.

## Obračunski periodi

Obračunski period definiše interval za koji se generiše račun.

Polja:

- `period_start`,
- `period_end`,
- `billing_scope`,
- `status`,
- `generated_at`,
- `approved_at`.

## Generisanje računa

Predloženi tok:

1. Izabrati ugovore obuhvaćene periodom.
2. Učitati merila za svaki ugovor.
3. Učitati validiranu telemetriju.
4. Izračunati potrošnju po tarifama.
5. Primijeniti cene, tier-e, poreze i naknade.
6. Kreirati stavke računa.
7. Sačuvati dijagnostiku verifikacije.
8. Ostaviti račun u statusu `draft` ili ga automatski izdati ako je poslovno pravilo takvo.

## Statusi računa

Predloženi statusi:

- `draft`,
- `issued`,
- `paid`,
- `overdue`,
- `cancelled`,
- `corrected`.
