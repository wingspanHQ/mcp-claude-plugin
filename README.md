# Wingspan for Claude

Ask Claude about the contractors you pay through Wingspan, what you owe them,
and what is holding up a payment — then onboard contractors and create draft
payables without leaving the conversation. Everything it creates is a draft:
it never sends money, and paying stays in the Wingspan app.

Wingspan is a payments platform. A company that pays contractors uses it to
bring them on board, collect their tax and compliance paperwork, and pay them.
This plugin connects Claude to your own Wingspan records and adds the guidance
Claude needs to use them well.

Everything here is written from the paying company's side. A contractor who
gets paid *through* Wingspan will not find their own records here.

## What you can ask for

- "Who do we pay?" — and any search over your roster by name, email, company
  or your own reference.
- "Can we pay this contractor, and if not, why?"
- "Who has not signed up yet?" and "who is missing their certificate of
  insurance?"
- "What do we owe this month?" and "what did we pay last quarter?"
- "What happened to this payment?" — the full history of one payment.
- "Add these five contractors to the design retainer and invite them."
- "Log twelve hours at $85 for this contractor, due on the 30th."

The last two write to your Wingspan account, and Claude always shows you a
preview of exactly what it would do and waits for you to approve it.

## Install

In Claude Code:

```
/plugin marketplace add wingspanHQ/mcp-claude-plugin
/plugin install wingspan@wingspan
```

Then connect your Wingspan account:

1. Run `/mcp` and choose the `wingspan` server.
2. Complete the Wingspan login that opens in your browser, and approve the
   access request.
3. Run `/wingspan:connect` to confirm which Wingspan account Claude is reading.

Access is renewed in the background, so a working connection normally stays
working. If it ever stops, run `/mcp` again and log in.

You need a Wingspan account on the paying side, and permission to see your
company's contractors and payments. Claude sees exactly what your own Wingspan
sign-in sees, and nothing more.

This repository also ships a Codex plugin manifest carrying the eight skills.
Codex does not read the connection from this plugin, so a Codex user adds the
Wingspan MCP server themselves, as an HTTP server at
`https://api.wingspan.app/mcp-api-v3` with OAuth, and then signs in the same
way.

## The eight tools

| Tool | What it answers | Reads or writes |
| --- | --- | --- |
| `who_am_i` | Does the connection work, and which account is it reading? | Reads |
| `search_contractors` | Who do we pay? Who matches this name? Who is held up? | Reads |
| `get_contractor` | Can we pay this one contractor, and if not, why? | Reads |
| `search_requirements` | What must a contractor satisfy before we can pay them? | Reads |
| `search_payables` | What do we owe, and what have we paid? | Reads |
| `get_payable` | What happened to this one payment? | Reads |
| `create_contractors` | Create contractors, assign an existing engagement, send invites. | **Writes** |
| `create_payables` | Log payments as drafts. | **Writes** |

Full detail, including every filter and the rules the tools follow:
[`docs/tool-surface.md`](docs/tool-surface.md).

## Commands

| Run these yourself | Does |
| --- | --- |
| `/wingspan:connect` | Checks the connection and reports which Wingspan account the tools read, plus the signed-in person's email address when one is available. |
| `/wingspan:outstanding <requirement name>` | Lists the contractors who still have one named requirement outstanding, such as a W-9 or a certificate of insurance. Whether that blocks their payment is confirmed in the Wingspan app. |

## What it cannot do

**No money moves from here.** Paying a payable, paying an invoice and
transferring funds are protected by an extra identity challenge, and there is
nowhere in a conversation to complete one. The same applies to creating or
rotating an API key and to changing who can access your account.

**Payments are created as drafts.** `create_payables` stops at a draft your
contractor cannot see, with no payment scheduled. Opening it, approving it and
paying it happen in the Wingspan app.

**Setup stays in the app.** Creating engagements, groups, worksites, custom
fields, rate cards and requirement definitions; attaching a requirement;
approving or rejecting what a contractor submitted; payroll runs and funding
sources; invoices and accounting integrations.

**The banking rail is not visible.** The tools report what the Payables screen
reports, which stops at the date a deposit was confirmed. A confirmed deposit
means the sending bank finished processing, not that the money is available or
final — a payment can still come back afterwards, and that shows up on the
payment itself, where reading it names the state Returned. A question about a
bank transfer after that belongs with Wingspan support.

**Everything a contractor does themselves.** Signing up, signing a document,
uploading a certificate, verifying identity, adding a payout method. Sharing
tax information is usually the contractor's step too, but a company can instead
record and verify a contractor's taxpayer details itself — in the Wingspan app,
not from here.

## Privacy

The tools return your roster the way your own screens show it: your
contractors' names, email addresses, company names and city, state and country.
One contractor read also returns the legal name and tax classification on their
W-9, whether they have a payout method and what kind it is, and the bank or card
brand behind it. Account and card numbers, phone numbers and internal notes are
never returned, and the write tools report each row by a label you choose rather
than by anyone's name or email address.

## Included guidance

The plugin ships eight skills. Claude loads these six on its own when they are
relevant; you do not need to invoke them. The other two are the commands in the
table above, which only run when you ask for them.

| Skill | Covers |
| --- | --- |
| `using-wingspan-tools` | The rules every tool call follows, and a glossary. |
| `finding-contractors` | Searching the roster, and who still has which requirement outstanding. |
| `checking-payments` | What is owed, what was paid, and one payment's history. |
| `onboarding-contractors` | Adding contractors and inviting them. |
| `creating-draft-payables` | Creating new payment obligations as drafts. |
| `troubleshooting-connection` | Authentication errors and wrong-account answers. |

## Troubleshooting

**No Wingspan tools appear.** The plugin is installed but the server is not
connected. Check `/mcp`.

**Connecting reports that the endpoint cannot be found.** This account does not
have Wingspan's Claude connection turned on. Your Wingspan account team can
turn it on.

**Answers are about the wrong company.** Run `/wingspan:connect`. It reports
which account the tools read as, and which other accounts your sign-in can
reach.

**A list looks empty.** Contractor searches exclude archived contractors and
payment searches exclude cancelled payments unless you ask for them, and a
contractor with no engagement assigned has no paperwork status to filter on.

For anything else, contact your Wingspan account team or reach Wingspan
support from the Help menu in the Wingspan app.

## License

MIT. See [LICENSE](LICENSE).
