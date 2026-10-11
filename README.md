# Dispensa

A personal fridge and pantry dashboard run by Claude, built to minimize food waste and recommend healthy recipes based on your preferences. *Dispensa* is Italian for pantry.

- Gets your food in two ways, your choice at setup: reading grocery receipts from a Gmail inbox (last 48 hours only), or just photos and manual entry with no email access
- Tracks quantities in metric units and conservative use-by dates
- Proposes 3 quick and 3 longer recipes, scaled to the number of people, using first what expires first
- Learns from thumbs up and down, "I cooked this", notes typed or dictated, and fridge photos
- Works in Italian, English, French and Spanish

## Before you install

At setup you choose how food gets into your inventory:

| Mode | How food gets in | What you need |
|---|---|---|
| Email | Grocery receipts reaching a Gmail inbox, read once a day and when you tap refresh, only from the last 48 hours. Photos and notes work too | A Claude plan with Artifacts and scheduled tasks, the Gmail connector, the Claude Code Remote connector for the refresh button |
| Photos and manual | Receipt photos, fridge photos, or writing what you bought | A Claude plan with Artifacts |

You can switch later by saying "connect my email to Dispensa" (or asking to disconnect it). In email mode, a dedicated Gmail account that only receives forwarded grocery receipts is strongly recommended. See [PRIVACY.md](PRIVACY.md).

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
| `dispensa-setup` | Create your dashboard and its database, connect or disconnect email, and (in email mode) schedule the daily and manual checks |
| `dispensa-cucina` | Ask what to cook or what is in the fridge, and update the inventory from chat |

## Sharing

Every user who installs Dispensa gets their own dashboard, inventory, tasks and email on their own Claude account.

You can also share your dashboard with the people you live with. They can add notes, photos and ratings, mark recipes as cooked and change use-by dates, all applied right away with their own Claude. Only the person who set it up can start an email check, because the scheduled task and the Gmail connection belong to their account.

## Notes

- Switch the language from the menu at the bottom of the dashboard. The interface, Claude's replies, new recipes and new item names all follow it. Existing items keep their names.
- Use-by dates are estimates. The package label always wins.
- Recipes are general suggestions, not medical or dietary advice.

## Privacy

Dispensa has no server of its own. Everything stays in your Claude account and your private dashboard. Full details in [PRIVACY.md](PRIVACY.md).

## License

[MIT](LICENSE)
