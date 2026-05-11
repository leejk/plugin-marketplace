---
name: code-greeter
description: Use this skill to add a friendly greeting comment block at the top of a source file, or to update an existing one. Trigger when the user says things like "add a greeting to this file", "apply greeter", or explicitly mentions "code-greeter".
---

# Code Greeter Skill

Inserts or refreshes a standardized greeting comment block at the top of a source file.

## When to use

- When creating a new file and you want a consistent header comment.
- When the user asks "add a greeting/header to this file".
- When the user explicitly invokes the `code-greeter` skill.

## Greeting block format

Insert the following block at the **very top** of the file, using the comment syntax for the file's language. If the file already has a shebang or a directive like `"use client"`, place the block right after it.

```
{COMMENT_START}
👋 Hello from code-greeter!

File:    {filename}
Updated: {YYYY-MM-DD}
{COMMENT_END}
```

Comment syntax by language:
- JavaScript / TypeScript / Java / Go / Rust / C / C++: `/* ... */`
- Python / Shell / Ruby: `#` (prefix on each line)
- HTML / Markdown: `<!-- ... -->`

## Procedure

1. **Read the target file**: Use Read on the file the user specified.
2. **Detect existing greeter**: Search the first 30 lines for `Hello from code-greeter!`.
   - If found → only update the `Updated:` line to today's date.
   - If not found → insert a new block at the top of the file (after any shebang/directive).
3. **Use Edit** to insert/update. Do not use Write to overwrite the whole file.
4. When done, report in one line: `code-greeter applied to {filename}`.

## Notes

- Do not apply to binary files (images, PDFs, etc.).
- Do not apply to files where comments are unsupported or risky (`.json`, `.lock`, `.toml`) — tell the user instead.
- If the user lists multiple files, repeat the procedure for each.
