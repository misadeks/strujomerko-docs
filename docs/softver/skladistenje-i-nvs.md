# Skladištenje i NVS

Firmware koristi NVS za kritične perzistentne podatke i SPIFFS particiju za kratkoročni merni log. Cilj je da uređaj preživi reset, gubitak napajanja i privremeni prekid mreže bez gubitka ključnog stanja.

## Podaci u NVS-u

Tipični zapisi:

| Zapis | Namena |
| --- | --- |
| `calib` | Kalibracioni koeficijenti i CRC validacija. |
| `emerg` | Emergency zapis pri power-fail događaju. |
| `cfg` | Runtime konfiguracija. |
| `dev_cfg` | Stariji objedinjeni per-device zapis. |
| `fleet_cfg` | Fabrička konfiguracija zajednička za seriju uređaja. |
| `unit_cfg` | Per-unit identitet, claim code i fabrički metapodaci. |
| `chkpt` | Poslednji checkpoint mernog stanja. |
| `tmpr_lat` | Perzistentni tamper latch. |

## SPIFFS merni log

Merni log nije obračunsko skladište. Koristi se za servisnu analizu, post-event pregled i validaciju tokom testiranja.

Log treba čitati paginirano preko:

- USB fabričkog protokola: `log_info`, `log_read`, `log_clear`,
- lokalnog servisnog API-ja: `/api/v1/measurement-log/info`, `/api/v1/measurement-log/read`, `/api/v1/measurement-log/clear`.

## NVS dump

Za očitavanje NVS particije koristi se alat iz firmware repozitorijuma:

```powershell
& "$env:USERPROFILE\.platformio\penv\Scripts\python.exe" tools\read_nvs.py --port COM3
```

Korisne opcije:

```powershell
& "$env:USERPROFILE\.platformio\penv\Scripts\python.exe" tools\read_nvs.py --port COM3 --dump blobs
& "$env:USERPROFILE\.platformio\penv\Scripts\python.exe" tools\read_nvs.py --port COM3 --project-json-out nvs_project.json
& "$env:USERPROFILE\.platformio\penv\Scripts\python.exe" tools\read_nvs.py --port COM3 --integrity-check
```

## Dijagnostika

Ako alat ne vidi očekivane zapise, moguće je da ih firmware još nije kreirao. Na primer, `calib` postoji tek nakon uspešne kalibracije, a `emerg` tek nakon power-fail čuvanja.

Ako je serijski port zauzet, zatvoriti monitor, Arduino Serial Monitor ili drugi alat koji drži COM port.
