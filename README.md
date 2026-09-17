# EOS audiobook gifting — two Claude skills

Two skills for running an audiobook gift programme with Claude, shared by
Matt Galt (Certified EOS Implementer®, EOS Worldwide).

The idea in one line: **Audible claim codes are bearer instruments.** They
aren't tied to a name or an email, and they don't expire. So buy with credits
when you have them, keep every code in one ledger, and reuse anything that goes
unclaimed. A first sweep of a four-year-old account found 126 unredeemed gifts,
every one still usable.

| Skill | Job |
|---|---|
| [`client-book-gift`](skills/client-book-gift/SKILL.md) | Send someone a book: draw the oldest usable code from the ledger, draft one email to one person. If the pool is empty, drive Audible to the gift page and let the owner press Buy. |
| [`audible-gift-sync`](skills/audible-gift-sync/SKILL.md) | Keep the ledger honest: read Audible's gift history in the owner's browser, add unclaimed codes, mark redeemed ones. |

## Using them

1. Copy the `skills/` folder into your project's `.claude/skills/`.
2. Make a Notion database with the fields listed in `client-book-gift` and put
   its URL where the files say "Put its URL here".
3. Swap in your own sending address and Audible store.

"The owner" throughout means whoever runs the programme — you.

Both skills draft; they never send. Neither presses a purchase button.
