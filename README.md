# Audiobook gifting — two Claude skills

Two skills for running an audiobook gift programme on Audible with Claude.

The idea in one line: **Audible claim codes are bearer instruments.** They
aren't tied to a name or an email, and they don't expire. So buy with credits
when you have them, keep every code in one ledger, and reuse anything that goes
unclaimed.

| Skill | Job |
|---|---|
| [`book-gift`](skills/book-gift/SKILL.md) | Send someone a book: draw the oldest usable code from the ledger, draft one email to one person. If the pool is empty, drive Audible to the gift page and let the owner press Buy. |
| [`audible-gift`](skills/audible-gift/SKILL.md) | Keep the ledger honest: read Audible's gift history in the owner's browser, add unclaimed codes, mark redeemed ones. |

## Using them

1. Copy the `skills/` folder into your project's `.claude/skills/`.
2. Make a Notion database with the fields listed in `book-gift` and put
   its URL where the files say "Put its URL here".
3. Swap in your own sending address and Audible store.
4. Run `audible-gift` once across the account's full history to find any
   unredeemed gifts already sitting there — they go into the ledger as usable
   stock. After that, a rescan of the last 12 months is enough.

"The owner" throughout means whoever runs the programme — you.

Both skills draft; they never send. Neither presses a purchase button.
