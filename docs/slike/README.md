# Slike i dijagrami

Ovaj folder je predviđen za slike, dijagrame, šeme i screenshot-ove koji će se koristiti u dokumentaciji.

## Predložene slike

| Fajl | Gde se koristi |
| --- | --- |
| `sistem-arhitektura.png` | Opšti pregled sistema. |
| `merni-hardverski-podsistem.png` | Merni hardverski podsistem. |
| `firmware-podsistem.png` | Firmware podsistem. |
| `lokalni-ui-podsistem.png` | Lokalni korisnički podsistem. |
| `servisni-podsistem-uredjaja.png` | Servisni režim uređaja. |
| `komunikacioni-podsistem.png` | GSM/GPRS uplink. |
| `backend-arhitektura.png` | Backend arhitektura. |
| `backend-podsistemi.png` | Backend podsistemi. |
| `backend-api-grupe.png` | Backend API grupe. |
| `backend-modeli-podataka.png` | Modeli podataka. |
| `tok-telemetrije.png` | Tok telemetrije. |
| `tarife-i-obracun.png` | Tarifni i obračunski tok. |
| `plan-implementacije-backenda.png` | Plan implementacije backend-a. |
| `proizvodni-podsistem.png` | Proizvodni provisioning i kalibracija. |

## Kako linkovati sliku

Iz dokumenta koji je u podfolderu, na primer `docs/backend/pregled-backenda.md`:

```markdown
![Backend arhitektura](../slike/backend-arhitektura.png)
```

Iz početne strane `docs/index.md`:

```markdown
![Arhitektura sistema](slike/sistem-arhitektura.png)
```
