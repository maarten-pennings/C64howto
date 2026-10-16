[![MCC](https://github.com/maarten-pennings/C64howto/blob/main/MCC/mcc240x160.png)](https://github.com/maarten-pennings/C64howto/tree/main/MCC#overview)

Maarten Pennings MCC#05


# Recursie in BASIC

Kunnen we recursie gebruiken in Commodore 64 BASIC?
Het korte antwoord is ja, maar het is wat lastiger dan in moderne talen.

## Introductie

Mijn eerst programma's schreef ik in BASIC. Ik kende de officiële termen nog 
niet, maar ik leerde natuurlijk het _assignment statement_ (`LET`) dat de 
volgorde van _operaties in expressies_ kende (`*` voor `+`) en het resultaat 
toekent aan een _variabele_. Die kun je ook `PRINT`en of `INPUT`en.

Er was een hoog niveau `FOR`-`NEXT` statement, maar ook een `IF`. Die is iets 
lager in de pikorde want de `THEN` moet op één regel en een `ELSE` is er niet.
Beide tekortkomingen kun je oplossen met `GOTO`, maar die staat op het 
laagste niveau van programmeer abstracties.

Ten slotte was er nog de magische `GOSUB`, the `GOTO` die onthoudt waarheen 
de `RETURN` terug moet springen. Gemaakt voor _subroutines_.

Ik had geen idee dat subroutines een naam kunnen hebben (_function_), of dat ze
argumenten of lokale variabelen kunnen hebben (op de _stack_). Laat staan 
dat ik enig benul had van zoiets als recursie.

Veel later, hoorde ik van recursie. Terugkijken vroeg ik mij af of recursie 
mogelijk is op de Commodore 64 (of andere BASIC dialecten). Het gebrek aan 
een _stack_ en argumenten lijkt roet in het eten te gooien.

In dit artikel bekijken we eerst drie mini problemen die we met recursie 
oplossen. Geen van de mini problemen zou je in de echte wereld met recursie 
oplossen. Maar ze zijn bekend en eenvoudig en zo leren we de valkuilen 
en oplossingen bij recursie in BASIC. Als we die kennen dan pakken we wat 
serieuzere problemen aan. 


## 1e mini probleem: machten van 2

Om 2 tot de macht `N` uit te rekenen kun je in Commodore BASIC 
gewoon `2↑N` typen. Maar wij gaan een recursieve subroutine schrijven 
die dit uitrekent.

In BASIC zijn er geen functies met argumenten of een _return_ waarde.
We gaan voor beide globale variabelen gebruiken. Onze subroutine heeft 
als input `N`. De aanroeper moet `N` een waarde geven voor de `GOSUB` 
naar de subroutine. De subroutine zal 2 tot de macht `N` met recursie 
berekenen en het antwoord toekennen aan de globale variabele `R` ("resultaat").
Na de `RETURN` moet de aanroeper `R` inspecteren.

De vorige paragraaf beschrijft de _signatuur_ van de subroutine:
welke waardes erin gaan (en in welke variabelen) en welke waardes erut gaan 
(en in welke variabelen). Eén ding ontbreekt nog: welke (globale) variabelen 
worden door de subroutine gebruikt als "kladblok". Ze worden overschreven 
door de subroutine maar krijgen geen bekende of betekenisvolle waarde.
Dit is een belangrijk aspect van een subroutine omdat alle variabelen globaal 
zijn. Als de aanroeper van een subroutine een variabele in gebruik heeft die 
door de subroutine als kladblok wordt gebruikt, dan ontstaan er problemen.

In onze subroutine wordt `N` gewijzigd - een kladblok.
De complete signatuur staat op regel 200 (met `MODS:N`).

```BASIC
100 PRINT "POW2"
110 I=0
120 :N=I:GOSUB 200:PRINT "2↑";I;"=";R
130 :I=I+1
140 GOTO 120
150 :
200 REM INP:N; OUT:R=2↑N; MODS:N
210 IF N=0 THEN R=1:RETURN
220 N=N-1
230 GOSUB 200
240 R=R*2
250 RETURN
```

De subroutine staat op 200-250.
Regel 210 implementeert de zogeheten _basis_ van de recursie: `N=0`.
Regels 220 tot 240 bevatten de _stap_ van de recursie:

```
pow2(0)= 1
pow2(n)= pow2(n-1)*2, if n>0
```

Dat is alles. Dat viel best mee. Dit programma print de machten van 2 tot
`2↑23`, die geeft een `OUT OF MEMORY ERROR` om dat er een _stack overflow_
plaats vindt.


## 2e mini probleem: faculteit

Dit probleem lijkt bedrieglijk veel op het vorige, maar is toch net een 
stapje anders. We gaan `N!` recursief uitrekenen. Voor de volledigheid

```
fac(0)= 1
fac(n)= fac(n-1)*n, if n>0
```

dus 4! = 4×3×2×1 = 24.

Onze subroutine voor faculteit heeft als input `N`. 
De subroutine zal `N!` met recursie berekenen en het antwoord 
toekennen aan de globale variabele `R`.
Laten we aannemen dat net als in het vorige probleem `N` 
een "kladblok" is. En laten we dan ook de subroutine hergebruiken.

```BASIC 
200 REM INP:N; OUT:R=N!; MODS:N
210 IF N=0 THEN R=1:RETURN
220 N=N-1
230 GOSUB 200
240 R=R*N
250 RETURN
```

Dit werkt _niet_. Het probleem zit in regel 240.
Na de `GOSUB 200` is globale variabele `N` gewijzigd,
dus de toekenning `240 R=R*N` is incorrect.

We moeten de signatuur van de subroutine wijzigen dat `N` niet langer een 
kladblok is, maar dat hij zijn waarde behoudt. Dat is de nieuwe specificatie,
zie regel 200 (`KEEP:N`). 

```BASIC 
100 PRINT "FAC"
110 N=0
120 :GOSUB 200
130 :PRINT N;CHR$(157);"! =";R
140 :N=N+1
150 GOTO 120
160 :
200 REM INP:N; OUT:R=N!; KEEP:N
210 IF N=0 THEN R=1:RETURN
220 N=N-1
230 GOSUB 200
240 N=N+1:REM RESTORE N
250 R=R*N
260 RETURN
```

De strengere eis voor `N` is makkelijk te realiseren.
Voor de basis `N=0` (regel 210) is er geen toekenning aan `N`, 
dus is er geen wijziging nodig.
Voor de stap als `N>0`, is er wel een toekenning aan `N`, namelijk 
op regel 220: `N=N-1`. De reparatie is op regel 240, waar `N` hersteld
wordt met `N=N+1`. Deze wijziging zorgt ervoor dat ook de stap aan
de nieuwe signatuur voldoet.
En dan is regel `250 R=R*N` wel correct.

Dit programma print een tabel met faculteiten tot `23!`.
Ook hier krijgen we `OUT OF MEMORY ERROR` vanwege een _stack overflow_.


## 3e mini probleem: Fibonacci

Bij `FAC(N)` zagen we dat het nodig is oude waardes te herstellen.
Dat kon daar eenvoudig (met `N=N+1`), maar bij het probleem in deze 
sectie lukt dat niet.

Fibonacci was een wiskundige die een beroemde rij getallen introduceerde: 
0, 1, 1, 2, 3, 5, 8, ..., elke getal is de som zijn twee directe voorgangers.
Iets formeler als volgt te formuleren.

```
fib(0)= 0
fib(1)= 1
fib(n)= fib(n-1) + fib(n-2), if n>1
```

Dat kunnen we nu wel.

Onze subroutine heeft als input `N`.
Hij berekent (recursief) `FIB(N)` en kent dat toe aan `R`.
We zullen er voor zorgen dat de waarde van `N` behouden blijft.
De subroutine heeft twee kladblok variabelen `R0` and `R1`.

```basic 
200 REM INP:N; OUT:R=FIB(N); KEEP:N; MODS:R0,R1
210 IF N<2 THEN R=N:RETURN
220 N=N-1:GOSUB 200:R0=R
240 N=N-1:GOSUB 200:R1=R
250 R=R0+R1
260 N=N+2:REM RESTORE N
270 RETURN
```

Deze poging is mislukt. Het was makkelijk om `N`s waarde te behouden, 
maar zowel `R0` als `R1` staat op de kladblok lijst. De `GOSUB 200` op regel 
240 wijzigt dus `R0` en dus is regel 2050 `R=R0+R1` incorrect. Er is geen 
probleem met `R1`, dat is snel te zien.

```
220 N=N-1:GOSUB 200:R0=R
240 N=N-1:GOSUB 200:R=R0+R
```

We moeten `R0` herstellen na de tweede `GOSUB 200`. Hoe doen we dat?
`R0` wordt vers berekend, dus een herstel is niet zo eenvoudig als een 
ophoging met 1.

De oplossing is om zelf een stack te maken.
We introduceren een array`S()` met _stackpointer_ `S`.
Merk op dat arrays (zoals `S()`) en getallen (zoals `S`) dezelfde naam 
mogen hebben in BASIC (net als getal `A` en string `A$`). De stack 
wordt aangemaakt op regel 110.

```basic
100 PRINT "FIB"
110 DIM S(30):S=0:REM SETUP STACK
120 N=0
130 :T0=TI:GOSUB 200:T1=TI:T=(T1-T0)/60
140 :PRINT "FIB(";N;")=";R
170 :N=N+1
180 GOTO 130
190 :
200 REM INP:N; OUT:R=FIB(N); KEEP:N,S
210 IF N<2 THEN R=N:RETURN
220 N=N-1:GOSUB 200
230 S(S)=R:S=S+1:REM PUSH R
240 N=N-1:GOSUB 200
250 S=S-1:R=R+S(S):REM POP R
260 N=N+2:REM RESTORE N
270 RETURN
```

- Regel 210 berekent beide basis gevallen (`N=0` and `N=1`); 
  de basis gevallen veranderen `N` en `S` niet.
  
- Regel 220 berekent `FIB(n-1)` en het resultaat `R` wordt _gepushed_ op de stack (regel 230).

- Regel 240 berekent `FIB(n-2)` en het resultaat `R` wordt verhoogs met de waarde die _gepopped_ wordt van de stack (line 250).
  Deze regel hersteld ook de stackpointer `S`.
  
- Regel 260 herstelt  `N` en beëindigt de subroutine.

We hebben eenzelfde hoofdprogramma als eerst, behalve dat we nu tijdmetingen doen.
Ook dit programma stopt bij 23 met een `OUT OF MEMORY ERROR`.
Het stopt overigens na 1128 secondes ofwel 20 minuten.
Een recursieve subroutine is een erg slechte aanpak voor Fibonacci.


## Puzzel: torens van Hanoi


## Puzzel: 8 koninginnen 


## Ten slotte 

Dit artikel is een verkorte Nederlandse versie van een Engels artikel.
Dat bevat meer details en bijvoorbeeld ook source files.

> [https://github.com/maarten-pennings/C64howto/blob/main/recursion/readme.md](https://github.com/maarten-pennings/C64howto/blob/main/recursion/readme.md). 

Daar staan ook nog twee extra voorbeelden. de eerste is een expressie evaluator die 
de string `"10+4*5+5*(1+1)"` uitrekent tot `40`. De tweede lost een puzzel op:
maak het getal 21 met 1, 5, 6, en 7 en alleen de operatoren `+`, `-`, `*`, `/` en 
`(`..`)`. Het kan!

We zien dat recursie best kan in BASIC, maar omdat er alleen maar globale 
variabelen zijn is het net wat moeilijker dan in modernere programmeertalen.

(end)
