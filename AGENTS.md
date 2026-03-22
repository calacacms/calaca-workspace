# AGENTS.md (workspace root)

**Workspace GitHub repository:** [`calacacms/calaca-workspace`](https://github.com/calacacms/calaca-workspace) (zie [`README.md`](README.md) § Git).

**Authoritative agent instructions for the CMS live in [`cms/AGENTS.md`](cms/AGENTS.md)** — stack rules, commands, architecture, Context7 hints, and the **Learned User Preferences** / **Learned Workspace Facts** sections.

Use this root file only as the entry point Cursor or other tools expect at the monorepo root; edit behavior and project memory in `cms/AGENTS.md`.

**Documentation language:** root-level Markdown (zoals roadmap en wensen) mag **Nederlands**; alle documentatie en **marketingcopy** in `cms/`, `docs/`, `landing/` en `agent-context/` moet **Engels** — zie [`.cursor/rules/calacacms-documentation-language.mdc`](.cursor/rules/calacacms-documentation-language.mdc) en `cms/AGENTS.md`.

For cross-project context (docs site, landing), see [`README.md`](README.md) and [`agent-context/PROJECT_SSOT.md`](agent-context/PROJECT_SSOT.md).

**Astro (landing + docs):** Cursor uses [`.cursor/rules/calacacms-astro-landing-docs.mdc`](.cursor/rules/calacacms-astro-landing-docs.mdc) when editing files under `landing/` and `docs/`.

## Roadmap en wensen (bewaking)

- **Product-roadmap** (geplande features, o.a. plugins/marketplace): [`ROADMAP.md`](ROADMAP.md)
- **Wensen + implementatiestatus**: [`cms-wensen.md`](cms-wensen.md)

Bij **plannen**, **prioriteren** of **afronden** van grotere CMS-werkzaamheden: roadmap en wensen raadplegen; na oplevering `ROADMAP.md` (status + changelog) en waar nodig de status-tabel in `cms-wensen.md` bijwerken. Cursor past dit ook toe via [`.cursor/rules/calacacms-roadmap.mdc`](.cursor/rules/calacacms-roadmap.mdc) wanneer je in `cms/` of de genoemde bestanden werkt.
