[![MCC](https://github.com/maarten-pennings/C64howto/blob/main/MCC/mcc240x160.png)](https://github.com/maarten-pennings/C64howto/tree/main/MCC#overview)

Maarten Pennings MCC#04 


# Rijen en regels

Dit artikel is een verkorte Nederlandse versie van een Engels artikel.
Dat bevat meer details en bijvoorbeeld ook source files.

> [https://github.com/maarten-pennings/C64howto/blob/main/rowsvslines/readme.md](https://github.com/maarten-pennings/C64howto/blob/main/rowsvslines/readme.md). 


## Introductie

Regels in een Commodore 64 BASIC programma mogen 80 karakters lang zijn. 
Niet langer. Het scherm van de C64 bestaat uit 40 kolommen en 25 rijen.
Een programma _regel_ die langer is dan 40 karakters is verdeeld over twee 
scherm _rijen_. Dat twee rijen een regel (kunnen) vormen is niet een eigenschap 
van BASIC; het is een eigenschap van de (programmeer) omgeving. Ik noem het 
in dit artikel een eigenschap van de _terminal_ (de code in de ROM die 
het scherm beheert en bijvoorbeeld het scrollen implementeert).

In dit artikel bekijken we hoe de terminal met rijen en regels omgaat.

Ik ga in dit artikel proberen het woord _rij_ te gebruiken voor een scherm rij 
en het woord _regel_ voor een logische (programma of ge`PRINT`te) regel. Ik 
hoop dat het gelukt is.


## Tab

We beginnen ons onderzoek met een analyse van de `TAB()` functie.
Die blijkt kennis te hebben van rijen en regels in de terminal.

De `TAB(X)` functie is geen functie zoals `SIN(X)`. `TAB(X)` kan alleen 
in de context van een `PRINT` statement gebruikt worden. Het verplaatst 
de cursor dan naar kolom `X` van de huidige regel (tellend vanaf 0).
De vergelijkbare functie `SPC(X)` is _relatief_; die verschuift de cursor `X` 
posities vanaf de huidige. `TAB(X)` springt naar aan _absolute_ positie: 
naar kolom `X`. Helaas weigert `TAB(X)` _terug_ te springen (persoonlijke mening). 
Als de cursor in kolom 30 staat, dan heeft een `TAB(25)` geen effect.


We proberen het volgende programma.

```basic
100 UP$=CHR$(145)
110 A10$="---------+"
120 A30$=A10$+A10$+A10$
130 A60$=A30$+A30$
140 :
200 PRINT A30$
210 PRINT UP$;">";TAB(6);"X"
220 PRINT:PRINT
230 :
300 PRINT A60$
310 PRINT UP$;">";TAB(6);"X"
320 PRINT:PRINT
330 :
400 PRINT A60$
410 PRINT UP$;">";TAB(46);"X"
420 PRINT:PRINT
430 :
500 PRINT A60$
510 PRINT UP$;UP$;">";TAB(46);"X"
520 PRINT:PRINT
530 :
600 PRINT A60$;A30$
610 PRINT UP$;">";TAB(6);"X"
620 PRINT:PRINT
```

Het programma begin met het definiëren van de string `UP$` die de cursor 
één rij omhoog beweegt. Daarna definieert het strings met 10, 30 en 60 
karakters (streepjes).

Het programma voert vijf tests uit, die alle vijf hetzelfde patroon volgen.
Eerst `PRINT` het een regel met streepjes (zie programma regels 200, 
300, 400, 500 en 600). Merk op dat de `PRINT` niet afsluit met een `;`. Dit 
betekent dat na de `PRINT` de cursor naar de eerste kolom van de volgende rij 
gaat. 

De volgende regel van de test (210, 310, 410, 510 en 610) stuurt de cursor een 
rij omhoog; print een `>` ter oriëntering, springt naar een `TAB()` positie 
en print daar een `X`. Twee witregels (220, 320, 420, 520 en 620) scheiden 
de tests.

Dit is de uitvoer van het programma.

![Output of the TAB program](tab1.png)

De eerste test heeft geen verrassingen. Hij print 30 streepjes, schuift de 
cursor omhoog naar de rij met 30 streepjes, en met een `TAB(6)` verschijnt er
een `X` in kolom 6 (tellend vanaf 0).

De tweede test was voor mij een verrassing. Hij print 60 streepjes; dat is 
een regel bestaande uit 2 scherm rijen. Regel 310 verschuift de cursor een 
rij omhoog (zie de `>`). Dit betekent (zo bleek) dat de cursor op kolom 40 
van de _regel_ staat. Een `TAB(6)` zou terug springen, maar dat doet `TAB` 
nooit; de `TAB` instructie wordt genegeerd. De cursor verplaatst niet; 
de `X` verschijnt meteen na de `>`.

Om de kolom-40 hypothese te testen bevat de derde test een `TAB(46)`.
Deze test print ook 60 streepjes (twee scherm rijen), en verschuift de cursor 
een rij omhoog (zie de `>`). De cursor staat weer op kolom 40 van de tweerijige 
regel. De `TAB(46)` wordt uitgevoerd (46 is immers groter dan 40), die 
verschuift de cursor 6 kolommen; daar komt de `X`.

De vierde test is vrijwel hetzelfde als de derde. Het verschil is dat na het 
printen van de 60 streepjes, de cursor _twee_ in plaats van _een_ rij omhoog 
wordt geschoven (zie de `>`). De cursor is dan in kolom 0 van de regel. 
De `TAB(46)` schuift de cursor naar kolom 46, dat wil zeggen naar kolom 6 op 
de tweede regel. Daar wordt de `X` geprint.

De vijfde test laat zien dat de terminal geen regels van 3 rijen kan hebben.
Nadat er 80 (van de 90) streepjes geprint zijn begint de terminal een verse 
regel. Op het scherm staat dus een regel van twee rijen gevolgd door een regel 
van een rij. De test schuift de cursor een rij omhoog, en geeft het commando 
`TAB(6)`, niet `TAB(126)`.

De conclusie is: **de terminal houdt bij welke (twee) scherm rijen samen een logische regel vormen**.

Ten slotte nog een detail. Een `TAB(X)` springt naar kolom `X` in de huidige 
regel, maar alleen als `X` groter of gelijk is aan de huidige cursor positie.
Met andere woorden er is een _minimum_ voor `X`, Tot mijn verassing is er geen 
_maximum_ voor `X`. Een `TAB(86)` verschuift de cursor naar de derde rij en 
een `TAB(126)` naar de vierde. Er is alleen een technisch maximum, `X` moet in 
een byte (8 bits) passen en is dus maximaal 255.


## Scrollen

Het was natuurlijk al lang bekend dat de C64 terminal kan _scrollen_. Als 
het scherm vol is, zorgen extra `PRINT`s dat de bestaande regels omhoog 
scrollen (de bovenste vallen weg), en de nieuwe worden onderaan bijgevoegd.
Wat ik me nooit zo gerealiseerd had: scrollen gaat per regel, niet per rij.

Het volgende programma heeft twee regels die op een scherm rij passen 
(regels 100 en 120), en een regel die over twee rijen verspreid staat 
(regel 110). 

```basic
100 PRINT "HELLO, WORLD!"
110 PRINT "THIS IS A LINE THAT SPANS TWO SREEN ROWS"
120 PRINT "SHORT AGAIN"
```

Dit programma runnen we niet, we `LIST`en het en drukken dan meerder malen 
op RETURN. Het plaatje hieronder laat een serie screenshots zien; tussen twee 
screenshots drukken we steeds op RETURN. Met rode blokken (links boven in elk 
screenshot) hebben we aangegeven of een regel uit een of twee rijen bestaat.
De serie screenshots begin met de cursor op de onderste rij.

![Repeatedly pressing RETURN for the SCROLL program](scroll1-6.png)

Wat we zien tussen screenshot 1 en 2 maar ook tussen 2 en 3 is dat de 
RETURN het scherm een rij omhoog scrollt.

Dat verandert bij de RETURN tussen screenshot 3 and 4. In screenshot 3 staat 
een tweerijige regel bovenaan. Na de RETURN scollt de terminal twee rijen 
omhoog zodat de hele regel verdwijnt. Merk ook een bijzonderheid op onderaan 
het scherm: de cursor is daar ook een regel omhoog geschoven.

De conclusie: **scrollen is niet per rij maar per regel** (bovenaan het scherm).

Dit effect is ook te zien bij het bekende 10 PRINT programma

```basic
10 PRINT CHR$(205.5+RND(1));:GOTO 10
```

Dit programma print schuine streep na schuine streep, alles op één regel 
(de `PRINT` heeft immers een `;`). De terminal ondersteunt echter geen regels 
langer dan 80 karakters. Als het 81 karakter geprint wordt begin the terminal
een nieuwe regel. De terminal is dus gevuld met logische regels van 80 
karakters, elke verdeeld over 2 rijen. Dit betekent dat elke keer als 
de onderste regel vol is, het scherm _twee_ regels scrollt.


## De _line link_ tabel

Hoe weet de terminal welke rijen samen een regel vormen?
Die informatie blijkt bijgehouden te worden in de _line link_ tabel.
Die staat op adressen $00D9-$00F2.

De _line link_ tabel heeft een byte per scherm rij.
Bit 7 van dat byte is de _first flag_.
Deze vlag is 1 als die rij het eerste deel is van een regel (of zelfs de hele regel bevat).
Deze vlag is 0 als die rij het tweede deel is van een regel.

De eerste rij heeft zijn byte op $00D9 (217) staan.
De tweede staat op $00DA (218), ... en de 25ste staat op $00F1 (241).
Het byte op $00F2 (242) is waarschijnlijk nodig om de scroll implementatie 
te vergemakkelijken.

De byte bevat twee _nibbles_.
De bovenste _nibble_ bevat alleen de _first flasg_, de overige 3 bits zijn 0.
De _nibble_ heeft dus waarde 8 voor "eerste deel of hele regel" en 
waarde 0 voor "tweede deel van de regel".
De onderste _nibble_ negeren we. Het bevat het pagina nummer van het screen 
memory waar die rij is opgeslagen. Bijvoorbeeld, de eerste rij staat op $0400,
dus het onderste nibble van de eerste rij is 4.

Het volgende programma laat de _line link_ tabel zien.

Het eerste deel van het programma print regels met schuine streepjes, net 
als het 10 PRINT programma. Sommige regels zijn minder dan 1 rij lang, andere 
beslaan tot wel 4 rijen. Aan het begin van elke regel print het programma 
een letter (`A`, `B`, ...) zodat we goed kunnen zien waar de logische regels 
beginnen.

Regel 110 initialiseert de _random number generator_ zodat de uitvoer steeds
hetzelfde is. Verwijder die regel als je dat niet wilt. 

> Dit programma is met `petcat` afgedrukt: vandaar geen hoofdletters.
> Het bevat wel de speciale tekens zoals `{home}`, `{down}` en `{rvon}`.

```basic
100 rem print lines with random length
110 c=rnd(-123)
120 for i=0 to 13
130 :print
140 :print chr$(65+i);
150 :for c=0 to rnd(1)*40*3.5
160 ::print chr$(205.5+rnd(1));
170 :next c
180 next i
190 :
200 rem show line link info for rows
210 d$="{home}{right}{right}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}{down}"
220 for i=0 to 24
230 :l=peek(217+i):rem line link row r
240 :lh=int(l/16):ll=l and 15
250 :print left$(d$,i+3);"{rvon}";lh;ll;"{rvoff}";
260 next i:print "{home}"
270 get a$:if a$="" then 270
```

Het interessante deel is het tweede deel, regels 200-270 die de _line link_ 
tabel afdrukken nadat het scherm gevuld is. Er is een `FOR` loop (regel 220) 
die  over alle rijen itereert (`I` van 0 tot 24). Op regel 230 wordt de 
byte voor de `I`-de rij uit de _link link_ tabel gehaald.
Regel 240 splitst de byte in bovenste _nibble_ (`LH`) en onderste nibble `LL`.
 
Regel 210 construeert een constante string `D$` die gebruikt wordt om 
de cursor op rij `I` te plaatsen. dat gebeurt op regel 250, waar de 
bovenste and onderste _nbble_ in _reverse video_ worden geprint.
 
Regel 270 wacht op een toets, die het programma beëindigd.
De `print"{home}"` op regel260 voorkomt een scroll.

Dit is de uitvoer van het programma.

![The output of LINELINKTABLE](linelinktable1.png)

We zien dat regel A kort is, hij past op een regel.
De bovenste _nibble_ is inderdaad 8.

Regel B is ongeveer 90 karakters lang. Hij beslaat 3 rijen.
De eerste rij is de eerste helft van een regel, dus bovenste _nibble_ is 8.
De tweede rij is de tweede helft van een regel, dus bovenste _nibble_ is 0.
De derde rij is het derde deel van regel B. Maar omdat de terminal geen regels 
langer dan 80 ondersteunt, is hier een nieuwe regel begonnen.
Het is dus de eerste (en enige deel) van een regel, dus bovenste _nibble_ is 8.
  

## Regelmatige scroll in 10 PRINT

Het is ook mogelijk om te _schrijven_ naar de _line link_ tabel.
We kunnen een lange regel (van twee rijen) doormidden breken door voor de 
tweede regel de bovenste _nibble_ op 8 te zetten. 

We voegen een `POKE` toe die dat doet. 
Hij zet de _first flag_ aan voor de tweede rij, 
en knipt daarmee de tweede rij los van de eerste.

```basic
10 PRINTCHR$(205.5+RND(1));:POKE218,128:GOTO 10
```

Als je die programma runt krijg je een regelmatiger scroll van een rij.
Zonder de `POKE` krijg je steeds een scroll van twee regels.


Merk op dat de `POKE` de onderste _nibble_ ook overschrijft.
Veiliger zou zijn `POKE 218,PEEK(218) OR 128` maar dat is langzamer,
en de huidige code lijkt geen zichtbaar effect te hebben.
De rij scrollt snell uit het zicht.


(end)
