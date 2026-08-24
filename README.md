# Notify Raumlüften

Home Assistant Blueprint für Lüftungshinweise anhand der Luftfeuchtigkeit.

## Funktionen

- Überwachung eines oder mehrerer Luftfeuchtigkeitssensoren
- Konfigurierbarer Luftfeuchtigkeits-Grenzwert
- Konfigurierbare Mindestdauer über dem Grenzwert, um kurze Messwertspitzen zu ignorieren
- Hysterese gegen Benachrichtigungs-Spam bei Werten rund um den Grenzwert
- Berücksichtigung von Tür- und Fenstersensoren
- Keine Lüftungsempfehlung, wenn bereits ein Fenster oder eine Tür geöffnet ist
- Optionaler Hinweis, dass bereits gelüftet wird
- Optional erneute Benachrichtigung nach dem Schließen, wenn die Luftfeuchtigkeit weiterhin zu hoch ist
- Optional eine Entwarnung, sobald die Luftfeuchtigkeit wieder im Normalbereich liegt
- Beliebige Home-Assistant-`notify`-Entities als Ziel, z. B. Telegram, Alexa oder die Home Assistant Mobile App

## Installation

Die Blueprint-Datei kann direkt über Home Assistant importiert werden:

```text
https://raw.githubusercontent.com/sanftleben/notify-raumlueften/main/blueprints/automation/notify_raumlueften.yaml
```

Alternativ die Datei unter

```text
config/blueprints/automation/notify_raumlueften.yaml
```

ablegen und Home Assistant die Automationen bzw. Blueprints neu laden lassen.

## Konfiguration

### Luftfeuchtigkeitssensoren

Es können mehrere Sensoren ausgewählt werden. Für die Entscheidung, ob gelüftet werden soll, wird der höchste aktuell verfügbare Messwert verwendet.

### Grenzwert

Beispiel:

```text
Grenzwert: 60 %
```

Eine Benachrichtigung wird ausgelöst, wenn die Luftfeuchtigkeit den Wert für die konfigurierte Mindestdauer überschreitet.

### Mindestdauer

Standardmäßig 5 Minuten.

Damit wird verhindert, dass ein einzelner kurzer Messwertsprung sofort eine Benachrichtigung erzeugt.

### Hysterese

Standardmäßig 3 %.

Bei einem Grenzwert von 60 % und einer Hysterese von 3 % gilt:

- über 60 % → Lüften erforderlich
- unter 57 % → wieder normal
- zwischen 57 % und 60 % → Zustand bleibt unverändert

Dadurch wird verhindert, dass die Automation bei Messwerten wie `59 → 60 → 59 → 60 %` ständig zwischen den Zuständen wechselt.

### Tür- und Fenstersensoren

Alle ausgewählten Tür- und Fenstersensoren werden gemeinsam betrachtet.

Sobald mindestens einer davon geöffnet ist, gilt der Raum als bereits gelüftet. Eine normale Lüftungsempfehlung wird deshalb unterdrückt.

Wenn mehrere Fenster vorhanden sind, bleibt der Raum so lange im Lüftungszustand, wie mindestens eines davon geöffnet ist.

### Benachrichtigungen

Als Ziel werden Home-Assistant-`notify`-Entities ausgewählt. Dadurch ist der Blueprint nicht auf einen bestimmten Messenger festgelegt.

Beispiele:

- Telegram
- Home Assistant Mobile App
- Alexa
- andere `notify`-Integrationen

Mehrere Ziele können gleichzeitig ausgewählt werden.

## Verhalten

```text
                 Luftfeuchtigkeit > Grenzwert
                              │
                     Mindestdauer erreicht?
                         /            \
                       nein            ja
                        │              │
                        │       Fenster/Tür offen?
                        │          /          \
                        │        ja            nein
                        │        │               │
                        │        │        Benachrichtigung
                        │        │               │
                        │        └── Lüften ─────┘
                        │
                        └──────── warten

Fenster/Tür wird geschlossen
              │
              ▼
       noch über Grenzwert?
          /           \
        ja             nein
        │                │
        ▼                ▼
   erneut melden       nichts
```

Die Entwarnung verwendet die Hysterese. Bei 60 % Grenzwert und 3 % Hysterese wird erst unter 57 % eine Normalmeldung ausgelöst.

## Beispiel

Konfiguration:

- Schlafzimmer
- Luftfeuchtigkeit: 60 %
- Mindestdauer: 5 Minuten
- Hysterese: 3 %
- Fensterkontakt: Schlafzimmerfenster
- Benachrichtigung: Telegram

Ablauf:

1. Luftfeuchtigkeit steigt auf 61 %.
2. Nach fünf Minuten wird geprüft, ob der Wert weiterhin über 60 % liegt.
3. Ist das Fenster geschlossen, wird eine Telegram-Nachricht gesendet.
4. Das Fenster wird geöffnet.
5. Es wird keine weitere normale Lüftungsaufforderung gesendet.
6. Das Fenster wird geschlossen.
7. Liegt die Luftfeuchtigkeit noch über 60 %, wird erneut zum Lüften aufgefordert.
8. Sinkt sie anschließend unter 57 %, ist der Zustand wieder normal.

## Hinweis zu mehreren Sensoren

Wenn mehrere Luftfeuchtigkeitssensoren ausgewählt sind, wird für die Benachrichtigung und die Lüftungsentscheidung der höchste gültige Messwert verwendet. So löst beispielsweise ein feuchter Sensor im Badezimmer auch dann eine Meldung aus, wenn ein zweiter Sensor einen niedrigeren Wert meldet.
