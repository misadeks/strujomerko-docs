# USB proizvodni provisioning

USB proizvodni provisioning koristi USB Serial/JTAG CDC kanal za komunikaciju između računara i uređaja. Isti fizički USB priključak može služiti za flash firmware-a, logove i proizvodni protokol, ali protokol mora biti jasno odvojen od običnih log linija.

## Transport

Preporučeni format je newline-delimited JSON sa prefiksom:

```text
@SMF:{"id":1,"cmd":"hello"}
@SMF:{"id":1,"ok":true,"chip_mac":"...","fw":"..."}
```

PC alat ignoriše sve linije koje ne počinju sa `@SMF:`. Firmware sve protokolske odgovore šalje istim prefiksom.

## Minimalni skup komandi

| Komanda | Namena |
| --- | --- |
| `hello` | Firmware verzija, chip MAC, flash veličina i schema verzije. |
| `read_records` | Čitanje fleet, unit, calibration i uplink sažetaka. |
| `write_fleet_cfg` | Upis fleet konfiguracije. |
| `write_unit_cfg` | Upis per-device konfiguracije. |
| `read_meter` | Čitanje skaliranih mernih vrednosti i sirovih ATM90E26 polja. |
| `log_info` | Informacije o mernom logu. |
| `log_read` | Paginirano čitanje mernog loga. |
| `log_clear` | Brisanje mernog loga. |
| `cal_start` | Ulazak u fabrički kalibracioni režim. |
| `cal_apply_point` | Slanje referentne tačke. |
| `cal_fit` | Izračunavanje koeficijenata. |
| `cal_write` | Perzistentni upis kalibracije. |
| `verify` | Provera zapisa, CRC-a i statusa. |
| `lock_factory` | Zaključavanje fabričkog stanja. |
| `reboot` | Restart uređaja. |

## PC proizvodni alat

PC alat može imati komande:

```powershell
python tools/factory_tool.py flash --port COM3
python tools/factory_tool.py provision --port COM3 --fleet factory/fleet.batch.json --batch BATCH-001
python tools/factory_tool.py calibrate --port COM3 --points factory/cal_points.demo.json
python tools/factory_tool.py verify --port COM3 --out factory/reports/MTR-0001.json
```

## Proizvodni web interfejs

Proizvodni web interfejs na računaru ne treba direktno da pristupa USB-u iz browser-a. Lokalni backend proces, na primer FastAPI, treba da poseduje COM port, šalje `@SMF:` komande i preko WebSocket-a prosleđuje podatke browser-u.

## Zaključavanje i bezbednost

Nakon uspešnog provisioninga i kalibracije, uređaj može biti označen kao proizvodno zaključen. Komande koje brišu provisioning, menjaju kalibraciju ili resetuju backend stanje treba zaštititi servisnom autentifikacijom i jasnim audit tragom.
