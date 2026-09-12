# SolarWorth (Omi)

**SolarWorth** is a website that answers one question: if I put solar panels on this roof, is that a good use of money?

You talk to **Omi**, a small sun-shaped helper. Omi asks a few questions about the place, the roof, the land, and the electric bill. Then it looks up sunshine and prices, runs the money math, and shows a plain-English yes / maybe / no — plus a picture of that roof from above.

This is a hackathon estimate, not financial advice and not a quote from an installer.

Browse the codebase: [cursor.com/codebase/anything-omar/solarworth](https://cursor.com/codebase/anything-omar/solarworth)

---

## What you can do

- Ask in a city **or** a full street address (address is better if you want the real roof).
- Pick which way the roof faces, how steep it is (including a **completely flat** 0° roof), and whether a hill or trees steal the sun.
- Enter a monthly bill in **that country’s money** (yen in Japan, euros in France, US dollars only in the United States). Skip uses a typical local bill, not a US $150 default.
- See payback years, 25-year return, year-one savings, and “what if” stress tests.
- See a year of production as a bar chart with the kWh number on each month.
- See the roof from satellite, and (with a Fal key) the same roof with panels drawn on the sunny slope.
- Paste API keys in the app at `/keys` — no need to edit code.

The site works **without any keys**. Sunshine and maps come from free public APIs. Extra keys unlock live prices, a written verdict, panel drawings, and saving a report.

---

## How a check works

Omi walks through five questions, then the server runs a pipeline (you see each step as it happens):

1. **Pin the place** — geocode the city or address.
2. **Sunshine** — a year of weather at that pin, then a solar map aimed at *this* roof tilt and compass direction.
3. **Prices** — typical regional rates, or a live web search if Exa / Firecrawl keys are present.
4. **Money math** — payback, 25-year net, ROI. The US 30% tax credit is applied **only** for US locations. Amounts are converted to local currency.
5. **Stress tests** — bills rising faster, cheap power, cost overruns, extra shade, wrong roof face, no tax credit (US only).
6. **Grok** (optional) — a short written take on the verdict.
7. **Roof photo** — overhead satellite. Fal can edit that photo to add panels. Street-front photos are not the main picture.
8. **Save** (optional) — store the report in Convex.

---

## How it was built

This started as an empty repo and was built as a Next.js app with a chatbot-style questionnaire instead of a long form.

- The **UI** is one question at a time (city → facing → pitch → land → bill), then a results page.
- The **analysis** is a Node route (`/api/analyze`) that streams progress over SSE so Omi can talk through each step.
- **Geocoding, weather, PV maps, and FX** are called from the server. Optional sponsor APIs are skipped cleanly when a key is missing.
- **Finance math** lives in TypeScript (`src/lib/finance.ts`). If a Daytona key is present, the same numbers can run in a sandbox.
- **Currency** is chosen from the country of the place you picked. Math is done in USD internally, then converted for display.

Main product files:

| Piece | Where |
| --- | --- |
| Chat UI | `src/components/solar-app.tsx` |
| Omi face | `src/components/omi-face.tsx` |
| Analysis pipeline | `src/lib/pipeline.ts`, `src/app/api/analyze/route.ts` |
| Roof / sun aiming | `src/lib/orientation.ts` |
| Payback math | `src/lib/finance.ts` |
| Currency | `src/lib/currency.ts` |

---

## What it was built with

| Layer | Tools |
| --- | --- |
| App | [Next.js](https://nextjs.org/) 15 (App Router), React 19, TypeScript |
| Style | Tailwind CSS 4, [shadcn/ui](https://ui.shadcn.com/) |
| Motion | Motion (Framer) |
| Icons | Lucide |
| Optional compute | [Daytona](https://www.daytona.io/) SDK |
| Optional database | [Convex](https://www.convex.dev/) |
| Hosting (example) | Any Node host (Render, Vercel, etc.) |

Dev server default port: **43127**.

---

## APIs and data sources

### Always used (no key)

| Service | What it does |
| --- | --- |
| [Open-Meteo geocoding](https://open-meteo.com/en/docs/geocoding-api) | City search and pin |
| [Open-Meteo archive](https://open-meteo.com/en/docs/historical-weather-api) | A year of sunshine / radiation |
| [Photon](https://photon.komoot.io/) (OpenStreetMap) | Street / address autocomplete |
| [Nominatim](https://nominatim.org/) | Address search and reverse geocode |
| [PVGIS](https://re.jrc.ec.europa.eu/pvg_tools/en/) (EU JRC) | kWh per kW for this tilt, direction, and nearby terrain |
| [Esri World Imagery](https://www.arcgis.com/) | Overhead satellite of the lot |
| [open.er-api.com](https://www.exchangerate-api.com/) | USD → local currency rates |

Built-in **typical electricity and install costs** (US state averages, world fallback) are used when live web search is off.

### Optional keys (paste on `/keys` or in `.env.local`)

| Key | Service | What it unlocks |
| --- | --- | --- |
| `XAI_API_KEY` | [x.ai Grok](https://console.x.ai) | Written verdict in plain English |
| `XAI_MODEL` | x.ai | Optional model name (default tries `grok-4.6`, then older Grok models) |
| `EXA_API_KEY` | [Exa](https://dashboard.exa.ai) | Live web search for local rates and panel costs |
| `FIRECRAWL_API_KEY` | [Firecrawl](https://www.firecrawl.dev/app) | Reads the utility / installer pages Exa finds |
| `FAL_KEY` | [Fal.ai](https://fal.ai/dashboard) | Draws panels onto the real aerial roof (`flux-pro/kontext`, then image-to-image fallback) |
| `DAYTONA_API_KEY` | [Daytona](https://app.daytona.io) | Runs the calculator in a sandbox (math still works without this) |
| `CONVEX_URL` / `NEXT_PUBLIC_CONVEX_URL` | [Convex](https://dashboard.convex.dev) | Saves a finished check |
| `GOOGLE_MAPS_API_KEY` | Google Geocoding / Street View | More precise rooftop pin; Street View is **not** shown as the main house photo |

You do not need all of them. Add one, reload, and that piece lights up.

x.ai keys from console.x.ai usually start with `xai-`.

---

## Run it on your computer

You need Node.js 18 or newer.

```bash
npm install
cp .env.example .env.local
npm run dev
```

Open [http://localhost:43127](http://localhost:43127). Paste keys at [http://localhost:43127/keys](http://localhost:43127/keys) if you have them.

### Clone with Origin CLI (WSL on Windows)

Origin CLI is for macOS, Linux, and WSL — not PowerShell.

```bash
# Run in WSL
curl -fsSL https://downloads.cursor.com/origin/install.sh | sh
origin auth login
origin repo clone anything-omar/solarworth
```

If `origin` is missing after install:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

CLI docs: [cursor.com/docs/origin/cli](https://cursor.com/docs/origin/cli)

---

## What the numbers assume

- Default system size: **8 kW**.
- Bill is in **local currency**; skip uses a typical bill for that currency.
- **30% US federal tax credit** only when the pin is in the United States.
- Panels lose a little power each year (~0.5%).
- Electricity prices rise about **3% a year** in the base case (stress tests try 1% and 5%).
- Extra power you do not use is credited at a weaker price than retail.

---

## Deploy

1. Point a Node web service at this repo.
2. Build: `npm install && npm run build`
3. Start: `npm run start`
4. Set the same keys as environment variables on the host.

Do not commit `.env.local`.
