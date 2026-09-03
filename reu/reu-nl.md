[![MCC](https://github.com/maarten-pennings/C64howto/blob/main/MCC/mcc240x160.png)](https://github.com/maarten-pennings/C64howto/tree/main/MCC)

Maarten Pennings MCC#03 


# REU in BASIC

Dit artikel is een verkorte Nederlandse versie van een Engels artikel.
Dat bevat meer details en bijvoorbeeld ook source files.

> [https://github.com/maarten-pennings/C64howto/blob/main/reu/readme.md](https://github.com/maarten-pennings/C64howto/blob/main/reu/readme.md). 


## Introductie

In 1985 heeft Commodore de eerste REU (**R**AM **E**xpansion **U**nit) op de markt 
gebracht. Een REU breidt het geheugen van de Commodore 64 behoorlijk uit, met 
128 kB (REU 1700), 256 kB (REU 1764), of zelfs 512 kB (REU 1750).
Ook moderne systemen zoals VICE, de Commodore Ultimate, The C64, en Kung Fu Flash 2
hebben ondersteuning voor een REU. In dit artikel bekijken we hoe een REU werkt,
en we schrijven een BASIC programma ("10 PRINT") dat de REU gebruikt.

Een REU gebruikt geen _banking_ mechanisme. We spreken van banking wanneer er 
meerdere RAM chips op hetzelfde adres gebied zitten, en er een mechanisme is 
om te kiezen welke chip (welke _bank_) op enig moment actief is. Bij een banking 
oplossing kiest de CPU, in ons geval de 6510, welke bank actief is, en 
daarna kan de CPU de bank lezen en schrijven.

De REU werkt anders. Een REU is een memory mapped I/O device. Het wordt bediend
met zeven registers. Via deze registers geeft de 6510 commando's (met `POKE`s) 
om data uit het C64 geheugen ergens in het REU geheugen te _stashen_, of om 
data uit het REU geheugen te _fetchen_ en ergens in het C64 geheugen te plaatsen.
Er zijn nog twee andere commando's: _swap_ en _compare_ maar die zullen we in 
dit artikel niet bekijken. We gebruiken de term kopiëren als overkoepelend 
begrip voor alle 4 de commando's.

De 6510 kan dus _niet_ het geheugen van de REU lezen of schrijven. Het kan 
alleen een kopieer opdracht aan de REU geven. De REU is dus een soort DMA device
(_direct memory access_) dat data kopieert tussen de REU en de C64. Dat klinkt 
langzaam, en ten opzichte van _bank switching_ is het dat ook. _Bank switching_ 
kost een of twee 6510 instructies. Maar in de praktijk is het snel. De REU 
kopieert op volle C64 klok snelheid: elke tik van de ongeveer 1MHz klok wordt 
1 byte gekopieerd. Dat betekent dat 1 miljoen bytes in 1 seconde worden 
gekopieerd, ofwel 1000 bytes in 1 ms. Het hele C64 geheugen kopiëren kost 
dus 64 ms.


## Registers

De REU registers zijn _memory mapped_ in de "I/O 2" regio; die begint op $DF00.
De REU heeft 7 registers variërend in grootte van 1 tot 3 bytes.

  | Register   | Size | Offset | Hex       | Dec         |
  |:----------:|:----:|:-------|:----------|:------------|
  | `status`   |   1  |  0     | DF00      | 57088       |
  | `command`  |   1  |  1     | DF01      | 57089       |
  | `c64base`  |   2  |  2,3   | DF02-DF03 | 57090-57091 |
  | `reubase`  |   3  |  4,5,6 | DF04-DF06 | 57092-57094 |
  | `translen` |   2  |  7,8   | DF07-DF08 | 57095-57096 |
  | `irqmask`  |   1  |  9     | DF09      | 57097       |
  | `addrctrl` |   1  |  10    | DF0A      | 57098       |

Het `status` register is eigenlijk niet belangrijk. Het geeft aan of de REU 
al klaar is met het uitvoeren van het kopieer commando (via bit 6). In de 
praktijk blokkeert (_stalls_) de REU de 6510 tijdens het kopiëren. 
Als de 6510 dus weer begint te lopen is het niet nodig het `status` register 
te bekijken; de REU is klaar.

We slaan `command` even over en springen naar registers `c64base`, `reubase`, 
en `translen`. Het register `c64base` is het start adres van de kopieer actie 
aan de C64 kant, het register `reubase` is het start adres van de kopieer actie 
aan de REU kant, en `translen` is het aantal te kopiëren bytes. Merk op dat 
`c64base` (en `translen`) de hele 64 kB van de 6510 omvat, terwijl het REU 
geheugen maximaal 256 × 64 kB kan zijn.

  | Bits | Function     | Details                                                             |
  |:----:|:------------:|:--------------------------------------------------------------------|
  |   7  | EXECUTE      | 1 start een kopieer actie (van type TRANSFERTYPE)                   |
  |   6  |              |                                                                     |
  |   5  | LOAD         | 1 = `c64base`, `reubase`, `translen` zet start waardes terug        |
  |   4  | NOFF00       | 1 = start meteen, 0 = start na schrijven naar $FF00                 |
  |  3:2 |              |                                                                     |
  |  1:0 | TRANSFERTYPE | 00=_stash_ (C64→REU), 01=_fetch_ (REU→C64), 10=_swap_, 11=_compare_ |

Het `command` register bestuurt de kopieer actie. De onderste twee bits 
`TRANSFERTYPE` bepalen welk van de 4 kopieer acties wordt uitgevoerd. 
Het `EXECUTE` bit (bit 7: 128) start de actie. Tijdens de kopieer actie zal 
de REU de registers `c64base`, `reubase` stap voor stap ophogen en `translen` 
aflagen. Als `LOAD` (bit 5: 32) hoog is zal de REU de start waardes herstellen 
aan het einde van de kopie. De vlag `NOFF00` stelt je in staat de start nog 
even uit te stellen; die begint pas na een schrijfactie naar $FF00. Dit is 
nodig in een speciaal geval: als de REU de C64 RAM moet lezen of schrijven,
die _onder_ het I/O 2 gebied ligt. 

Het `irqmask` register geeft de mogelijkheid een interrupt te genereren als 
de kopieer actie klaar is. Zoals bij `status` vermeld _stallt_ de REU de 6510
tot het kopiëren klaar is dus interrupts zijn niet nodig.

Het laatste register `addrctrl` bepaalt of de registers `c64base` en `reubase` 
tijdens de kopieer actie opgehoogd worden of niet. Bit 7 (128) is voor `c64base` 
en bit 6 (64) voor `reubase`. Niet ophogen is zinvol als er een _memory fill_ 
nodig is: steeds hetzelfde byte schrijven naar alle adressen in het doel 
geheugen.

Zie het Engelse artikel op GitHub voor meer details en BASIC voorbeelden 
voor alle registers.


## 10 PRINT met REU 

Nu we weten hoe de REU werkt gaan we hem gebruiken in een BASIC programma.
We gaan het klassieke 10 PRINT programma nabouwen omdat iedereen het kent 
en omdat het goed past bij de mogelijheden van de REU.


### 10 PRINT

Het klassieke 10 PRINT programma bestaat uit één regel:

```basic
  10 PRINT CHR$(205.5+RND(1));:GOTO 10
```
  
Het gebruikt de `RND(1)` functie om een willekeurig getal tussen 0 en 1 te 
genereren. Hierdoor ligt `205.5+RND(1)` tussen 205.5 en 206.5. De `CHR$()` 
functie kapt het argument af naar 205 of 206, waardoor een `\` of `/` wordt 
afgedrukt. De GOTO 10 vormt een lus, wat leidt tot het afdrukken van 
oneindig veel schuine strepen, zodat er een doolhof patroon ontstaat.


### REU en 10 PRINT

De REU kan snel 1000 tekens kopiëren van zijn geheugen naar het C64-geheugen.
We zullen de REU opdracht geven te kopiëren naar $0400, het scherm geheugen 
van de C64. Zoals we eerder zagen, de REU kopieert 1000 tekens, één scherm, 
in 1 ms.

Het idee is dat we veel schuine strepen van het 10 PRINT-programma opslaan 
in het REU-geheugen. We slaan ze op vanaf adres 0 ($000000) in de REU. 

Voor de animatie halen we 1000 bytes op vanaf REU adres 0 en plaatsen deze 
op C64 adres $0400 (het scherm). Daarna halen we 1000 bytes op uit REU adres 
40 en plaatsen die opnieuw op $0400. Dit scrolt het scherm één rij omhoog. 
Vervolgens halen we 1000 bytes op vanaf REU adres 80 en plaatsen die op $0400; 
weer _scrollt_ het scherm een rij omhoog. En zo verder. 

Dit gaat snel. Het is niet meer nodig om strepen te berekenen en ook de 
kernel-scroll is niet meer nodig. Elke kopie duurt slechts 1 ms 
(we hebben nog wel een paar BASIC-commando's nodig om de kopie te starten).


### REU geheugen indeling

Er zijn twee problemen met de hierboven geschetste REU versie van 10 PRINT.

Het eerste probleem is dat elke schuine streep, eerst in de REU moet staan; 
we moeten deze dus toch berekenen. 
Om dit op te lossen, spelen we een beetje vals. We genereren slechts 256 
schuine strepen en slaan meerder kopieën achtereenvolgens op in de REU. 
Als je goed kijkt kun je dat zien, maar 256 schuine strepen is 6 scherm 
rijen (van 40) plus nog 16 tekens. Hierdoor is de tweede reeks van 256 
schuine strepen met 16 tekens verschoven. Dit maakt het lastiger om het 
herhalende patroon te zien. Goed genoeg voor onze eenvoudige demo. 
Vanaf nu noemen we een reeks van 256 schuine strepen één pagina.

Het tweede probleem is dat we, net als in het originele 10 PRINT programma, 
willen dat onze versie oneindig door loopt. Het is duidelijk dat we geen 
oneindige hoeveelheid (identieke) pagina's van 256 schuine strepen kunnen hebben.
Hier schiet de wiskunde ons te hulp. Het kleinste gemene veelvoud van de 
rijgrootte 40 en de paginagrootte 256 is 1280. We kunnen precies 5 pagina's 
(van 256 tekens) kwijt in 1280, én we kunnen precies 32 rijen (van 40 tekens) 
kwijt in 1280. Als de REU gevuld is met voldoende pagina's en we beginnen 
met het ophalen van (1000 bytes vanaf) rij 0 (adres 0), vervolgens 1000 vanaf 
rij 1 (adres 40), ..., en op een gegeven moment 1000 bytes vanaf rij 32 
(adres 1280), dan bereiken we een cyclus: de bytes (vanaf) rij 32 zijn gelijk 
aan die op rij 0. Dus na het ophalen van het scherm vanaf rij 31, halen we het 
volgende scherm niet op vanaf rij 32, maar springen we terug en halen we het 
scherm weer op vanaf rij 0. Dat is immers identiek aan dat op rij 32.

We moeten ons realiseren dat we een scherm van 1000 bytes ophalen, waarvan 
de laatste begint op rij 31. Daarom moeten er vanaf rij 31 nog minimaal 1000 
schuine strepen in de REU staan. Zoals de het geheugen plaatje hieronder 
laat zien, hebben we 9 pagina's in de REU nodig om genoeg schuine strepen 
te hebben om helemaal naar rij 31 te scrollen. Wij denken dat dit genoeg 
variatie biedt voor onze demo.

![REU geheugen](memorysmall.png)


(end)
