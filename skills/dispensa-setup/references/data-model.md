# Dispensa data model

All data lives in the dashboard artifact's database (ArtifactData tool, the artifact's `url`).

| Collection / doc | Fields | Written by |
|---|---|---|
| `meta/state` | `owner` (first name), `cuisine` (e.g. "Italian home cooking"), `lang` (it, en, fr, es; the dashboard, Claude in the page and the sync tasks all follow it), `mode` (`email`: receipts read from Gmail by the two scheduled tasks; `manual`: no email, food comes in only from photos and notes, no scheduled tasks; missing means `email`), `servings` (1 to 4, 4 = 4+), `manualTriggerId` (trig_…, empty in manual mode), `inbox` (Gmail address the connector reads, empty in manual mode), `timezone` (IANA), `lastSync` (ISO 8601 with offset), `processedEmails` [Gmail message ids, most recent 500] | setup, sync tasks, page (servings, lang) |
| `inventory/<slug>` | `name` (short, no brand, in the language active when it was added), `qty` (metric), `bought`, `useBy` (YYYY-MM-DD), `useBySetBy` ("user" if set by hand), `storage`, `status` (full, half, low, gone), `source` (email receipt, photo receipt, photo fridge, note; never a store name, sender or price) | sync tasks, page |
| `recipes/<slug>` | `tier` (simple, complex), `rank`, `slot` (tonight = the "suggested menu of the day" card, quick, slow; if missing, derived from tier and rank), `batch` (shared by recipes written together; if missing, `generated` is used), `servings`, `title`, `time_min`, `uses` [inventory ids], `why`, `ingredients` [strings starting with the metric quantity], `steps`, `generated`, `source` | sync tasks, page |
| `ratings/<slug>` | `title`, `vote` (1 or -1), `at` | page |
| `feedback/<id>` | `text`, `prompt`, `at`, `status` (open, applied), `reply`, `model` | page, sync tasks |
| `meta/shown` | `titles` [last 40 proposed dishes] | page |
| `meta/prefs` | `rules` [short standing preferences in the user's words, at most 30, e.g. "No vinegar"] | page (from notes, removable), sync tasks, chat |
| `meta/shopping` | `items` [{`name`, `qty`, `kind` (restock: keep the pantry stocked, unlock: enable more recipes), `why`}], `at` | page, sync tasks |

Seed only `meta/state` at setup. Leave inventory empty until the first receipt arrives, unless the user lists what they have, in which case create items with conservative use-by dates from today.
