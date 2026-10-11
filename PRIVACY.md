# Privacy

Dispensa has no server, no account and no analytics of its own. It is a set of instructions that runs inside your own Claude account, with the connectors and permissions you grant there. This page explains what it reads, what it keeps, and how to remove it.

## What Dispensa reads

- **Your Gmail inbox, through the Gmail connector.** The daily check searches emails from the last 3 days, the manual check the last 30 days. Each new email is opened once to decide whether it is a grocery receipt. The Gmail connector technically gives Claude access to the whole mailbox, so the safest setup is a dedicated Gmail account that only receives forwarded grocery receipts.
- **Photos you upload in the dashboard** (receipts or fridge shelves). They are sent to Claude to read the items and are not stored.
- **Notes you type or dictate** in the dashboard, and the ratings you give recipes.

## What Dispensa keeps

Everything is stored in the database of your private dashboard page on claude.ai:

| Data | Why |
|---|---|
| Item names, quantities, purchase dates, use-by dates, storage place | The inventory |
| Recipes, ratings, recently proposed dish titles | Recipe proposals and learning your taste |
| Notes and Claude's one-line replies | Applying your changes and showing the history |
| First name, cuisine, number of people, language, time zone, the inbox address | Personalizing the dashboard and the checks |
| IDs of the last 500 emails already checked | Not opening the same email twice. Only the ID is kept, never the content, sender or subject |

Dispensa does not copy emails, receipts, prices, store names, payment details or photos into its database. Your browser also remembers the text size and an in-progress check on that device only.

## Who can see it

- The dashboard is private to you by default.
- If you share the page, the people you share it with can see and change the inventory, recipes, notes and ratings. They never get access to your Gmail. Only you can start a receipt check.
- Claude processes your data under your own agreement with Anthropic. Gmail stays with Google as before.
- The dashboard loads its font from Google Fonts, which means Google sees the viewer's IP address when the page opens. No other external service is contacted.

## How long it is kept

Until you delete it. The list of checked email IDs is capped at the most recent 500.

## How to remove everything

1. Delete the two scheduled tasks named "Dispensa" (daily check and manual check).
2. Delete the "Dispensa" dashboard page. Its database goes with it.
3. Optionally disconnect the Gmail connector and uninstall the plugin.

## Good to know

- Use-by dates are conservative estimates. The date on the package always wins.
- Recipes aim to be balanced, but they are general suggestions, not medical or dietary advice. Dispensa does not know about allergies or intolerances unless you tell it in a note, and even then, check every recipe yourself.
- Dispensa is not designed for or directed at people under 18.

Questions: open an issue on the repository.
