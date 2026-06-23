# Formatierungsregeln für Rezepte auf Bruzl

## 1. Wichtige Formatierungsregeln

Halte dich genau an die Vorgaben aus dem Rezept, kürze und vereinfache den Text aber überall, wo keine wesentlichen Informationen verloren gehen.

## 2. Sprache & Einheiten

- Übersetze alle Rezepte auf Deutsch
- Ausnahme: bekannte Eigennamen von Gerichten (z.B. Pad Thai, Ratatouille)
- Konvertiere Maßangaben in deutsche Formate:
  - Löffelmaße: tablespoon/tbsp → EL, teaspoon/tsp → TL (niemals in ml umrechnen)
  - Gewichte: lb/oz → g/kg
  - Cups/fl oz bei Flüssigkeiten → ml
  - Cups bei festen Zutaten (Gemüse, Kräuter, Mehl …) → g
  - Bereits metrische Angaben (ml, l, g, kg, EL, TL) unverändert übernehmen

## 3. Struktur des `content`-Felds

```
## ZUTATEN

### [Komponente 1]

- **[MENGE]** [ZUTAT], [Zubereitungshinweis]
- **[MENGE]** [ZUTAT], [Zubereitungshinweis]
- [...]

### [Komponente 2]

- **[MENGE]** [ZUTAT]
- [...]

## ZUBEREITUNG

### 1. [Hauptschritt] (⏱️ [Zeitdauer])

1. _[Zutat]_ ([Menge]) [Anweisung].
2. _[Zutat]_ ([Menge], [Zubereitungshinweis]) hinzufügen, ⏱️ [Zeit] [Anweisung].
3. [...]

### 2. [Hauptschritt] (⏱️ [Zeitdauer])

1. [Anweisung].
2. [...]


## HINWEISE

- [EMOJI] [Hinweis zur Aufbewahrung/Haltbarkeit]
- [EMOJI] [Hinweis zu Alternativen]
```

## 4. Detailregeln

**Zutaten (`## ZUTATEN`):**
- Jede Zutat: `- **[Menge]** [Zutat]` (Menge fett)
- Optionale Zubereitungshinweise nach Komma: `- **200 g** Kirschtomaten, halbiert`
- Verwende `### Unterüberschriften` für Zutatgruppen (Sauce, Teig, etc.) — aber NUR wenn die Zutatenliste lang genug ist und natürliche Gruppen hat
- Teile die gleiche Zutat NICHT über mehrere Gruppen auf

**Zubereitung (`## ZUBEREITUNG`):**
- Gliedere in nummerierte `### Hauptschritte` mit Zeitangabe: `### 1. Die Sauce (⏱️ ~1 Stunde)`
- Innerhalb der Schritte: nummerierte Einzelschritte (1. 2. 3.)
- Referenziere Zutaten kursiv mit Menge und Zubereitungshinweis: `_Knoblauch_ (2 Zehen, dünn geschnitten)`
- Zutaten ohne Mengenangabe werden ohne Klammer referenziert: `_Salz_`, `_Pfeffer_`
- Inline-Zeitangaben: `⏱️ 1 Min.`

**Hinweise (`## HINWEISE`):**
- Nur wenn relevante Hinweise im Rezept vorhanden sind (Aufbewahrung, Alternativen, Tipps)
- Jeder Hinweis mit passendem Emoji-Prefix (❄️, 🌶️, 💡, 🔄, etc.)
- Weglassen wenn keine Hinweise vorhanden — dann **kein** `## HINWEISE` Abschnitt

## 5. Wichtige Leerzeilen

- Eine Leerzeile vor und nach jeder Überschrift (`##`, `###`)
- Eine zusätzliche Leerzeile zwischen dem letzten Zutateneintrag und `## ZUBEREITUNG`
- Eine zusätzliche Leerzeile zwischen dem letzten Zubereitungsschritt und `## HINWEISE`