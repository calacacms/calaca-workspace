# CalacaCMS (workspace)

**GitHub (workspace-root):** repository **`calaca-workspace`** — [github.com/calacacms/calaca-workspace](https://github.com/calacacms/calaca-workspace); bevat o.a. roadmap, agent-context en Cursor-regels; **niet** `cms/`, `docs/` of `landing/` (eigen repos).

Deze map is de **workspace** rond **CalacaCMS**: een site-first PHP-CMS met aparte projecten voor documentatie en marketing. Elk onderdeel heeft eigen dependencies en scripts; start en bouw ze vanuit de betreffelde subdirectory.

---

## Mappenoverzicht

| Map | Wat het is | Waar je begint |
|-----|------------|----------------|
| [`cms/`](cms/) | Het **CMS zelf**: PHP-backend (Twig, routing, opslag), admin onder `/admin`, thema’s met Vite + Tailwind v4 + Alpine. | Zie [`cms/calaca/README.md`](cms/calaca/README.md) voor installatie, CLI (`calaca …`) en architectuur. In deze checkout: `bun install` in `cms/`, daarna o.a. `bun run start` (PHP op `localhost:8000` + Vite watch voor thema’s). |
| [`docs/`](docs/) | **Documentatiesite** (Astro + Starlight): gepubliceerde pagina’s als `.mdx` in [`docs/src/content/docs/`](docs/src/content/docs/) (getting-started, user-guide, developer, reference). | [`docs/README.md`](docs/README.md) (Starlight-starter). In `docs/`: `bun install`, `bun dev` (typisch `localhost:4321`). |
| [`agent-context/`](agent-context/) | **Kennisbank voor mensen en AI-agenten** (Markdown): SSOT, onderzoek, plannen — **niet** de Starlight-site. | [`agent-context/README.md`](agent-context/README.md), [`agent-context/PROJECT_SSOT.md`](agent-context/PROJECT_SSOT.md). |
| [`landing/`](landing/) | **Marketing**: Astro + Tailwind v4; **`/`** = Coming Soon (hoofdpagina), **`/landing`** = volledige landingspagina (niet voor zoekmachines). | [`landing/README.md`](landing/README.md). In `landing/`: `bun install`, `bun run dev` / `bun run build`. |

---

## Productvisie, wensen en roadmap

- **Wensen + implementatiestatus:** [`cms-wensen.md`](cms-wensen.md)
- **Roadmap** (features om later op te pakken, o.a. plugins/marketplace): [`ROADMAP.md`](ROADMAP.md)
- **Open punten en onderzoek:** o.a. [`agent-context/research/`](agent-context/research/)

---

## Snelstart (per project)

**CMS** (`cms/`):

```bash
cd cms && bun install && bun run start
```

Admin: `http://localhost:8000/admin` (na lokale setup zoals in de CMS-readme).

**Bundle-analyse:** vanuit `cms/` na `bun run build` (of `bun run build:admin` / `build:default`) de gegenereerde HTML openen: `themes/admin/bundle-stats.html` en `themes/site/default/bundle-stats.html` — interactieve treemap en geschatte gzip/brotli-groottes (`rollup-plugin-visualizer`). Zie ook [`agent-context/PROJECT_SSOT.md`](agent-context/PROJECT_SSOT.md) § Key Commands.

**Documentatie** (`docs/`):

```bash
cd docs && bun install && bun dev
```

**Landing** (`landing/`):

```bash
cd landing && bun install && bun run dev
```

Gebruik overal **Bun** zoals in de package-scripts; Node-versie: zie `engines` in `landing/package.json` waar van toepassing.

---

## Git: workspace-root op GitHub (zonder cms/docs/landing)

Deze **bovenliggende map** wordt op GitHub de repository **`calaca-workspace`**. Daarin horen gedeelde zaken: `AGENTS.md`, `ROADMAP.md`, `cms-wensen.md`, `agent-context/`, `.cursor/`, enzovoort.

**`cms/`**, **`docs/`** en **`landing/`** horen **niet** in die root-repo: ze staan in [`.gitignore`](.gitignore) en krijgen elk een **aparte** GitHub-repository. Lokaal kun je ze naast elkaar houden (zoals nu) of als losse clones in die mappen plaatsen.

### Lokale checkout (volledige workspace)

1. Clone **`calaca-workspace`** naar bijvoorbeeld `calacacms/`:

```bash
git clone https://github.com/calacacms/calaca-workspace.git calacacms
cd calacacms
```

2. Clone de drie app-repositories **in** de verwachte mappen (namen moeten `cms`, `docs` en `landing` blijven voor de paden in `AGENTS.md` en agent-regels):

```bash
git clone https://github.com/calacacms/cms.git cms
git clone https://github.com/calacacms/docs.git docs
git clone https://github.com/calacacms/landing.git landing
```

**CMS-repo:** [github.com/calacacms/cms](https://github.com/calacacms/cms) — SSH (standaard): `git@github.com:calacacms/cms.git`. Als je lokaal een **SSH-hostalias** gebruikt (bijv. `github.com-calacacms`), mag `origin` die host blijven gebruiken zolang het pad `calacacms/cms.git` hetzelfde blijft.

Als je al één grote map hebt met alles erin: maak in `cms/`, `docs/` en `landing/` aparte repos (`git init` of `git clone`), en gebruik voor de bovenliggende map alleen de root-repo — dan blijven de app-mappen uit de root-remote door `.gitignore`.
