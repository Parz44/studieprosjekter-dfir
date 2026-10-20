# Studieprosjekter i digital etterforskning

Utvalgte prosjekter og lab-øvelser fra bachelorgraden min i digital etterforskning (Digital Forensics) ved Noroff (2026), skrevet om til kortere porteføljeversjoner. Hvert prosjekt viser problemstillingen, oppsettet, hvordan jeg gikk frem, hva jeg fant og hva jeg lærte.

CTF-oppgaver og generell metodikk ligger i et eget repo: [ctf-playbook](https://github.com/Parz44/ctf-playbook).

## Prosjekter

| Prosjekt | Emne | Hva det viser |
| --- | --- | --- |
| [Artefakter fra Tor Browser på Android](tor-browser-android/) | Advanced Device Forensics | Eget eksperiment med fire snapshots, ADB-uttrekk, sammenligning med WinMerge, SQLite-analyse og en tidslinje |
| [Lab-øvelser i hendelseshåndtering](irf-labber/) | Incident Response Fundamentals | Minneanalyse med Volatility og nettverksanalyse av Emotet-trafikk i Wireshark |

## Ferdigheter

| Område | Verktøy og teknikker |
| --- | --- |
| Mobil- og appanalyse | ADB, rootet Android-emulator, `/data/data`, `/data/system`, `packages.xml`, systemlogger |
| Filsystem og databaser | Autopsy, DB Browser for SQLite, WAL- og journalfiler, HxD |
| Sammenligning over tid | Snapshots før og etter hver handling, WinMerge, tidslinje fra tidsstempler |
| Bevisintegritet | SHA-256-hashing av hvert uttrekk, logging av alle kommandoer |
| Minneanalyse | Volatility 2.6: imageinfo, pslist, pstree, cmdscan, consoles, connscan, printkey, svcscan, malfind |
| Nettverksanalyse | Wireshark: Conversations, DNS- og HTTP-filtre, TLS ClientHello, følge TCP-strømmer |
| Trusseletterretning | VirusTotal for IP-adresser, domener og filhasher |

## Om innholdet

- Alle prøvefiler i lab-øvelsene er offentlige treningsdata. Ingen ekte saker eller personopplysninger er med.
- Teksten er skrevet om fra innleveringene og notatene mine. Den er ikke den fulle innleverte oppgaven.
- Domener og IP-adresser fra skadevare er gjort ufarlige med `[.]`, så de ikke blir klikkbare lenker.
