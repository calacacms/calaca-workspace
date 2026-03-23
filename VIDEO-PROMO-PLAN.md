# CalacaCMS — plan: promotionele productvideo

**Doel:** een professionele video die **vertrouwen** en **enthousiasme** opwekt bij potentiële kopers en adoptanten (developers, agencies, eigenaren van kleine tot middelgrote sites).

**Status:** ideeën en structuur — geen scriptdefinitief. Herzie dit document wanneer de **landing**, **pricing** en **kernfeatures** (zie `ROADMAP.md`) verder zijn uitgekristalliseerd.

---

## 1. Doel en KPI’s (vastleggen vóór opname)

| Vraag | Notities |
|--------|----------|
| **Primaire CTA** | Demo aanvragen, repo/docs bezoeken, wachtlijst, direct kopen — één duidelijke actie per videoversie. |
| **Doelgroep** | Eerst **developers / agencies** (site-first, flat-file, Twig); eventueel tweede cut voor **site-eigenaren** (eenvoud, geen database nodig). |
| **Succes** | Watch-through rate, kliks op CTA, kwalitatieve feedback (“begrijp ik wat het is binnen 60 sec?”). |

---

## 2. Kernboodschappen (afgestemd op product)

Gebruik deze punten consistent; kies er **2–3** als hoofdlijn per video.

1. **Site-first** — Je begint met je HTML/CSS; de CMS-laag past zich aan je site aan, niet omgekeerd.
2. **Flat-file first** — Content in eenvoudige bestanden; **geen database verplicht** om te starten (wel opschaalbaar naar DB-storages waar relevant).
3. **Progressive enhancement** — Start simpel; breid uit wanneer je dat nodig hebt.
4. **Developer-ergonomie** — Bekende stack: PHP 8.3+, Twig, Symfony-componenten, moderne admin (Vite, Tailwind, TypeScript, Alpine).
5. **Controle en eigendom** — Geen vendor lock-in in de “je site is van jou”-zin; past bij self-hosted / file-based workflows.

Vermijd in de eerste video te veel roadmap-toekomst (bijv. marketplace); focus op **wat vandaag werkt** en voelt als “af”.

---

## 3. Welke video(s)? — formaten

| Format | Lengte | Gebruik |
|--------|--------|---------|
| **Hero / trailer** | 30–60 s | Homepage, social ads, conferenties. Emotie + één duidelijke belofte. |
| **Product demo** | 3–6 min | Website “Watch demo”, sales-gesprekken. Schermopname + voice-over. |
| **Feature deep-dive** | 5–15 min per onderwerp | Docs-kanaal, developers die vergelijken. |
| **“Voor agencies” cut** | 60–90 s | Hergebruik van hero-beelden + nadruk op thema’s, blueprints, snelle iteratie. |

**Advies:** begin met **één** sterke hero-trailer + **één** korte demo; later uitbreiden.

---

## 4. Storyboard-ideeën (hero 45–60 sec)

**Opening (0–8 s)**  
- Snelle beelden: rommelige “te veel dashboards” / generieke CMS-UI (abstract, geen concurrerende merknamen).  
- Tekst op scherm: *“Je site. Niet hun template.”* of neutraal: *“CMS zonder je front-end te verliezen.”*

**Probleem (8–18 s)**  
- Kort: klassieke CMS’en dwingen een model op je site; upgrade-stress; database-complexiteit voor kleine projecten.

**Oplossing — CalacaCMS (18–40 s)**  
- Split screen of over-shoulder: echte **Twig/HTML** in editor → zelfde pagina in browser.  
- Knip naar **admin**: duidelijk, rustig paneel (blueprints/content — wat op dat moment representatief is).  
- Zin: *“Flat-file content. Voeg CMS toe waar je het wilt.”*

**Bewijs / vertrouwen (40–52 s)**  
- Logo’s of “PHP 8.3 · Twig · Symfony” als tekstbalk (geen overweldigende tech-stack-rant).  
- Optioneel: korte quote placeholder voor latere testimonial.

**CTA (52–60 s)**  
- Duidelijke knop + URL. *“Bekijk de demo”* / *“Start met de docs”* — afhankelijk van je funnel.

---

## 5. Demo-video — hoofdstukken (suggestie)

1. **Installatie / eerste start** (kort — of “al draaiend” als dat strakker is).  
2. **Een pagina uit je theme** → markeren welke delen “content” worden.  
3. **Blueprint / content type** (als dat de sterkste story is) of **bewerken in admin** → opslaan → front-end refresh.  
4. **Thema / site vs admin** — waar developers aanpassen.  
5. **Optioneel:** één opslagpad (flat files) tonen — transparantie zonder te technisch te worden.  
6. **Afsluiting:** waar documentatie en community/support te vinden zijn.

---

## 6. Tone en stijl

- **Visueel:** rustig, veel witruimte, strakke typografie; donkere editor-scènes mogen, maar admin en site in **helder contrast** zodat het “professioneel product” uitstraalt.  
- **Audio:** lichte, moderne bed (geen cliché “corporate whoosh”-overkill); voice-over **duidelijk en rustig** (NL of EN afhankelijk van primaire markt — landing is EN; overweeg **twee voice-overs** of ondertiteling).  
- **Tempo:** iets langzamer dan je denkt — kijkers moeten UI-lezen kunnen.

---

## 7. Productie-checklist (praktisch)

- [ ] **Script** + **shot list** (per scene: scherm / B-roll / tekst overlay).  
- [ ] **Demo-omgeving** met vaste seed-data (geen persoonlijke paden, geen secrets in beeld).  
- [ ] **Schermopname:** 4K of 1080p, consistente schaal, grote cursor of duidelijke focus.  
- [ ] **Muziek:** royalty-free licentie vastleggen.  
- [ ] **Toegankelijkheid:** ondertitels (SRT) voor alle publieke versies.  
- [ ] **Versies:** 16:9 (YouTube/site), 9:16 of 1:1 (social cuts).  
- [ ] **Legal:** geen logo’s van derden zonder toestemming; gebruik eigen voorbeeldsite of generieke mock.

---

## 8. Afstemming met roadmap (wanneer filmen?)

Opname is het overtuigendst wanneer de **zichtbare** admin- en site-flow strak zijn. Houd rekening met `ROADMAP.md`: grote UI-wijzigingen of geplande onboarding (bijv. R5) kunnen een video **dated** maken — plan een **update-cyclus** (bijv. jaarlijks of per major release).

---

## 9. Open punten om intern te beslissen

1. **Primair taalgebied** voor eerste release (NL vs EN vs beide).  
2. **Pricing / licentie** op het moment van publicatie — CTA moet juridisch en commercieel kloppen.  
3. **Hosting** van de video (YouTube niet-ingelogd, Vimeo, eigen CDN).  
4. **Persona:** alleen developer-first, of ook een “marketing manager”-hoek?

---

## 10. Volgende stap

1. Kiezen: hero **of** demo als eerste deliverable.  
2. Eén pagina **storyboard** (tekst + ruwe schetsen) laten goedkeuren.  
3. Script schrijven op **woordniveau** (± 130 woorden per minuut voice-over).  
4. Demo-data en omgeving klaarzetten, dan opnemen.

---

*Aangemaakt als werkdocument voor CalacaCMS-marketing. Bij productwijzigingen: kernboodschappen en demo-hoofdstukken herschikken.*
