# Artefakter fra Tor Browser på Android

Emne: Advanced Device Forensics, Noroff (høsten 2025). Eget eksperiment.

## Problemstilling

Tor Browser er laget for å ikke etterlate spor. Hvilke spor finnes likevel på en Android-enhet etter at Tor Browser er installert, brukt og avinstallert? Og hva kan en etterforsker faktisk bevise i hver av disse situasjonene?

## Kort oppsummert

- Tor Browser lagret ingen nettleserhistorikk, informasjonskapsler eller økter. Databasene fantes, men var tomme.
- Mens appen var installert, fantes det tydelige spor av at enheten hadde koblet seg til Tor-nettverket (`files/tor/`).
- Ved avinstallering forsvant hele app-mappen og alle registreringer i pakkefilene.
- Det eneste sporet som overlevde avinstalleringen var Androids egen varslingslogg (`notification_log.db`), med tidsstempler som viser at appen hadde vært i bruk.

Konklusjon: Tor Browser beviser **at** enheten koblet seg til Tor, ikke **hva** brukeren gjorde. Etter avinstallering er det operativsystemet, ikke appen, som har de siste sporene.

## Oppsett

| Innstilling | Verdi |
| --- | --- |
| Emulator | LDPlayer 9 (9.1.63.0) |
| Android-versjon | Android 9 (høyeste LDPlayer støtter) |
| Emulert enhet | Samsung Galaxy S24 Ultra, 4 kjerner, 4096 MB RAM |
| Root | Aktivert |
| App | Tor Browser for Android 14.5.9 (x86_64), APK fra torproject.org |

| Verktøy | Versjon | Bruk |
| --- | --- | --- |
| ADB | | Uttrekk av filsystemet |
| Autopsy | 4.22.1 | Gjennomgang, tidsstempler, nøkkelordsøk |
| DB Browser for SQLite | 3.13.1 | SQLite-databaser, WAL- og journalfiler |
| WinMerge | 2.16.52.0 | Sammenligning av mapper mellom snapshots |
| HxD | 1.7.7.0 | Kontroll av filsignaturer på filer uten endelse |

Hvorfor emulator: en ren og kontrollert enhet uten annen brukeraktivitet gjør det mulig å knytte hver endring i filsystemet til én bestemt handling. Root trengs for å lese `/data/data`.

## Fremgangsmåte

Eksperimentet ble delt i fire faser. Etter hver fase tok jeg et snapshot i LDPlayer og hentet ut de samme tre mappene med ADB:

```
adb devices
adb root
adb shell whoami

adb pull /data/data/org.torproject.torbrowser/ C:\TorAnalysis\<Fase>\data_data\
adb pull /data/system/ C:\TorAnalysis\<Fase>\data_system\
adb pull /sdcard/ C:\TorAnalysis\<Fase>\sdcard\
```

Hvert uttrekk ble pakket i en `.tar` og hashet med SHA-256, og hashene ble ført i en egen logg.

| Fase | Hva jeg gjorde |
| --- | --- |
| A. Baseline | Ren emulator, ingenting installert |
| B. Installasjon | Installerte APK-en med `adb install`, åpnet appen én gang og lukket den |
| C. Bruk | Koblet til Tor, besøkte tre nettsteder, gjorde tre søk på DuckDuckGo, åpnet og lukket faner, lukket appen helt |
| D. Avinstallering | Avinstallerte via Innstillinger, slik en vanlig bruker ville gjort |

Bruksscenarioet var det samme hver gang, slik at resultatet kan gjenskapes:

| Handling | Innhold |
| --- | --- |
| Nettsteder | `proton.me/mail`, `duckduckgo.com`, `check.torproject.org` |
| Søk på DuckDuckGo | «cute puppies», «burger king», «terminator» |

Etterpå sammenlignet jeg snapshotene parvis i WinMerge (A mot B, B mot C, C mot D), gikk gjennom filene i Autopsy, søkte etter nøkkelord som `duckduckgo`, `proton`, `terminator`, `https` og `.onion`, og åpnet alle SQLite-databasene i DB Browser.

## Hva uttrekkene viste

| | Baseline | Installasjon | Bruk | Avinstallering |
| --- | --- | --- | --- | --- |
| Størrelse på snapshot | ca. 1,05 GB | ca. 1,50 GB | ca. 1,62 GB | ca. 1,63 GB |
| Filer i `/data/system` | 161 | 192 | 207 | 222 |
| Filer i `/sdcard` | 7 | 7 | 7 | 7 |
| Filer i Tor-mappen | finnes ikke | 73 | 127 | finnes ikke |

`/sdcard` var uendret gjennom alle fasene. Tor Browser skrev ingenting til den delte lagringen.

## Funn

### Installasjon (A mot B)

- `packages.list` fikk en ny linje: `org.torproject.torbrowser 10060 0 /data/user/0/org.torproject.torbrowser default:targetSdkVersion=35`. Appen fikk UID 10060.
- `packages.xml` fikk en hel blokk for Tor Browser med tillatelser, signeringssertifikat (utstedt til The Tor Project) og tidsstempler for installasjonen.
- `notification_policy.xml` fikk en varslingskanal som heter «Received Tabs». Den kommer fra Firefox, som Tor Browser bygger på.
- App-mappen `/data/data/org.torproject.torbrowser/` ble opprettet med undermapper for cache, databaser og innstillinger.

### Bruk (B mot C)

- **Tor-nettverket:** `files/tor/` inneholdt `cached-microdesc-consensus`, `cached-microdescs`, `cached-certs` og `state`. Dette er vanlige filer for en Tor-klient og viser at enheten lastet ned informasjon om reléer og bygde kretser.
- **Databaser:** `places.sqlite` (historikk), `cookies.sqlite` (tabellen `moz_cookies`) og `tab_collections` hadde tabeller, men ingen rader. Ingen URL-er, søk eller informasjonskapsler.
- **Nylig lukkede faner:** filen under `mozac.feature.recentlyclosed` inneholdt bare en `about:`-side.
- **Ikoner:** `cache/mozac_browser_icons/` viste at nettsider var blitt lastet, men ikke hvilke.
- **Systemloggen:** `notification_log.db` fikk tabellen `log` med tre hendelser for `org.torproject.torbrowser`, med tidsstempel.
- **Innstillinger:** nye filer i `shared_prefs`, blant annet `fenix.onboarding.xml` og `org.torproject.torbrowser_preferences.xml`, viser at appen var åpnet og satt opp.
- **Nøkkelordsøk:** ingen treff på nettstedene eller søkeordene fra scenarioet.

### Avinstallering (C mot D)

- Hele `/data/data/org.torproject.torbrowser/` var borte, også alle Tor-filene.
- Tor Browser var fjernet fra `packages.xml`, `packages.list` og `notification_policy.xml`.
- En ny, tom `packages-warnings.xml` dukket opp, som Android lager når den rydder.
- `notification_log.db` hadde fortsatt hendelsene for Tor Browser.

## Tidslinje

Tidsstemplene i `packages.xml` (`ft`, `it`, `ut`) er Unix-tid i millisekunder skrevet som hex. Ved å regne dem om, sammen med tidsstempler fra startskjermens database og systemloggen, kan appens livsløp settes sammen:

| Tid (UTC), 4. desember 2025 | Hendelse | Kilde |
| --- | --- | --- |
| 15:39:50 | APK-filen skrevet til enheten (`ft="19aea050970"`) | `packages.xml` |
| 15:39:52 | Appen installert (`it="19aea0514ba"`) | `packages.xml` |
| 15:41:50 | Snarvei til Tor Browser lagt på startskjermen | Startskjermens database |
| 15:55:02 | Oppdateringsforsøk for en innebygd utvidelse | `addons_updater_attempts_database` |
| 15:55:21 | Hendelse fra Tor Browser registrert av systemet | `notification_log.db` |
| 16:02:58 | Tor-klientens tilstand sist skrevet | `files/tor/state` |

Lærdom: tidsstempler som ser uleselige ut, er ofte bare et annet format. Hex-verdiene i `packages.xml` ga eksakte tidspunkter for installasjonen.

## Hva dette betyr for en etterforsker

1. **Er appen fortsatt installert, må uttrekket skje før noe annet.** Tor-filene og pakkeinformasjonen forsvinner ved avinstallering.
2. **Etter avinstallering er systemloggene det viktigste.** `notification_log.db` var det eneste som viste at Tor Browser hadde vært i bruk.
3. **Tradisjonell nettleseranalyse gir ingenting.** Historikk, informasjonskapsler og økter kan ikke gjenskapes fra lagringen.

## Begrensninger

- Én emulator og én Android-versjon. LDPlayer støtter bare Android 9, og nyere versjoner eller fysiske enheter kan oppføre seg annerledes.
- Ingen minneanalyse. Åpne faner, øktnøkler og annen data som bare finnes i RAM ble ikke undersøkt.
- Et kort og enkelt bruksscenario med standardinnstillinger. Nedlastinger, utvidelser og lengre økter kan gi andre spor.
- Sikkerhetskopier, skysynkronisering og andre kilder utenfor enheten var ikke med.

## Videre arbeid

Gjenta forsøket med minneanalyse og på flere enheter og Android-versjoner, for å fange spor som bare finnes i RAM og se hvor mye resultatet varierer.

## Hva jeg lærte

- Et snapshot før og etter hver handling er en enkel og sterk måte å knytte spor til en bestemt hendelse på.
- At en database er tom, er også et funn. Det bekrefter hva appen ikke lagrer.
- Operativsystemet logger ting appen selv ikke har kontroll over, og det er ofte der de siste sporene ligger.
