---
name: dispensa-setup
description: Sets up "Dispensa", a personal fridge and pantry dashboard that reads grocery receipts from Gmail, tracks use-by dates, minimizes food waste and proposes healthy recipes based on the user's preferences. Use when the user says "set up Dispensa", "set up my fridge inventory", "configura la dispensa", "configurer Dispensa", "configurar Dispensa", or asks to start the food inventory and recipe system for the first time.
---

# Dispensa setup

Build the user's own Dispensa in one guided pass: a dashboard artifact, its database, and two scheduled tasks that read grocery receipts from Gmail. Talk to the user in their language, in plain words, without tool names or file paths. The dashboard is available in Italian, English, French and Spanish and can be switched later from the bottom of the page. Everything the user sees (questions, task names, the dashboard description, the closing message) is in the language they choose in step 2. "Dispensa" is the product name and stays the same in every language.

Use a task list. Its last step is verification.

## 1. Check what is available

Check silently, then report only what is missing:

- **Gmail connector**: a tool like `search_threads` from a Gmail connector. If absent, search the registry for Gmail and suggest connecting it. Without it the automatic receipt reading cannot work. Continue anyway, the dashboard still works with photos and notes.
- **Artifact and ArtifactData tools**: required. If absent, stop and say this Claude surface cannot host the dashboard.
- **Scheduling tools** (`create_trigger`, `update_trigger`, `fire_trigger`): needed for the daily sync and the refresh button. If absent, continue and say receipts will only be read when the user asks in chat.

## 2. Ask the setup questions

Ask once, in a single AskUserQuestion call where possible (at most four questions per call, ask the optional one only if the user seems interested in cost or speed):

1. Dashboard language: Italiano, English, Français, Español (preselect the one the user is writing in) and first name to use in the dashboard.
2. Cuisine preference (default "Italian home cooking"). Store it in English.
3. How many people they usually cook for (1, 2, 3, 4+).
4. Time of the daily receipt check (default 18:00).
5. Optional: a specific model for the manual refresh (leave the default unless they name one).

Take the time zone from the conversation context. Find the Gmail address the connector reads by running one search (`newer_than:7d`) and reading the recipient, or ask if unclear.

## 3. Explain how receipts arrive

Tell the user, briefly:

- Online grocery receipts (Whole Foods, Amazon Fresh, Instacart, supermarkets) must reach the connected Gmail inbox. Best setup: a dedicated Gmail account for receipts plus an automatic forwarding rule from their main mailbox for the store senders.
- Paper receipts: email a photo to that inbox, or upload photos directly in the dashboard's photo section.

Then tell them, in two or three lines, what Dispensa reads and keeps (see `references/privacy.md`): only grocery receipts are kept as item lists, every other email is skipped without being stored, and the data lives in their private dashboard.

## 4. Publish the dashboard

1. Copy `references/dashboard.html` (read it fully first) to the scratchpad as `dispensa.html`. Do not change its code.
2. Publish it with the Artifact tool: `icon: "recipe"`, title "Dispensa", the description from the table below in the chosen language, and capabilities:
   `{"db": {}, "user": {}, "sample": {"images": true}, "mcp": {"servers": [{"server": "Claude Code Remote", "tools": ["fire_trigger"]}]}}`.
   If the publish is refused because of the `mcp` entry, publish again without it and tell the user the refresh button will not work.
3. Keep the returned artifact URL.

## 5. Create the scheduled tasks

Read `references/sync-prompt.md`. Fill its placeholders: `{{OWNER}}`, `{{ARTIFACT_URL}}`, `{{INBOX}}`, `{{TIMEZONE}}`, plus the per-task values in its table.

Name both tasks in the chosen language:

| Language | Dashboard description | Manual task | Daily task |
|---|---|---|---|
| it | Dispensa: inventario, scadenze e ricette | Dispensa: controllo manuale | Dispensa: controllo giornaliero |
| en | Dispensa: inventory, use-by dates and recipes | Dispensa: manual check | Dispensa: daily check |
| fr | Dispensa : inventaire, dates limites et recettes | Dispensa : vérification manuelle | Dispensa : vérification quotidienne |
| es | Dispensa: inventario, fechas de caducidad y recetas | Dispensa: revisión manual | Dispensa: revisión diaria |

1. **Manual refresh**: `create_trigger` with no schedule (fired only by the dashboard button), named as in the table, `initiation: "human_request"`, notifications push on. Keep its id. Change its model with `update_trigger` only if the user asked for a specific model (for example a faster, cheaper one for refreshes).
2. **Daily sync**: `create_trigger` with `cron_expression: "CRON_TZ=<tz> <MM> <HH> * * *"` at the chosen time (shift a whole-hour time 1 to 15 minutes earlier), named as in the table, `initiation: "human_request"`, notifications push on.

Say in one sentence whether runs need approval, as each result reports.

## 6. Seed the database

`ArtifactData` `set` on collection `meta`, doc `state`:

```json
{"owner": "<name>", "cuisine": "<cuisine>", "lang": "<it|en|fr|es>", "servings": <n>, "manualTriggerId": "<manual trigger id>", "inbox": "<gmail address>", "timezone": "<IANA tz>", "lastSync": "", "processedEmails": []}
```

See `references/data-model.md` for every collection. Do not invent inventory. If the user lists what they have, add those items with conservative use-by dates from today.

## 7. First run and verification

1. Offer to run the first check now with `fire_trigger` on the manual task. It reads the last 30 days of receipts.
2. Verify: `ArtifactData get meta/state` returns the seeded values, and `list_triggers` shows both tasks enabled.
3. Close with four short lines, in the chosen language: what was created, how receipts get in, that the first time they tap refresh or send a note the page asks permission to use Claude, and that if they share the dashboard, the other person can add notes, photos and ratings but only they can start a receipt check.

## Rules carried into every run

- Privacy: store only what `references/privacy.md` lists. Never copy email text, senders, store names, prices or photos into the database.

- Metric units, item names in the dashboard language without brands, conservative use-by dates (table in the sync prompt).
- Rice, pasta and condiments are always assumed in stock and never tracked.
- Never mark items finished on the user's behalf unless they said so.
- Expiry dates are estimates. Remind the user once, at the end of setup, to trust the package label over the estimate.
