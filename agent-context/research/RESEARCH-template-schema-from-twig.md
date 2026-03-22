# Onderzoek: schema en admin uit Twig-templates (template-gedreven velden)

**Status:** gepland onderzoek — nog geen besluit of implementatie  
**Aanleiding:** verschil tussen `cms-wensen.md` (Twig-filter markeert velden → backend vult admin) en de huidige CalacaCMS-aanpak (expliciete definities via YAML + PHP).  
**Gerelateerd:** [CalacaCMS-filosofie](../01-knowledge-base/00-philosophy/calacacms-philosophy.md) (“site first, CMS second”), [Twig best practices](../01-knowledge-base/02-twig-templates/twig-best-practices.md), [storage drivers](../technical/storage-drivers.md).

---

## 1. Doel van dit document

Dit document bundelt **vragen, opties en onderzoeksstappen** om later te bepalen of en hoe CalacaCMS **template-gedreven veldmarkering** moet ondersteunen — als **aanvulling** op (niet noodzakelijk als vervanging van) `site/blueprints/pages/*.yml` en `calaca/types/*.php` (`ContentTypeManager`).

**Succescriteria voor het onderzoek (later invullen):**

- Duidelijke aanbeveling: *niet doen / alleen validatie / hybride / beperkte auto-discover*.
- Lijst met **risico’s en mitigaties** die het team accepteert.
- Indien “ja”: schets van **syntax**, **merge-regels**, **moment van extractie** (build vs runtime) en **migratiepad** voor bestaande sites.

---

## 2. Huidige situatie (korte feiten)

| Aspect | Nu in CalacaCMS |
|--------|------------------|
| Schema / contenttypen | YAML-blueprints onder `site/blueprints/pages/`, overschrijfbaar met PHP in `calaca/types/` |
| Twig-extensie (`CmsExtension`) | Helpers: o.a. `slugify`, `excerpt`, `asset`, `url` — **geen** veld-markering die schema voedt |
| Filosofische spanning | Wensen beschrijven “markup als signaal naar admin”; codebase kiest **expliciete** definities (vergelijkbaar met o.a. Kirby/Statamic-richting) |

---

## 3. Probleemstelling

**Kernvraag:** Levert het toevoegen van “Twig (of markup) markeert velden → admin/schema wordt daaruit afgeleid” **meetbare** voordelen voor CalacaCMS-gebruikers, zonder **voorspelbaarheid, versioning en complexe sites** te ondermijnen?

**Secundaire vragen:**

- Voor **welke** doelgroep (alleen kleine sites, alleen prototypes, ook enterprise-achtige sites)?
- Is de echte pijn “**dubbel onderhoud** template vs blueprint”, of iets anders (bijv. page builder, blokken, meertaligheid)?
- Hoe verhoudt dit zich tot **headless** (meerdere consumers van dezelfde content)?

---

## 4. Conceptuele modellen (te vergelijken)

### 4.1 Model A — Schema uitsluitend uit templates

- Extractie uit `.twig` (regex/AST/static analysis) of uit speciale tags/filters.
- Blueprint/YAML wordt optioneel of gegenereerd.

**Te onderzoeken:** volledigheid, conflicten bij `extends`/`include`/`embed`, conditionele velden, loops.

### 4.2 Model B — Blueprint blijft bron van waarheid; Twig alleen ergonomie

- Filters/functies zoals `cms_field('body')` **renderen** content; veld `body` **moet** in blueprint bestaan.
- Geen auto-schema; wél één duidelijke API in templates.

**Te onderzoeken:** DX-winst vs. huidige `{{ content.body }}`-achtige patronen.

### 4.3 Model C — Hybride merge

- Blueprint definieert validatie, labels, geavanceerde opties.
- Template-scan voegt **ontbrekende** velden toe of waarschuwt bij mismatch.

**Te onderzoeken:** merge-prioriteit (PHP > YAML > scan), conflict-resolutie, migraties.

### 4.4 Model D — Validatie / sync (geen auto-schema)

- CLI of CI: “alle gemarkeerde velden in template X komen voor in blueprint Y”.
- Fouten falen de build of tonen een rapport.

**Te onderzoeken:** implementatie-kosten vs. waarde; false positives bij dynamische includes.

### 4.5 Model E — Gegenereerde stub-YAML (mens commit)

- Scan op `calaca sync` / deploy genereert of update een **voorstel**-blueprint; developer reviewt in git.

**Te onderzoeken:** workflow, noise bij grote templates, hoe vaak handmatige correctie nodig is.

---

## 5. Referentiesystemen (extern onderzoek)

| Systeem | Relevant mechanisme | Lessen voor onderzoek |
|---------|---------------------|------------------------|
| CouchCMS | `<cms:editable />` e.d. in templates | Waar “template = regio’s” sterk is; waar toch configuratie nodig is |
| Kirby | Blueprints / YAML, los van views | Expliciet model dat goed schaalt |
| Statamic | Blueprints, Antlers/Blade | Scheiding content model ↔ presentatie |
| WordPress + ACF | Velden in code of UI | Geen template als enige bron; wel bekende editor-flow |

**Actie (later):** per systeem 1–2 pagina’s notities: *hoe definieer je herhaalbare regio’s, relaties, validatie?*

---

## 6. Technische onderzoekspunten (CalacaCMS-specifiek)

1. **Moment van extractie**
   - Build-time (CLI): reproduceerbaar, past bij git-review.
   - Runtime (eerste request): eenvoudiger voor gebruikers, moeilijker voor caching en security.

2. **Template-graaf**
   - Hoe koppel je een **contenttype** aan een **set templates** (bijv. `page` → `pages/default.twig` + partials)?
   - Hoe ga je om met **gedeelde partials** die voor meerdere types gebruikt worden?

3. **Syntax**
   - Filter vs. Twig-function vs. custom tag (indien ooit ondersteund).
   - Hoe encodeer je **type** (`text`, `richtext`, `image`, `relation`), **required**, **max**, **groepen**?

4. **Integratie met `ContentTypeManager`**
   - Uitbreiding van laadvolgorde: bijv. blueprints → PHP → template-scan (alleen als vlag aan staat).
   - API voor plugins: kunnen derden een extractor registreren?

5. **Conflict met filosofie “save = live”**
   - Als schema uit scan komt: wanneer is admin “bij”? Invalidatie van cache, migraties van opslag.

6. **Storage**
   - Flatfile vs database: heeft schema-wijziging **migratie** van bestaande JSON/SQL nodig? Hoe automatiseren?

---

## 7. Empirisch onderzoek (eigen projecten)

**Te doen op 3–5 echte of representatieve sites:**

- Frequentie van schema-wijzigingen vs. template-wijzigingen.
- Aantal velden per type; hoeveel **conditioneel** of **herhaalbaar**.
- Of editors vooral **één template per type** hebben of veel varianten.

**Output:** korte case-tabellen + conclusie “template-first zou hier X% dubbel werk besparen” (schatting).

---

## 8. Risico’s (checklist)

- [ ] Schema-drift tussen omgevingen als extractie niet deterministisch is.
- [ ] **Security:** runtime schema-wijziging op basis van requests.
- [ ] Onderhoudslast van een **Twig-parser** of fragiele regex op complexe templates.
- [ ] Verwarring voor gebruikers: twee manieren om hetzelfde te definiëren.
- [ ] Headless / tweede front-end: welke template is leidend voor het model?

---

## 9. Mogelijke uitkomsten (vooraf geen keuze)

| Uitkomst | Korte omschrijving |
|----------|-------------------|
| **Niet bouwen** | Huidige blueprints + documentatie; eventueel betere voorbeelden. |
| **Alleen DX** | Twig-helpers gekoppeld aan bestaande velden; geen extractie. |
| **Alleen validatie** | CLI/CI template ↔ blueprint. |
| **Hybride** | Merge-regels + optionele stub-generatie. |
| **Volledig template-first** | Alleen als onderzoek aantoont dat het binnen afgebakende use cases blijft. |

---

## 10. Open vragen (invullen tijdens onderzoek)

1. Moet het systeem **zonder build-stap** blijven werken voor eindgebruikers (cf. filosofie)?
2. Is **page builder / blokken** een betere investering dan veld-extractie uit lineaire templates?
3. Hoe past dit in een toekomstige **headless API** (één canoniek model onafhankelijk van Twig)?

---

## 11. Volgende stappen (wanneer dit onderzoek start)

1. Vastleggen **eigenaar** en **deadline** van het onderzoek.
2. Invullen sectie 7 (empirisch) en sectie 5 (extern).
3. Een **1-pagina besluit** (ADR-achtig) met aanbeveling + “niet gedaan omdat …”.
4. Indien positief: koppelen aan een implementatie-RFC (syntax, CLI-commando’s, teststrategie).

---

## 12. Changelog

| Datum | Wijziging |
|-------|-----------|
| 2026-03-22 | Initieel onderzoeksdocument aangemaakt (context uit teamdiscussie + huidige architectuur). |
