# Module 15 Completion Report

## Script Metadata
- Filename: tools/validate_markdown.py
- Language: Python
- Purpose: Validates Markdown files one file at a time by checking for common formatting issues such as missing final newlines, CRLF line endings, tab characters, trailing whitespace, duplicate blank lines, malformed separator lines, and overly long lines outside fenced code blocks.

## Script Contents
```python
#!/usr/bin/env python3
"""Validate Markdown files one file at a time."""

import argparse
import sys
from pathlib import Path


def iter_markdown_files(root: Path):
    """Yield Markdown files under a root, skipping dependency folders."""
    for path in sorted(root.rglob("*.md")):
        if "node_modules" in path.parts or ".git" in path.parts:
            continue
        yield path


def validate_markdown_file(path: Path):
    """Check a single Markdown file and return a list of issue strings."""
    issues = []
    try:
        text = path.read_text(encoding="utf-8")
    except UnicodeDecodeError:
        return ["non-UTF8 content"]
    except OSError as exc:
        return [f"read error: {exc}"]

    lines = text.splitlines()
    if text and not text.endswith("\n"):
        issues.append("missing final newline")
    if "\r" in text:
        issues.append("CRLF line endings")
    if "\t" in text:
        issues.append("tab characters present")

    in_fence = False
    for index, line in enumerate(lines, start=1):
        if line.startswith("```"):
            in_fence = not in_fence
            continue

        if in_fence:
            continue

        if line.rstrip() != line:
            issues.append(f"trailing whitespace on line {index}")
        if len(line) > 100:
            issues.append(f"line {index} exceeds 100 characters")
        if index > 1 and lines[index - 2] == "" and line == "":
            issues.append(f"multiple blank lines around line {index}")

    if lines and set(lines[0].strip()) <= {"_"} and lines[0].strip():
        issues.append("malformed separator line at start of file")

    return issues


def main() -> int:
    parser = argparse.ArgumentParser(
        description="Validate Markdown files in a project, processing each file individually."
    )
    parser.add_argument(
        "path",
        nargs="?",
        default=".",
        help="File or directory to validate (defaults to current working directory).",
    )
    args = parser.parse_args()

    target = Path(args.path).resolve()
    if target.is_file():
        files = [target]
    elif target.is_dir():
        files = list(iter_markdown_files(target))
    else:
        print(f"Error: path not found: {target}", file=sys.stderr)
        return 2

    has_errors = False
    for path in files:
        issues = validate_markdown_file(path)
        if not issues:
            print(f"OK  {path}")
            continue

        has_errors = True
        print(f"BAD {path}")
        for issue in issues:
            print(f"  - {issue}")

    if has_errors:
        return 1
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

## Parameters
| Parameter | Description | Default |
|-----------|-------------|---------|
| path | File or directory to validate. A single file or a directory tree can be processed. | . |

## Test Run Output
```text
$ python3 tools/validate_markdown.py .github
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/copilot-instructions.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-create-instructions.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-create-status-report.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-evaluate-labeled-pr-corpus.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-identify-repeated-task-patterns.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-use-github-api-client.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-use-github-pr-api.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-use-pr-context-collector.prompt.md
OK  /Users/Ravi_Prakash/Downloads/heelo-genai/.github/prompts/to-validate-instructions.prompt.md
```
