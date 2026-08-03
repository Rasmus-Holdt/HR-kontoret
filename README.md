# HR-Kontoret — hjemmeside

Statisk 4-siders site. Ingen build, ingen frameworks, ingen JavaScript.

## Filer

| Fil | Indhold |
|---|---|
| `index.html` | Forside |
| `ydelser.html` | 9 ydelser + abonnement (`#abonnement`) |
| `om.html` | Om virksomheden + stifteren |
| `kontakt.html` | Kontaktoplysninger |
| `404.html` | Fejlside |
| `assets/style.css` | Al styling (~10 kB) |
| `assets/favicon.svg` | Favicon |
| `robots.txt`, `sitemap.xml` | SEO |
| `netlify.toml` | Hosting: pæne URL'er, cache-headers, sikkerhedsheaders |

Hele sitet fylder ca. **35 kB** uden billeder. Ingen webfonts, ingen eksterne requests — det loader stort set med det samme, også på mobil.

## Sådan ser du den lokalt

Dobbeltklik på `index.html`. Alle interne stier er relative, så siden virker direkte fra filsystemet — også når du sender mappen videre.

## Status

På plads: Michaels baggrund, uddannelse og certificeringer, de tre abonnementer (Basis / Plus / Partner) og fra-priser på alle ni enkeltydelser.

CVR-nummer (46669320) og Michaels baggrundstekst er på plads.

**Der mangler kun én ting: mailadressen.** Den oprettes sammen med domænet. Indtil da står `kontakt@hr-kontoret.com` som gul placeholder fire steder — find dem med `grep -n 'class="ph"' *.html`.

**Prisforklaringerne under fra-priserne** (`.pris-note`) er formuleret af mig ud fra hvad der normalt driver omfanget i den slags opgaver. Michael bør læse dem igennem og rette, hvis noget ikke passer med hans måde at prissætte på.

## Målt ydelse

Målt i headless Chrome på 390 px viewport:

| | |
|---|---|
| First Contentful Paint | 32 ms |
| Requests pr. sidevisning | 2 (HTML + CSS) |
| Cumulative Layout Shift | 0,0000 |
| Første sidevisning, gzippet | 9,6 kB |
| Hele sitet, gzippet | ~22 kB uden OG-billedet |

Ingen webfonts, ingen JavaScript, ingen tredjeparter, ingen cookies. OG-billedet (17 kB) hentes kun af Facebook/LinkedIn, aldrig af besøgende.

Ting der holder tallene nede — pas på ikke at ødelægge dem:

- **Ét stylesheet.** Tilføjer du et bibliotek eller en webfont, ryger både hastigheden og den nuværende cookie-frihed.
- **Ingen billeder i sidernes indhold.** Kommer der fotos ind, skal de komprimeres til maks ~150 kB og have `width`, `height` og `loading="lazy"`, ellers ryger CLS op.
- **Alt animation er transform-baseret**, aldrig opacity — se nedenfor.

## SEO

- Unik `<title>` og meta description pr. side
- `lang="da"`, canonical, `color-scheme`
- Open Graph med `og:image` (`assets/og-billede.png`, 1200×630) og `summary_large_image`
- JSON-LD: `ProfessionalService` med adresse, åbningstider og pris på forsiden · `OfferCatalog` + `FAQPage` (6 spørgsmål) + `BreadcrumbList` på ydelser · `AboutPage` med `Person` og certificeringer på om · `ContactPage` på kontakt
- Semantisk HTML, ét `<h1>` pr. side, skip-link, `aria-current` på aktiv menu
- Intern krydslinkning mellem alle fire sider

**FAQ-schemaet genereres ud fra den synlige tekst.** Retter du et spørgsmål eller svar i `ydelser.html`, skal JSON-LD-blokken i `<head>` rettes tilsvarende — ellers er strukturerede data ude af trit med siden, og det straffer Google.

## Overgange og animation

Sideskift sker med `@view-transition` (krydstoning). Sektioner glider på plads via `animation-timeline: view()`.

Animationen rører **kun** `transform`, aldrig `opacity` — så indhold kan aldrig ende usynligt i print, i browsere uden understøttelse, eller hvis animationen ikke kører. `prefers-reduced-motion` slår det hele fra.

## Før den går live — tjekliste

**1. Fjern alle placeholders.** Alt der er markeret gult på siden er `<span class="ph">…</span>`. Find dem med:

```bash
grep -n 'class="ph"' *.html
```

Når indholdet er på plads, slettes `<span class="ph">`-tags'ene (selve `.ph`-reglen i CSS må gerne blive stående — den gør ikke noget uden tags).

**2. Bekræft domænet.** `https://hr-kontoret.com` står i canonical-tags, Open Graph, `sitemap.xml`, `robots.txt` og JSON-LD. Bliver det et andet, så ret dem alle:

```bash
grep -rln 'hr-kontoret.com' . --include='*.html' --include='*.xml' --include='*.txt'
```

**3. Billede.** Siden kører helt uden billeder. Kommer der et portræt af Michael, sættes det ind på Om-siden lige efter `<h2>Michael</h2>`:

```html
<img src="assets/michael.jpg" alt="Michael, stifter af HR-Kontoret" width="560" height="700" loading="lazy" style="max-width:280px;border-radius:4px">
```

Komprimér billedet først (maks ~150 kB, gerne `.webp`).

**4. Efter launch:** opret Google Business-profil og send `sitemap.xml` ind i Google Search Console.

## Kontaktformularen

Formularen på `kontakt.html` bruger **Netlify Forms** — den virker helt uden backend, men **kun når sitet ligger på Netlify**. Åbner man `kontakt.html` lokalt, sker der ingenting ved at trykke Send.

Sådan hænger det sammen:

- `data-netlify="true"` og `name="kontakt"` gør, at Netlify opdager formularen ved deploy
- `<input type="hidden" name="form-name" value="kontakt">` er påkrævet
- `netlify-honeypot="firma-web"` er spamfælden — feltet er skjult med `.hp` i CSS'en, så kun bots udfylder det
- `action="tak.html"` sender folk videre til kvitteringssiden

Efter første deploy: gå ind i Netlify → Forms → Form notifications og sæt en mailadresse på, ellers får Michael ikke besked. Gratisplanen dækker 100 indsendelser om måneden.

Skal sitet hostes et andet sted, skal formularen laves om — enten til Formspree (indsæt deres URL i `action`) eller til en `mailto:`-formular.

## Hosting: One.com + Netlify

Domæne og mail ligger hos **One.com**. Selve siden ligger på **Netlify** (gratis, CDN, automatisk HTTPS). `netlify.toml` er klar — forbind repoet, eller træk mappen ind.

### ⚠️ Peg IKKE navneserverne over på Netlify

Netlify foreslår selv, at man flytter DNS'en til dem. **Gør det ikke her.** Mailen ligger hos One.com, og flytter du navneserverne, forsvinder MX-recordene — så holder mailen op med at virke, uden at der kommer en fejl nogen steder. Det er den slags, man opdager en uge senere, når en kunde siger, at de aldrig fik svar.

Behold DNS'en hos One.com, og tilføj i stedet disse to records:

| Type | Navn | Værdi |
|---|---|---|
| A | `@` | `75.2.60.5` (Netlifys load balancer) |
| CNAME | `www` | `<sitenavn>.netlify.app` |

MX-recordene til One.com rører du ikke. Netlify slår selv HTTPS til, når DNS'en er slået igennem — det tager typisk under en time.

Tjek den præcise A-record i Netlifys panel under Domain settings, før du opretter den — den kan ændre sig.

### Efter deploy

1. **Netlify → Forms → Form notifications:** sæt Michaels mailadresse på. Uden det lander beskederne kun i dashboardet, hvor ingen kigger.
2. Send en testbesked gennem formularen og bekræft, at den kommer frem.
3. Send `sitemap.xml` ind i Google Search Console.

## Tilgængelighed

Klikflader er min. 44-48 px høje, fokus-markering er synlig, og `prefers-reduced-motion` slår al animation fra. Menu og FAQ-accordion virker uden JavaScript.

Farvekontrasten er regnet igennem på hele paletten — alle elleve kombinationer består WCAG AA. Laveste er prisnoterne på hvide kort (4,55:1) og hvid tekst på okker-knapper (3,69:1, hvor kravet for stor fed tekst er 3:1). Gør du okkeren lysere, ryger knapperne under grænsen.

Testet uden vandret scroll på 320, 360, 390 og 430 px.
