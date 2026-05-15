# API integracija, podaci i realtime

Frontend ima dva API sloja: novi meter API za aktivni smart-meter domen i stariji/mock API za legacy delove aplikacije.

## Novi meter API

Glavni fajl:

```text
src/lib/api/meter-client.ts
```

Ovo je glavni klijent za smart meter backend. Izvozi:

- `API_BASE_URL`,
- tipove za meter domen,
- helper-e `recordId`, `asItems`, `clearMeterApiCache`,
- `meterApi` objekat sa metodama.

Karakteristike:

- centralizovan `fetch`,
- automatski `Authorization: Bearer <token>`,
- JSON body handling,
- FormData upload bez ručnog `Content-Type`,
- CSV response handling,
- GET cache za određene customer endpoint-e,
- cache invalidation posle non-GET zahteva,
- normalizacija list response-a kroz `asItems`,
- tolerancija na `id`, `_id` i domenske ID ključeve kroz `recordId`.

## Najvažnije grupe metoda

Dashboard:

- `getFleetDashboard`,
- `getElectricalAverages`,
- `getOfflineMeters`,
- `getFaultedMeters`,
- `getTamperAlarms`.

Customers/users:

- `listCustomers`,
- `getCustomer`,
- `createCustomer`,
- `patchCustomer`,
- `listUtilityUsers`,
- `createUtilityUser`.

Service/delivery/contract:

- `listServicePoints`,
- `listContracts`,
- `listPointsOfDelivery`.

Meters/devices:

- `listMeters`,
- `getMeter`,
- `getMeterLatest`,
- `assignMeterToPoint`,
- `createDeviceCommand`,
- `clearTamperLatch`,
- `createMeterClaimCode`,
- `updateDeviceConfig`.

Events:

- `listEvents`,
- `getEvent`,
- `acknowledgeEvent`,
- `resolveEvent`,
- `ignoreEvent`,
- delete helpers.

Field:

- `listServiceTickets`,
- `createServiceTicket`,
- `patchServiceTicket`,
- `listWorkOrders`,
- `createWorkOrder`,
- `patchWorkOrder`.

Tariffs:

- `listTariffPlans`,
- `getTariffPlan`,
- `createTariffPlan`,
- `patchTariffPlan`.

Billing:

- `listBillingSchedules`,
- `materializeSchedule`,
- `listBillingPeriods`,
- `generateBills`,
- `regenerateBills`,
- `listBills`,
- `getBill`.

Customer portal:

- `getProfile`,
- `getMyContracts`,
- `getMyMeters`,
- `getMyUsageSummary`,
- `exportMyUsageSummary`,
- `getMyBills`,
- `getMyAlerts`.

Technician:

- `getTechnicianWorkOrders`,
- `patchTechnicianWorkOrder`,
- `addTechnicianWorkOrderNote`,
- `addTechnicianWorkOrderAttachment`,
- `getTodayRoute`.

Media:

- `getMedia`,
- `getMediaFileUrl`.

## Stariji/mock API

Fajl:

```text
src/lib/api/client.ts
```

Ovaj klijent sadrži:

- mock login korisnike,
- mock dashboard/container/device/crew/route data,
- starije container endpoint-e,
- `createOperationsStream`,
- media helper-e.

Koristi se za starije komponente i legacy delove. Ne treba ga proširivati za nove smart-meter funkcionalnosti ako već postoji odgovarajući meter backend endpoint.

## Endpoint katalog

Detaljan katalog endpoint-a je u:

```text
docs/frontend_api_endpoints.md
```

Za tačne sheme uvek proveriti backend OpenAPI:

```text
http://127.0.0.1:8080/docs
http://127.0.0.1:8080/openapi.json
```

## List response normalizacija

Backend može vratiti:

- niz objekata,
- `{ items: [...] }`,
- `{ results: [...] }`.

Koristi se:

```ts
asItems(response)
```

Za ID:

```ts
recordId(record)
```

`recordId` proverava `id`, `_id`, `meter_id`, `contract_id`, `point_of_delivery_id`, `pod_id`, `device_id`, `customer_account_id`, `tariff_plan_id`, `billing_schedule_id`, `billing_period_id`, `bill_id`, `event_id`, `ticket_id`, `work_order_id`.

## Cache

`meter-client.ts` ima in-memory request cache za GET endpoint-e koji se pozivaju kroz `cachedApi`.

Primeri TTL-a:

- profile i contracts: 5 minuta,
- my meters: 2 minuta,
- latest meter: 15 sekundi,
- usage summary: 60 sekundi,
- hourly telemetry: 60 sekundi,
- daily telemetry: 5 minuta,
- raw telemetry: bez cache-a.

Svaki non-GET zahtev čisti meter API cache.

## Polling

Hook:

```text
src/hooks/use-polling.ts
```

Koristi se za periodično osvežavanje. Dashboard layout ga koristi za broj otvorenih događaja na 30 sekundi.

## Realtime

Stariji klijent ima `createOperationsStream` za SSE:

```text
/realtime/operations
```

Podržani event listener-i:

- `crew.location.updated`,
- `alarm.created`,
- `alarm.updated`.

Ovo trenutno pripada starijem operations/container sloju. Za nove realtime meter događaje treba definisati novu integraciju u `meter-client.ts` ili jasno migrirati postojeću.

## Mesto za slike

Ovde dodati:

```markdown
![Frontend API slojevi](../slike/frontend-api-slojevi.png)
![Frontend cache i realtime tok](../slike/frontend-cache-realtime.png)
```
