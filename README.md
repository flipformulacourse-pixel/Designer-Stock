# Designer Stock – webshop

En hurtig, ren webshop med betaling via Stripe Checkout og et dashboard til ejerne, klar til Vercel. Produkterne gemmes i jeres Stripe-konto, billederne i Vercel Blob, og lageret tælles automatisk ned ved salg. Kunder kontakter jer ved at oprette en ticket i jeres Discord.

## Sådan kommer I online

1. **Læg projektet på GitHub.** Opret et nyt repository, og upload alle filerne i denne mappe.
2. **Importér i Vercel.** Gå til vercel.com → Add New → Project → vælg jeres repository. Framework Preset: **Other**. Lad Build Command og Output Directory stå tomme, og tryk Deploy.
3. **Tilføj Stripe-nøgler.** I Vercel: Settings → Environment Variables. Tilføj variablerne fra `.env.example`:
   - `STRIPE_SECRET_KEY` – fra Stripe → Udviklere → API-nøgler. Start med `sk_test_...`.
   - `SITE_URL` – jeres domæne, fx `https://designerstock.dk` (kan udelades).
   - `STRIPE_WEBHOOK_SECRET` – se trin 4.
   - `STRIPE_TERMS_CONSENT` – se trin 5.
   Tryk derefter **Redeploy**, så nøglerne bliver brugt.
4. **Dashboard.** Tilføj `ADMIN_PASSWORD` (mindst 10 tegn) i Vercel. Gå til Vercel → Storage → Create Database → **Blob**, og forbind den til projektet – så oprettes `BLOB_READ_WRITE_TOKEN` automatisk, og I kan uploade billeder. Redeploy, og log ind på `https://<jeres-domæne>/admin.html`.
5. **Opsæt webhook (vigtigt – tæller lageret ned).** Stripe → Udviklere → Webhooks → Tilføj endpoint: `https://<jeres-domæne>/api/webhook`, hændelse `checkout.session.completed`. Kopiér signeringshemmeligheden (`whsec_...`) til `STRIPE_WEBHOOK_SECRET`. Nye ordrer kan så ses i Vercel → Logs.
6. **Handelsbetingelser i Stripe (anbefalet).** Stripe → Indstillinger → Offentlige oplysninger → Servicevilkår: indsæt `https://<jeres-domæne>/handelsbetingelser.html`. Sæt `STRIPE_TERMS_CONSENT=true`. Så skal kunden også sætte kryds i Stripes betalingsside.
7. **Discord-link** er sat til https://discord.gg/2h8kXedT. Skifter I link, så ret `discord` øverst i `public/assets/js/products.js`.
8. **Test et køb** med kortnummer `4242 4242 4242 4242`, en fremtidig udløbsdato og en vilkårlig CVC.
9. **Gå live:** skift `STRIPE_SECRET_KEY` til `sk_live_...` og webhook-nøglen til live-versionen, og redeploy.
10. **Eget domæne:** Vercel → Settings → Domains.

## Dashboardet

Log ind på `/admin.html` med `ADMIN_PASSWORD`.

- **Oversigt:** omsætning, antal ordrer, gennemsnitlig ordre og solgte stk for 7, 30 eller 90 dage, omsætning pr. dag, mest solgte varer og seneste ordrer med link til Stripe.
- **Produkter:** opret, ret og arkivér varer. Upload billeder (de gøres automatisk mindre), sæt pris og vejl. pris, lager, minimumskøb, størrelser eller farver, Bulk Bundle-antal og pre-order med forventet levering. Nye varer ligger i butikken inden for et halvt minut.
- **Betalingslink:** ét klik laver et Stripe-link til en vare, som I kan dele i Discord, på Instagram eller i en besked. Kunden kan vælge antal og størrelse, betaler hos Stripe, og lageret tælles ned som ved et almindeligt køb.

Produkterne gemmes som produkter i jeres Stripe-konto, så I kan også se dem under Produktkatalog i Stripe. Rediger dem helst i dashboardet, så butikkens felter følger med.

Indtil der er en Stripe-nøgle, viser butikken eksempelvarerne i `public/assets/js/products.js`. Øverst i samme fil står fragt, fri-fragt-grænse, lande, Discord-link og firmaoplysninger.

## Godt at vide

- **Lager** tælles ned af webhooken, når en betaling er gennemført – både fra webshoppen og fra betalingslinks. Uden webhook skal lageret rettes i dashboardet.
- **Én adgangskode** deles af ejerne. Skift `ADMIN_PASSWORD` (eller `ADMIN_SECRET`) i Vercel for at logge alle ud.
- **Kvitteringer:** slå "E-mail kunder om gennemførte betalinger" til i Stripe → Indstillinger → Kundemails.
- **Ingen returret** står kun i afkrydsningen i kurven, ved betalingsknappen i Stripe og i handelsbetingelserne. Accepten og tidspunktet gemmes på betalingen i Stripe.

## Lokal udvikling og test

```bash
npm i -g vercel        # én gang
cp .env.example .env.local
vercel dev             # siden kører på http://localhost:3000
npm test               # tester kurv, Stripe-kald, dashboard-login, produkter, salgstal, betalingslinks og lager
```

## Filer

```
public/                 hjemmesiden (forside, shop, produkt, kurv, tak, betingelser, returpolitik)
public/admin.html       dashboardet
public/assets/js/       products.js (indstillinger og eksempelvarer), app.js (butik), admin.js (dashboard)
api/products.js         butikkens varer fra Stripe
api/checkout.js         opretter Stripe Checkout
api/session.js          viser ordren på tak-siden
api/webhook.js          modtager besked fra Stripe ved betaling og tæller lageret ned
api/admin/              login, produkter, billed-upload, salgstal og betalingslinks
lib/                    fælles kode til Stripe og validering
tests/                  automatiske tests
```

Drives af Flip Formula · CVR 46163834
