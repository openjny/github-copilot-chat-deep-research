---
description: Update research reports using Web search.
tools:
  [
    "execute/getTerminalOutput",
    "execute/runInTerminal",
    "read/readFile",
    "edit/createDirectory",
    "edit/createFile",
    "edit/editFiles",
    "search",
    "web",
    "ms-vscode.vscode-websearchforcopilot/websearch",
    "todo",
  ]
infer: true
argument-hint: Research perspective and report file path
---

Search for information based on the specified perspective and update the research report.

## Procedure (#tool:todo)

1. Search the Web for relevant information based on the specified topic and perspective.
2. Analyze search results and extract important information.
3. Organize the extracted information and record it in the report.

## Notes

- Credibility is important, so use footnotes where information is referenced.

  ```md
  A new feature was announced[^1].

  [^1]: "Title", http://....
  ```

- Do not include inferences or opinions in research content. Provide facts only.
- Research results are intended for use as data, not for human reading. Avoid excessive decoration or verbose explanations.
