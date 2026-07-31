# Repository Working Rules

## Prompt Logging

- For every instruction received, append an entry to prompts.txt.
- Use the machine's current timestamp in ISO 8601 format, including timezone when available.
- Include the complete user prompt.
- Include a concise and accurate summary of the work performed.
- Include files changed, commands executed, validation performed, assumptions, and remaining risks.
- Never rewrite, reorder, backdate, or delete previous prompt entries.
- Never include access tokens or secrets in prompts.txt.
- Ensure the first and second prompts from this session are both represented in prompts.txt.

## Engineering Workflow

- Inspect the repository before modifying files.
- Plan before implementing ambiguous or multi-step changes.
- Use only the official DummyJSON documentation as the authoritative API contract.
- Clearly distinguish documented contract behavior from experimentally observed behavior.
- Treat undocumented edge cases as characterization tests.
- Do not invent expected HTTP status codes based only on REST conventions.
- Do not weaken a failing assertion merely to make a test pass.
- Identify the root cause before changing a failing test.
- Prefer small, reviewable implementation steps.
- Run the relevant Newman folder after each meaningful implementation change.
- Run the complete suite before considering the work complete.
- Keep runtime tokens empty in committed files.
- Never commit secrets, private credentials, generated reports, or node_modules.
- Do not create Git commits unless I explicitly request it.
- At the end of every response, report files changed, commands run, test results, assumptions, and remaining risks.
