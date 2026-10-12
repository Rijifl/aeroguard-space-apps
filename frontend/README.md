# AeroGuard web app

The AeroGuard user interface: a [Next.js 15](https://nextjs.org/) App Router application written in TypeScript, styled with Tailwind CSS 4 and [shadcn/ui](https://ui.shadcn.com/) components.

## Run

```bash
npm install
npm install --no-save leaflet
npm run dev
```

Open <http://localhost:3000>. Leaflet is imported by the map but is not listed in `package.json`, so it is installed separately. Pages that forecast or give advice also need the [API](../backend/README.md) running on port 8000.

Create `frontend/.env.local` from [`.env.example`](../.env.example) to enable live OpenAQ sensor data.

| Script | Action |
|---|---|
| `npm run dev` | Development server with Turbopack |
| `npm run build` | Production build (currently blocked by type and lint errors) |
| `npm run start` | Serve the production build |
| `npm run lint` | ESLint |

## Pages

| Route | Source | What it shows |
|---|---|---|
| `/` | [`app/page.tsx`](app/page.tsx) | Landing page with links into each tool |
| `/map` | [`app/map/page.tsx`](app/map/page.tsx) | Wildfire map of GA, NC, SC and TN with Citizen and EMS views, a toggle to a global AQI map, and an inline wildfire report form |
| `/aqi-anderson` | [`app/aqi-anderson/page.tsx`](app/aqi-anderson/page.tsx) | Air quality dashboard: OpenAQ sensor charts, NO₂ forecast and city AQI forecast |
| `/chat` | [`app/chat/page.tsx`](app/chat/page.tsx) | Advice chat with wildfire and pollution modes |
| `/report` | [`app/report/page.tsx`](app/report/page.tsx) | Environmental emergency report form (errors on server render; use the form on `/map`) |

## API routes

These run on the Next.js server and keep third-party calls out of the browser.

| Route | Source | Behaviour |
|---|---|---|
| `GET /api/openaq` | [`app/api/openaq/route.ts`](app/api/openaq/route.ts) | Fetches measurements for five OpenAQ sensors |
| `POST /api/openaq` | same | Fetches ten daily NO₂ averages from OpenAQ and forwards them to the API's `/predict-no2` |
| `POST /api/ai` | [`app/api/ai/route.ts`](app/api/ai/route.ts) | Validates `mode` (`wildfire` or `pollution`) and proxies to the API's advice endpoints |
| `POST /api/report` | [`app/api/report/route.ts`](app/api/report/route.ts) | Validates a report and returns a generated report ID |

## Key components

| Component | Role |
|---|---|
| [`components/region-map.tsx`](components/region-map.tsx) | Leaflet map: state outlines, NASA EONET wildfire markers, simulated citizen position, safe-location routing, EMS help-request markers |
| [`components/map-viewer.tsx`](components/map-viewer.tsx) | Citizen / EMS view switch around the map |
| [`components/aqi-visualization.tsx`](components/aqi-visualization.tsx) | D3 time-series charts, summary statistics and forecast cards |
| [`components/inline-report-form.tsx`](components/inline-report-form.tsx) | Wildfire report form shown under the map |
| [`components/ui/`](components/ui) | shadcn/ui primitives and the navigation bar |

## Layout

```
frontend/
├── app/            Routes, API handlers, global styles, root layout
├── components/     Feature components and UI primitives
├── constants/      Shared constants (Clemson coordinates)
├── lib/            Utilities (class name helper)
├── public/         Static assets, including the embedded AQI map (message.html)
└── types/          Shared TypeScript types
```
