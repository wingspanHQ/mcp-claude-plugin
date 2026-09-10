---
name: connect
description: Check the Wingspan connection and report who it is signed in as. Run this after installing the plugin or after re-authorizing, or whenever you want to confirm Claude is reading the right Wingspan account.
disable-model-invocation: true
---

# Check the Wingspan connection

Call `who_am_i`. It takes no arguments and performs a real authenticated read,
so a successful answer proves the whole connection works end to end.

Then report, in plain sentences:

- **Connected as** — the person's email address, when the connection
  identifies an individual. No name is returned. An authorization granted
  through the browser identifies the company account rather than the
  individual, and the person comes back unavailable; say that it identifies the
  account, and do not present it as a problem.
- **Reading as** — the account from `account` when present, otherwise the
  matching entry in `accounts`. `actingAsAccountId` is always null here,
  because `who_am_i` takes no `accountId`; ignore it.
- **Accounts reachable** — the entries in `accounts`, by name, noting any that
  sit under a parent account. Mention that a tool can be pointed at one of them
  with its `accountId` if the user needs a different one. An empty `accounts`
  list is legitimate, not a fault.

Then say what the connection is for in one line: these tools read the
contractors this account pays, what it owes them, and their onboarding
paperwork, and they can create contractor records and draft payments after
showing a preview. Paying, approving and anything requiring an extra identity
challenge stay in the Wingspan app.

## If it fails

**An authentication failure** means the connection needs re-authorizing. Tell
the user to run `/mcp`, pick the `wingspan` server, and complete the login in
the browser. If the tool reported a request id, pass it on — that is the handle
Wingspan support needs.

**No Wingspan tools available at all** means the plugin is installed but its
server is not connected. Point at `/mcp` to see the `wingspan` entry and its
status. If connecting reports that the endpoint cannot be found, this account
does not have Wingspan's Claude connection turned on; the user's Wingspan
account team can turn it on.

The `troubleshooting-connection` skill covers the rest.
