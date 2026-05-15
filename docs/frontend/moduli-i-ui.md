# Moduli, UI sistem i domeni

Ovaj dokument opisuje glavne frontend module, UI sistem, admin panel, customer portal, field technician interfejs, tarife, billing, mape i legacy/mock sloj.

## UI sistem i dizajn

Globalni CSS:

```text
src/app/globals.css
```

Koristi:

- Tailwind CSS v4,
- shadcn theme token-e,
- light/dark CSS variables,
- font family kroz theme token-e,
- Leaflet popup custom styling.

UI komponente su u:

```text
src/components/ui
```

Tipične komponente:

- `Button`,
- `Card`,
- `Dialog`,
- `DropdownMenu`,
- `Input`,
- `Label`,
- `Select`,
- `Tabs`,
- `Table`,
- `Tooltip`,
- `Badge`,
- `Alert`,
- `Sheet`,
- `Skeleton`.

Dizajn aplikacije je operativan, ne marketinški:

- navigacija je sidebar/bottom-nav, ne landing hero,
- tabele, filteri i detalj stranice su glavni radni model,
- forme treba da budu jasne, guste i validirane,
- prazna, loading i error stanja treba imati na svakom data view-u,
- tekst u admin i portal delu treba da koristi meter/electricity/utility/customer terminologiju,
- ne uvoditi novu container/bin/waste terminologiju u aktivan smart-meter UI.

## Admin panel

Glavna komponenta:

```text
src/components/meter-platform/admin-workspace.tsx
```

`AdminWorkspace` prima `section` i učitava podatke kroz `useAdminData(section)`.

`AdminState` obuhvata:

- fleet dashboard,
- electrical averages,
- offline/faulted meters,
- tamper alarms,
- customers,
- service points,
- points of delivery,
- contracts,
- meters,
- events,
- device connection errors,
- service tickets,
- work orders,
- route plans,
- tariffs,
- billing schedules,
- billing periods,
- bills,
- users.

Glavni view-evi:

- `DashboardView`,
- `AdminListView`,
- `EventsView`,
- `DeviceConnectionErrorsView`,
- `FieldView`,
- `RoutesView`,
- `TariffsView`,
- `BillingView`,
- `SettingsView`.

Kreiranje entiteta:

- `CustomerWizardDialog`,
- `ContractWizardDialog`,
- `MeterWizardDialog`,
- `SimpleCreateDialog`,
- `TariffCreationWizard`.

Detalj stranice:

```text
src/components/meter-platform/entity-detail-page.tsx
```

Ona renderuje više tipova entiteta preko `kind`: customer, contract, point, meter, device, tariff, schedule, period i bill.

### Slike admin panela

![Admin overview](../slike/admin-portal-UI/overview.png)
![Svi korisnici](../slike/admin-portal-UI/svi-korisnici.png)
![Prikaz korisnika](../slike/admin-portal-UI/prikaz-korisnika.png)
![Ugovori](../slike/admin-portal-UI/ugovori.png)
![Svi uređaji](../slike/admin-portal-UI/svi-uredjaji.png)
![Prikaz uređaja](../slike/admin-portal-UI/prikaz-uredjaja.png)
![Događaji i greške](../slike/admin-portal-UI/error-dogadjaji.png)
![Tiketi održavanja](../slike/admin-portal-UI/tiketi-odrzavanja.png)
![Sve tarife](../slike/admin-portal-UI/sve-tarife.png)
![Prikaz tarife - osnovni podaci](../slike/admin-portal-UI/prikaz-tarife-1.png)
![Prikaz tarife - detalji](../slike/admin-portal-UI/prikaz-tarife-2.png)
![Računi](../slike/admin-portal-UI/racuni.png)

## Customer portal

Glavna komponenta:

```text
src/components/meter-platform/customer-portal.tsx
```

Portal je read-only ili ograničeno self-service iskustvo za korisnike. Layout daje navigaciju, account menu i password change flow.

`portalNav`:

- Home,
- Contracts,
- Supply points,
- Bills,
- Consumption,
- Meters.

Portal koristi `/me/*` backend endpoint-e, što znači da backend radi scoping podataka prema autentifikovanom customer nalogu.

Najvažniji tokovi:

- dashboard overview,
- pregled ugovora,
- pregled supply point-a,
- pregled brojila,
- detalj brojila sa latest state i telemetrijom,
- usage summary i export,
- lista računa,
- detalj računa sa breakdown-om,
- alerts/događaji relevantni za korisnika.

Password change u `PortalLayout`:

1. Korisnik unosi trenutnu lozinku.
2. Frontend pokušava login za verifikaciju.
3. Patch-uje customer account sa novom lozinkom.
4. Ponovo radi login sa novom lozinkom.
5. Zatvara dialog i prikazuje toast.

### Slike korisničkog portala

![Početni prikaz portala](../slike/user-portal-UI/pocetni-prikaz.png)
![Svi ugovori](../slike/user-portal-UI/svi-ugovori.png)
![Prikaz ugovora](../slike/user-portal-UI/prikaz-ugovora.png)
![Svi uređaji korisnika](../slike/user-portal-UI/svi-uredjaji.png)
![Prikaz uređaja korisnika](../slike/user-portal-UI/prikaz-uredjaja.png)
![Potrošnja - pregled](../slike/user-portal-UI/potrosnja-1.png)
![Potrošnja - detalji](../slike/user-portal-UI/potrosnja-2.png)
![Svi računi](../slike/user-portal-UI/svi-racuni.png)
![Prikaz računa - osnovno](../slike/user-portal-UI/prikaz-racuna-1.png)
![Prikaz računa - detalji](../slike/user-portal-UI/prikaz-racuna-2.png)

## Field technician interfejs

Glavna komponenta:

```text
src/components/meter-platform/technician-workspace.tsx
```

Field interfejs je optimizovan za mobilni rad:

- bottom navigation,
- work overview,
- route screen,
- stops/work orders,
- work order detail,
- settings.

API metode:

- `getTechnicianWorkOrders`,
- `getTechnicianWorkOrder`,
- `patchTechnicianWorkOrder`,
- `addTechnicianWorkOrderNote`,
- `addTechnicianWorkOrderAttachment`,
- `getTodayRoute`.

Geolocation hook:

```text
src/hooks/use-crew-location.ts
```

`useCrewLocationTracking` koristi browser geolocation API i šalje poziciju uz throttling. Trenutno šalje kroz stariji `sendCrewLocation` iz `src/lib/api/client.ts`, što treba imati u vidu ako se field route endpoint-i potpuno migriraju na novi meter backend.

## Tarife i billing

Tarife i billing su centralni deo utility domena. Ne sme se hardkodovati logika kao green/blue/red zone. UI treba da tretira tarifu kao konfigurabilan skup:

- `time_bands`,
- `usage_tiers`,
- `fixed_charges`,
- `fees`,
- `discounts`,
- `taxes`,
- public service fee,
- vulnerable customer discount,
- effective dates,
- active/inactive state.

Tariff wizard:

```text
src/components/meter-platform/tariff-creation-wizard.tsx
```

Bill detail panels:

```text
src/components/meter-platform/bill-detail-panels.tsx
```

Tipičan billing tok:

1. Definisati tariff plan.
2. Kreirati billing schedule.
3. Materialize schedule za ciljnu godinu/mesec.
4. Dobiti billing period.
5. Pokrenuti dry-run bill generation.
6. Pregledati rezultat.
7. Pokrenuti generate/regenerate bills.
8. Pregledati bills list i bill detail.

## Mape, geolokacija i rute

Mape koriste Leaflet/React Leaflet.

Komponente:

- `src/components/meter-platform/meter-map-view.tsx`,
- `src/components/meter-platform/meter-map-client.tsx`,
- `src/components/maps/city-map-client.tsx`,
- `src/components/maps/operations-map.tsx`,
- `src/components/maps/container-location-picker.tsx`,
- `src/components/maps/container-location-picker-client.tsx`,
- `src/app/crew/route/crew-route-map-client.tsx`.

Next.js server rendering i Leaflet ne idu direktno zajedno, pa su mape client-only wrapper-i. Kod novih mapa treba paziti da:

- Leaflet import bude u client komponenti,
- nema SSR pristupa `window`/`document`,
- marker podaci imaju validan lat/lng,
- empty state bude jasan kada nema lokacije,
- ruta i marker-i ne zaklanjaju osnovne kontrole.

Rute i route planning:

- admin lista route planova: `/dashboard/routes`,
- detalj route plana: `/dashboard/routes/[routePlanId]`,
- field današnja ruta: `/crew/route`,
- backend endpoint-i: `/routes/plan`, `/routes/plans`, `/routes/plans/{id}`, `/routes/plans/{id}/stops/{stop_id}`.

## Komponente i helperi

Resource UI:

```text
src/components/meter-platform/resource-ui.tsx
```

Sadrži:

- `PageHeader`,
- `MetricTile`,
- `ResourceTable`,
- `RawRecordCard`,
- `SimpleCreateDialog`,
- `StatusBadge`,
- `textValue`,
- `numberValue`,
- `AnyRecord`.

Creation dialogs:

```text
src/components/meter-platform/creation-dialogs.tsx
```

Device config:

```text
src/components/devices/device-config-dialog.tsx
src/lib/device-config.ts
```

Events:

```text
src/components/meter-platform/event-overview-sheet.tsx
src/components/meter-platform/event-delete-dialog.tsx
src/components/alarms/alarm-severity-badge.tsx
src/components/alarms/alarm-detail-sheet.tsx
```

Dates:

```text
src/lib/dates.ts
```

Važni helper-i:

- `parseApiDate`,
- `formatApiDateTime`,
- `formatDateDDMMYYYY`,
- `formatTimeInTimeZone`,
- `safeTimeZone`,
- `weekdayInTimeZone`.

Podrazumevana tarifa/timezone:

```text
Europe/Belgrade
```

## Legacy i mock sloj

U projektu postoje stariji container/waste modeli:

- `Container`,
- `CleaningCrew`,
- `MaintenanceTicket`,
- `FillState`,
- `CameraState`,
- `MOCK_CONTAINERS`,
- `MOCK_CREWS`,
- `MOCK_MAINTENANCE_TICKETS`.

Ovaj sloj je koristan za demo/mock razvoj, starije ekrane i reference za UI obrasce, ali za novi EKS smart-meter frontend:

- ne dodavati nove container endpoint-e,
- ne uvoditi garbage/bin tekst u aktivan UI,
- nove tipove dodavati uz meter domen,
- nove API metode dodavati u `meter-client.ts`.

