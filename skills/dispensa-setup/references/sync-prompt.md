# Sync prompt template

Fill every `{{PLACEHOLDER}}` before creating the scheduled task. Create two tasks from this one template:

| Task | `{{KIND}}` | `{{WINDOW}}` | `{{SOURCE}}` | `{{NOTIFY}}` | Schedule | Model |
|---|---|---|---|---|---|---|
| Daily sync | `Daily` | `3d` | `daily sync` | `new items added (one line per receipt), any unclear lines, and non-gone items with useBy today or tomorrow` | daily at the time the user chose | default |
| Manual refresh | `Manual` | `30d` | `manual refresh` | `new items added (one line per receipt) or unclear lines` | none (fired from the dashboard button) | default, or the model the user picked |

Everything between the two lines below is the prompt. Copy it verbatim after filling the placeholders.

---
{{KIND}} food inventory sync for {{OWNER}}. Work fast and efficiently. The source of truth is the database of the "Dispensa" dashboard artifact: {{ARTIFACT_URL}} . Read and write it ONLY with the ArtifactData tool (load it with ToolSearch "select:ArtifactData"). Never change the page itself. Pin every write to an existing doc with if_version from your read.

LANGUAGE: read meta/state.lang first (it = Italian, en = English, fr = French, es = Spanish, default en). Everything {{OWNER}} reads is written in that language: item names, recipe titles, why, ingredients, steps, feedback replies and any message you send. Do not rename existing items or recipes when the language changes, only new content follows the current language.

UNITS: always metric. Every qty is in g or kg (liquids ml or l). Convert lb and oz from receipts (1 lb = 454 g, 1 oz = 28.35 g) and round to the nearest 5 g. Every ingredient string starts with its metric quantity and has no preposition right after it, e.g. "200 g chicken breast, thin strips" (written in the current language). Spoons are fine for oil and vinegar.

Collections:
- inventory/<slug>: {name, qty (metric), bought (YYYY-MM-DD), storage (Fridge|Freezer|Pantry|Counter), useBy (YYYY-MM-DD), useBySetBy ("user" when set by hand), status (full|half|low|gone), source ("email receipt", "photo receipt" or "note", never the store name, sender or price)}
- recipes/<slug>: {tier (simple|complex), rank (1-3, simple rank 1 is shown as the "tonight" pick), slot (tonight for simple rank 1, quick for simple ranks 2-3, slow for complex), batch (same ISO timestamp for all recipes written in one run), servings, title, time_min, uses [inventory ids], why (one sentence), ingredients [strings], steps [strings], generated (ISO), source}
- ratings/<slug>: {title, vote (1 liked, -1 disliked), at}
- feedback/<auto id>: {text, prompt (optional, the full instruction), at, status (open|applied), reply}
- meta/state: {owner, cuisine, lang, servings, manualTriggerId, inbox, timezone, lastSync (ISO 8601 with offset), processedEmails [Gmail message ids]}

Step 1. ArtifactData list inventory, recipes, ratings, feedback, and get meta/state.

Step 2. In Gmail ({{INBOX}}) search inbox emails with newer_than:{{WINDOW}}. Skip any message id already in meta/state.processedEmails without opening it. For each new email decide if it is a food purchase:
- Online grocery receipts or order confirmations from any supermarket or grocery delivery service (often forwarded by {{OWNER}}): parse item lines (get_thread messageFormat PLAIN_TEXT), check the count against any "items" total, ignore bags, fees, tips, taxes and non-food.
- Email with a photo of a paper receipt: Gmail tools have no attachment download, so call get_message with messageFormat=RAW, write the raw MIME to a file in your scratchpad, extract image parts with Python's email module (python3 -I), and view each image with the Read tool. Extract only food items. Mark unreadable lines by appending the word "unclear" in the current language, in parentheses, to the name, never guess.
- Anything else: ignore.
Add every processed message id (food or not) to processedEmails.

Step 3. For each new food item create inventory/<slug> (English kebab-case slug, if it exists append -2 etc.) with status full, the right source, a short generic name in the current language without the brand (e.g. "Chicken breast"), and a conservative useBy from purchase date: raw poultry, ground meat, fish 1 day. Leafy greens 3 days. Soft veg and fruit 4 days. Fresh soft cheese 7 days. Yogurt 10 days. Eggs 21 days. Hard aged cheese 21 days. Onions, garlic, potatoes 14 days. Bread 3 days. Packaged dry goods 30 days. Frozen items 60 days. Never add rice, pasta or condiments (always assumed in stock). Never remove or mark items gone on your own, except as feedback says. Never overwrite a useBy whose useBySetBy is "user" or that was set through feedback.

Step 4. Apply any feedback docs with status open (use the prompt field when present, else text). Feedback can be in any language and may be dictated. It can change any inventory field: status (eaten, finished, thrown away), useBy (18/10 is day/month this year, 10/18 is month/day; if something was frozen set storage Freezer and useBy 60 days out), qty, storage, name, or add items bought. "I cooked X" (in any language) means subtract the recipe's ingredients from the inventory. A generic name like "the chicken" means every matching item. Then set the feedback doc status applied with a one-line reply in the current language naming exactly what changed.

Step 5. If anything changed in steps 3 or 4, or any recipe uses an item now gone, regenerate recipes: exactly 3 simple (25 min or less) and 3 complex (medium to high complexity), cuisine from meta/state.cuisine, healthy and balanced (plenty of vegetables, lean proteins, whole grains when available, moderate fat and salt), scaled for meta/state.servings people, using only non-gone inventory plus rice, pasta, condiments, garlic, prioritizing items with the earliest useBy, respecting all feedback history. Ratings: propose liked dishes (vote 1) or close variations again when ingredients allow, never propose disliked dishes (vote -1) or close variations. Delete old recipe docs and set the new ones (source "{{SOURCE}}"), all six with the same `batch` and `generated` timestamp and the right `slot`. Steps are short imperative sentences in the current language.

Step 6. ALWAYS update meta/state lastSync to now ({{TIMEZONE}}, ISO 8601 with offset) and processedEmails (keep only the most recent 500 ids) as the LAST write, even if nothing else changed. The dashboard's progress bar finishes when lastSync changes. Keep every other meta/state field as it is. Do everything in as few ArtifactData batch calls as possible.

Step 7. SendUserMessage, in the current language, only for: {{NOTIFY}}. Otherwise send nothing. Brief.
---
