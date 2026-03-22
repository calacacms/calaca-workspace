# CMS-wensen (CalacaCMS)

Visie en functionele wensen voor het CMS. Dit document beschrijft **wat** we nastreven; de actuele implementatie kan hiervan afwijken. Open punten en onderzoek backlogs staan in [agent-context/research/RESEARCH-open-wensen-cms-wensen.md](agent-context/research/RESEARCH-open-wensen-cms-wensen.md).

### Implementatiestatus (overzicht)

Status ten opzichte van de huidige codebase (`cms/`), handmatig bij te werken wanneer features landen.

| Legenda | Betekenis |
|---------|-----------|
| **Ja** | Wens is in de kern aanwezig en bruikbaar. |
| **Gedeeltelijk** | Deels gerealiseerd, of afwijkend detail (poort, workflow, diepgang). |
| **Nee** | Nog niet gerealiseerd of alleen als idee / onderzoek. |

| § | Onderwerp | Status | Korte toelichting |
|---|-----------|--------|-------------------|
| 1 | Bundling (CSS/JS entry) | **Ja** | Per thema o.a. Vite-entry (`themes/admin`, `themes/site/default`); gecompileerde assets in `web/assets/`. |
| 2 | DRY / modulair | **Ja** | Services, repositories, thema-scheiding; doorontwikkeling blijft nodig zoals bij elk systeem. |
| 2 | Geen barrel files | **Ja** | Geen `index.ts`-aggregators of `export *`-lagen; Vite-entry heet expliciet `admin-entry.ts`; types in o.a. `types/shared-types.ts`. |
| 3 | Admin Tailwind + Alpine | **Ja** | Admin-bundle met Tailwind v4 en Alpine. |
| 4 | Installatie voor niet-technische gebruikers | **Gedeeltelijk** | CLI/installatie-services aanwezig; geen volledig “guided” traject op dat niveau. |
| 4 | Eén commando, watch, vaste dev-URL | **Ja** | `bun run start`: proxy (standaard `localhost:8080`) naar PHP (`127.0.0.1:8000`) en Vite-paden `/__vite/admin` + `/__vite/default`; `VITE_DEV` + `APP_URL` in `.env`. Roadmap **R4**. `.local` niet standaard uit de doos. |
| 4 | Admin onder `/admin`, beveiligd | **Ja** | Routes onder `/admin`, loginflow. |
| 5 | Twig template inheritance | **Ja** | Layouts en blocks in site-thema. |
| 5 | Markering in Twig → auto velden in admin | **Nee** | Schema o.a. via YAML (`site/blueprints/pages`) en PHP (`calaca/types`); geen template-naar-schema extractie. |
| 5 | Minder handwerk template ↔ admin | **Gedeeltelijk** | Blueprints en formulieren koppelen velden; geen automatische sync uit markup. |
| 6 | Hergebruik componenten/site | **Ja** | Sluit aan bij modulaire opzet en thema’s. |
| 7 | Vrijheid publieke frontend-stack | **Ja** | Eigen thema’s en tooling mogelijk. |
| 7 | Presets (Tailwind, Alpine, …) | **Ja** | O.a. default/admin stubs en Vite-configuraties. |
| 7 | Frontend-stack kiezen via UI | **Nee** | Geen menu/wizard; configuratie via projectbestanden. Intentie: [§ 7 — R6](#frontend-stack-kiezen-via-ui-roadmap-r6). Roadmap: [ROADMAP.md](ROADMAP.md) **R6**. |
| 8 | Filosofie (site eerst) | **Ja** | Beschreven in kennisbank; dit document sluit daarop aan. |
| 9 | Keuze flatfile vs database | **Ja** | o.a. `StorageFactory` met meerdere drivers; default flatfile-achtig. |
| 9 | Flatfile als automatische fallback bij DB | **Nee** | Geen vastgelegd automatisch terugvallen; wel driverkeuze per configuratie. |
| 9 | Headless gebruik | **Gedeeltelijk** | JSON onder o.a. `/admin/api/…` (met auth); geen volledig publiek headless-product (API-keys, versioning, enz.). |
| 10 | Centrale plugins / marketplace | **Nee** | Nog geen centrale installatie of marketplace. |
| 11 | Core field suite (ACF Pro-achtig), alles in core | **Gedeeltelijk** | Blueprints en typed fields aanwezig; nog geen volledige first-party parity (o.a. flex layouts, rijke conditionals, herbruikbare veldgroepen-UX zoals ACF). Roadmap: [ROADMAP.md](ROADMAP.md) **R10**. |

---

## 1. Bundling en frontend-entrypoints

- **CSS:** één duidelijk entrybestand waarin alle benodigde stijlen samenkomen, zodat styling overzichtelijk beheerd en efficiënt gecompileerd kan worden.
- **JavaScript / TypeScript:** één entrybestand waarin scripts gebundeld en geoptimaliseerd worden voor gebruik in de applicatie.

---

## 2. Codekwaliteit en architectuur

- **DRY (Don't Repeat Yourself):** code en functionaliteit zo opzetten dat niets onnodig wordt herhaald, zodat onderhoud en uitbreiding eenvoudiger blijven.
- **Modulair:** het systeem moet zich lenen voor onderverdeling in modules. Onderdelen moeten zich laten scheiden, hergebruiken en onderhouden zonder onnodige kruisafhankelijkheden of conflicten.
- **Geen barrel files:** geen `index.ts` die vooral dient om andere modules te her-exporteren; imports wijzen naar concrete bestanden (bv. `admin-entry.ts` als bundel-startpunt, geen map-naar-`index`-facade).

---

## 3. Admin-thema en technologie

- Het **adminpanel** wordt gebouwd met **Tailwind CSS** en **Alpine.js**: een moderne, snelle en responsieve interface die goed uit te breiden en te onderhouden is.

---

## 4. Installatie en lokale ontwikkeling

- **Installatie:** een helder installatieproces, zodat ook mensen met beperkte technische achtergrond het CMS zonder onduidelijke stappen kunnen opzetten.
- **Lokale ontwikkeling:** met **één terminalcommando** de webserver starten, inclusief **watch** voor live verwerking van bestandswijzigingen. De site moet direct bereikbaar zijn, bij voorkeur op een vaste basis-URL zoals `http://localhost:3000` of een lokale hostnaam (bijv. `naam-site.local`), **zonder** dat ontwikkelaars eerst veel moeten configureren.
- **Admin-URL:** inloggen en beheren via hetzelfde basisadres met het pad `/admin` (bijv. `http://localhost:3000/admin` of `https://naam-site.local/admin`), met een **beveiligde** en **bruikbare** interface voor content en instellingen.

---

## 5. Templates, Twig en koppeling met het admin

- **Twig en overerving:** de website-templates gebruiken **template inheritance** in Twig, zodat layouts hiërarchisch en overzichtelijk zijn met herbruikbare basiselementen.
- **Markering van content in templates:** dynamische content wordt in het template **zichtbaar gemarkeerd** (bijvoorbeeld via een Twig-filter of vergelijkbaar mechanisme), zodat duidelijk is welke variabelen en contenttypen waar horen.
- **Automatische aansluiting op het admin:** de backend moet die markeringen **automatisch** kunnen vertalen naar de juiste velden en contenttypen in het adminpanel, **zonder** dat elke koppeling handmatig hoeft te worden ingesteld.
- **Doel:** een efficiënte samenwerking tussen **frontend-templates** en **contentbeheer**, met minder handwerk voor ontwikkelaars en redacteuren en minder kans op inconsistenties.

> **Onderzoek:** dit onderdeel wijkt in de huidige aanpak mogelijk af (schema via blueprints/PHP). Zie [agent-context/research/RESEARCH-template-schema-from-twig.md](agent-context/research/RESEARCH-template-schema-from-twig.md).

---

## 6. Modulariteit en hergebruik (CMS en site)

Componenten en functies moeten **herbruikbaar** zijn in verschillende delen van zowel het CMS als de publieke site, in lijn met de modulaire architectuur hierboven.

---

## 7. Vrijheid in de frontend van de website

- **Keuze bij de bouwer:** hoe de **publieke** website-templates worden opgebouwd, bepaalt de gebruiker van het CMS. Die vrijheid omvat onder meer platte HTML/CSS, JavaScript-frameworks of andere templating-aanpakken.
- **Ondersteunde presets:** het systeem moet **vooraf ingestelde configuraties** kunnen bieden (bijv. Tailwind CSS, Alpine.js en vergelijkbare moderne stacks) om snel te starten.
- **Optionele configuratie-UI:** idealiter kan de gebruiker de frontend-omgeving ook **via een interface** (bijv. een menu of wizard) samenstellen, zodat niet altijd configuratiebestanden handmatig bewerkt hoeven te worden.

### Frontend-stack kiezen via UI (roadmap R6)

Deze wens is uitgewerkt als roadmap-item **R6** ([ROADMAP.md](ROADMAP.md)). Hieronder staat **wat** we daarmee willen bereiken (doelbeeld, geen implementatiespecificatie).

**Doel**

- Een **tweede, optioneel pad** naast het huidige pad (alles via projectbestanden: Vite-config, themamappen, o.a. `calaca init`): de **publieke frontend-stack en thema-bootstrap** ook **via het CMS of een gekoppelde wizard** kunnen kiezen en laten aanmaken — zonder de **vrijheid van de bouwer** (eigen stack, maatwerk) te schrappen.

**Wat dit wél is**

- **Presets zichtbaar maken:** herkenbare keuzes (bijv. Tailwind, Alpine, vergelijkbare stacks) die vandaag al als stubs/config bestaan, worden **expliciet kiesbaar** i.p.v. alleen af te leiden uit de bestandsstructuur.
- **Minder drempel:** voor wie **liever niet** alleen door configuratiebestanden werkt, is er een **menu- of wizardflow** die dezelfde uitkomst triggert als een correcte handmatige setup.
- **Concrete bootstrap:** de UI is geen los scherm — ze moet **project-/thema-opzet** opleveren (vergelijkbaar met wat je via CLI en bestanden nu regelt: juiste entries, thema-skelet, aansluiting op `calaca init` waar dat past).

**Wat dit niet is**

- **Geen verplichting** om alles browser-only te configureren; **bestanden blijven de bron van waarheid** voor wie dat wil.
- **Geen afdwingen** van één framework: maatwerk en afwijkende stacks blijven mogelijk binnen de filosofie “site eerst, CMS volgt” ([§ 8](#8-filosofie)).

**Samenhang**

- Sluit aan op **guided installatie** (roadmap **R5**): onboarding kan op termijn dezelfde “geen onnodig handwerk”-lijn volgen, maar R6 richt zich specifiek op **frontend-tooling en thema-start**, niet op de hele CMS-installatie.
- **Page builder** (roadmap **R9**): een duidelijke stack-/thema-keuze helpt later om visuele bouw en templates consistent te houden; volgorde staat in de roadmap.

---

## 8. Filosofie

Het CMS moet **zo eenvoudig mogelijk** zijn voor de gebruiker en **zo flexibel mogelijk** voor de ontwikkelaar. De **website staat centraal**; het CMS volgt de site en past zich daaraan aan — niet andersom.

Uitwerking in de kennisbank: [agent-context/01-knowledge-base/00-philosophy/calacacms-philosophy.md](agent-context/01-knowledge-base/00-philosophy/calacacms-philosophy.md).

---

## 9. Opslag en headless

- **Opslag:** keuze tussen een **flatfile**-gedreven CMS en een **database**-gedreven variant, met de mogelijkheid om **flatfile als fallback** te gebruiken waar dat past.
- **Headless:** het CMS moet ook **headless** te gebruiken zijn (content en structuur beschikbaar voor andere clients dan de standaard Twig-site), binnen een duidelijk af te grenzen API- of integratiemodel.

---

## 10. Plugins en uitbreidingen

Plugins en add-ons hebben een **centrale, herkenbare plek** voor installatie en beheer. Op termijn kan dat uitmonden in een **marketplace** of vergelijkbaar ecosysteem — te bepalen bij implementatie.

---

## 11. Uitgebreide velden en groepen (vergelijkbaar met ACF Pro)

**Doel:** het CMS moet **out of the box** de flexibiliteit bieden die bouwers kennen van **Advanced Custom Fields (ACF / ACF Pro)** in WordPress: uitbreidbare veldtypen, **herbruikbare veldgroepen**, **repeaters**, **flexibele layouts** (blokken binnen een veld), **voorwaardelijke weergave** van velden, en een duidelijke koppeling naar templates — **zonder** dat een **externe commerciële plugin** nodig is voor gangbare agency- en site-workflows.

- **First-party:** deze mogelijkheden horen in de **kern** van CalacaCMS, zodat je niet afhankelijk bent van een derde app of marketplace voor standaard contentmodellering.
- **Aansluiting op de bestaande stack:** uitbreiding en aanscherping van **blueprints**, **PHP field types** (`calaca/types`) en adminformulieren; geen verplicht ecosysteem van betaalde add-ons voor basisfuncties.
- **Afbakening:** overlap met automatische template → admin-sync ([§ 5](#5-templates-twig-en-koppeling-met-het-admin), roadmap **R3**) en met visuele page building (roadmap **R9**): in ontwerp en RFC’s expliciet scheiden wat “datastructuur + velden” is versus “visuele layout”.

---

## Gerelateerde documenten

| Document | Doel |
|----------|------|
| [ROADMAP.md](ROADMAP.md) | Geplande features (backlog, horizon, changelog); bewaking via `AGENTS.md` en Cursor-regel |
| [RESEARCH-open-wensen-cms-wensen.md](agent-context/research/RESEARCH-open-wensen-cms-wensen.md) | Openstaande wensen t.o.v. de codebase; onderzoeksvragen en prioritering |
| [RESEARCH-template-schema-from-twig.md](agent-context/research/RESEARCH-template-schema-from-twig.md) | Diepgaand onderzoek: template-gedreven schema en admin |
