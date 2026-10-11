---
name: dispensa-cucina
description: Answers everyday questions about the user's Dispensa food inventory and updates it from chat. Use when the user asks "what should I cook", "what's in my fridge", "what's expiring", "cosa cucino stasera", "cosa ho in frigo", "qu'est-ce que je cuisine", "qué cocino hoy", says they ate, finished, froze or bought something, or asks to refresh or check their grocery receipts.
---

# Dispensa in chat

The user's data lives in the database of their "Dispensa" dashboard artifact. Read and write it only with the ArtifactData tool.

## Find the dashboard

1. Use the Artifact tool's `list` and look for a page titled "Dispensa". If there are several or none, ask the user for the link. If there is none, offer to run the `dispensa-setup` skill.
2. `ArtifactData get meta/state` for owner, cuisine, `lang`, servings, inbox and `manualTriggerId`. Write any new data (item names, recipes, feedback replies) in the `lang` language.

## Answer questions

- **What to cook**: list `recipes`. Group them by `slot` (if missing: simple rank 1 is tonight, other simple recipes are quick, complex ones are slow) and take the newest `batch` per slot (if missing, use `generated`). Lead with the tonight recipe, then the others, each with time and why. If the user wants something different, propose new recipes from non-gone inventory plus rice, pasta and condiments, prioritizing the earliest `useBy`, following `ratings` (never disliked dishes), scaled to `servings`.
- **What is in the fridge / what expires**: list `inventory` with status not "gone", sorted by `useBy`, with days left. Flag anything due today, tomorrow or overdue.

Answer in the language the user is writing in, briefly, as a short list.

## Update from chat

When the user reports a change, apply it the same way the dashboard does:

- Ate, finished, threw away: set `status` (half, low, gone).
- Froze: `storage: "Freezer"`, `useBy` 60 days from today.
- New expiry: set `useBy` and `useBySetBy: "user"`.
- Bought something: new `inventory/<english-slug>` doc with a short name in the dashboard language (`meta/state.lang`), metric qty, `bought` today, conservative `useBy`, status full, `source: "note"`.
- Cooked a recipe: subtract its ingredients and set statuses accordingly.

Read each doc first and pin writes with `if_version`. Use one `batch` for several writes. Then also add a `feedback` doc `{text, at, status: "applied", reply}` so the change shows in the dashboard history. Confirm in one sentence what changed.

## Refresh receipts

If the user asks to check receipts now and `meta/state.mode` is not `manual`, call `fire_trigger` with `meta/state.manualTriggerId` and say it takes a few minutes, it reads only the last 48 hours of email, and the dashboard updates by itself. In manual mode, say email is not connected: they can upload a receipt photo in the dashboard, tell you what they bought, or say "connect my email to Dispensa" to switch (the `dispensa-setup` skill does it).
