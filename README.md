# SB-vacature-tracker

Vacature-tracker voor SB, gebouwd naar het patroon van [vacature-tracker](https://github.com/arjen-rave/vacature-tracker)
(Arjens eigen tracker) en [eclipse2026](https://github.com/arjen-rave/eclipse2026): statische site op
GitHub Pages, pushmeldingen via een eigen Cloudflare Worker + web-push, periodieke check via een
Cowork scheduled task. Zelfde functionaliteit en interface als Arjens tracker, met twee verschillen:
Deloitte-groen (`#86BC24`) in plaats van blauw als accentkleur, en een tweewekelijks in plaats van
dagelijks checkritme (maandag + donderdag, 09:00).

Gehost en beheerd door Arjen (eigen GitHub-account, eigen Cloudflare Worker, eigen Cowork scheduled
task) — SB heeft zelf geen accounts nodig om de tracker te gebruiken, alleen om 'm te bekijken en de
site-eigen instellingen (meldingen aan/uit, sollicitatiestatus) te bedienen.

## Onderdelen

- `index.html`, `data.json`, `sw.js`, `manifest.json` — de site zelf (GitHub Pages). Uitklapbare
  kaarten voor actief/archief/niet-controleerbaar, met een "Sollicitatie status"-dropdown per actieve
  kaart die direct naar `data.json` schrijft via de Cloudflare Worker.
- `subscriptions.json` — pushsubscripties, wordt geschreven door de Cloudflare Worker.
- `.github/workflows/send-push.yml` — verstuurt een pushmelding naar iedereen in `subscriptions.json`.
  Wordt getriggerd via `workflow_dispatch` door de Cowork-taak, bij élke succesvolle run.
- `cloudflare-worker/worker.js` — eigen Worker-instance (aparte VAPID-sleutels en GitHub-token van
  Arjens eigen tracker), met dezelfde drie endpoints: `/subscribe`, `/unsubscribe`, `/update-status`.

## Data bijwerken

`data.json` wordt tweewekelijks (maandag + donderdag) bijgewerkt door een Cowork scheduled task, op
dezelfde manier als bij Arjens eigen tracker: vacaturesites checken, 2-4 Engelse tags per actieve
vacature toekennen, vacatures met status "Afgewezen"/"Niet interessant" naar het archief verplaatsen,
committen, en daarna de pushmelding triggeren.

**Let op:** deze scheduled task staat bij aanmaak bewust nog UIT (disabled) en de prompt bevat nog
TODO-placeholders voor SB's rolprofiel, cv en bedrijvenlijst — die moeten eerst worden ingevuld voordat
de taak wordt aangezet, anders heeft de dagelijkse/tweewekelijkse check niets om op te screenen.

## data.json schema

Identiek aan Arjens eigen tracker:

```json
{
  "lastUpdated": "<ISO-timestamp met +02:00>",
  "active": [
    {
      "bedrijf": "",
      "titel": "",
      "link": "",
      "beschrijving": "",
      "fit": "",
      "status": "Niet gesolliciteerd | Gesolliciteerd | Sollicitatie begonnen | Afgewezen | Aanbod | Niet interessant",
      "tags": ["energy", "strategy", "..."]
    }
  ],
  "archive": [
    { "bedrijf": "", "titel": "", "reden": "", "datum": "YYYY-MM-DD" }
  ],
  "notControlled": [
    { "bedrijf": "", "titel": "", "reden": "", "link": "" }
  ]
}
```

## Eenmalige setup (door Arjen)

1. GitHub Secrets toevoegen aan dit repo (Settings → Secrets and variables → Actions):
   - `VAPID_PUBLIC_KEY` = `BJ9LsXBpKr_THPpoA6ngAYdfvNDXxi-xdhc-UWpERQeC83zmEVivkVxLo-7arWVEm9MGLrnDdKwf6TW7NnRF-nU`
   - `VAPID_PRIVATE_KEY` = `DVQpiwVUbA2hKGrrV7VBaicHLfQLZD1OILrfLsUmGnk`
   - Deze sleutels zijn speciaal voor SB's tracker gegenereerd, los van Arjens eigen VAPID-sleutels.
2. Een nieuwe Cloudflare Worker deployen met `cloudflare-worker/worker.js`. **Noem de Worker exact
   `sb-vacature-tracker-subscribe`** — de `WORKER_URL` in `index.html`/`sw.js` is al voor-ingevuld
   met `https://sb-vacature-tracker-subscribe.arjen-ravestein.workers.dev`, dus met deze naam hoeft
   er verder niets aangepast te worden in de sitecode.
   Worker-secrets/vars instellen:
   - Secret `GITHUB_TOKEN` — een GitHub Personal Access Token met `repo`-scope (fine-grained: Contents
     read/write, alleen op dit repo). Kan hetzelfde token zijn als voor Arjens eigen tracker, of een
     apart token — beide werken.
   - Vars: `GITHUB_OWNER=arjen-rave`, `GITHUB_REPO=SB-vacature-tracker`, `GITHUB_BRANCH=main`,
     `ALLOWED_ORIGIN=https://arjen-rave.github.io`
3. GitHub Pages aanzetten op de `main`-branch (Settings → Pages) van dit repo.
4. Zodra SB's cv, rolprofiel en bedrijvenlijst binnen zijn: de placeholder-tekst in de Cowork
   scheduled task (`SB-vacature-tracker-biweekly-check`) vervangen door haar echte gegevens, en de
   taak aanzetten (enabled).
5. Site testen: open `https://arjen-rave.github.io/SB-vacature-tracker/`, zet de meldingen-toggle aan
   op SB's telefoon om te bevestigen dat push-meldingen werken (net als bij Arjens eigen tracker,
   eventueel eerst een `test_send`-run via de Actions-tab).
