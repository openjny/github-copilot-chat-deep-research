# General Guidelines

This repository provides GitHub Copilot custom agents for answering user queries and generating research reports using trusted sources.

## Agents

| Agent                  | Description                                                                                                                         |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `mshelp.ask`           | Quick search agent for Microsoft Learn Docs and Web sources. Used for simple queries and understanding user intent.                 |
| `mshelp.research`      | Main orchestrator for research using Microsoft Learn MCP and Web search. Coordinates subagents and generates comprehensive reports. |
| `mshelp.websearch`     | Main orchestrator for web-focused research. Similar to `mshelp.research` but relies primarily on web search.                        |
| `mshelp.sub.research`  | Subagent for researching specific perspectives using Microsoft Learn Docs and Web search.                                           |
| `mshelp.sub.websearch` | Subagent for researching specific perspectives using Web search only.                                                               |
| `mshelp.sub.review`    | Subagent for analyzing reports and identifying missing or incomplete information.                                                   |

## Directory Structure

```
├── downloads/
│   ├── download-file.pdf
│   └── download-file.meta.json
├── research/
│   └── YYYY-MM-DD-slug.md
└── AGENTS.md
```

## Research Report File Conventions

- File names must follow the format `YYYY-MM-DD-<slug>.md`.
- Do not include inferences or opinions in research content. Provide facts only.
- Research results are primarily intended for use as data, not for human reading. Avoid excessive decoration or verbose explanations.
- Note that, when `ask` agent mode is used, no report file will be created. Only `research` and `websearch` agent modes generate report files.

## Citation Requirements

Credibility is important, so cite sources using footnotes when referencing data. Compile the footnote list at the end of the report and always include URLs (specify URLs explicitly, not in markdown format).

```md
A new feature was announced[^1]. However, it has also been noted that...[^2].

[^1]: Title 1, https://....

[^2]: Title 2, https://....
```

## Microsoft Knowledge

Microsoft updates are constantly in progress, and your Microsoft knowledge is outdated. Do not use any information whose documentation source is unclear.

## Downloads

- For files like PDFs that are more efficient to download locally before reading, save them to the `downloads/` folder.
- Always create a metadata file (`.meta.json`) for downloaded files. The metadata file should include the following information:

  ```json
  {
    "url": "https://...", // Source URL
    "downloaded_at": "YYYY-MM-DDTHH:MM:SSZ", // Download timestamp (ISO 8601 format)
    "type": "pdf" // File type (e.g., pdf, docx, etc.)
  }
  ```
