# Lab-øvelser i hendelseshåndtering

Emne: Incident Response Fundamentals, Noroff (våren 2026). Alle prøvefilene er offentlige treningsdata.

| Øvelse | Type | Hovedfunn |
| --- | --- | --- |
| [Zeus](zeus-minneanalyse.md) | Minne, Windows XP | Kode injisert i `svchost.exe`, kontakt med ondsinnet IP-adresse, persistens via Winlogon |
| [Mem00.raw](mem00-minneanalyse.md) | Minne, Windows 7 | Et Python-skript som skrev ut en kodet streng, funnet via kommandohistorikken |
| [laqma](laqma-minneanalyse.md) | Minne, Windows XP | Én prosess med tilkoblinger til mange eksterne IP-adresser |
| [Emotet](emotet-pcap-analyse.md) | Nettverk | Infisert maskin, nedlasting av ondsinnet DLL og trafikk til kommandoserver |

## Fremgangsmåten jeg brukte

**Minne:**

1. `imageinfo` for å finne operativsystem og profil.
2. `pslist` og `pstree` for å se hvilke prosesser som kjørte.
3. `cmdscan` og `consoles` for å se hvilke kommandoer som ble kjørt og hva de skrev ut.
4. `connscan` for nettverkstilkoblinger, også lukkede, koblet til prosess-ID.
5. Registeret og tjenestene for persistens.
6. `malfind` på den mistenkelige prosessen.

**Nettverk:**

1. Statistics > Conversations for å finne maskinen med mest trafikk på det interne nettet.
2. DNS-oppslag fra den maskinen for å finne domener som ikke ser legitime ut.
3. HTTP GET-forespørsler for å finne nedlastinger.
4. Følge TCP-strømmen for å se hva som faktisk ble overført.
5. Slå opp domener, IP-adresser og hasher i VirusTotal.
