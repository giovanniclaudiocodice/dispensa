# Privacy

Dispensa has no server, no account and no analytics of its own. It is a set of instructions that runs inside your own Claude account, with the connectors and permissions you grant there. This page explains what it reads, what it keeps, and how to remove it.

## What Dispensa reads

- **Your Gmail inbox, only if you choose email mode.** In photos-and-manual mode Dispensa never touches your email. In email mode, both the daily check and the refresh button search only emails from the last 48 hours, never older (emails dated earlier are skipped even if the search returns them). Each new email is opened once to decide whether it is a grocery receipt. The Gmail connector technically gives Claude access to the whole mailbox, so the safest setup is a dedicated Gmail account that only receives forwarded grocery receipts.
- **Photos you upload in the dashboard** (receipts or fridge shelves). The page sends them to Claude through its "ask Claude" permission, which claude.ai asks you to allow the first time. Claude reads the items and the photos are not stored.
- **Photos of paper receipts attached to emails** (email mode). The check copies the email into its temporary workspace to open the image. That workspace is private to the run and discarded when it ends. Only the extracted food items are kept.
- **Notes you type or dictate** in the dashboard, and the ratings you give recipes.

## What Dispensa keeps

Everything is stored in the database of your private dashboard page on claude.ai:

| Data | Why |
|---|---|
| Item names, quantities, purchase dates, use-by dates, storage place | The inventory |
| Recipes, ratings, recently proposed dish titles, preferences you state in notes (such as "no vinegar"), the shopping list | Recipe proposals, learning your taste and suggesting what to buy |
| Notes, the instruction sent to Claude with each note, Claude's one-line replies and the model that answered | Applying your changes and showing the history |
| First name, cuisine, number of people, language, mode, time zone, the inbox address (email mode only) | Personalizing the dashboard and the checks |
| IDs of the last 500 emails already checked (email mode only) | Not opening the same email twice. Only the ID is kept, never the content, sender or subject |

Dispensa does not copy emails, receipts, prices, store names, payment details or photos into its database. Your browser also remembers the text size and an in-progress check on that device only.

In email mode, two scheduled tasks are also saved in your Claude account. Their instructions contain your first name, the dashboard link, the inbox address and your time zone. After a check that finds new receipts, you get a short notification from Claude listing what was added.

## Who can see it

- The dashboard is private to you by default.
- If you share the page, the people you share it with can see and change the inventory, recipes, notes and ratings. They never get access to your Gmail. Only you can start an email check.
- Claude processes your data under your own agreement with Anthropic. Gmail stays with Google as before.
- The dashboard loads its font from Google Fonts, which means Google sees the viewer's IP address when the page opens. No other external service is contacted.

## How long it is kept

Until you delete it. The list of checked email IDs is capped at the most recent 500.

## How to remove everything

1. In email mode, delete the two scheduled tasks named "Dispensa" (daily check and manual check).
2. Delete the "Dispensa" dashboard page. Its database goes with it.
3. Optionally disconnect the Gmail connector and uninstall the plugin.

## Good to know

- Use-by dates are conservative estimates. The date on the package always wins.
- Recipes aim to be balanced, but they are general suggestions, not medical or dietary advice. Dispensa does not know about allergies or intolerances unless you tell it in a note, and even then, check every recipe yourself.
- Dispensa is not designed for or directed at people under 18.

Questions: open an issue at https://github.com/giovanniclaudiocodice/dispensa/issues.
