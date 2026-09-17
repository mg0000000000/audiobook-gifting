---
name: book-gift
description: Send someone an Audible audiobook by drawing a claim code from the pool in Notion and drafting the email. Use whenever the owner says he wants to send a book to someone, names a book and a person, asks what books have gone out, asks which codes are still available or unclaimed, asks to chase an unredeemed one, or says he has just bought a book on Audible.
---

# Book gift

> "The owner" is whoever runs the gift programme — swap in your own ledger,
> mailbox and Audible store.


the owner sends audiobooks to people — usually the business canon
(*Traction*, *Good to Great*, *The Five Dysfunctions of a Team*, *The
Four Obsessions of an Extraordinary Executive*, *Rocket Fuel*, *How to
Be a Great Boss*).

## How this works

**Audible claim codes are bearer instruments.** A code isn't tied to an
email address or a name — whoever has it redeems it. The recipient name
on the gift certificate is cosmetic. So a code bought today can go to
anyone, months later.

That means the buying and the sending are separate jobs:

- **the owner buys**, in his own browser, whenever credits build up or a title
  runs out. Gift purchases are paid with **Audible credits** — his
  credits expire, so buying ahead converts an expiring credit into a
  code that doesn't.
- **This skill draws a code down** when he names a person, and writes
  the email.

Everything the skill *records* comes from **Notion and Gmail** — or whatever
your ledger and mailbox are. The
browser is used for one thing: when the pool has no code for the title
he wants, the skill drives Audible to the gift page (section 5) and
**the owner completes the purchase himself**.

**The ledger is the pool:** a Notion database — one row per code, with the fields in the table below. Put its URL here.

`Status` drives everything: **Available** (in the pool, unassigned) →
**Sent** (given to someone, not yet redeemed) → **Redeemed**.
**Unclaimed** is a Sent code still unredeemed after **3 months**.

---

## 1. Logging new codes — from his inbox, never from him

**the owner never pastes a code in.** When he buys a gift, Audible emails him
**"Your Audible gift is here 🎉"** within a minute or two, and the claim
code is in the body in plain text. The skill reads that email.

Whenever he says he's bought something, or before drawing from the pool
if the last sweep is more than a day old, search Gmail:

```
from:audible subject:"Your Audible gift is here"
```

For each gift email whose code is **not already in the ledger**, add one
**Available** row:

| Field | What goes in it |
|---|---|
| **Gift** | `<Book> — available` |
| **Book** | Title as it appears in the Audible email |
| **Code / link** | The claim code, plus `redeem at <your Audible store's redeem URL>` |
| **Status** | `Available` |
| **Date bought** | The email's date |
| **Paid with** | `1 credit` |
| Recipient, Company, Why, Date sent | Leave blank — filled in when it's drawn down |

One row per code. Never merge two codes into one row. Check the ledger
for the code before adding — the same email will be seen on every sweep.

## 2. When the owner names a person and a book

**a. Sweep the inbox first** (section 1), so a book he bought this
morning is in the pool.

**b. Draw from the pool.** Look for `Status = Available` matching the
title. Take the **oldest** one. If there's an `Unclaimed` code for that
title, offer it ahead of an Available one — reallocating a dead gift
beats spending a fresh code.

**c. If the pool is empty for that title, go and buy it** — section 5.
Don't draft an email promising a book that has no code behind it.

**d. Update the row:** Status → `Sent`, Recipient, Company, `Date sent` =
today, `Gift` → `<Book> — <Name>`, and **Why** — why this book, this
person, now.

**Update the ledger before drafting.** If the email step fails, the code
is already accounted for and can't be handed out twice.

## 3. Draft the email

Gmail, from the owner's own sending identity.

**One person, one email.** A fresh message to the recipient alone — no
reply-all, no cc, no thread. A gift goes to a person, not to a
conversation.

Shape:

- **Why this book, for them, now** — one or two lines tied to something
  they actually said or are wrestling with. This is the whole email.
  Generic "thought you'd enjoy this" is a failure.
- The redeem URL and the claim code, on their own line so it's easy to
  copy.
- A light close. No CTA — it's a gift, not a touch.

Four or five lines. Sound like the owner talks, not like a gift receipt.

If the owner supplies wording, **use it verbatim** — don't improve it.

**Leave it as a draft. the owner always presses send.**

Gmail drafts made through the API don't pick up his signature block —
mention it so he can paste it before sending.

## 4. The standing list — sent, unredeemed, 3 months on

Keep a **standing list of every book sent more than 3 months ago that
is still `Sent`**, and flip each of them to `Unclaimed`. Run it whenever
he asks what's outstanding, and whenever the skill is used for anything
else — it costs one query.

Unredeemed usually means the email went to spam, not that they didn't
want it. Report the list to the owner as: recipient, book, date sent, code.
He decides whether to nudge or reallocate.

Unclaimed codes are the first place to look next time he wants to send
that title.

## 5. Buying — the skill drives, the owner pays

When the pool is empty for a title, the skill takes the browser to the
gift page and stops there. **the owner completes the purchase.** Always
**Buy Now with 1 Credit**.

On your Audible store, signed in as the owner:

1. Search the title. **Check the author** — the search dropdown puts
   knock-offs and same-titled books right under the real one (there's a
   different *Traction* by Gabriel Weinberg & Justin Mares sitting
   directly below Gino Wickman's).
2. On the product page: **More options** → **Give as a gift**. It's not
   on the main button stack.
3. Delivery: choose **Email**, and put **the owner's own address** in the email
   field — not the recipient's.
4. Recipient's name / your name / note — **all cosmetic**, the code
   doesn't care. Put anything. Continue.
5. Stop at the purchase button and hand over. **the owner presses Buy Now with
   1 Credit.** If no credit is available, he decides whether to pay by
   card — the skill never presses either.
6. Audible sends **"Your Audible gift is here 🎉"** within a minute or
   two. Go back to section 1 and log the code from the inbox, then carry
   on with section 2.

**Never pick Email with the *recipient's* address in the field** — that
sends them a bare Audible template with none of the owner's words in it, and
the message is the point of the gift. Emailing *himself* is the fast
path to the code; emailing *them* skips him out of his own gift.

The **Print** option ("Print and hand-deliver yourself") also works, but
it's slower: the code isn't on the confirmation screen, so it means
opening the "Your recent Audible gift purchase" email, clicking **Print
your gift**, and reading the code off the certificate. Only use it if
the Email path is unavailable.

Under two minutes per book, and several can be done in one sitting.

## Hard rules

- **Never send.** Draft only.
- **Never press a purchase button.** The skill stops at the gift page.
- **Never hand out a code that isn't in the ledger as Available**, and
  never give the same code to two people.
- **Don't guess or reconstruct a claim code.** If it's not in the inbox
  or the ledger, it doesn't exist — say so.
- Record every code. An untracked code is a forfeited code.
