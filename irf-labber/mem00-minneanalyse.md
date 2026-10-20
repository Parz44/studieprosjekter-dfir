# Minneanalyse: Mem00.raw (Windows 7)

Verktøy: Volatility 2.6 standalone på Windows.

## Først: ta et minnebilde selv

Før analysen tok jeg et minnebilde av lab-maskinen og regnet ut hashen:

```
certutil.exe -hashfile <minnebilde>.raw MD5
```

Lærdom: RAM endrer seg hele tiden. Et nytt minnebilde av den samme maskinen et sekund senere får en helt annen hash. Derfor må hashen regnes ut med en gang, og den beviser bare at akkurat denne filen er uendret.

## 1. Identifisere bildet

```
volatility -f Mem00.raw imageinfo
Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
Image date and time       : 2018-10-23 08:30:51 UTC+0000
Image local date and time : 2018-10-23 14:00:51 +0530
```

Forskjellen mellom UTC og lokal tid viser at maskinen sto i tidssonen UTC+5:30.

## 2. Prosesser

```
volatility -f Mem00.raw --profile=Win7SP1x86 pslist
```

Tre prosesser skilte seg ut:

- `cmd.exe`: kommandolinjen, kan vise hva som ble kjørt
- `DumpIt.exe`: verktøyet som ble brukt til å ta minnebildet
- `explorer.exe`: Filutforsker

## 3. Kommandohistorikk

```
volatility -f Mem00.raw --profile=Win7SP1x86 cmdscan
Cmd #0 @ 0x2f43c0: C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
```

Et Python-skript med `.txt`-endelse ble kjørt fra skrivebordet. Neste spørsmål er hva det skrev ut.

## 4. Det skriptet skrev ut

`consoles` viser innholdet i konsollvinduet, ikke bare kommandoene:

```
volatility -f Mem00.raw --profile=Win7SP1x86 consoles
C:\Users\hello>C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
<hex-streng>
```

Strengen besto bare av 0-9 og a-f, så den så ut som hex. Dekodet fra hex i CyberChef ga den uleselige bytes, som viste seg å være XOR-kodet. Brute force av XOR-nøkkelen ga en lesbar tekst som så ut som siste del av et flagg.

## Hva jeg lærte

- `cmdscan` viser kommandoene, men `consoles` viser også det som ble skrevet ut på skjermen. Begge bør kjøres.
- Filendelser lyver. `demon.py.txt` var et Python-skript.
- Kodinger kommer ofte i lag: her hex, så XOR.
- Tidssonen kan leses ut av forskjellen mellom UTC og lokal tid i `imageinfo`.
