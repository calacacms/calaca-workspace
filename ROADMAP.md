# CalacaCMS — product-roadmap

Geplande features en thema’s **na** de huidige basisimplementatie. Dit is een **levend document**: prioriteiten verschuiven; werk het bij wanneer iets wordt opgepakt of afgerond.

**Gerelateerd**

- Wensen en status t.o.v. code: [`cms-wensen.md`](cms-wensen.md)
- Open punten en onderzoeksvragen: [`agent-context/research/RESEARCH-open-wensen-cms-wensen.md`](agent-context/research/RESEARCH-open-wensen-cms-wensen.md)
- Template ↔ admin (diepgaand onderzoek): [`agent-context/research/RESEARCH-template-schema-from-twig.md`](agent-context/research/RESEARCH-template-schema-from-twig.md)

---

## Tijdshorizon (suggestie)

| Horizon | Bedoeling |
|---------|-----------|
| **Nu** | Huidige release: bugs, docs, kleine verbeteringen (niet hieronder als grote feature). |
| **Next** | Eerstvolgende focus: 1–2 grote stukken die het meeste opleveren. |
| **Later** | Duidelijke meerwaarde, maar meer ontwerp of afhankelijkheden. |
| **Onderzoek** | Eerst besluit/RFC; pas daarna inschatten in roadmap. |

---

## Feature-backlog

Status: **Idee** | **Onderzoek** | **Gepland** | **In progress** | **Gedaan** (verplaats naar changelog-onderdeel of verwijder uit actieve lijst).

**Rijvolgorde** = aanbevolen **implementatievolgorde** (afhankelijkheden en lagen: contentmodel → kwaliteitsborging → onboarding → template-automatisering → presentatie-keuzes → visuele bouw → publieke API → ecosystem → dev-ergonomie). **ID’s** blijven stabiel ter referentie.

> **Context7** ([Strapi-documentatie](https://github.com/strapi/documentation), o.a. content-type/schema als basis, REST/GraphQL als laag erboven, plugins die de content-manager uitbreiden): zelfde idee — **schema en kern-admin eerst**, **publieke API** en **plugin-uitbreidingen** pas op een stabiel contentmodel.

| ID | Feature | Horizon | Status | Koppeling `cms-wensen.md` | Notities |
|----|---------|---------|--------|----------------------------|----------|
| R10 | **Core field suite (ACF Pro-achtig)**: uitbreidbare veldtypen, herbruikbare groepen, repeaters, flexibele layouts, voorwaardelijke velden — **first-party in core**, geen afhankelijkheid van derde partij voor standaard redactie-/bouw-workflows | Next | Onderzoek | § 11 | **Eerste stap:** canoniek veld- en blueprint-model; overlapt deels met R3/R9 — die volgen hierna. |
| R8 | **CI: blueprint ↔ template-validatie** (geen schema uit Twig, wel “geen mismatch”) | Next | Idee | § 5 | **Tweede stap:** borgt consistentie terwijl R10 en templates evolueren; blijft nuttig naast R3. |
| R5 | **Guided installatie** (wizard of heldere first-run voor niet-technische gebruikers) | Next | Idee | § 4 | **Derde stap:** onboarding die aansluit op het stabielere kernmodel (na R10/R8). |
| R3 | **Template-gedreven velden / sync met admin** | Onderzoek | Onderzoek | § 5 | **Vierde stap:** automatisering op basis van vast blueprint/template-spoor; zie research-doc. |
| R6 | **Frontend-stack kiezen via UI** (menu/wizard i.p.v. alleen bestanden) | Later | Idee | § 7 | **Vijfde stap:** project-/thema-bootstrap vóór zware visuele builders; koppeling `calaca init`. **Uitwerking bedoeling:** [cms-wensen.md § 7 — R6](cms-wensen.md#frontend-stack-kiezen-via-ui-roadmap-r6). |
| R9 | **Page builder (drag-and-drop)**: visuele pagina-opbouw met blokken/componenten in de admin | Later | Idee | — | **Zesde stap:** profiteert van R10-velden en heldere stack-keuze (R6); a11y; overlap met R3 afbakenen. |
| R2 | **Headless-product**: publieke content-API (auth, versioning, CORS) | Later | Onderzoek | § 9 | **Zevende stap:** publieke API nadat contentmodel en publicatieflow helder zijn. |
| R1 | **Plugin-ecosysteem**: centrale installatie, registry of marketplace | Later | Idee | § 10 | **Achtste stap:** ecosystem wanneer kern-API’s en extensiepunten stabiel zijn (o.a. na R10/R2). |
| R4 | **Eén dev-URL / proxy** (bijv. alles via één host:poort i.p.v. 8000 + Vite) | Later | Gedaan | § 4 | `bun run start`: proxy op `DEV_PROXY_PORT` (default 8080); PHP op `127.0.0.1:8000`; Vite onder `/__vite/admin` en `/__vite/default`. Zie `cms/AGENTS.md` en `cms/.env.example`. |

Voeg nieuwe rijen toe met het volgende vrije ID (`R11`, …).

---

## Changelog (roadmap)

Kort vastleggen wanneer een item de status **Gedaan** bereikt of wordt ingetrokken.

| Datum | Item | Gebeurtenis |
|-------|------|-------------|
| 2026-03-22 | R4 | Geïmplementeerd: dev reverse proxy + Twig/Vite `base` paden; ene publieke URL voor PHP + HMR. |

---

## Bewaking van deze roadmap (rol “roadmap-bewaker”)

Er is **geen losse software-“projectmanager”** in de repo; wél een **vaste werkwijze** zodat niemand (en geen agent) de roadmap vergeet.

### Mens (eigenaar)

- Wijs **één persoon** aan als eigenaar van `ROADMAP.md` (product owner, tech lead, of jijzelf).
- **Minimaal eens per twee weken:** statuskolom nalopen, “Next” herijken, afgeronde items naar changelog.
- Bij start van een grotere feature: issue of ticket koppelen (GitHub/GitLab/Jira) in de notitieskolom — optioneel maar handig.

### AI-agents (Cursor e.d.)

- Staat beschreven in [`.cursor/rules/calacacms-roadmap.mdc`](.cursor/rules/calacacms-roadmap.mdc) en in [root `AGENTS.md`](AGENTS.md): bij **plannen**, **prioriteren** of **afronden** van CMS-features eerst deze roadmap en `cms-wensen.md` raadplegen; na oplevering **roadmap + implementatiestatus** bijwerken.

### Waarom zo?

Zo blijft “Centrale plugins / marketplace” en andere grote wensen **zichtbaar en traceerbaar**, zonder dat alles alleen in losse chats staat.
