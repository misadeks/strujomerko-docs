# Telemetrija i događaji

Telemetrija je glavni tok podataka od uređaja ka backend-u. Backend mora primiti podatke, proveriti ih, sačuvati sirovi payload, normalizovati korisna polja i proizvesti događaje ili agregate.

## Telemetrijski payload

Payload treba da sadrži:

- `schema_version`,
- `device_id`,
- `meter_id`,
- `seq`,
- `timestamp`,
- merenja,
- tarifna polja,
- statusna polja,
- quality flags,
- informacije o firmware-u kada je potrebno.

Primer:

```json
{
  "schema_version": "1.1",
  "seq": 12345,
  "timestamp": "2026-05-15T12:00:00Z",
  "voltage_v": 230.1,
  "current_a": 2.5,
  "frequency_hz": 50.0,
  "active_power_w": 575.0,
  "active_energy_wh": 120045,
  "active_tariff_code": "T1",
  "tariff_config_revision": 7,
  "tariff_energy_wh": {
    "T1": 80000,
    "T2": 40045
  },
  "tariff_delta_wh": {
    "T1": 25,
    "T2": 0
  },
  "quality_flags": []
}
```

## Obrada telemetrije

Preporučeni tok:

1. Proveriti uređajski token.
2. Proveriti da uređaj postoji i da je aktivan.
3. Proveriti schema verziju.
4. Proveriti sekvencu i deduplikaciju.
5. Sačuvati sirovi payload.
6. Normalizovati merenja.
7. Proveriti tarifnu konzistentnost.
8. Upisati događaje ako postoje alarmi.
9. Ažurirati `last_seen_at`.
10. Vratiti potvrdu i eventualne komande.

## Deduplikacija

Uređaj može ponoviti slanje ako ne dobije odgovor zbog loše GSM veze. Backend mora tretirati par `device_id + seq` kao jedinstven za telemetrijski paket.

Ako duplikat stigne sa istim payload-om, backend vraća potvrdu bez ponovnog obračuna. Ako duplikat ima drugačiji payload, treba ga označiti kao konflikt.

## Događaji

Događaji se mogu poslati eksplicitno ili izvesti iz telemetrije.

Tipični događaji:

- tamper,
- power-fail,
- emergency save,
- restart uređaja,
- clock unsynced,
- modem greška,
- metering fault,
- config rejected,
- firmware update status,
- offline buffer flush.

## Agregati

Za brz prikaz u portalu backend treba da računa agregate:

- potrošnja po satu,
- potrošnja po danu,
- potrošnja po mesecu,
- potrošnja po tarifi,
- maksimalna snaga po periodu,
- broj i tipovi događaja.

Agregati se mogu računati background worker-om nakon upisa telemetrije.

## Mesto za slike

Ovde dodati tok telemetrije:

```markdown
![Tok telemetrije](../slike/tok-telemetrije.png)
```
