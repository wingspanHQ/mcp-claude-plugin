# Changelog

## Unreleased

- Two read-only tools for the licenses and certifications contractors share:
  `search_credentials` and `get_contractor_credentials`.
- `search_contractors` filters by a shared credential's type, jurisdiction
  and validity status.
- `search_requirements` and the requirement catalogue know the credential
  check (`CredentialCheck`).

## 0.1.0 — 2026-09-10

First release.

- Connects Claude to the Wingspan API through the Wingspan MCP server, over an
  authorization the user grants in their browser.
- Ten tools: `who_am_i`, `search_contractors`, `get_contractor`,
  `search_requirements`, `search_payables`, `get_payable`,
  `get_payroll_preview`, `create_contractors`, `create_payables` and
  `open_payables`. The three write tools preview by default and change nothing
  until they are applied.
- Eight skills, six of which Claude loads on its own: `using-wingspan-tools`,
  `finding-contractors`, `checking-payments`, `onboarding-contractors`,
  `creating-draft-payables` and `troubleshooting-connection`.
- The other two skills are commands you run yourself: `/wingspan:connect` and
  `/wingspan:outstanding <requirement name>`.
