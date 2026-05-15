# Arhitektura firmware-a

Firmware je organizovan kao skup FreeRTOS servisa koji razmenjuju podatke preko snapshot modela, događaja, namenskih redova i sistemskih event bitova. Cilj arhitekture je da merenje, prikaz, skladištenje, uplink i servisni režim ostanu razdvojeni, ali sinhronizovani.

## Model komunikacije

### Merni snapshot

`svc_metering` periodično čita merno kolo i objavljuje najnoviji `meter_snapshot_t`. Snapshot sadrži sekvencu, monotoni timestamp, Unix vreme kada je dostupno, merne vrednosti, aktivnu tarifu, kvalitet podataka i statusne zastavice.

Potrošači snapshot-a su:

- `display_task`,
- `uplink_task`,
- `matter_task`,
- `diagnostics_task`,
- servisni API.

Potrošači se obaveštavaju direktnim task notifikacijama, a najnovije stanje čitaju preko funkcije `svc_metering_get_latest_snapshot()`.

### Sistemski događaji

Za sporije sistemske promene koristi se `esp_event` petlja sa `APP_EVENT_BASE`. Tipični događaji su:

- metering spreman ili u grešci,
- modem spreman,
- mreža registrovana,
- uplink online ili offline,
- Wi-Fi povezan ili prekinut,
- Matter commissioned,
- promenjena konfiguracija,
- ažurirana kalibracija,
- tamper otvoren ili vraćen,
- power-fail detektovan,
- emergency save pokrenut ili završen,
- ulazak u degradirani režim.

### Vlasnički redovi komandi

Servisi koji poseduju određeni resurs imaju sopstvene redove komandi. Primeri su `storage_cmd_q`, `modem_cmd_q`, `display_cmd_q`, `uplink_cmd_q`, `wifi_cmd_q`, `matter_cmd_q`, `supervisor_cmd_q` i `tamper_cmd_q`.

Ovaj model sprečava da više delova firmware-a istovremeno direktno upravlja istim resursom.

## Sistemski event bitovi

Firmware koristi event group bitove za brzu proveru globalnog stanja:

- `SYS_BOOT_DONE`
- `STORAGE_READY`
- `METER_READY`
- `LCD_READY`
- `MODEM_READY`
- `CELL_READY`
- `UPLINK_READY`
- `WIFI_READY`
- `MATTER_READY`
- `POWER_FAIL`
- `EMERGENCY_SAVE_DONE`
- `TAMPER_LATCHED`
- `DEGRADED_MODE`

## Power-fail putanja

Kada hardver detektuje gubitak glavnog napajanja:

1. GPIO prekid budi `power_guard_task`.
2. Postavljaju se `POWER_FAIL` i `DEGRADED_MODE`.
3. Objavljuju se sistemski događaji.
4. Isključuju se nebitni potrošači, na primer LCD backlight i modem.
5. Traži se zamrznuti merni snapshot.
6. Formira se `emergency_record_t`.
7. `STORAGE_CMD_EMERGENCY_FLUSH` se šalje storage servisu.
8. `storage_task` upisuje emergency zapis u NVS.
9. Uređaj ostaje u degradiranom režimu.

## Merni log

Merni log je namenjen bench analizi i dijagnostici, a ne obračunskom skladištenju. Podrazumevani kapacitet od `30000` zapisa pri kadenci od `200 ms` pokriva oko 100 minuta istorije.

Svaki zapis sadrži:

- sekvencu i timestamp,
- napon, struju, frekvenciju, aktivnu snagu i energiju,
- sirove ATM90E26 vrednosti,
- quality flags,
- fault code,
- tamper stanje,
- CRC.

Log se može čitati preko USB fabričkog protokola ili lokalnog servisnog API-ja.

## Tamper putanja

Tamper događaj prolazi kroz sledeći tok:

1. GPIO edge budi `tamper_task`.
2. Debouncing se radi u task kontekstu.
3. Otvaranje kućišta postavlja latched stanje.
4. Stanje se čuva kroz storage queue.
5. Događaj se šalje UI-ju i telemetriji.
6. Brisanje latch-a mora biti eksplicitna servisna komanda.
