# Modeli podataka

Ovaj dokument opisuje predložene glavne modele podataka za backend. Nazivi kolekcija i polja su predlog i mogu se prilagoditi izabranoj bazi i framework-u.

## Pregled modela

| Model | Namena |
| --- | --- |
| `users` | Nalozi za admin, operatere, servis i korisnike. |
| `customers` | Krajnji korisnici ili pravna lica. |
| `locations` | Lokacije na kojima su uređaji instalirani. |
| `devices` | Fizički Strujomerko uređaji. |
| `meters` | Logička merna mesta vezana za uređaje i ugovore. |
| `contracts` | Ugovorni odnos korisnika, lokacije, merila i tarife. |
| `telemetry_readings` | Merni podaci primljeni od uređaja. |
| `device_events` | Tamper, power-fail, greške i statusni događaji. |
| `device_configs` | Verzije konfiguracije poslate uređajima. |
| `tariff_plans` | Tarifni planovi, vremenski opsezi i cene. |
| `billing_periods` | Obračunski periodi. |
| `bills` | Računi i stavke računa. |
| `audit_log` | Trag administrativnih i servisnih akcija. |

## `devices`

Čuva fizički uređaj.

Ključna polja:

- `id`,
- `serial`,
- `model`,
- `hardware_revision`,
- `firmware_version`,
- `status`,
- `claim_code_hash`,
- `device_token_hash`,
- `last_seen_at`,
- `last_config_revision`,
- `created_at`,
- `updated_at`.

## `meters`

Predstavlja merno mesto. U jednostavnom sistemu jedan uređaj ima jedno merilo, ali odvajanje modela olakšava zamenu uređaja bez gubitka istorije mernog mesta.

Ključna polja:

- `id`,
- `device_id`,
- `location_id`,
- `obis_code`,
- `status`,
- `installed_at`,
- `removed_at`.

## `telemetry_readings`

Čuva normalizovana očitavanja.

Ključna polja:

- `id`,
- `device_id`,
- `meter_id`,
- `seq`,
- `timestamp`,
- `received_at`,
- `voltage_v`,
- `current_a`,
- `frequency_hz`,
- `active_power_w`,
- `active_energy_wh`,
- `active_tariff_code`,
- `tariff_config_revision`,
- `tariff_energy_wh`,
- `tariff_delta_wh`,
- `quality_flags`,
- `raw_payload`.

## `device_events`

Čuva servisne i alarmne događaje.

Primeri tipova:

- `tamper_opened`,
- `tamper_restored`,
- `power_fail`,
- `emergency_saved`,
- `meter_fault`,
- `clock_unsynced`,
- `config_rejected`,
- `device_offline`.

## `tariff_plans`

Čuva tarifne planove.

Ključna polja:

- `id`,
- `name`,
- `timezone`,
- `time_bands`,
- `usage_tiers`,
- `fixed_charges`,
- `fees`,
- `taxes`,
- `discounts`,
- `valid_from`,
- `valid_to`,
- `status`.

## `bills`

Čuva obračun i finalne iznose.

Ključna polja:

- `id`,
- `customer_id`,
- `contract_id`,
- `billing_period_id`,
- `total_energy_kwh`,
- `usage_breakdown`,
- `line_items`,
- `tax_breakdown`,
- `total_amount`,
- `status`,
- `issued_at`,
- `due_at`.

## Indeksi

Preporučeni indeksi:

- `devices.serial` jedinstven,
- `telemetry_readings.device_id + seq` jedinstven,
- `telemetry_readings.meter_id + timestamp`,
- `device_events.device_id + created_at`,
- `bills.customer_id + issued_at`,
- `contracts.customer_id + status`.
