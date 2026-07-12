# Blinky1541
Maarten Pennings, Juli 2026

_Blinky1541_ is een machinetaal programma dat runt op de 1541 disk drive;
het laat de activity LED van de 1541 vijf keer knipperen.

Dit artikel is een verkorte Nederlandse versie van een gedetailleerder engels artikel 
[https://github.com/maarten-pennings/C64howto/blob/main/blinky1541/readme.md](https://github.com/maarten-pennings/C64howto/blob/main/blinky1541/readme.md).


## Introductie

De Commodore 1541 disk drive wordt een "smart peripheral" genoemd.
Het is eigenlijk een computer op zichzelf; het kleine broertje van de C64:
het heeft een 6502 CPU, 16 kB ROM, 2 kB RAM en 2 VIAs.

In dit artikel gaan we een programma voor de 1541 schrijven.
Zoals "Hello, World" het eerste programma is dat je schrijft voor een 
nieuwe _programmertaal_, is "Blinky" het eerste programma voor nieuwe _hardware_.
Een blinky laat een LED knipperen.

_Blinky1541_ wordt een 6502 machine taal programma. 
De C64 "upload" het naar de 1541. Als de 1541 het programma uitvoert
zal de "activity LED" van de 1541 vijf keer knipperen.


## Activity LED

Op [https://www.zimmers.net](https://www.zimmers.net)
vinden we schemas en documentatie voor de 1541 drive.
Daar leren we dat de activity LED met de anode aan de 5V verbonden is
en met de cathode aan pin PB3 van VIA 2. De LED is dus _low active_,
dat will zeggen als de pin laag gezet wordt (op 0) gaat de LED aan.

Op dezelfde site vinden we de _memory map_ van de 1541.
Daaruit leren we dat `control port B` (om de PBx pinnen laag of hoog te trekken)
op adres $1C00 is gemapped. PB3 is dus bit drie op dat adres.

We weten nu dat het op 0 zetten van bit 3 op adres $1C00 de LED aan zet 
terwijl een 1 op die plek de LED uit zet. Rest nog een vraag, waar 
plaatsen we het blinky programma zelf?

In memory map op Zimmers zien we dat de 1541 vijf buffers heeft.
Deze RAM buffers worden gebruikt om disk sectors te lezen of te schrijven.
Zolang we geen file `LOAD`,`SAVE` of `OPEN` doen heeft de 1541 firmware die 
buffers niet nodig. De buffers staan op pagina 3 (0300-03FF) tot en met 
pagina 7 (0700-07FF). Wij gaan de eerste buffer gebruiken.
 

## Het blinky programma


Zoals een blinky betaamd, schrijven we een _kort_ programma.
Omdat we alleen bit 3 van $1C00 willen veranderen doen we een zogeheten 
_read-modify-write_: we lezen $1C00, maskeren bit 3 naar nul en schrijven 
het resultaat terug naar $1C00. In assembly wordt dat 
`LDA $1C00; AND #$F7; STA $1C00` (omdat we bits vanaf 0 tellen staat PB3 op de _vierde_ plaats).

We doen hetzelfde om het bit weer naar één te schrijven. Omdat het
anders te snel gaat roepen we na welke schrijf actie een subroutine aan (`JSR $0320`) die executie 
vertraagt (een "wait").

Dit is het complete programma:

```
0300 | 162,5     | LDX #$5

0302 | 173,0,28  | LDA $1C00
0305 | 41,247    | AND #$F7
0307 | 141,0,28  | STA $1C00
030A | 32,32,3   | JSR $0320

030D | 173,0,28  | LDA $1C00
0310 | 9,8       | ORA #$08
0312 | 141,0,28  | STA $1C00
0315 | 32,32,3   | JSR $0320

0318 | 202       | DEX
0319 | 208,231   | BNE $0302
031B | 96        | RTS

031C | 234       | NOP
031D | 234       | NOP
031E | 234       | NOP
031F | 234       | NOP

0320 | 169,0     | LDA #$00
0322 | 160,0     | LDY #$00
0324 | 136       | DEY
0325 | 208,253   | BNE $0324
0327 | 56        | SEC
0328 | 233,1     | SBC #$01
032A | 208,246   | BNE $0322
032C | 96        | RTS
```

- De wacht ("wait") routine bevindt zich op $0320. 
  Deze routine gebruikt register X niet; dat is belangrijk omdat het 
  hoofdprogramma die gebruikt om af te tellen dat er 5 "blinks" zijn.
  De wachttijd van twee geneste 256-loops is ongeveer 1/3 seconde.
- Het hoofdprogramma staat op $0300.
- Register X telt de "blinks" af: 
  X krijgt zijn beginwaarde op $0300, de aftelling is op $0318 en de loop sprong terug staat op $0319.
- Op 0302-030A wordt de activity LED 1/3 seconde aangezet (bit 3 van $1C00 wordt gewist).
- Op 030D-0315 wordt de activity LED 1/3 seconde uitgezet (bit 3 van $1C00 wordt hoog gemaakt).


## De C64 uploader

In deze sectie bekijken we het "upload" programma voor de C64.
Het bevat het blinky programma van de vorige sectie in `DATA` statements.
We gebruiken het DOS commando "M-W" (memory write) om blinky in het 
1541 geheugen te schrijven, en daarna "M-E" (memory execute) om blinky uit 
te laten voeren. Ter controle hebben we ook nog een "M-R" (memory read)
om het geschreven programma terug te lezen.

Alle drie de memory opdrachten moeten gevolgd worden door een extra argument:
het adres (waar de bytes geschreven, gelezen, uitgevoerd moeten worden).
Dat zijn twee bytes in _little endian_ vorm, dat wil zeggen het 
_least significant byte_ eerst.

De schrijf en lees opdracht hebben daarna nog een argument: een byte 
die de lengte van de te schrijven/lezen data aangeeft.

Hieronder volgt het hele programma.
De `.` op regels 30, 32 and 44 is een cursor-links symbol.

```basic 
10 PRINT "BLINKY1541"
12 OPEN 1,8,15,"I0":REM PENNINGS 202607
20 AL=0:AH=3:A$=CHR$(AL)+CHR$(AH):D$=""
22 READD:IFD>=0THEND$=D$+CHR$(D):GOTO22
30 L=LEN(D$):PRINT "M-W";L;".#";
32 FOR I=0 TO L-1 STEP 32:PRINTI;".@";
34 :C$=MID$(D$,I+1,32):L$=CHR$(LEN(C$))
36 :PRINT#1,"M-W"CHR$(I);CHR$(AH);L$;C$
38 NEXT I:PRINT
40 PRINT"M-R";:PRINT#1,"M-R"A$;CHR$(L);
42 FOR I=1 TO L
44 :GET#1,B$:PRINT ASC(B$+CHR$(0));".";
46 NEXT I:PRINT
50 PRINT "M-E (BLINK 1541 LED 5 TIMES)"
52 PRINT#1,"M-E";A$
60 CLOSE 1:END
70 DATA 162,5,173,0,28,41,247,141,0,28
72 DATA 32,32,3,173,0,28,9,8,141,0,28
74 DATA 32,32,3,202,208,231,96,234,234
76 DATA 234,234,169,0,160,0,136,208,253
78 DATA 56,233,1,208,246,96,-1
```

- De regels 1x (regels 10 en 12) openen het disk drive commando kanaal (kanaal 15 op apparaat 8).
- De regels 2x bouwen de string `D$` die het blinky programma bevat, gelezen van de `DATA` regels.
  String `A$` is het adres ($0300) dat elk commando mee krijgt.
- De regels 3x doen meerdere memory writes "M-W" naar de disk van `D$` 
  (meerdere writes omdat een commando maximaal 40 bytes mag bevatten).
- De regels 4x zijn overbodig, ze lezen het zojuist geschreven blinky programma 
  terug van de disk drive - ter controle.
- Op regels 5x wordt het programma uitgevoerd.
  De activity LED blinkt vijf keer.
  
Dit program werkt op de echte C64 met de echte 1541 drive;
maar ook op de echte C64 met de Pi1541 drive emulator; 
en zelfs op VICE.

(end)
