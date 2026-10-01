- Validate instruction files only against the current project rules in
  `./instructions/main.agent.md` and `./instructions/creating-instructions.agent.md`.
- Accept either a specific instruction file path or the `./instructions/` folder;
  read the complete file before making conclusions.
- Confirm each instruction file uses a verb-first, hyphen-separated name and the
  `.agent.md` extension.
- Confirm the content is a single focused workflow with concise, actionable
  bullets; avoid duplicated logic, long explanatory prose, and conflicting
  requirements.
- Confirm each instruction is present in `./instructions/main.agent.md` with a
  one-line description and useful trigger keywords.
- Confirm any matching `.github/prompts/to-[name].prompt.md` wrapper exists and
  references the instruction with the correct relative path; keep wrappers thin
  and non-duplicative.
- Check for broken links, stale references, malformed Markdown, trailing
  whitespace, duplicate blank lines, and invalid separator patterns that do not
  match the project’s formatting conventions.
- If a file or catalog entry is missing, stale, or inconsistent, report the
  exact fix to make rather than inventing content.
- If the instruction is valid, state the result clearly and concisely; do not
  add unsupported claims or extra project assumptions.
- Keep the validation output in Markdown and focused on actionable findings.
