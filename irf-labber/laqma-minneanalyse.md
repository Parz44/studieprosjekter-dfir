# Minneanalyse: laqma.vmem (Windows XP)

Verktøy: Volatility 2.6. Profil: WinXPSP2x86.

## Det jeg fant

`connscan` viste mange tilkoblinger fra én og samme prosess, PID 1340, til ulike eksterne IP-adresser på port 80:

```
65.54.x.x:80
207.46.170.x:80
96.6.41.211:80
12.120.180.24:80
```

En prosess som snakker med mange forskjellige eksterne adresser på samme port kan tyde på kommunikasjon med en kommandoserver eller jevnlige oppdateringsforsøk (beaconing).

## Neste steg

```
volatility -f laqma.vmem --profile=WinXPSP2x86 cmdline -p 1340
Command line : "C:\Program Files\Internet Explorer\iexplore.exe"

volatility -f laqma.vmem --profile=WinXPSP2x86 malfind -p 1340
```

PID 1340 var Internet Explorer. `malfind` fant ingen injisert kode i prosessen.

## Vurdering

Mange tilkoblinger fra en nettleser er ikke mistenkelig i seg selv. Uten funn fra `malfind` er neste steg å se om noen av IP-adressene er kjent som ondsinnede, og å sjekke andre prosesser og registeret for persistens.

## Hva jeg lærte

- Mønsteret «én prosess, mange IP-adresser» er et godt sted å starte, men må vurderes ut fra hvilken prosess det er.
- Et tomt resultat fra `malfind` betyr ikke at maskinen er ren. Det betyr bare at det ikke ble funnet injisert kode i akkurat den prosessen.
