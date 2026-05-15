# Pregled frontend aplikacije

Frontend aplikacija za EKS / Strujomerko platformu je Next.js aplikacija za rad sa pametnim električnim brojilima, korisničkim portalom i operativnim utility procesima. Namenjena je developerima koji održavaju, proširuju ili povezuju frontend sa backend-om.

## Glavna korisnička iskustva

Frontend pokriva tri glavna toka:

| Tok | Ruta | Namena |
| --- | --- | --- |
| Admin / utility operator panel | `/dashboard` | Upravljanje korisnicima, brojilima, događajima, tarifama, računima i terenskim radom. |
| Customer / consumer portal | `/portal` | Korisnički pregled potrošnje, ugovora, brojila i računa. |
| Field technician interfejs | `/crew` | Mobilni interfejs za terenske tehničare, work orders i rute. |

## Domen aplikacije

Glavni domen je elektroenergetska infrastruktura:

- korisnici i customer accounts,
- service points i points of delivery,
- ugovori,
- brojila i uređaji,
- poslednje električne vrednosti i telemetrija,
- događaji, alarmi i greške konekcije uređaja,
- terenski rad, work orders i rute,
- tarife, billing schedules, billing periods i računi.

U projektu postoji i stariji sloj za kontejnere/otpad (`containers`, `crews`, `maintenance`) koji je delom zadržan zbog kompatibilnosti i mock podataka. Novi razvoj treba usmeravati na `meter-platform` komponente i `meterApi` klijent.

## Tehnički stack

Osnovne tehnologije:

- Next.js `16.2.4`,
- React `19.2.4`,
- TypeScript `5`,
- Tailwind CSS `4`,
- shadcn-style UI komponente,
- Radix UI primitives,
- `lucide-react` ikone,
- Sonner toast notifikacije,
- `next-themes` za light/dark temu,
- Recharts za grafikone,
- Leaflet i React Leaflet za mape,
- TanStack Table / React Table za tabele,
- React Hook Form i Zod za forme i validaciju gde se koriste,
- `ra-core` kao data-provider kompatibilni sloj za deo admin resursa.

## Lokalno pokretanje

Zahtevi:

- Node.js 20+,
- npm.

Instalacija:

```bash
npm install
```

Development server:

```bash
npm run dev
```

Podrazumevani URL:

```text
http://localhost:3000
```

Ako je port zauzet:

```bash
npm run dev -- -p 3001
```

Production build:

```bash
npm run build
npm run start
```

## Konfiguracija okruženja

Lokalna konfiguracija se drži u `.env.local`.

Najvažnije promenljive:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
NEXT_PUBLIC_USE_MOCK_API=false
```

`NEXT_PUBLIC_API_URL` je bazni URL za backend API. Novi meter API klijent ima fallback:

```text
https://api.eks.misa.tel/api/v1
```

`NEXT_PUBLIC_USE_MOCK_API` kontroliše stariji mock client u `src/lib/api/client.ts`.

Važno:

- Za realni smart-meter backend koristi se `src/lib/api/meter-client.ts`.
- Za starije container/mock ekrane koristi se `src/lib/api/client.ts`.
- Ako `NEXT_PUBLIC_USE_MOCK_API` nije eksplicitno `false`, stariji klijent radi u mock režimu.
- `NEXT_PUBLIC_*` promenljive su dostupne u browser bundle-u, pa se u njima ne smeju čuvati tajne.

## Struktura projekta

```text
.
|-- docs/
|-- public/
|-- src/
|   |-- app/
|   |-- components/
|   |-- hooks/
|   |-- lib/
|   |-- types/
|-- package.json
|-- next.config.ts
|-- tsconfig.json
|-- eslint.config.mjs
|-- components.json
```

Najvažniji folderi:

- `src/app` - Next.js App Router rute, layout-i i page entrypoints.
- `src/components/app-shell` - layout/navigation shell za dashboard, portal i crew tokove.
- `src/components/meter-platform` - glavni feature modul za smart-meter domen.
- `src/components/ui` - shadcn/Radix bazne komponente.
- `src/components/maps` - Leaflet map komponente i client-only map wrapper-i.
- `src/components/devices` - device config dijalozi.
- `src/hooks` - reusable hook-ovi kao polling i geolocation tracking.
- `src/lib/api` - API klijenti i data provider.
- `src/lib/auth` - auth context.
- `src/lib/mock-data` - mock podaci za stariji client.
- `src/types` - stariji/globalni tipovi; deo njih je još u container terminologiji.

## Mesto za slike

Ovde dodati:

- opšti screenshot dashboard-a,
- screenshot korisničkog portala,
- screenshot field technician interfejsa,
- dijagram strukture frontend aplikacije.

Primer:

```markdown
![Frontend pregled](../slike/frontend-pregled.png)
```
