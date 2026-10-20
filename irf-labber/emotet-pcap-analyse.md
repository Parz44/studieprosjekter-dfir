# Nettverksanalyse: Emotet-infeksjon

Prøvefiler: offentlige pcap-filer med Emotet-trafikk fra januar 2021, laget for opplæring i Wireshark. Domener og IP-adresser er gjort ufarlige med `[.]`.

## Svar

| Spørsmål | Svar |
| --- | --- |
| Infisert maskin | `10.1.6[.]206` |
| IP-adresse som leverte skadevaren | `103.93.235[.]25` |
| Fil som ble lastet ned | `nDUrg8uFD5hl.dll` |
| Protokoll for nedlastingen | HTTP |
| Mulig kommandoserver | `5.2.136[.]90`, port 80 |

## 1. Finne den infiserte maskinen

Statistics > Conversations, fanen IPv4, sortert på antall pakker. Den største samtalen var mellom `10.1.6[.]206` (privat adresse) og `5.2.136[.]90` (offentlig adresse), med 2992 pakker. En intern maskin med så mye trafikk mot én ekstern adresse er sannsynligvis den infiserte.

## 2. DNS-oppslag

```
dns && ip.src == 10.1.6.206
```

De første oppslagene var etter domener som ikke ser ut som noe en vanlig bruker besøker:

```
hangarlastik[.]com
padreescapes[.]com
sarture[.]com
seo.udaipurkart[.]com
```

## 3. Nedlastinger

```
http.request.method == "GET"
```

Fem forespørsler, blant annet:

```
hangarlastik[.]com   GET /cgi-bin/Ui4n/
padreescapes[.]com   GET /blog/0I/
seo.udaipurkart[.]com GET /rx-5700-6hnr7/Sgms/
sarture[.]com        GET /wp-includes/JD8/
```

Flere av domenene var markert som ondsinnede i VirusTotal. Nedlastingen av DLL-filen kom fra `103.93.235[.]25` over HTTP.

Et filter som gir god oversikt over web-trafikk uten støy:

```
(http.request or http.response or tls.handshake.type eq 1) and !(ssdp)
```

## 4. Trafikk etter infeksjonen

Første POST-forespørsel til `5.2.136[.]90` sendte rundt 66 KB med data som så kryptert ut. Det passer med en infisert maskin som sender data til en kommandoserver.

## 5. TLS: nettleser eller automatikk?

Med `tls.handshake.type == 1` (ClientHello) så jeg på håndtrykkene:

- TLS 1.2 ble brukt hele veien.
- Rundt 19 cipher suites ble tilbudt.
- Utvidelsene inkluderte `supported_groups` og `signature_algorithms`.
- Servernavnene (SNI) pekte til legitim Microsoft-infrastruktur.

Håndtrykkene så ut som vanlig klienttrafikk og ikke som et nedstrippet, automatisert verktøy. Den krypterte trafikken i dette opptaket var derfor ikke det mistenkelige. Det var HTTP-trafikken.

## Del 2: Emotet med spambot

I et annet opptak fra samme kampanje viste HTTP GET-filteret to mistenkelige URL-er. Ved å følge TCP-strømmen så jeg at en fil ble lastet ned fra `infoprocenter[.]com` (pakke 162). I del 2 av opptaket viste `smtp.data.fragment` ukryptert e-post sendt fra den infiserte maskinen, som er typisk for en spambot.

## Hva jeg lærte

- Start med Conversations. Den interne maskinen med mest trafikk mot én ekstern adresse er et godt utgangspunkt.
- DNS-oppslag rett før nedlastinger viser hvilke domener som ble brukt til å levere skadevaren.
- En GET uten et `HTTP/1.1 200 OK` som svar kan bety at serveren ikke svarte, at opptaket stoppet, eller at svaret ligger i en annen strøm.
- Kryptert trafikk er ikke mistenkelig i seg selv. Se på mønsteret i håndtrykket før du konkluderer.
