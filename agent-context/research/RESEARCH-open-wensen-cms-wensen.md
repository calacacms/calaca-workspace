# Openstaande wensen uit `cms-wensen.md` — onderzoek backlog

**Status:** backlog voor later onderzoek en prioritering  
**Bron:** [cms-wensen.md](../../../cms-wensen.md) (repository root)  
**Roadmap (planning & prioriteit):** [ROADMAP.md](../../../ROADMAP.md)  
**Laatst afgestemd op codebase:** 2026-03-22  

Dit document bevat **alleen** wensen die **nog niet** of **duidelijk incompleet** zijn ten opzichte van de huidige CalacaCMS-implementatie (`cms/`). Voor het diepgaande onderzoek naar **template-gedreven velden** zie [RESEARCH-template-schema-from-twig.md](./RESEARCH-template-schema-from-twig.md).

---

## Samenvatting (wat wél grotendeels aanwezig is)

Ter context — **niet** onderdeel van dit backlog: één CSS/JS-entry per thema (Vite), Twig inheritance, admin op `/admin` met Tailwind en Alpine, modulaire PHP-kern, opslagkeuze (flatfile / DB-drivers), basis `bun run start`-achtige dev-workflow (PHP + Vite watch). Zie eerdere projectvergelijking met `cms-wensen.md` voor details.

---

## 1. Installatie en onboarding (niet-technische gebruikers)

**Wens (parafrase):** Duidelijk installatieproces; nieuwe gebruikers zonder sterke technische achtergrond moeten eenvoudig kunnen starten.

**Huidige situatie:** Er is CLI/installatie-gerelateerde code (`InstallationService`, `FileInstallerService`, enz.), maar geen aantoonbaar **eind-tot-eind** “guided setup” op het niveau dat de wens beschrijft.

**Onderzoeksvragen:**

- Wie is de primaire “niet-technische” gebruiker (site-eigenaar, redacteur, agency-klant)?
- Welke stappen moeten **geautomatiseerd** vs. **gedocumenteerd** blijven?
- Hoe verhoudt dit tot hosting (shared hosting, Docker, alleen lokaal)?

**Mogelijke uitkomsten:** wizard in admin, één pagina quickstart, video/checklist, gehoste demo, of bewust “developers only” met duidelijke positionering.

---

## 2. Dev-URL en één-commando-ervaring (poort / `.local`)

**Wens:** Eén terminalcommando; site op `http://localhost:3000` of vergelijkbaar zoals `naam-site.local`; watch live; minimale extra configuratie.

**Huidige situatie:** O.a. `bun run start` start PHP op **localhost:8000** en Vite op vaste poorten (**5173** admin, **5174** default). Dat wijkt af van poort **3000** en van een enkele URL voor alles.

**Onderzoeksvragen:**

- Is **3000** een harde eis (conventie Next/React) of “willekeurige voorbeeld-URL”?
- Wil het team **één proxy** (bijv. Caddy/Vite middleware) zodat alles onder één host/poort lijkt?
- Hoe moet `naam-site.local` werken (documentatie-only vs. script dat `/etc/hosts` of dnsmasq voorstelt)?

**Mogelijke uitkomsten:** proxy + unified dev URL; documenteren van huidige poorten; optioneel `.env` `DEV_PORT`.

---

## 3. Twig-filter markeert velden → backend vult admin automatisch

**Wens:** Dynamische content via Twig-filter markeren; backend pikt dit op en mapt naar admin-velden zonder handmatige mapping.

**Huidige situatie:** Schema via YAML (`site/blueprints/pages`) en PHP (`calaca/types`); `CmsExtension` levert geen schema-koppelde veld-markering.

**Verdieping:** [RESEARCH-template-schema-from-twig.md](./RESEARCH-template-schema-from-twig.md)

---

## 4. “Slim systeem” template ↔ contentbeheer (minder handwerk, minder fouten)

**Wens:** Efficiënte samenwerking tussen frontend-templates en backend; minder handmatige instellingen; fouten minimaliseren.

**Huidige situatie:** Deels gedekt door **expliciete** blueprints en `FormBuilder`-achtige stromen; **geen** automatische synchronisatie op basis van templates.

**Onderzoeksvragen:**

- Lost **sectie 3** dit op, of is het echte probleem iets anders (bijv. validatie op deploy, preview, blokken)?
- Welke fouten zien gebruikers nu in de praktijk (missing fields, verkeerde types)?

**Koppeling:** overlapt met template-onderzoek en met eventuele **CI-validatie** blueprint ↔ template.

---

## 5. Frontend-stack kiezen via UI (“menu”) zonder config-bestanden

**Wens:** Ondersteuning met vooraf ingestelde stacks (Tailwind, Alpine, …) **of** samenstellen via een menu; geen handmatig bewerken van configbestanden nodig.

**Huidige situatie:** Voorbeeldthema’s en stubs met Vite/Tailwind; **geen** admin- of CLI-“menu” dat de frontend-tooling van een site wisselt.

**Onderzoeksvragen:**

- Is dit **product** (wizard bij `calaca init`) of **runtime** (instelbaar in admin)?
- Hoe voorkom je gebroken builds bij wissel van stack?
- Hoe verhoudt dit tot “volledige vrijheid” (wens: gebruiker bepaalt tools)?

---

## 6. Filosofie-document (link in wensen)

**Status:** In `cms-wensen.md` verwijst sectie 8 nu naar [calacacms-philosophy.md](../01-knowledge-base/00-philosophy/calacacms-philosophy.md). Eerdere verwijzing naar `chaincms-philosophy.md` is daarmee rechtgezet.

**Eventueel onderzoek:** inhoudelijke afstemming tussen `cms-wensen.md` en de filosofiepagina (één bron van waarheid voor formuleringen).

---

## 7. Database vs flatfile mét flatfile als fallback

**Wens:** Keuze tussen flatfile en database-gedreven CMS; **flatfile als fallback**.

**Huidige situatie:** `StorageFactory` biedt meerdere drivers; default is flatfile-achtig. **Automatische fallback** (bijv. bij DB-storing) is niet als duidelijk productfeature gedocumenteerd/geverifieerd.

**Onderzoeksvragen:**

- Bedoelen jullie **runtime-fallback** (DB weg → lees JSON) of **installatiekeuze** met flatfile als “safe default”?
- Hoe ziet **datasynchronisatie** eruit bij wissel of fallback?
- Wat zijn acceptatiecriteria voor data-integriteit?

**Gerelateerde doc:** [storage drivers](../technical/storage-drivers.md) (controleren en aanvullen na onderzoek).

---

## 8. Headless CMS

**Wens:** Mogelijkheid om het CMS **headless** te gebruiken.

**Huidige situatie:** JSON-endpoints onder `/admin/api/...` (met auth), gericht op admin/dashboard-gebruik — **geen** volledig uitgewerkt publiek headless-model (API-keys, versioning, CORS voor publieke apps, content API los van admin).

**Onderzoeksvragen:**

- Doelgroep: **eigen SPA** / mobiele app / externe integraties?
- Authenticatie: sessies, JWT, API keys?
- Read vs write; webhooks; GraphQL vs REST?
- Relatie met “site first” (Twig-site blijft canoniek of niet)?

---

## 9. Plugins, add-ons, centrale installatie, marketplace

**Wens:** Centrale plek om plugins/add-ons te installeren; eventueel marketplace.

**Huidige situatie:** Geen marketplace; plugin-architectuur eventueel in losse modules/README’s — **geen** eenduidig “Calaca package registry”-verhaal in product.

**Onderzoeksvragen:**

- Composer-only vs. ingebouwde UI vs. curated directory?
- Security model (ondertekening, reviews)?
- Compatibiliteit met core-versies en migraties.

---

## Prioriteringsmatrix (invullen tijdens planning)

| # | Thema | Impact (geschat) | Complexiteit (geschat) | Notities |
|---|--------|------------------|-------------------------|----------|
| 1 | Installatie niet-tech | | | |
| 2 | Dev URL / één poort | | | |
| 3 | Template → schema | | | Zie apart onderzoeksdocument |
| 4 | Slim template–CMS | | | Hangt samen met 3 |
| 5 | Frontend via menu | | | |
| 6 | Filosofie-link | | | Snelle doc-fix |
| 7 | DB + flatfile fallback | | | Data / opslag |
| 8 | Headless | | | API-product |
| 9 | Plugins / marketplace | | | Ecosysteem |

---

## Volgende stappen

1. Product/engineering: **prioriteiten** invullen in de matrix.  
2. Per rij: eigen **ADR of mini-RFC** of opname in bestaande planningdocs.  
3. `cms-wensen.md` desgewenst opsplitsen in “visie” vs. “backlog” met links naar dit bestand en naar [RESEARCH-template-schema-from-twig.md](./RESEARCH-template-schema-from-twig.md).

---

## Changelog

| Datum | Wijziging |
|-------|-----------|
| 2026-03-22 | Eerste versie: openstaande wensen uit `cms-wensen.md` als onderzoek backlog. |
