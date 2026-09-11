# The Wingspan tool surface

Every tool this plugin provides, and the rules all of them follow.

Terms used below: a **payer** is the company doing the paying; a
**contractor** (called a *payee* inside Wingspan) is a person or business it
pays; an **engagement** is a named working arrangement a contractor is
assigned to; a **payable** is one payment owed to one contractor; a
**requirement** is something a contractor must satisfy before they can be
paid.

## The eight tools

| Tool | What it answers | Reads or writes |
| --- | --- | --- |
| `who_am_i` | Does the connection work, and which account is it reading? | Reads |
| `search_contractors` | Who do I pay, and who matches "X"? Lists or searches the Contractors table, filtered by screen tab, onboarding state, engagement, import batch or a named requirement; sorted and paginated. | Reads |
| `get_contractor` | Can I pay contractor Y, and if not why? One contractor in full, in the detail screen's own words, with the screen's single alert as the verdict. | Reads |
| `search_requirements` | What must a contractor satisfy before I can pay them, and which of those actually block? Lists the payer's requirement definitions. | Reads |
| `search_payables` | What do I owe, or have I paid, and which payments match "X"? Lists or searches the Payables views; sorted and paginated, with a summary for the whole filtered set when totals are available. | Reads |
| `get_payable` | What happened to payment Y? One payment in full, in the detail panel's own words, with its activity timeline. | Reads |
| `create_contractors` | Onboard these contractors: create each one, assign an existing engagement, send the invite. | **Writes** |
| `create_payables` | Log payments for contractors against their engagements, as drafts. | **Writes** |

`who_am_i` answers a question about the connection rather than about the
business. It is there because it is the only honest way to answer "does this
work?" — it performs a real authenticated read scoped to the caller
themselves, which cannot succeed for the wrong identity. That also makes it the
first thing to run when anything looks wrong.

## Listing and searching

`search_contractors` and `search_payables` share one contract.

- **One page per call.** Neither tool fetches more than one page. Each returns
  `pagination.nextPageToken` and `pagination.nextPageArgs` — the complete
  argument object for the follow-up call — so the next request cannot drift
  from the one that produced the token. A token is only valid for the exact
  filters and sort that produced it.
- **The `note` says what to do next**, as plain sentences: how many rows of
  roughly how many, that the assistant should show these and offer the next
  batch rather than fetching it unasked, how the list is sorted and which
  fields are sortable, and that a free-text `query` makes relevance the
  primary order. The `sorting` block carries the same facts as data.
- **Filters are the screens' own tabs.** The one addition is `search_payables`'
  `cancelled`, because cancelled payments are hidden unless the filter asks for
  them. What each Payables view holds, including the two kinds of row every
  view leaves out, is written once in the `checking-payments` skill.
- **Contractors have two separate status axes.** `status` is the tab strip, and
  it mixes the working relationship with paperwork — `incomplete` and
  `complete` are a summary of requirements. `onboarding` is the relationship
  on its own: `Pending`, `Active`, `Inactive`, and every row reports it. The
  two combine. The one contradictory pair, `status: "active"` with
  `onboarding: "Inactive"`, is refused rather than answered with a confident
  empty page. `onboarding: "Inactive"` is also the one value the underlying
  search cannot express exactly: it means "archived or the invite was
  rejected", and a filter cannot express "or" here, so it matches the archived
  half and the `note` says so.
- **"Who still has X outstanding" is a requirement-scoped filter.**
  `search_contractors` takes `requirement` — a `requirementDefinitionId`, or a
  definition's exact name from `search_requirements`, matched
  case-insensitively — plus `requirementState`: `incomplete` by
  default, or `complete`, `pendingReview`, `expiring`, `expired`. The
  requirement and the state travel together as one condition, so a contractor
  whose insurance is complete and whose tax form is outstanding is never
  matched by a search for "insurance outstanding".
- **The words are the app's words.** The requirements and tax-ID labels, the
  payment status pill and the detail headline are all the wording the Wingspan
  app shows. Where the app writes a date into a label ("Payout cleared • Sep
  04"), the date is reported separately from the label. Calendar dates are
  `YYYY-MM-DD`; timestamps carry a time.

## Reading one record

`get_contractor` and `get_payable` each answer for one record, and each takes
the id its search sibling reported: `contractorId` or `payableId`, plus
nothing but the optional `accountId`. **Every search row carries the id its
`get` sibling takes**, and ids from anywhere else are generally rejected.

`get_contractor` returns `alert`, the detail screen's single verdict. At most
one of its outcomes applies, a fixed order decides which, and when no check
fires it is empty — a contractor who
is archived and never shared tax information reads "Tax information not
shared", because that check comes first. Wingspan has no single "can I pay this
person" field; this verdict is the closest thing to one. Every outcome, and what
each one means, is listed once in the `finding-contractors` skill.

`get_payable` returns the panel's alerts, the gross-to-net totals with a named
row per deduction, and `activity`, the full event timeline, newest first. Some
of the timeline's rules are easy to lose. At most two views of the invoice link
are ever listed — the first, and the first more than an hour after it — so a
short timeline is not evidence the contractor stopped looking. Views after the
payment was paid are dropped. And a "due today" reminder sent the same day the
payment opened is suppressed.

What it does **not** report is where the money is in the banking system. The
Payables screen reads no payout record and neither does this tool; the
furthest either goes is the date the deposit was confirmed.

There is no filter for a single eligible-or-not verdict, because eligibility
only reads as satisfied when every active assignment is payable. The
requirement-scoped filter
above answers the operational version instead — which named requirement is
holding whom up — and `search_requirements` reports, per definition, the
`blocking` value Wingspan holds for it and the `gracePeriodDays` window: the
days from when a requirement is attached to a contractor before an outstanding
copy starts holding up a payment. Inside that window an unfinished requirement
does not block.

## Writing

`create_contractors` and `create_payables` change state; everything else
reads. Both follow the same rails.

- **`mode` defaults to `preview`.** A preview resolves every id, validates
  every row and reports exactly what `apply` would do, without writing
  anything. The client's own approval prompt is a second gate, not the first.
- **`apply` requires a `requestId` you choose**, from which every write derives
  a key of its own. Retrying an apply with the same `requestId` and the same
  refs returns the result of the original writes; nothing new is created. A
  different request under the same id is refused rather than silently accepted.
- **Rows are independent.** One bad row never stops the others, and every row
  reports its own outcome.
- **Rows are identified by `ref`**, your own label. Refs and resource ids go
  into the output; names and email addresses do not, error messages included.
  Send the same refs on `apply` that you sent on `preview` — refs, not row
  order, are what the keys are built from.
- **Fifty rows per call.** Over that is an error naming the limit, never a
  silent truncation. These are synchronous calls, not bulk importers.

`create_contractors` invites everyone not yet on board: the contractors it
created, and any that already existed but never came on board. That second
group is what makes a retry safe — a retry of a half-finished attempt finds
the records the first attempt created, reports them as skipped, and still gets
them invited. Contractors already on board are left alone.

`create_payables` stops at a draft the contractor cannot see, with no payment
scheduled. Opening and approving happen in the Wingspan app, and paying is
permanently out of reach here.

## Two things that cannot be done differently

**A payment's engagement is fixed when it is created.** There is no way to
change it afterwards and no bulk reassignment. That is why `create_payables`
creates payments *under* an engagement rather than moving existing ones into
one, and why "add these payments to an engagement" and "log payments against
this engagement" are the same call.

**Onboarding cannot be one request.** A contractor record carries no
engagement and no invite of its own, so onboarding is three separate steps: the
record, an assignment per engagement, and the invite. `create_contractors` does
all of that in one tool call.

## Rules every tool follows

- **The app is the source of truth.** Status labels, eligibility verdicts and
  date arithmetic match what the Wingspan app shows on screen.
- **Answer a question, not an endpoint.** The tools return what the screen
  says, so an answer can be read straight to the user.
- **Amounts are in dollars**, never cents.
- **Contact details are reported narrowly.** `who_am_i` returns the caller's
  own email address, because that is the answer it exists to give, and the
  roster reads return the contractor's, because a roster without contact
  addresses is not the roster the payer sees on screen. Account and card
  numbers, phone numbers and internal notes are excluded; the kind of payout
  method and the institution behind it are reported, because the screen shows
  them.
- **Honest uncertainty.** Where the API cannot supply something, the tool says
  so in its note rather than inventing a value — a payable reports the dates it
  has and no bank-arrival estimate.
- **A missing section is named, not hidden.** When one part of an answer could
  not be fetched, the tool says so in its note; read the note before trusting a
  blank field.
- **Anything requiring an extra identity challenge is unreachable.** Paying a
  payable, paying an invoice, moving money between accounts, creating or
  rotating an API key and every similar action cannot be reached from here.
  There is nowhere in a conversation to complete such a challenge.
