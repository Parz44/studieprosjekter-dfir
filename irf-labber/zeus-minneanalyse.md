# Minneanalyse: Zeus (Windows XP)

Bildet er den offentlige `zeus.vmem`-prøven. Verktøy: Volatility 2.6 standalone på Windows.

## Sammendrag

| Funn | Verdi |
| --- | --- |
| Operativsystem | Windows XP SP2 x86 |
| Mistenkelig prosess | `svchost.exe`, PID 856 |
| Ekstern adresse | `193.104.41.75:80`, markert som ondsinnet på VirusTotal |
| Persistens | `Userinit` i Winlogon kjører også `sdra64.exe` |
| Injisert kode | `malfind` gir treff i PID 856 på baseadresse `0xb70000` |
| Konklusjon | Zeus (Trojan.Zbot), skadevare som stjeler innloggings- og bankinformasjon |

## 1. Identifisere minnebildet

```
volatility -f zeus.vmem imageinfo
Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86
Image date and time  : 2010-08-15 19:17:56 UTC+0000
```

## 2. Prosesstreet

```
volatility -f zeus.vmem --profile=WinXPSP2x86 pstree
```

Treet ser normalt ut ved første øyekast: `smss.exe` > `winlogon.exe` > `services.exe` > flere `svchost.exe`. Ingen navn skiller seg ut, så neste steg er å se på nettverkstrafikken.

## 3. Nettverkstilkoblinger

`connections` viste ingen aktive tilkoblinger da dumpen ble tatt. Det betyr ikke at det ikke har vært noen tidligere, så jeg kjørte `connscan`, som også finner lukkede tilkoblinger.

```
volatility -f zeus.vmem --profile=WinXPSP2x86 connscan
Local Address          Remote Address        Pid
172.16.176.143:1054    193.104.41.75:80      856
0.0.0.0:1056           193.104.41.75:80      856
```

Begge tilkoblingene tilhører PID 856. I prosesstreet er det:

```
0x80ff88d8:svchost.exe   856   676   29   336   2010-08-11 06:06:24
```

En `svchost.exe` som snakker med en ekstern IP-adresse på port 80 er verdt å se nærmere på. VirusTotal markerte `193.104.41.75` som ondsinnet og koblet den til skadevare som endrer registernøkler.

## 4. Persistens i registeret

```
volatility -f zeus.vmem --profile=WinXPSP2x86 printkey -K "Microsoft\Windows NT\CurrentVersion\Winlogon"
```

Verdien `Userinit` kjører `sdra64.exe` i tillegg til den vanlige `userinit.exe`. Det betyr at filen starter hver gang en bruker logger på. `sdra64.exe` er en kjent fil som legges inn av Trojan.Zbot (Zeus).

## 5. Tjenester

```
volatility -f zeus.vmem --profile=WinXPSP2x86 svcscan > svcscan.txt
```

Et søk etter PID 856 i utskriften viste at den var registrert som en terminaltjeneste. At `svchost.exe` lager utgående nettilkoblinger passer ikke med den rollen.

## 6. Injisert kode

```
volatility -f zeus.vmem --profile=WinXPSP2x86 -p 856 malfind
```

`malfind` fant et mistenkelig minneområde i `svchost.exe` (PID 856) på baseadresse `0xb70000`. Det er her den injiserte koden kjører. `impscan` på den samme prosessen viser hvilke funksjoner den skjulte koden importerer.

## Hva jeg lærte

- Start bredt (prosesser), og følg så én tråd (nettverk) frem til en bestemt prosess.
- `connections` viser bare øyeblikket dumpen ble tatt. Kjør alltid `connscan` i tillegg.
- Run-nøkler og Winlogon-verdier i registeret er et vanlig sted for persistens.
- Et legitimt prosessnavn betyr ikke legitim kode. Skadevare injiserer seg i `svchost.exe` for å gli inn.

## Neste steg jeg ville tatt

- Dumpe det injiserte området med `malfind -D <mappe>` og skanne det med YARA-regler for Zeus.
- Sjekke `handles` og `mutantscan` for prosessen for å finne flere indikatorer.
- Skrive ned indikatorene (IP-adresse, filnavn, registerverdi) i et format en SOC kan bruke til deteksjon.
