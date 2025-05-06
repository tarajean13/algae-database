# 🌊 **Algenia — North-American Marine Micro-Algae Library**

> **Status**: 🚧 MVP inception  
> **Owner**: Tara Jean (product lead)  
> **AI builders**: Cursor IDE, Prometheus Swarm agent-orchestration

---

## 1  Project Vision
A **zero‑cost, open‑access web app** (<https://algenia.co>) that lets anyone **search, filter and explore _marine_ micro‑algae species recorded in Canada and the USA**. We highlight each species’ ecological roles and climate‑mitigation potential.

---

## 2  Scope (MVP)
| Included | Excluded (v1) |
|----------|---------------|
| Marine micro‑algae in Canada & USA | Freshwater, macro‑algae |
| Filters: name, taxonomy, province/state, colour group, core traits | Omics data, lab protocols |
| Static heat‑map PNG per species (see §7) | Interactive global map |
| Weekly automated data refresh | Real‑time ingest |
| Public, read‑only app | Auth, user uploads |

> **Species Universe Estimate**: ~5–8 k names (GBIF + OBIS).  
> **Data footprint**: 25 fields × 8 k rows ≈ 6 MB parquet (images external).

---

## 3  Target Users
* Marine & phycology researchers
* Climate‑tech entrepreneurs (biofuels, carbon credits)
* Educators, students & citizen scientists

---

## 4  Data Model
### 4.1 `species` table
| Column | Type | Example | Notes |
|--------|------|---------|-------|
| id | UUID | `8e5f…` | primary key |
| scientific_name | text | `Nannochloropsis oculata` | |
| common_names | text[] | `["yellow-green algae"]` | |
| phylum / class / order / family / genus | text |  | taxonomy ranks |
| color_group | enum | Green / Brown / Red / Blue-green | |
| geo_distribution | jsonb | `{ "Canada": ["BC"], "USA": ["CA"] }` | provinces & states |
| temperature_optimum_c | numeric | 22 | growth optimum |
| lipid_content_pct_dw | numeric | 45 | dry‑weight basis |
| traits | jsonb | `{ "cell_size_um": 3 }` | flexible extra traits |
| food_web_role | text | `Primary producer; copepod diet` | |
| biogeochemical_role | text | `Silica cycling; CO2 sink` | |
| climate_uses | text | `Biofuel; carbon sequestration` | |
| images | text[] | | external URLs |
| primary_source | text | GBIF | provenance |

### 4.2 (optional) `occurrences`
Raw point occurrences reserved for future live‑map; not stored in MVP to stay below 500 MB.

---

## 5  Data Sources & Licensing (public only)
| Source | License | Fields used |
|--------|---------|-------------|
| GBIF Occurrence API | CC‑BY 4.0 | taxonomy, coords |
| OBIS Bulk CSV | CC‑BY 4.0 | coords |
| iNaturalist API | CC‑BY etc. | images, sightings |
| Culture collections (NIES, CCMP) | varies | strain IDs |

Each record stores its original licence and citation.

---

## 6  Tech Stack & Runtime
| Layer | Choice | Reason |
|-------|--------|--------|
| Node.js | >= 20 | matches Vercel default |
| Python | 3.11 | latest pandas & cartopy |
| Database | Supabase Postgres + PostGIS (< 500 MB) | zero cost |
| API | Supabase auto REST/GraphQL | no backend code |
| Front‑end | Next.js (TypeScript) | SSG + SSR |
| UI | Tailwind CSS + shadcn/ui | rapid dev |
| ETL | Python (pandas, supabase-py) | AI friendly |
| Static maps | Cartopy -> PNG (600×400) | small storage |
| Monitoring | Sentry | error capture |

**Upgrade path**: if DB exceeds 500 MB, migrate to Neon.tech (10 GB free).

---

## 7  Visual Design
Generate static PNG heat‑maps:
```bash
python etl/render_heatmap.py --species_id <uuid> --out public/maps/
```
PNG size: 600×400 px, < 60 kB.

---

## 8  Branding
| Element | Value |
|---------|-------|
| Domain | algenia.co |
| Palette | Alga-bud `#DAFFB4`, White `#FFFFFF`, Black `#02081F`, accents Porple `#C381CD`, Ocean Blue `#3758B6`, Hadal Zone `#002165` |
| Typography | Inter |
| Logo | `/public/logo.svg` |

---

## 9  Development Conventions
### Folder structure
```
/app/               # Next.js source
  pages/
  components/
/etl/               # Python pipelines & map renderer
/public/maps/       # PNG heat‑maps
.github/workflows/  # CI + ETL
```

### Environment variables (`.env.example`)
```
SUPABASE_URL=<url>
SUPABASE_ANON_KEY=<key>
MAPBOX_TOKEN=
```

### Dependencies
`package.json` excerpt:
```json
{
  "name": "algenia",
  "engines": { "node": ">=20" },
  "dependencies": {
    "next": "latest",
    "react": "latest",
    "@supabase/supabase-js": "latest",
    "tailwindcss": "latest",
    "clsx": "latest"
  }
}
```
`requirements.txt`:
```
python>=3.11
pandas
supabase-py
cartopy
numpy
pytest
```

### Testing
At least one `pytest` unit test per module; CI fails if tests or linting fail.

---

## 10  Local Setup
```bash
git clone https://github.com/<org>/algenia.git
cd algenia
cp .env.example .env
npm install
npm run dev
pip install -r requirements.txt
python etl/etl_gbif.py --sample
```

---

## 11  Automated Data Ops
Workflows run on **main**.
* ETL (cron `0 3 * * 2`): fetch, clean, upsert; generate heat‑maps; post Slack report.
* QA: validate; open `data-alert` issues if anomalies.
* DocBot: verify doc‑strings for non‑tech clarity.

Human reviews Slack digest & GitHub issues, then marks ✅ / ❌.

---

## 12  Roadmap (Phase 1)
- [ ] Define trait list
- [ ] GBIF + OBIS ETL
- [ ] Search UI
- [ ] Heat‑map generator
- [ ] Weekly pipelines + QA
- [ ] Accessibility & responsive
- [ ] Freshwater phase

---

## 13  Contributing
* Branch names: `feat/*`, `fix/*`
* Commits: Conventional Commits
* PR checklist: tests, lint, doc‑strings

---

## 14  License
* Code: MIT  
* Data: upstream CC (see §5)

---

## 15  Acknowledgements
GBIF, OBIS, iNaturalist and the open‑science community.
