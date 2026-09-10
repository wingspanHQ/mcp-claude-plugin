---
name: checking-payments
description: Look up what a company owes its contractors through Wingspan and what has happened to a particular payment. Use for "what do we owe", "what did we pay last month", "how much is outstanding", "what is awaiting approval", "show me [name]'s payments", "what happened to this payment", "why has this not been paid yet", "has this been paid", "is this payment disputed".
---

# Checking payments

A **payable** is one payment owed to one contractor — the row on the Payables
screen in the Wingspan app. A **contractor** is a person or business the company
pays. An **engagement** is the named working arrangement a payment is filed
under.

Two tools cover this: `search_payables` lists and filters, `get_payable` reads
one payment in full. Every row a search returns carries a `payableId`, and that
is the id `get_payable` takes.

## Listing and searching

`search_payables` with no arguments lists every payment except cancelled ones,
25 to a page.

| Argument | What it does |
| --- | --- |
| `query` | Free-text search over the contractor's name, email and company, plus the invoice number. At least two characters; a full email address matches exactly. |
| `status` | Which screen view to read: `all`, `draft`, `toApprove`, `scheduled`, `paid`, `cancelled`. |
| `contractor` | One contractor's payments only — their `contractorId`, your external id for them, or their email. |
| `dueDateFrom`, `dueDateTo` | Bound the due date, as `YYYY-MM-DD`, inclusive. |
| `paidDateFrom`, `paidDateTo` | Bound the date paid, as `YYYY-MM-DD`, inclusive. |
| `sortBy` | `dueDate`, `createdAt`, `updatedAt`, `openedAt`, `paidAt`, `amount` or `scheduledPaymentDate`. One field per call. |
| `sortDirection` | `asc` or `desc`. Defaults to `desc`. Refused without `sortBy`. |
| `limit` | Rows per page, 1 to 100. Defaults to 25. |
| `pageToken` | Continue a previous page. Pass the previous result's `pagination.nextPageArgs` back unchanged. |
| `accountId` | Read one child account of an organization instead of the signed-in account. Only when the user names one; `who_am_i` lists them. |

There are no other filters. Do not invent one.

What each `status` view holds:

- `all` — every payment except cancelled ones.
- `draft` — created but not yet opened, so the contractor cannot see it.
- `toApprove` — open and waiting for someone at the company to approve it.
- `scheduled` — approved and waiting for a payroll run.
- `paid` — paid or in transit only. Payments recorded off-platform, partially
  paid or refunded are not in this view; use `all` for a complete history.
- `cancelled` — cancelled. Hidden everywhere else, which is why totals stay
  honest.

Every view, `all` included, also leaves out two kinds of row: the batch-level
total a payroll run creates, which would double-count against the individual
payments, and personal payment-link invoices. Neither can be read by id either.

## Totals

A list normally carries a `summary` with a count and an amount total, covering
**every** payment matching the filters, not only this page. Use it for "how
much do we owe" instead of adding up a page, and say which filters it covers.
When `summary` is null the totals were unavailable — say so rather than summing
the page and presenting it as the total.

## The words on a row

Each row's status is the wording the Wingspan app shows, not a raw code, so it
can be read to the user as-is.

| Row says | Means |
| --- | --- |
| Draft | Created and not yet opened. |
| Action required | Waiting on someone at the company: approval, a dispute the contractor raised, or something the contractor resubmitted. |
| Awaiting contractor | Open, but the contractor was not eligible for payment when it came up. |
| Scheduled | Approved and queued for a future payroll. |
| Paid | Paid, or the payment is in route. |
| Refunded | Refunded, in whole or in part. |
| Off-platform | A historical record of a payment made outside Wingspan. |
| Cancelled | Cancelled. |

One more value can appear on a row: `Unknown`, when the payment is in a state
the row wording has no pill for — a returned deposit is the common case. Read
`get_payable` for it; the detail headline names it, Returned. The money did not
land, the payment stays on the payroll run it was part of, and it is final for
that payment: a replacement is a new payment, created in the Wingspan app.

**Approval is a separate field from status.** A payment can be open and
unapproved, or open and approved; the status wording above folds that in, but
they are two different things underneath. Approving happens in the app.

## One payment in full

`get_payable` returns what the Payables detail panel shows:

- The panel headline and the row's status wording.
- Any alert on it. Two exist: the contractor has not finished setting up
  digital payments, and the contractor disputed the invoice — their reason is
  in the activity timeline.
- The amount, with the breakdown from gross to net and a named row per
  deduction.
- The line items, the due date, whether the due date was rescheduled, and the
  original date if it was.
- How it was paid, the attachments, any notes and purchase-order or project
  labels.
- `activity` — the full timeline of everything that has happened to it,
  newest first.

Quirks of the timeline are worth knowing before you conclude something did not
happen. At most two views of the invoice link are ever listed — the first, and
the first more than an hour after it — so a short timeline is not evidence the
contractor stopped looking. Views after the payment was paid are dropped. A
"due today" reminder sent on the same day the payment was opened is suppressed.

## What these tools cannot see

**Where the money is in the banking system.** No payout or bank-transfer
record is read, matching what the Payables screen itself shows. The furthest
either tool goes is the date the deposit was confirmed. A confirmed deposit
means the sending bank finished processing, not that the money is available or
final — a payment can still come back afterwards, and that shows up on the
payment itself, where `get_payable` names the state Returned. A question like
"has the bank transfer landed" or "why did the transfer fail" belongs in the
Wingspan app, or with Wingspan support.

**Payroll runs and funding sources.** A payroll run is the batch that funds and
pays a set of approved payments, and the funding source is the account Wingspan
debits to fund it. Neither is readable here. When a user asks why a scheduled
batch has not gone out, or which account funded it, send them to the Payroll
screens in the app.

## Why is this not paid yet

Work down this list.

1. `get_payable`. If the status is Draft, it was never opened. If it says
   action required, the secondary line on the row says which of the three it
   is — approval, a dispute or a resubmission. If it says awaiting contractor,
   the contractor was not eligible when the payment came up.
2. If the answer points at the contractor, `get_contractor` with the
   `contractorId` on the payment. Its `alert` gives the single reason —
   invited and not signed up, tax information not shared, archived, payments
   eligibility pending, an outstanding requirement, or one expired or
   expiring. The `finding-contractors` skill lists all eight outcomes.
3. If the payment is paid but the contractor says the money has not arrived,
   that is the banking question above: Wingspan app, or Wingspan support.

**"Awaiting contractor" is not a reason to cancel anything.** It means the
contractor was not eligible at the moment payment came up. Fix the contractor
and the payment carries on. Cancelling and recreating loses the record and,
because a payment's engagement is fixed when it is created, is sometimes
unrecoverable.

## Finish these in the Wingspan app

- Opening a draft, approving or unapproving a payment, rescheduling it,
  cancelling it, and paying it.
- Payroll runs, funding sources, invoices, payment splits and accounting
  integrations.
- Resolving a dispute with a contractor.
- Anything about a bank transfer after Wingspan has sent the payment.
