---
name: audible-gift-sync
description: Sweep Audible's gift history for unclaimed gifts, pull each claim code, and sync them into the Notion Book Gift Ledger so unredeemed gifts can be reallocated instead of lost. Use when the owner asks to check or refresh the gift inventory, asks what's unclaimed or recoverable, asks to scrape or sync Audible, or wants to know how many codes are in stock before buying more. Runs in Chrome, in the owner's logged-in browser.
---

# Audible gift sync

> Shared by Matt Galt, EOS Worldwide, Sept 2026. "The owner" is whoever runs
> the gift programme — swap in your own ledger, mailbox and Audible store.


Audible tracks whether each gift has been redeemed. An implementer's account has
years of gifts that were sent and never claimed — every one is a paid
credit sitting idle, and **every one can be reallocated to someone
else**, because a claim code isn't bound to the person it was sent to.

This skill harvests them into the ledger so they become usable
inventory.

## Where this runs

**In Chrome, in the owner's logged-in browser** — Claude in Chrome, the
extension on his machine. Audible needs his session, and it's
unreachable from Claude Code sessions (egress-blocked, returns a
connection failure).

The work is read-only: no purchases, no account changes. Just reading
the gift table.

## The ledger

The same Notion database the `client-book-gift` skill uses. Put its URL here.

## Flow

### 1. Open the gift history — last 12 months only

`<your Audible store>/account/gift-history`

The default view is **Last 365 days**, and that is the right scope for a
rescan: it catches everything bought since the last
run and anything recently redeemed. Older gifts don't change state — if
someone hasn't redeemed in 12 months, they're not going to — and they are
already in the ledger.

**First run:** sweep every year the account has been gifting (year filter:
`?tf=giftpurchased&df=YYYY`) to build the ledger. After that, the last 365
days is enough.

The table has one row per gift sent:

| cover | Title | Date purchased | Recipient | Date sent | **Status** | Order details · **Print gift** |

**Status is the whole point.** It reads `Unclaimed` for anything not yet
redeemed. Anything else has been taken up and needs no action.

Some rows show an email address as the recipient; others show a name and
`Printed card` as the delivery, which is the print-your-own path. Both
behave the same — both have a claim code behind them.

### 2. Read the codes off the table — no certificates needed

**The claim code is in the "Print gift" link's URL** — the
`claimCode=` query parameter — on every Unclaimed row. Nothing has to be
clicked. Pull the whole table in one go with the Chrome `javascript_tool`:

```js
[...document.querySelectorAll('table tr')].slice(1).map(tr=>{
  const c=[...tr.querySelectorAll('td')].map(td=>td.innerText.trim().replace(/\s+/g,' '));
  const a=tr.querySelector('a[href*="claimCode="]');
  return {t:c[1]||c[0],d:c[2],to:c[3],s:c[5],code:a?new URL(a.href).searchParams.get('claimCode'):''};
}).filter(r=>r.s==='Unclaimed')
```

The tool's output truncates at ~1000 characters, so stash the array on
`window.__u` and return it in slices of ~13 rows, abbreviating titles.
`get_page_text` returns nothing on this page; `read_page` works but is
50KB a year — the script is the way.

Only the claimed/unclaimed status and the code matter. Claimed rows need
no action beyond the reverse reconcile in step 3.

Collect the whole page before writing to Notion, then write once.

### 3. Sync to the ledger

For each unclaimed gift, match against the ledger on **claim code
first**, then on recipient + title.

**Already in the ledger** → update it: set `Status` = `Unclaimed`, and
fill `Code / link` if it was a `PENDING` placeholder.

**Not in the ledger** → add a row. `notion-create-pages` takes up to 100
rows in one call — use it:

| Field | What goes in it |
|---|---|
| **Gift** | `<Book> — UNCLAIMED (<recipient>)` |
| **Recipient** | Whatever the history shows — email or name |
| **Book** | Title as Audible has it |
| **Code / link** | `<CODE>  —  redeem at <your Audible store's redeem URL>` |
| **Status** | `Unclaimed` |
| **Date bought** / **Date sent** | From the history columns |
| **Paid with** | `1 credit` unless the order says otherwise |
| **Why** | Where it came from and that it's recoverable |

**Also reconcile the other way.** Any ledger row marked `Sent`,
`Reallocated` or `Unknown` whose code is *not* in the Unclaimed set has
been redeemed — flip it to `Redeemed`. That's how the ledger stops
overstating what's recoverable.

**And the 3-month rule:** a ledger row still `Sent`
more than 3 months after its send date, whose code Audible still shows
Unclaimed, flips to `Unclaimed`. Rows sent within the last 3 months stay
`Sent` even if unredeemed — give people time.


### 4. Report

Tell the owner:

- how many unclaimed gifts there are, **broken down by title** — that's
  the number that decides whether he needs to buy anything
- anything newly discovered that wasn't in the ledger
- anything that flipped to Redeemed
- any row where the code couldn't be read

## Watch for

- **Duplicate claim codes.** If the same code turns up on two ledger
  rows, one of them is a copying error from the legacy sheet — Audible
  has exactly one gift per code, so find that gift and correct the row.
- **Truncated recipient emails.** The history column clips long
  addresses (`admin@thesu…`). **Order details** shows the full address
  if it matters; otherwise record what's visible and say it's truncated.
- **Codes don't expire.** Gifts bought in 2022 were still redeemable in
  2026, so an old unclaimed gift is as good as a new one. Never write
  one off for age.

## Hard rules

- **Read-only.** Never buy, never cancel a gift, never change account
  settings. This skill only reads and clicks "Print gift".
- **Never sign in as the owner.** If the session has expired, stop and say so.
- **Never transcribe a claim code by hand from a blurry render** — open
  the certificate and copy the text. A wrong character is a dead code.
- **Don't email anyone.** Sending a recovered code to a person is the
  `client-book-gift` skill's job, and the owner presses send.
