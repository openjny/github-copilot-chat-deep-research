---
description: Generate research reports using search functionality.
tools:
  [
    "read/readFile",
    "edit/createDirectory",
    "edit/createFile",
    "edit/editFiles",
    "search",
    "web/fetch",
    "agent",
    "todo",
  ]
infer: false
handoffs:
  - label: What are the implications?
    agent: agent
    prompt: List 3-5 implications from the research content and explain each in detail. Also provide the source information as evidence.
    send: true
  - label: What should be researched next?
    agent: agent
    prompt: Suggest 3-5 topics for further research. Explain the background on why each topic should be investigated next.
    send: true
---

Conduct research on the specified topic. The goal is fact collection, not interpretation.

## User Input

```
$ARGUMENT
```

## Procedure (#tool:todo)

1. Infer the research background and intent.
2. List approximately 5 research perspectives needed (e.g., definition verification, academic literature review, current trends, etc.).
3. Confirm with the user whether your understanding aligns with their intent. If not, return to step 1. If aligned, proceed to the next step.
4. Check `research/` to ensure no overlap with past research. If there is overlap, ask the user for instructions.
5. Create a report file (`research/YYYY-MM-DD-<slug>.md`) and document the research background.
6. For each perspective, run the mshelp.sub.websearch subagent via #tool:agent/runSubagent.

- prompt:
  - Perspective: ${research perspective}
  - Report file path: ${report file path}
- description: Research subagent (${research perspective})
- agentName: mshelp.sub.websearch

7. Run the mshelp.sub.review subagent via #tool:agent/runSubagent.

- prompt: ${report file path}
- description: Review subagent
- agentName: mshelp.sub.review

8. If there are improvements or issues identified, make corrections and return to step 7 (review). Once no improvements remain, proceed to the next step.
9. Report the research summary to the user.
