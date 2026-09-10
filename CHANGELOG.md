# Changelog

## 0.1.0 — 2026-09-10

First release.

- Connects Claude to the Wingspan API through the Wingspan MCP server, over an
  authorization the user grants in their browser.
- Eight tools: `who_am_i`, `search_contractors`, `get_contractor`,
  `search_requirements`, `search_payables`, `get_payable`, `create_contractors`
  and `create_payables`. The two write tools preview by default and change
  nothing until they are applied.
- Eight skills, six of which Claude loads on its own: `using-wingspan-tools`,
  `finding-contractors`, `checking-payments`, `onboarding-contractors`,
  `logging-payments` and `troubleshooting-connection`.
- The other two skills are commands you run yourself: `/wingspan:connect` and
  `/wingspan:blocked <requirement name>`.
