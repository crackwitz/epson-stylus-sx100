# Contact Image Sensor (CIS) aus einem Epson Stylus SX100

Beschriftungen: FC11B913F56 F KTH0351-2

Die LEDs brauchen so 2-3 Volt, Strom ~20 mA bringt schon Licht.

die TO Pins am PCB sind: V+, Blau, Rot, Gruen.

Numerierung:
* Rohm: Pin 1 aussen.
* PCB: Pin 1 innen <- die nehmen wir

Spannung 1 mA:

| + \ - |  1 |  2 |  3 |  4 |  5 | Beschreibung des Pins              |
|-------|----|----|----|----|----|------------------------------------|
|     1 |  x |    |    |    |    | N/C (so weit sich erkennen laesst) |
|     2 |    |  x |    |    |    | LED GND Rot |
|     3 |    |    |  x |    |    | LED GND Gruen |
|     4 |    |    |    |  x |    | LED GND Blau |
|     5 |    |  R |  G |  B |  x | LED + |

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

Vermutungen:
* 7,9: Versorgung
* 12,11,10,8,6: Daten

sieht verdammt nach http://rohmfs.rohm.com/en/products/databook/datasheet/module/contact_image_sensor/flatbed/lsh3008-ca10a.pdf aus

## coloring code

    import re, colorsys
    
    rex = re.compile('(?:<span[^>]*>)?([0-9].[0-9]{2})(?:</span>)?')
    
    def color(value, min, max):
        value = (value - min) / (max - min)
        (r,g,b) = [int(v*255) for v in colorsys.hls_to_rgb(value, 0.85, 1.0)]
        return "#{:02x}{:02x}{:02x}".format(r,g,b)
    
    def colorize(match):
        value = match.group(1)
        return '<typo bg:{1}>{0}</typo>'.format(value, color(float(value), 0.4, 2.5))
    
    source = open("foo.txt").read()
    open("foo.txt", "w").write(rex.sub(colorize, source))

