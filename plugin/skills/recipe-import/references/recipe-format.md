# Bruzl recipe formatting spec

> **Kept in sync with the AI service `recipe-transform` prompt** (`ai-service/src/cookbook_ai/llm/prompts.py` in the private `lksrpp/cookbook` repo). The German formatting rules below are lifted near-verbatim from that prompt; only the output framing differs — here you **call `create_recipe`** instead of emitting `RecipeResult` JSON. When that prompt's shared domain rules change, carry them over here; output-framing changes do not cross.

## Contents

1. [Output target — the reframe](#1-output-target--the-reframe)
2. [Sprache & Einheiten](#2-sprache--einheiten)
3. [Struktur des `content`-Felds](#3-struktur-des-content-felds)
4. [Detailregeln](#4-detailregeln)
5. [Wichtige Leerzeilen](#5-wichtige-leerzeilen)
6. [Metadaten-Felder](#6-metadaten-felder)
7. [Field mapping → `create_recipe` args](#7-field-mapping--create_recipe-args)
8. [Worked example](#8-worked-example)
9. [Surface to the user](#9-surface-to-the-user)

---

## 1. Output target — the reframe

The AI service runs these rules through a separate LLM call that returns a `RecipeResult` JSON object. **You don't.** You are the model, already in the loop. Apply the exact same German formatting rules, then **call the `create_recipe` tool** with the result. The recipe `content` is one Markdown string (`## ZUTATEN` / `## ZUBEREITUNG` / `## HINWEISE`); everything else maps to individual camelCase tool args (§7). There is no `review` field — surface those notes in chat instead (§9).

`content` ≤ 50 000 chars; `title` ≤ 200; `duration`/`author`/`source`/`sourceUrl`/`cover` ≤ 500 each.

---

## 2. Sprache & Einheiten

Halte dich genau an die Vorgaben aus dem Rezept, kürze und vereinfache den Text aber überall, wo keine wesentlichen Informationen verloren gehen.

- Übersetze alle Rezepte auf Deutsch
- Ausnahme: bekannte Eigennamen von Gerichten (z.B. Pad Thai, Ratatouille)
- Konvertiere Maßangaben in deutsche Formate:
  - Löffelmaße: tablespoon/tbsp → EL, teaspoon/tsp → TL (niemals in ml umrechnen)
  - Gewichte: lb/oz → g/kg
  - Cups/fl oz bei Flüssigkeiten → ml
  - Cups bei festen Zutaten (Gemüse, Kräuter, Mehl …) → g
  - Bereits metrische Angaben (ml, l, g, kg, EL, TL) unverändert übernehmen

---

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

---

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

---

## 5. Wichtige Leerzeilen

- Eine Leerzeile vor und nach jeder Überschrift (`##`, `###`)
- Eine zusätzliche Leerzeile zwischen dem letzten Zutateneintrag und `## ZUBEREITUNG`
- Eine zusätzliche Leerzeile zwischen dem letzten Zubereitungsschritt und `## HINWEISE`

---

## 6. Metadaten-Felder

Regeln für die Metadaten-Felder (alles außer `content`):

**`duration` (Gesamtdauer):**
- Format: `X Std. Y Min.` (z.B. `1 Std. 30 Min.`, `45 Min.`, `2 Std.`)
- Keine `~`-Näherung und keine Dezimalstellen

**Allgemeine Regeln:**
- Lass diese Felder (`duration`, `author`, `source`, `sourceUrl`, `cover`) **weg** (omit the arg), wenn sie nicht eindeutig aus dem Input hervorgehen
- Erfinde keine Quellen, Namen oder URLs

---

## 7. Field mapping → `create_recipe` args

The AI-service prompt's JSON keys are snake_case; the `create_recipe` tool args are **camelCase** and some are renamed. Map by this table — do **not** copy the JSON key names:

| `recipe-transform` JSON | `create_recipe` arg | Notes |
|---|---|---|
| `title` | `title` | — |
| `content` | `content` | the Markdown body (§3) |
| `duration` | `duration` | `X Std. Y Min.` (no `~`, no decimals) |
| `author` | `author` | omit when not unambiguous; never invent |
| `source` | `source` | — |
| `source_url` | **`sourceUrl`** | rename |
| `cover_url` | **`cover`** | rename; **must be a public http(s) image URL or omitted** — MCP has no upload path, and the tool rejects non-http(s) URLs. You cannot attach a local image. |
| `suggested_categories` | **`categories`** | rename; names from `list_categories` only; omit or `[]` if none fit |
| `review` | *(none)* | report in chat instead — see §9 |

`cookbookId` is also required — it comes from `list_cookbooks` at runtime (the default cookbook unless the user named another), not from the recipe.

---

## 8. Worked example

Input recipe: *"Spicy Cherry Tomato Sauce with Fettuccine"* by Yotam Ottolenghi (NYT Cooking).

Call `create_recipe` with these args (`cookbookId` from `list_cookbooks`):

- **`title`**: `Scharfe Cherry Tomatensauce mit Fettuccine oder Spaghetti`
- **`duration`**: `1 Std. 30 Min.`
- **`author`**: `Yotam Ottolenghi`
- **`source`**: `NYT Cooking`
- **`sourceUrl`**: `https://cooking.nytimes.com/recipes/spiced-cherry-tomato`
- **`cover`**: `https://static01.nyt.com/images/2025/05/01/multimedia/cherry-tomato-jhpt/cherry-tomato-jhpt-master768.jpg`
- **`categories`**: `["Pasta", "Italienisch"]`
- **`content`**:

```markdown
## ZUTATEN

### Sauce

- **75 ml** Olivenöl
- **2** Knoblauchzehen, dünn geschnitten
- **1 kg** Kirschtomaten, halbiert
- **½ TL** feiner Zucker (je nach Süße der Tomaten)
- **1** getrocknete Ancho-Chili, zerbrochen
- **20 g** Basilikumblätter
- **200 ml** Wasser
- Salz

### Pasta

- **400 g** Fettuccine (oder Spaghetti)
- **35 g** Parmesan, fein gerieben
- Salzwasser zum Kochen


## ZUBEREITUNG

### 1. Die Sauce (⏱️ ~1 Stunde)

1. _Öl_ (75 ml) in großer Bratpfanne bei mittlerer bis hoher Hitze erhitzen.
2. _Knoblauch_ (2 Zehen, dünn geschnitten) hinzufügen, ⏱️ 1 Min. braten bis karamellisiert.
3. _Tomaten_ (1 kg, halbiert), _Zucker_ (½ TL), _Chili_ (1, zerbrochen) und _Salz_ vorsichtig hinzufügen.
4. _Wasser_ (200 ml) darübergießen, ⏱️ 4 Min. rühren bis Tomaten weich werden.
5. Hitze reduzieren, ⏱️ 1 Std. köcheln lassen, gelegentlich umrühren bis eingedickt.
6. _Basilikum_ (20 g, ganz) unterrühren, warmhalten.

### 2. Die Pasta (⏱️ ~15 Minuten)

1. Großen Topf _Salzwasser_ zum Kochen bringen.
2. _Pasta_ (400 g) darin ⏱️ 10-12 Min. kochen (laut Packung, bis al dente).
3. Pasta abgießen und mit der Sauce vermischen.
4. Portionieren und mit _Parmesan_ (35 g, gerieben) bestreuen.


## HINWEISE

- ❄️ Sauce hält bis zu 5 Tage im Kühlschrank oder 1 Monat im Gefrierfach
- 🌶️ Ancho-Chili kann durch geräuchertes Paprikapulver ersetzt oder weggelassen werden
```

Note the inline `~` in stage headings (`⏱️ ~1 Stunde`) is allowed; the `~`-ban applies only to the `duration` **arg**.

---

## 9. Surface to the user

The AI service stores a `review` list; you don't have that field, so report the same notes **in chat** (briefly, after showing the formatted result) — only when noteworthy, no filler:

- Rounding from unit conversions (e.g. `1/3 cup → 80 ml (gerundet)`)
- Fields that couldn't be extracted (e.g. "no author in the source")
- Substantive translations/renames (e.g. "title translated from English")
- Ambiguous ingredients, unclear/missing steps, or unclear quantities

If there's nothing noteworthy, say nothing.
