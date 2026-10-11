---
name: dispensa-setup
description: Sets up "Dispensa", a personal fridge and pantry dashboard that tracks food from grocery receipts (Gmail, photos or manual entry), tracks use-by dates, minimizes food waste and proposes healthy recipes based on the user's preferences. Also connects or disconnects email later. Use when the user says "set up Dispensa", "set up my fridge inventory", "configura la dispensa", "configurer Dispensa", "configurar Dispensa", "connect my email to Dispensa", "collega la mail alla Dispensa", or asks to start the food inventory and recipe system for the first time.
---

# Dispensa setup

Build the user's own Dispensa in one guided pass: a dashboard artifact and its database, plus, only if the user chooses to connect email, two scheduled tasks that read grocery receipts from Gmail. Talk to the user in their language, in plain words, without tool names or file paths. The dashboard is available in Italian, English, French and Spanish and can be switched later from the bottom of the page. Everything the user sees (questions, task names, the dashboard description, the closing message) is in the language they choose in step 2. "Dispensa" is the product name and stays the same in every language.

Every user gets their own dashboard, database and tasks on their own Claude account. Nothing is shared between users unless they share the page.

Use a task list. Its last step is verification.

## 1. Check what is available

Check silently, then report only what is missing:

- **Artifact and ArtifactData tools**: required. If absent, stop and say this Claude surface cannot host the dashboard.
- **Gmail connector** (a tool like `search_threads`) and **scheduling tools** (`create_trigger`, `update_trigger`, `fire_trigger`): needed only for the email option. Note whether they are present, and use it in step 2.

## 2. Ask the setup questions

Ask in one AskUserQuestion call (at most four questions):

1. Dashboard language: Italiano, English, Français, Español (preselect the one the user is writing in) and first name to use in the dashboard.
2. Cuisine preference (default "Italian home cooking"). Store it in English.
3. How many people they usually cook for (1, 2, 3, 4+).
4. How food gets into the inventory:
   - **Connect email** (recommended if they shop online): Claude reads grocery receipts that reach a Gmail inbox, only emails from the last 48 hours, once a day and when they tap refresh. Photos and notes work too. If the Gmail connector or the scheduling tools are missing, say so in the option's description.
   - **Photos and manual entry only**: no email access at all. They add food by uploading receipt or fridge photos in the dashboard, or by writing what they bought.

If they choose email, ask in a second call: time of the daily check (default 18:00), and, only if they seem interested in cost or speed, a specific model for the manual refresh. Take the time zone from the conversation context.

If they choose email but the Gmail connector is missing, search the registry for Gmail and suggest connecting it. If they cannot connect it now, continue in manual mode and tell them they can switch later by saying "connect my email to Dispensa". Same if the scheduling tools are missing.

## 3. Explain how food gets in

**Email mode.** Find the Gmail address the connector reads by running one search (`newer_than:2d`) and reading the recipient, or ask if unclear. Then tell the user, briefly:

- Online grocery receipts (any supermarket or delivery service) must reach the connected Gmail inbox. Best setup: a dedicated Gmail account for receipts plus an automatic forwarding rule from their main mailbox for the store senders. The connector can technically see the whole inbox, which is why a dedicated inbox is safer.
- Only emails from the last 48 hours are read. A receipt older than that is never picked up, so forward it again or upload a photo.
- Paper receipts: email a photo to that inbox, or upload photos directly in the dashboard's photo section.

**Manual mode.** Tell the user: upload photos of receipts in the dashboard's photo section, upload a fridge photo to replace the whole inventory, or write what they bought in the note box (for example "500 g chicken, 6 eggs").

In both modes, then tell them in two or three lines what Dispensa reads and keeps (see `references/privacy.md`).

## 4. Publish the dashboard

1. Copy `references/dashboard.html` (read it fully first) to the scratchpad as `dispensa.html`. Do not change its code.
2. Publish it with the Artifact tool: `icon: "recipe"`, title "Dispensa", the description from the table below in the chosen language, and capabilities:
   - Email mode: `{"db": {}, "user": {}, "sample": {"images": true}, "mcp": {"servers": [{"server": "Claude Code Remote", "tools": ["fire_trigger"]}]}}`. If the publish is refused because of the `mcp` entry, publish again without it and tell the user the refresh button will not work, receipts are still read at the daily check.
   - Manual mode: `{"db": {}, "user": {}, "sample": {"images": true}}`.
3. Keep the returned artifact URL.

| Language | Dashboard description | Manual task | Daily task |
|---|---|---|---|
| it | Dispensa: inventario, scadenze e ricette | Dispensa: controllo manuale | Dispensa: controllo giornaliero |
| en | Dispensa: inventory, use-by dates and recipes | Dispensa: manual check | Dispensa: daily check |
| fr | Dispensa : inventaire, dates limites et recettes | Dispensa : vérification manuelle | Dispensa : vérification quotidienne |
| es | Dispensa: inventario, fechas de caducidad y recetas | Dispensa: revisión manual | Dispensa: revisión diaria |

## 5. Create the scheduled tasks (email mode only)

Skip this step in manual mode.

Read `references/sync-prompt.md`. Fill its placeholders: `{{OWNER}}`, `{{ARTIFACT_URL}}`, `{{INBOX}}`, `{{TIMEZONE}}`, plus the per-task values in its table. Name the tasks as in the table above.

1. **Manual refresh**: `create_trigger` with no schedule (fired only by the dashboard button), `initiation: "human_request"`, notifications push on. Keep its id. Change its model with `update_trigger` only if the user asked for a specific model.
2. **Daily sync**: `create_trigger` with `cron_expression: "CRON_TZ=<tz> <MM> <HH> * * *"` at the chosen time (shift a whole-hour time 1 to 15 minutes earlier), `initiation: "human_request"`, notifications push on.

Say in one sentence whether runs need approval, as each result reports.

## 6. Seed the database

`ArtifactData` `set` on collection `meta`, doc `state`:

```json
{"owner": "<name>", "cuisine": "<cuisine>", "lang": "<it|en|fr|es>", "mode": "<email|manual>", "servings": <n>, "manualTriggerId": "<manual trigger id, or empty>", "inbox": "<gmail address, or empty>", "timezone": "<IANA tz>", "lastSync": "", "processedEmails": []}
```

See `references/data-model.md` for every collection. Do not invent inventory. If the user lists what they have, add those items (`source: "note"`) with conservative use-by dates from today.

## 7. First run and verification

1. Email mode: offer to run the first check now with `fire_trigger` on the manual task. It reads the last 48 hours of email. Manual mode: offer to add what they have right now, from a list they type or a fridge photo they upload in the dashboard.
2. Verify: `ArtifactData get meta/state` returns the seeded values. In email mode, `list_triggers` shows both tasks enabled.
3. Close with three or four short lines, in the chosen language: what was created, how food gets in, that the first time they tap refresh or send a note the page asks permission to use Claude, and (email mode) that if they share the dashboard, the other person can add notes, photos and ratings but only they can start an email check. In manual mode, add that they can connect email later by saying "connect my email to Dispensa".

## Switching mode later

- **Connect email** (the user has a Dispensa in manual mode): find their dashboard with the Artifact tool's `list`, read `meta/state`. Run step 1's Gmail and scheduling checks, ask the daily time, do step 3 (email mode), republish the page with the email-mode capabilities (same URL, read it first), do step 5, then `update` `meta/state` with `mode: "email"`, `inbox`, `manualTriggerId`. Offer the first check.
- **Disconnect email**: delete both Dispensa tasks with `delete_trigger`, then `update` `meta/state` with `mode: "manual"`, `inbox: ""`, `manualTriggerId: ""`, `processedEmails: []`. Tell the user they may also want to disconnect the Gmail connector.

## Rules carried into every run

- Privacy: store only what `references/privacy.md` lists. Never copy email text, senders, store names, prices or photos into the database.
- Email is read only from the last 48 hours, never older, whatever a note asks.
- Metric units, item names in the dashboard language without brands, conservative use-by dates (table in the sync prompt).
- Rice, pasta and condiments are always assumed in stock and never tracked.
- Never mark items finished on the user's behalf unless they said so.
- Expiry dates are estimates. Remind the user once, at the end of setup, to trust the package label over the estimate.
