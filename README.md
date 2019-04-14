# Contact Image Sensor (CIS) aus einem Epson Stylus SX100

Beschriftungen: FC11B913F56 F KTH0351-2

Die LEDs brauchen so 2-3 Volt, Strom ~20 mA bringt schon Licht.

die TO Pins am PCB sind: V+, Blau, Rot, Gruen.

Numerierung:
* Rohm: Pin 1 aussen.
* PCB: Pin 1 innen <- die nehmen wir

## Beleuchtung 

Die LEDs brauchen so 2-3 Volt, Strom ~20 mA bringt schon Licht.

die TO Pins am PCB sind: V+, Blau, Rot, Gruen.

Bei 5V Versorgung sollte mit folgenden Widerständen nach Erde gezogen werden fuer weisses Papier: Rot:50, Grün:50, Blau:220.

Spannung 1 mA:

| + \ - |  1 |  2 |  3 |  4 |  5 | Beschreibung des Pins              |
|-------|----|----|----|----|----|------------------------------------|
|     1 |  x |    |    |    |    | N/C (so weit sich erkennen laesst) |
|     2 |    |  x |    |    |    | LED GND Rot |
|     3 |    |    |  x |    |    | LED GND Gruen |
|     4 |    |    |    |  x |    | LED GND Blau |
|     5 |    |  R |  G |  B |  x | LED + |

## Datensignale

Spannung 1 mA:

| + \ - |    6 |    7 |    8 |    9 |   10 |   11 |   12 | Vias | Geometrie  | relativ | Vermutung | Beschreibung des Pins |
|-------|------|------|------|------|------|------|------|------|------------|---------|-----------|-----------------------|
|     6 |    x | 0.61 | 1.85 | 1.51 | 1.85 | 1.85 | 1.85 | 1    | dünn       |         |           | zwischen oberer Flaeche und Schrauben |
|     7 | 1.22 |    x | 1.34 | 0.99 | 1.34 | 1.34 | 1.33 | 2    | breit      | oben    | 3V3       | |
|     8 | 1.85 | 0.61 |    x | 1.51 | 1.85 | 1.85 | 1.82 | 1    | dünn       |         |           | von unten hochstechend |
|     9 | 0.41 | 0.35 | 0.41 |    x | 0.41 | 0.41 | 0.40 | 3-5  | breitestes | unten   | GND       | ueber #8 |
|    10 | 1.84 | 0.61 | 1.84 | 1.50 |    x | 1.84 | 1.83 | 1    | breit      | mitte   | Vref      | wo die Schrauben durchgehen (aber kein kontakt) |
|    11 | 1.85 | 0.61 | 1.85 | 1.51 | 1.85 |    x | 1.85 | 1    | dünn       |         |           | unter der Mittleren Flaeche, ueber #12 |
|    12 | 1.84 | 0.61 | 1.84 | 1.50 | 1.83 | 1.84 |    x | 1    | dünn       |         |           | ueber der unteren Flaeche, unter #11 |

## Insgesamt

sieht verdammt nach http://rohmfs.rohm.com/en/products/databook/datasheet/module/contact_image_sensor/flatbed/lsh3008-ca10a.pdf aus

* 6 Startpuls
* 8 Clock
* 10 Vref (sollte auf GND fest)
    * wenns hoeher ist, gibts kein gescheites bild mehr
* 11 DPI Mode
    * macht auf jeden Fall, wieviele Clocks man pro Zeile rausholen kann
    * 0/L: 300 dpi ~2600 clocks
    * 1/H: 600 dpi ~5268 clocks
    * nach anderen datenblaettern hat man 82 clocks “output period”, dann kommen die pixel (inkl. vorne und hinten dummypixeln)
    * 600 dpi: 5184 pixels
    * 300 dpi: 2592 pixels
* 12 Analog Out
    * bei steigender Clock gehts scheinbar los (kommt von GND hoch fuer jeden Pixel neu, gefuehlte 100ns hier)
    * bei fallender Clock sampeln klingt sinnvoll

## Operation

Startpuls wird bei fallender Clock gesampelt. Wenn Startpuls, dann werden die Werte in den Ausgabepuffer geschickt. Dann mit Clock rauscyceln. Belichtungszeit haengt wohl vom Intervall zwischen Startpulsen ab.

Was ich mache (funktioniert, kann man bestimmt noch reduzieren):

* Werte → Shiftregister, neue Belichtung starten:
    * Puls hoch
    * Clock hoch
    * warte 500us
    * Clock runter
    * warte 500us
    * Puls runter
* Shiftregister rausholen:
    * Clock rauf (Signal geht los)
    * warte 1us
    * Clock runter (Signal sollte stabil sein, kann man sampeln)
    * warte 1us
    * so oft wie es pixel gibt
* restliche Belichtungszeit geben
    * warte, 0-20 Millisekunden haben geklappt

## Arduino

 #define START 2
 #define CLOCK 3
 #define DPIMODE 4
 
 bool dpimode = 1;
 uint16_t pixels = dpimode ? 5184 : 2592;
 
 uint32_t linetime = 20000; // us
 uint32_t sched = 0;
 
 void setup() {
   pinMode(START, OUTPUT);
   pinMode(CLOCK, OUTPUT);
   pinMode(DPIMODE, OUTPUT);
 
   digitalWrite(DPIMODE, dpimode);
 
   sched = micros();
 }
 
 void loop() {
   digitalWrite(START, HIGH);
   digitalWrite(CLOCK, HIGH);
   digitalWrite(CLOCK, LOW);
   digitalWrite(START, LOW);
 
   for (uint16_t counter = 0; counter < 82 + pixels; counter += 1)
   {
     PORTD |= _BV(PORTD3);
     //delayMicroseconds(1);
     PORTD &= ~_BV(PORTD3);
     //delayMicroseconds(1);
   }
 
   sched += linetime;
   int32_t dt = sched - micros();
   while (dt > 0x4000)
   {
     delayMicroseconds(0x4000);
     dt -= 0x4000;
   }
   if (dt > 0)
     delayMicroseconds(dt);
 }
