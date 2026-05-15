# Backend API ugovori

Backend treba da ima jasno razdvojene API grupe: uređajski API, administratorski API i korisnički API. Uređajski API mora biti stabilan i verzionisan jer firmware zavisi od njega.

## Verzije API-ja

Preporučena bazna putanja:

```text
/api/v1
```

Ako se promeni format payload-a koji uređaj šalje ili prima, povećati schema verziju u payload-u ili u API putanji.

## Uređajski API

Uređajski API koriste Strujomerko uređaji preko GSM/GPRS uplink-a.

### Bootstrap

```http
POST /api/v1/devices/bootstrap
```

Namena:

- prvo povezivanje uređaja,
- provera serijskog broja i claim code-a,
- izdavanje uređajskog tokena,
- vraćanje početne konfiguracije.

Minimalni zahtev:

```json
{
  "serial": "MTR-0001",
  "claim_code": "ABC123",
  "firmware_version": "1.0.0",
  "hardware_revision": "rev-a"
}
```

Minimalni odgovor:

```json
{
  "device_id": "dev_...",
  "meter_id": "mtr_...",
  "device_token": "...",
  "config_revision": 1,
  "config": {}
}
```

### Slanje telemetrije

```http
POST /api/v1/devices/{device_id}/telemetry
```

Namena:

- prijem mernih podataka,
- prijem statusa,
- potvrda primljenih sekvenci,
- slanje komandi ili nove konfiguracije ako je potrebno.

### Periodično javljanje uređaja

```http
POST /api/v1/devices/{device_id}/heartbeat
```

Namena:

- periodično javljanje uređaja,
- status firmware-a,
- RSSI i modem status,
- prijava trenutne konfiguracione revizije.

### Preuzimanje konfiguracije

```http
GET /api/v1/devices/{device_id}/config?known_revision=1
```

Ako uređaj već ima najnoviju konfiguraciju, backend može vratiti status bez punog payload-a. Ako postoji novija konfiguracija, backend vraća ceo konfiguracioni dokument.

### Potvrda konfiguracije

```http
POST /api/v1/devices/{device_id}/config/ack
```

Uređaj potvrđuje da je konfiguracija primljena, validirana i primenjena ili prijavljuje razlog neuspeha.

## Administratorski API

Administratorski API koriste admin portal, servisni panel i interni alati.

Predložene grupe:

| Grupa | Primeri endpoint-a |
| --- | --- |
| Uređaji | `GET /api/v1/admin/devices`, `GET /api/v1/admin/devices/{id}` |
| Korisnici | `GET /api/v1/admin/customers`, `POST /api/v1/admin/customers` |
| Lokacije | `GET /api/v1/admin/locations`, `POST /api/v1/admin/locations` |
| Ugovori | `GET /api/v1/admin/contracts`, `POST /api/v1/admin/contracts` |
| Tarife | `GET /api/v1/admin/tariff-plans`, `POST /api/v1/admin/tariff-plans` |
| Računi | `GET /api/v1/admin/bills`, `POST /api/v1/admin/billing-periods/{id}/generate-bills` |
| Događaji | `GET /api/v1/admin/events`, `PATCH /api/v1/admin/events/{id}` |

## Korisnički API

Korisnički API koristi portal ili mobilna aplikacija.

Predložene grupe:

| Grupa | Primeri endpoint-a |
| --- | --- |
| Profil | `GET /api/v1/me` |
| Moji uređaji | `GET /api/v1/me/meters` |
| Potrošnja | `GET /api/v1/me/meters/{id}/usage` |
| Računi | `GET /api/v1/me/bills`, `GET /api/v1/me/bills/{id}` |
| Obaveštenja | `GET /api/v1/me/notifications` |

## Pravila za greške

Svaka greška treba da ima stabilan format:

```json
{
  "error": {
    "code": "invalid_device_token",
    "message": "Token uređaja nije validan.",
    "details": {}
  }
}
```
