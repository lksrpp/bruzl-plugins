---
name: recipe-import
description: Transforms a user's recipe into a structured markdown format and imports it into their Bruzl cookbook. Use when the user wants to add, import, save, or format a recipe, or pastes/links a recipe and asks to put it in their cookbook. Requires the Bruzl MCP server.
---

# Bruzl Context

Bruzl (https://bruzl.de) is an online app to create, manage, and share recipes. Recipes are categorized and managed in cookbooks. 

The user needs your support to manage their recipes, specifically to format new recipes from various sources (URLs, pasted text, or images/PDFs) and import them into their cookbooks.

## Important

- **German is the default language for recipes.** Bruzl's default language is German. Regardless of the language you speak with the user, all imported recipes should be in German, unless explicitly stated otherwise by the user upfront.
- **Follow the workflow and format described below.** This ensures that all recipes are imported correctly and formatted consistently. Do not skip steps or improvise the format.
- **Never invent details**: Don't make up information that aren't included in the source recipe.

## Tools: Bruzl MCP server

The Bruzl MCP server exposes several tools you can leverage (plugin-namespaced as `mcp__plugin_bruzl_bruzl__<tool>` if installed via the `bruzl` plugin; bare tool names are used below):

- `list_cookbooks` — pick the target cookbook
- `list_categories` — get the valid `categories` names
- `create_recipe` — save the recipe to the db
- The MCP server provides various other tools you can use to support the user in managing their cookbook, but they are not required for importing a new recipe.

## Workflow

### 1. Precondition: MCP server availability

- Confirm the Bruzl MCP server is available and authenticated. 
- If the MCP server isn't connected, tell the user to install or enable it, and stop. You cannot save a recipe without it.

### 2. Resolve the target cookbook

- Call `list_cookbooks` for an overview of available cookbooks.
- Use the one with `isDefault: true` unless the user named a specific cookbook. Keep its `cookbookId` for `create_recipe`.

### 3. Get the source recipe

Depending on the user's input, you may need to fetch or read the recipe from different sources:
- **URL** — fetch it and read the recipe off the page.
- **Pasted text** — use it directly.
- **Image / PDF** — read the recipe from the attachment.

If you can't find a recipe in the source, say so and stop. If multiple recipes are present, ask which one you should import. You can only import one recipe at a time. 

### 4. Format the recipe

- Read the existing formatting rules in [`references/recipe-format.md`](references/recipe-format.md).
- Apply these rules to the source recipe. Don't improvise or deviate from these rules.
- The markdown format will be used as one string as the `content` argument for `create_recipe`.

### 5. Select valid categories

- Call `list_categories` for the chosen cookbook. 
- Select appropriate categories for the new recipe. If none genuinely fit, omit `create_recipe`'s `categories` argument.

### 6. Other metadata fields

- **`duration` (Gesamtdauer)**:
  - Format: `X Std. Y Min.` (e.g., `1 Std. 30 Min.`, `45 Min.`, `2 Std.`)
  - No `~`-approximation and no decimal places
- **General Rules**: 
  - Omit these fields (`duration`, `author`, `source`, `sourceUrl`, `cover`) if they cannot be unambiguously extracted from the source.
  - Never invent authors, sources, or URLs.

### 7. Confirm, then write the recipe

- Show the formatted result to the user: the title, the rendered Markdown body, chosen categories, and the metadata.
- Briefly highlight details the user might want to review in a concise bullet list. ONLY include:
  - Rounding from unit conversions (e.g., "1/3 cup → 80 ml (gerundet)")
  - Fields that couldn't be extracted (e.g., "Kein Autor im Rezepttext gefunden")
  - Substantive translations/renames (e.g., "Titel von Englisch übersetzt")
  - Ambiguous ingredients, unclear/missing steps, or unclear quantities
  - Don't highlight anything, if there is nothing noteworthy. No fillers, no trivial observations.
- Ask the user for changes or approval.
- On approval, call `create_recipe` and write the recipe to the db.
- If the call fails, surface the error message and offer to retry with a fix. Common errors include:
  - **Slug collision** (`A recipe with that title already exists…`) or **reserved slug**: surface the message and offer a different title; retry with the new one.
  - **Unknown category name**: drop the offending name(s) (re-check against `list_categories`) and retry.

### 8. Report

- Confirm the save and share the recipe's `url` with the user.
- Ask if you can support the user with any other tasks related to their cookbook or recipes with the capabilities of the Bruzl MCP server.

### 9. Full `create_recipe` example

Input recipe: *"Spicy Cherry Tomato Sauce with Fettuccine"* by Yotam Ottolenghi (NYT Cooking).

Calling `create_recipe` with the following args:

- **`cookbookId`**: <from `list_cookbooks`>
- **`title`**: `Scharfe Cherry Tomatensauce mit Fettuccine oder Spaghetti`
- **`duration`**: `1 Std. 30 Min.`
- **`author`**: `Yotam Ottolenghi`
- **`source`**: `NYT Cooking`
- **`sourceUrl`**: `https://cooking.nytimes.com/recipes/spiced-cherry-tomato`
- **`cover`**: `https://static01.nyt.com/images/2025/05/01/multimedia/cherry-tomato-jhpt/cherry-tomato-jhpt-master768.jpg`
- **`categories`**: `["Pasta", "Italienisch"]`
- **`content`**: the literal markdown string shown below (pass it verbatim — the MCP layer handles JSON/newline encoding; do not escape newlines yourself)

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