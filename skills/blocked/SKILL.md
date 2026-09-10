---
name: blocked
description: List the contractors held up by one named onboarding requirement, such as a W-9, a certificate of insurance or a background check. Run it with the requirement's name.
disable-model-invocation: true
argument-hint: "[requirement name]"
---

# Who is held up by: $ARGUMENTS

A **requirement** is something a contractor must satisfy before the company can
pay them. The company configures each one once, and every contractor placed on
an engagement gets their own copy. This answers "who has not finished
$ARGUMENTS".

If no requirement was named, list the company's requirements with
`search_requirements` and ask which one the user means. Do not guess.

## Step 1 — find the requirement

Call `search_requirements`. Match `$ARGUMENTS` against the `name` of each
result, case-insensitively and in full. A partial name does not count as a
match.

- **No match.** Show the names that came back and ask which one was meant. Do
  not substitute a similar-sounding one.
- **More than one match.** Show the candidates and ask. Requirement names can
  be near-identical across engagements, and answering for the wrong one is
  worse than asking.

Keep two fields from the match: its `requirementDefinitionId` and its
`blocking` value.

## Step 2 — find who has not finished it

Call `search_contractors` with `requirement` set to that
`requirementDefinitionId` and `requirementState` set to `incomplete`, which
means outstanding — the contractor has not finished it.

That returns one page. Report the page, then offer the next one using the
result's `pagination.nextPageArgs`; do not fetch further pages unasked.

## Step 3 — report

Lead with the count and the requirement's full name. Then list the contractors
by name, with their onboarding state, so the user can see who has not even
signed up yet versus who signed up and has not finished.

Then add whichever of these applies:

- **If the definition is not marked blocking**, say so: that is what Wingspan
  reports for the definition, so an outstanding copy is usually tracked rather
  than holding anyone up. Where the requirement was attached decides for one
  contractor: a placement that is not blocking tracks the copy without stopping
  a payment, and an engagement placement can also block eligibility while
  still allowing payment. `get_contractor` does not read that setting and
  treats every incomplete requirement as blocking, so confirm in the Wingspan
  app before telling someone they are clear.
- **If the user wants a different slice**, the same pair of calls answers it
  with a different `requirementState`: `pendingReview` for submitted and not
  yet reviewed, `expiring` for satisfied but about to lapse, `expired` for
  lapsed, `complete` for finished.
- **A rejected or revoked requirement reads as outstanding again.** One the
  company rejected, or one whose underlying record was revoked or failed, is
  reopened rather than failed, with a reason recorded. There is no failed
  state, so say "reopened after review" rather than "never started" about such
  a contractor.

For one contractor's full picture, `get_contractor` with their `contractorId`.

## What cannot be done from here

Nudging a contractor, approving or rejecting what they submitted, resetting or
renewing a requirement, and extending an expiry date all happen in the Wingspan
app. So does everything the contractor does themselves — signing, uploading,
verifying identity. Sharing tax information is usually the contractor's step
too, but a company can instead record and verify a contractor's taxpayer
details itself, in the app.
