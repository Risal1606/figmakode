Kort refleksion
===============

Efter feedbacken har jeg gået koden igennem igen for at rette de konkrete ting, der blev peget på: at subgrid ikke lå i noget, der faktisk blev vist på siden, at CSS'en var spredt ud på en måde, der ikke passede med det, jeg skrev i refleksionen, og at flere af de komponenter, der rent faktisk bliver brugt, stadig havde hardcodede farver og afstande. Nedenfor går jeg konkret igennem, hvad der er ændret, og hvorfor.

Subgrid i makrolayoutet
------------------------
Det var her, den forrige aflevering fejlede: subgrid-koden lå i en komponent (`ValueCard.astro`) og et CSS-udsnit i `about.css`, som aldrig blev importeret eller renderet nogen steder. Jeg har fjernet den døde kode og lagt subgrid ind i noget, der faktisk vises — Case Study-artiklen (`src/styles/case.css`):

```css
.case-study {
  display: grid;
  grid-template-columns:
    minmax(var(--space-lg), 1fr)
    minmax(0, 860px)
    minmax(var(--space-lg), 1fr);
}

.case-study__sections {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: subgrid;
}

.case-study__section {
  grid-column: 2;
}

.case-study__section-img {
  grid-column: 1 / -1; /* bryder ud af tekstkolonnen og fylder hele bredden */
  justify-self: center;
}
```

Artiklen sætter et 3-kolonne-makrolayout op (venstre luft / indhold / højre luft). `.case-study__sections` subgridder ind i de samme kolonner som resten af artiklen, så hver sektion som udgangspunkt ligger i den samme smalle tekstkolonne som overskriften — men billedet i sektionen (`.case-study__section-img`) kan bryde ud og fylde hele bredden, fordi det stadig sidder på samme grid som resten af artiklen. Det er den klassiske brug af subgrid: at et enkelt element kan bryde layoutet uden at man skal bygge et helt nyt gridsystem til det. Jeg har testet det i devtools og bekræftet at `grid-template-columns` rent faktisk beregnes som `subgrid` på elementet, og at det opfører sig korrekt helt ned til mobilbredde uden at noget skærer over kanten.

Tokens og hardcodede værdier i de komponenter, der faktisk bruges
------------------------------------------------------------------
Tidligere havde jeg tokenificeret nogle af de globale CSS-filer, men ikke tjekket om det egentlig var dem, der blev brugt på siden. Det var en fejl — flere steder lå den rigtige styling i stedet i komponentens egen `<style>`-blok (Astros scoped styles), som stadig havde hardcodede farver og mål. Det gælder blandt andet `TeamGrid.astro`, `TeamHero.astro`, `ExperienceSection.astro` og selve Team Single-siden (`src/pages/team/[id].astro`), som har sin egen `<style>`-blok direkte i sidefilen.

Eksempel fra `tokens.css`, hvor jeg har udvidet tokens med farver, der gik igen flere steder på tværs af filer (fx `#111827` og `#f0c24b`), i stedet for at lade dem stå som rå hex-koder hver gang:

```css
:root {
  --color-midnight: #111827;
  --color-gold-deep: #f0c24b;
  --color-ink: #111111;
}
```

Jeg tjekkede også `--step-*`-skalaen (den flydende typografi-skala i `tokens.css`) og fandt ud af, at den reelt aldrig blev brugt nogen steder på siden — den eneste reference lå i den døde `.hero__title`-regel i `global.css`, som blev overskrevet af `overrides.css` alligevel. Jeg har rettet det ved at lade `.case-study__section-title` i `case.css` bruge `var(--step-2)` i stedet for en fast `24px`, så overskrifterne i Case Study-artiklen nu skalerer flydende med skærmbredden (fra 24px op til 32px) i stedet for at være låst til én størrelse.

Under den her gennemgang opdagede jeg også noget, jeg ikke havde set før: `global.css` og `overrides.css` definerede begge `.hero`, `.button--primary`, `.services` og `.service-card` — men med helt forskellige værdier. Fordi `overrides.css` importeres sidst i `Layout.astro`, er det reelt dens version, der bliver vist, mens hele blokken i `global.css` bare lå der som død, overskrevet CSS. Jeg har fjernet duplikaterne fra `global.css` (og flyttet de par egenskaber, der ikke fandtes i `overrides.css`, som `flex-wrap: wrap` på `.hero__actions`, ind i `overrides.css` først), og tokenificeret `overrides.css`, fordi det er den fil, der reelt styrer forsidens hero- og services-sektion. Jeg har testet i browseren før og efter med et snapshot af de beregnede stilarter for at være sikker på, at intet ændrede sig visuelt.

og i `TeamGrid.astro`:

```css
.team-grid-heading {
  color: var(--color-ink);
  margin: 0 0 var(--space-sm);
}
```

Jeg har bevidst ikke tokenificeret absolut alt — nogle farver og mål bruges kun ét sted i hele projektet (fx nogle af accentfarverne på Contact-sektionens ikoner), og der giver det ikke mening at lave en variabel, bare fordi man kan. Reglen jeg har fulgt er: hvis en værdi går igen flere steder, bliver den en token; hvis den er unik for ét sted, står den som den er.

Container queries
------------------
`.financial__inner` i `global.css` er sat op med `container-type: inline-size`, og selve grid-layoutet for finansielle-korts-sektionen reagerer på sin egen bredde, ikke viewportets:

```css
.financial__inner {
  container-type: inline-size;
  container-name: financial-section;
}

@container financial-section (max-width: 62rem) {
  .financial__grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

Det er brugbart, fordi sektionen ville kunne genbruges i en smallere kontekst (fx en sidebar) og stadig layoute rigtigt, i modsætning til en almindelig media query, som kun kender til hele skærmens bredde.

Relative Color Syntax
-----------------------
Hover- og active-states på "Join our team"-knappen (`TeamGrid.astro`) er udregnet direkte ud fra brand-farven i stedet for en ny hardcodet farve:

```css
.team-grid-btn:hover {
  background: oklch(from var(--color-action) calc(l - 0.08) c h);
}

.team-grid-btn:active {
  background: oklch(from var(--color-action) calc(l - 0.14) c h);
}
```

Fordelen er, at hvis brand-farven (`--color-action`) ændres, følger hover- og active-farverne automatisk med, uden at man skal huske at opdatere dem separat.

`--flow-space` på artiklen
----------------------------
Case Study-artiklens sektioner bruger `--flow-space` til at style afstanden mellem overskrift, tekst og liste i stedet for at sætte margin på hvert enkelt element:

```css
.case-study__section {
  --flow-space: var(--space-md);
}

.case-study__section > * + * {
  margin-top: var(--flow-space);
}

.case-study__text + .case-study__text {
  --flow-space: 0.85em;
}
```

Det giver en konsekvent lodret rytme, og fordi `--flow-space` kan overskrives lokalt (som det sker mellem to på hinanden følgende afsnit), kan man justere afstanden ét sted uden at skulle ændre margin på hvert element for sig.

Anchor Positioning med fallback
---------------------------------
Login-panelet har en almindelig fallback med `position: absolute`, og bliver derefter forbedret med anchor positioning i browsere, der understøtter det:

```css
.login-panel {
  position: absolute;
  top: calc(100% + 1rem);
  right: 0;
}

@supports (anchor-name: --test) and (position-anchor: --test) {
  .site-header__login {
    anchor-name: --login-trigger;
  }

  .login-panel {
    top: auto;
    right: auto;
    position-anchor: --login-trigger;
    inset-block-start: calc(anchor(bottom) + 1rem);
    inset-inline-end: anchor(right);
  }
}
```

Det er et eksempel på progressive enhancement: grundoplevelsen virker i alle browsere, og browsere med understøttelse får en mere præcis placering, der følger login-knappen automatisk.

FAQ med details/summary og reduceret bevægelse
--------------------------------------------------
FAQ-sektionen er bygget med native `details`/`summary`, animeret hvor browseren understøtter `::details-content`, og deaktiveret helt hvis brugeren har slået reduceret bevægelse til:

```css
@supports selector(::details-content) {
  .faq-item::details-content {
    opacity: 0;
    block-size: 0;
    transition: content-visibility 0.3s allow-discrete, opacity 0.3s ease, block-size 0.3s ease;
  }
}

@media (prefers-reduced-motion: reduce) {
  .faq-icon,
  .faq-item::details-content {
    transition: none;
  }
}
```

Uden `::details-content`-understøttelse folder `details` bare ud/ind uden animation, hvilket stadig er fuldt funktionelt — det er endnu et sted, hvor grundfunktionaliteten ikke er afhængig af den nyeste CSS.

Defensive CSS
--------------
Login-panelets bredde er sat med `width: min(23rem, calc(100vw - 2rem))`, så det aldrig kan blive bredere end skærmen minus lidt luft, uanset skærmstørrelse. Samme mønster går igen i makrolayoutet i `case.css`, hvor `minmax(var(--space-lg), 1fr)` sikrer et minimum af luft i siderne, selv når indholdskolonnen skrumper på små skærme — jeg har testet ned til 375px bredde uden vandret scroll.

CSS-organisering
------------------
Efter oprydningen er organiseringen nu:

- **`tokens.css`** — alle custom properties (farver, `--space-*`, `--step-*`, radius, osv.), bruges globalt.
- **`reset.css`** — browser-reset, bruges globalt.
- **`global.css`** — delt styling for sideskelettet (header, footer, login-panel) samt de sektioner på forsiden, der ikke er blevet redesignet siden.
- **`overrides.css`** — importeres sidst i `Layout.astro` og ejer nu reelt hero- og services-sektionen på forsiden (se afsnittet ovenfor om, hvorfor den fil vandt over `global.css`).
- **`about.css` / `case.css`** — side-specifik CSS, importeres kun i den `.astro`-fil, den hører til, og indeholder kun styling for den ene side.
- **Komponent-scoped `<style>`** i selve `.astro`-filerne (`TeamGrid.astro`, `TeamHero.astro`, `ExperienceSection.astro`, `team/[id].astro`) — bruges når styling kun hører til én komponent, så den ikke skal ligge i en delt fil, som andre sider aldrig bruger.

Jeg fandt undervejs en gammel `team.css`-fil og en `ValueCard.astro`-komponent, som slet ikke blev importeret nogen steder — sandsynligvis rester fra en tidligere version af Team-siden. De er slettet, i stedet for at blive stående som forvirrende, ubrugt kode. Samme sted fandt jeg, at `.values-section`, `.value-card`, `.about-layout` og `.about-section` i `about.css` heller aldrig blev renderet af nogen komponent — det var faktisk her, den oprindelige (ugyldige) subgrid-forsøg lå, hvilket forklarer, hvorfor det ikke virkede. Den døde kode er fjernet.

Jeg stødte også på en mere teknisk fejl undervejs: `reset.css` havde en regel, `a[class]:any-link { color: currentColor; }`, som var mere specifik (element + attribut + pseudo-klasse) end en almindelig enkelt klasse som `.team-grid-btn`. Det betød, at flere knappers tiltænkte tekstfarve reelt aldrig blev vist — de arvede bare den omkringliggende tekstfarve i stedet. Jeg rettede det ved at pakke reset-reglen ind i `:where()`, som nulstiller dens specificitet, så komponent-klasser igen kan overskrive den, uden at ændre hvilke elementer reglen rammer.
