# Arhitektura aplikacije i rute

Ovaj dokument opisuje kako je frontend aplikacija organizovana kroz Next.js App Router, layout-e, zaštitu ruta i glavne module.

## Root layout

Root layout je:

```text
src/app/layout.tsx
```

On postavlja:

- HTML/body shell,
- `ThemeProvider`,
- `AuthProvider`,
- globalni `Toaster`,
- globalni CSS iz `src/app/globals.css`.

## Glavni tok renderovanja

1. Korisnik dolazi na `/`.
2. `src/app/page.tsx` radi redirect na `/dashboard`.
3. Ako korisnik nije autentifikovan, `RoleGuard` ga šalje na `/login`.
4. Login čuva token i user state.
5. Korisnik se usmerava prema roli:
   - `CUSTOMER` -> `/portal`,
   - `FIELD_TECHNICIAN` ili `CREW` -> `/crew`,
   - ostale utility role -> `/dashboard`.

Frontend je primarno client-driven za admin, portal i crew sekcije. Većina stranica u `src/app` je tanak wrapper koji prosleđuje `section`, `kind` ili `id` centralnoj komponenti.

## Root i auth rute

| Ruta | Modul | Opis |
| --- | --- | --- |
| `/` | `src/app/page.tsx` | Redirect na `/dashboard`. |
| `/login` | `src/app/login/page.tsx` | Login forma, prikaz/sakrivanje lozinke i role-based redirect. |

## Admin dashboard

Admin layout:

```text
src/app/dashboard/layout.tsx
```

Koristi:

- `DashboardSidebar`,
- `DashboardHeader`,
- `RoleGuard`,
- `usePolling` za broj otvorenih događaja/alarmi na 30 sekundi,
- `AdminWorkspace`.

Admin rute:

| Ruta | Sekcija / komponenta | Opis |
| --- | --- | --- |
| `/dashboard` | `AdminWorkspace section="dashboard"` | Fleet dashboard, KPI, električne prosečne vrednosti, alarmi i work orders. |
| `/dashboard/customers` | `section="customers"` | Customer accounts i povezani entiteti. |
| `/dashboard/customers/[customerId]` | `EntityDetailPage kind="customer"` | Detalj customer account-a. |
| `/dashboard/service-points` | `section="service-points"` | Fizičke lokacije / service sites. |
| `/dashboard/points-of-delivery` | `section="points"` | Delivery points / supply points. |
| `/dashboard/points-of-delivery/[podId]` | `EntityDetailPage kind="point"` | Detalj point of delivery. |
| `/dashboard/contracts` | `section="contracts"` | Ugovori, tariff code i billing cycle. |
| `/dashboard/contracts/[contractId]` | `EntityDetailPage kind="contract"` | Detalj ugovora. |
| `/dashboard/meters` | `section="meters"` | Brojila, latest state i device provisioning. |
| `/dashboard/meters/[meterId]` | `EntityDetailPage kind="meter"` | Detalj brojila, telemetrija i device operacije. |
| `/dashboard/devices/[deviceId]` | `EntityDetailPage kind="device"` | Detalj uređaja. |
| `/dashboard/map` | `section="map"` | Mapa brojila. |
| `/dashboard/events` | `section="events"` | Događaji, acknowledge, resolve i ignore. |
| `/dashboard/device-connection-errors` | `section="device-errors"` | Greške konekcije device ingest-a. |
| `/dashboard/field` | `section="field"` | Service tickets, work orders i terenski rad. |
| `/dashboard/routes` | `section="routes"` | Route plan lista i planiranje ruta. |
| `/dashboard/routes/[routePlanId]` | `RoutePlanDetail` | Detalj route plana i stop statusi. |
| `/dashboard/tariffs` | `section="tariffs"` | Lista tariff plans. |
| `/dashboard/tariffs/new` | `TariffCreationWizard` | Wizard za novu tarifu. |
| `/dashboard/tariffs/[tariffPlanId]` | `EntityDetailPage kind="tariff"` | Detalj tarife. |
| `/dashboard/billing` | `section="billing"` | Billing schedules, periods i bills. |
| `/dashboard/billing/schedules/[scheduleId]` | `EntityDetailPage kind="schedule"` | Detalj billing schedule-a. |
| `/dashboard/billing/periods/[periodId]` | `EntityDetailPage kind="period"` | Detalj billing period-a. |
| `/dashboard/billing/bills/[billId]` | `EntityDetailPage kind="bill"` | Detalj računa. |
| `/dashboard/users` | `section="users"` | Utility users. |
| `/dashboard/settings` | `section="settings"` | Frontend operational settings/API info. |

Legacy redirect rute:

- `/dashboard/containers` -> `/dashboard/meters`,
- `/dashboard/containers/[containerId]` -> `/dashboard/meters`,
- `/dashboard/alarms` -> events,
- `/dashboard/maintenance` -> field service,
- `/dashboard/crews` -> `/dashboard/field`,
- `/dashboard/crews/[crewId]` -> `/dashboard/field`.

## Customer portal

Portal layout:

```text
src/app/portal/layout.tsx
```

Koristi:

- `RoleGuard` za `CUSTOMER`, uz dozvolu admin rola za pregled,
- responsive sidebar/mobile bottom nav,
- account dropdown,
- password change dialog,
- supply point header,
- `CustomerPortal`.

Portal rute:

| Ruta | Sekcija | Opis |
| --- | --- | --- |
| `/portal` | overview | Customer dashboard: status brojila, potrošnja, alerts i najnoviji račun. |
| `/portal/contracts` | contracts | Ugovori korisnika. |
| `/portal/contracts/[contractId]` | contract detail | Detalj ugovora. |
| `/portal/points-of-delivery` | supply points | Delivery/supply points korisnika. |
| `/portal/points-of-delivery/[podId]` | supply point detail | Detalj supply point-a. |
| `/portal/meters` | meters | Brojila korisnika. |
| `/portal/meters/[meterId]` | meter detail | Latest state, telemetrija i read-only detalji. |
| `/portal/usage` | usage | Pregled potrošnje i export summary ako backend podržava. |
| `/portal/bills` | bills | Računi korisnika. |
| `/portal/bills/[billId]` | bill detail | Read-only detalj računa sa breakdown-om. |

## Field technician interfejs

Crew layout:

```text
src/app/crew/layout.tsx
```

Koristi:

- `RoleGuard` za `FIELD_TECHNICIAN`, `CREW`, `ADMIN`, `UTILITY_ADMIN`,
- header sa korisnikom i GPS indikatorom,
- `CrewBottomNav`,
- `TechnicianWorkspace`.

Crew rute:

| Ruta | Sekcija | Opis |
| --- | --- | --- |
| `/crew` | work | Assigned work overview. |
| `/crew/route` | route | Današnja ruta i mapa. |
| `/crew/stops` | stops/orders | Work orders / stops. |
| `/crew/stops/[stopId]` | detail | Detalj work order-a. |
| `/crew/settings` | settings | Field settings/GPS. |

## Autentifikacija i autorizacija

Auth state je u:

```text
src/lib/auth/context.tsx
```

`AuthProvider` čuva:

- `user`,
- `isLoading`,
- `login(email, password)`,
- `logout()`.

Login flow:

1. `/login` poziva `useAuth().login`.
2. `apiLogin` u `src/lib/api/client.ts` poziva `/auth/login` ili mock login.
3. Token se čuva u `localStorage` pod ključem `auth_token`.
4. User se čuva/dekodira iz JWT payload-a ili mock user storage-a.
5. Stranica radi redirect prema roli.

Role:

- `SUPER_ADMIN`,
- `UTILITY_ADMIN`,
- `ADMIN`,
- `OPERATOR`,
- `DISPATCHER`,
- `FIELD_TECHNICIAN`,
- `CUSTOMER`,
- `VIEWER`,
- `CREW`.

`RoleGuard`:

- prikazuje skeleton dok se auth učitava,
- preusmerava neulogovanog korisnika na `/login`,
- ako korisnik nema dozvoljenu rolu, šalje ga na podrazumevanu rutu za njegovu rolu.

Unauthorized handling:

- API helper prepoznaje `401` i greške tipa `UNAUTHORIZED`,
- briše lokalnu sesiju,
- emituje `eks:auth:unauthorized`,
- preusmerava na `/login`.
