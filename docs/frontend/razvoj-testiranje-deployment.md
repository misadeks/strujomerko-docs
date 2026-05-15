# Razvoj, testiranje i deployment

Ovaj dokument opisuje standarde za razvoj, lokalne provere, build, deployment, troubleshooting i korake za dodavanje nove funkcionalnosti.

## TypeScript standardi

`tsconfig.json` ima:

- `strict: true`,
- path alias `@/* -> ./src/*`,
- `moduleResolution: bundler`,
- `jsx: react-jsx`,
- `noEmit: true`.

Preporuke:

- koristiti postojeće tipove iz `meter-client.ts` ili dodati domenski tip blizu API klijenta,
- koristiti `Record<string, unknown>` samo kada backend shema još nije stabilna,
- normalizovati ID kroz `recordId`,
- normalizovati liste kroz `asItems`,
- ne pretpostavljati da backend uvek vraća isti omotač liste.

## React i Next standardi

Pravila:

- client komponente moraju imati `"use client"`,
- komponente koje koriste `useState`, `useEffect`, `window`, `localStorage`, Leaflet ili geolocation moraju biti client komponente,
- page wrapper-i treba da ostanu tanki,
- zajedničku logiku držati u `components/meter-platform`, `hooks` ili `lib`,
- redirect koristiti iz `next/navigation`.

Next 16 koristi async `params` u dynamic route page-evima:

```ts
export default async function MeterDetailPage({
  params,
}: {
  params: Promise<{ meterId: string }>;
}) {
  const { meterId } = await params;
  return <EntityDetailPage kind="meter" id={meterId} />;
}
```

## UI pravila

Preporuke:

- koristiti postojeće `ui` komponente,
- koristiti lucide ikone,
- koristiti `PageHeader`, `ResourceTable`, `StatusBadge`, `MetricTile`,
- za akcije koristiti toast iz Sonner-a,
- svaka data stranica treba da ima loading, empty i error stanje,
- destructive akcije treba da imaju potvrdu,
- posle create/update/delete pozvati `reload` ili očistiti cache,
- ne praviti novu paralelnu biblioteku komponenti ako se može proširiti postojeća.

## API pravila

Preporuke:

- sav novi smart-meter API kod ide u `src/lib/api/meter-client.ts`,
- koristiti `meterApi.raw` samo kada metoda još nije eksplicitno dodata,
- posle stabilizacije dodati imenovanu metodu u `meterApi`,
- za query string koristiti interni `qs` helper,
- za upload koristiti `FormData`,
- ne ručno postavljati `Content-Type` za `FormData`,
- greške prikazivati korisniku kroz toast ili error banner.

## Testiranje i provere

Trenutno `package.json` nema test skriptu. Dostupne provere su:

```bash
npm run lint
npm run build
```

Preporučena lokalna provera pre merge-a:

```bash
npm run lint
npm run build
```

Za UI promene ručno proveriti:

- `/login`,
- `/dashboard`,
- relevantnu admin listu,
- relevantnu detail stranicu,
- `/portal` za customer tok,
- `/crew` za technician tok ako promena utiče na field,
- responsive/mobile layout,
- loading i error stanja,
- tamnu temu ako promena dira globalne stilove.

Za API promene proveriti:

- auth token u request headers,
- list response mapping,
- empty response,
- `401` / `403` / `404`,
- cache invalidation posle mutacija,
- CSV response ako se radi export.

## Build i deployment

Production build:

```bash
npm run build
```

Start production servera:

```bash
npm run start
```

Deployment mora obezbediti:

- Node.js 20+,
- validan `NEXT_PUBLIC_API_URL`,
- HTTPS za production,
- backend CORS za frontend origin,
- stabilan auth/token flow.

`next.config.ts` trenutno sadrži:

```ts
allowedDevOrigins: ["192.168.1.160"]
```

Ako se testira sa druge lokalne IP adrese, dodati je ovde ili koristiti standardni localhost tok.

## Troubleshooting

### Korisnik se stalno vraća na login

Proveriti:

- da li token postoji u `localStorage` pod `auth_token`,
- da li backend vraća `401`,
- da li JWT payload ima `sub` i `role`,
- da li je `NEXT_PUBLIC_API_URL` tačan,
- da li browser blokira request zbog CORS-a.

### Admin stranica je prazna

Proveriti:

- rolu korisnika,
- network tab za failed request,
- da li backend vraća `{ items: [...] }`, niz ili drugi format,
- da li je `recordId` sposoban da nađe ID,
- da li komponenta ima error banner koji je možda izvan vidljivog dela.

### Portal ne prikazuje podatke

Proveriti:

- korisnik mora imati `CUSTOMER` rolu ili admin rolu za pregled,
- `/me/*` endpoint-i moraju scoping raditi prema tokenu,
- cache može držati staru vrednost; pozvati `clearMeterApiCache` ili osvežiti posle mutacije.

### Mapa se ne učitava

Proveriti:

- komponenta mora biti client component,
- Leaflet CSS/assets,
- validne koordinate,
- da nema SSR pristupa `window`,
- da li je container dobio visinu.

### Build pada na TypeScript-u

Proveriti:

- dynamic route `params` tip za Next 16,
- da li client-only API koristi `"use client"`,
- da li su `Record<string, unknown>` vrednosti cast-ovane pre renderovanja,
- da li import koristi alias `@/`.

### API radi u Postman-u, ali ne u frontendu

Proveriti:

- `NEXT_PUBLIC_API_URL`,
- CORS,
- `Authorization: Bearer <token>`,
- content type,
- da li frontend šalje field imena koja backend očekuje,
- da li backend response ima `detail.error.message`, `detail`, ili drugi error shape.

## Kako dodati novu admin sekciju

1. Dodati API metode u `src/lib/api/meter-client.ts`.
2. Dodati tipove ako backend shema postoji.
3. Dodati novu route page u `src/app/dashboard/<nova-sekcija>/page.tsx`.
4. Proširiti `AdminSection` u `admin-workspace.tsx`.
5. Proširiti `AdminState` ako sekcija ima svoj data set.
6. Dodati fetch u `useAdminData`.
7. Dodati view komponentu ili proširiti `AdminListView`.
8. Dodati metadata u `adminNavMetadata`.
9. Dodati item u `DashboardSidebar`.
10. Dodati loading/empty/error state.
11. Dodati create/edit flow ako je potreban.
12. Pokrenuti `npm run lint` i `npm run build`.

## Kako dodati novu detail stranicu

1. Dodati `kind` u `EntityDetailPage`.
2. Dodati fetch metodu kroz `meterApi`.
3. Dodati route `src/app/dashboard/<resource>/[id]/page.tsx`.
4. U page wrapper-u pročitati async `params`.
5. Vratiti `<EntityDetailPage kind="<kind>" id={id} />`.
6. Dodati link iz liste na detail route.

## Kako dodati customer portal stranicu

1. Dodati `/me/*` API metodu u `meter-client.ts`.
2. Proširiti `CustomerPortal` sekcije.
3. Dodati route u `src/app/portal`.
4. Dodati item u `portalNav`.
5. Proveriti desktop sidebar i mobile bottom/more navigaciju.
6. Obezbediti read-only prikaz ako korisnik nema pravo mutacije.

## Važne napomene za budući razvoj

- Smart-meter domen je aktivni pravac razvoja; container/waste sloj tretirati kao legacy.
- Ne hardkodovati tarifne zone kao fiksna polja; koristiti dinamičke nizove iz tariff plana.
- Backend list response može imati više oblika; uvek koristiti `asItems`.
- Backend ID može biti `id`, `_id` ili domenski ID; uvek koristiti `recordId` za UI liste/linkove.
- Customer portal treba da koristi samo `/me/*` endpoint-e za customer podatke.
- Device ingest endpoint-i nisu browser-user tokovi; prikazivati ih samo kao operativnu informaciju kada je korisno.
- Raw telemetry ne treba agresivno cache-irati.
- Mutacije treba da osveže UI i očiste cache.
- Role-based redirect i `RoleGuard` su centralna zaštita ruta; ne oslanjati se samo na skrivanje navigacije.

## Mesto za slike

Ovde dodati:

```markdown
![Frontend razvojni tok](../slike/frontend-razvojni-tok.png)
![Frontend deployment](../slike/frontend-deployment.png)
```
