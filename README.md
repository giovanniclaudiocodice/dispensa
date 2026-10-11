# Dispensa

A personal fridge and pantry dashboard run by Claude, built to minimize food waste and recommend healthy recipes based on your preferences. *Dispensa* is Italian for pantry.

- Reads grocery receipts that reach a Gmail inbox (online orders from any store, or photos of paper receipts)
- Tracks quantities in metric units and conservative use-by dates
- Proposes 3 quick and 3 longer recipes, scaled to the number of people, using first what expires first
- Learns from thumbs up and down, "I cooked this", notes typed or dictated, and fridge photos
- Works in Italian, English, French and Spanish

## Before you install

Dispensa needs all of these. Without them some parts stop working:

| Requirement | Used for | If missing |
|---|---|---|
| A Claude plan with Artifacts and scheduled tasks | The dashboard and the daily check | Dispensa cannot run |
| Gmail connector | Reading receipts automatically | You can still add food with photos and notes |
| Claude Code Remote connector | The refresh button on the dashboard | Receipts are read only at the daily check |

A dedicated Gmail account that only receives forwarded grocery receipts is strongly recommended. See [PRIVACY.md](PRIVACY.md).

## Install

In Claude Code:

```
/plugin install dispensa --marketplace GITHUB_USER/dispensa
```

Or from your shell:

```
claude plugin marketplace add GITHUB_USER/dispensa
claude plugin install dispensa@dispensa
```

Then say "set up Dispensa" (or "configura la dispensa", "configurer Dispensa", "configurar Dispensa"). Claude asks a few questions, publishes your private dashboard, and schedules the checks. Setup takes about five minutes.

## Skills

| Skill | Use it to |
|---|---|
| `dispensa-setup` | Create your dashboard, its database and the two scheduled tasks (daily check and manual check) |
| `dispensa-cucina` | Ask what to cook or what is in the fridge, and update the inventory from chat |

## Sharing

You can share the dashboard with the people you live with. They can add notes, photos and ratings, mark recipes as cooked and change use-by dates. Only the person who set it up can start a receipt check from the refresh button, because the scheduled task belongs to their account. Their notes are applied at the next daily check.

## Notes

- Switch the language from the menu at the bottom of the dashboard. The interface, Claude's replies, new recipes and new item names all follow it. Existing items keep their names.
- Use-by dates are estimates. The package label always wins.
- Recipes are general suggestions, not medical or dietary advice.

## Privacy

Dispensa has no server of its own. Everything stays in your Claude account and your private dashboard. Full details in [PRIVACY.md](PRIVACY.md).

## License

[MIT](LICENSE)
