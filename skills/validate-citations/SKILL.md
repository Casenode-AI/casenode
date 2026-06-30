---
name: validate-citations
description: Validate the citations in a document against the CaseNode corpus. Use when the user wants a fast existence check across every citation in a draft, that is, which citations are real, which are not in the corpus, which are ambiguous, and which carry negative treatment. This checks existence and treatment, not whether each authority supports its point.
---

# Validate citations in a document

Run a fast, single-pass existence and treatment check across every citation in a
document. This is the quick check. It confirms each citation resolves to a real
authority and reports negative treatment, but it does not check whether the
authority supports the proposition it is cited for. When the user needs that
usage check, use the cite-check skill instead, and say so if the document looks
like it needs it.

All work uses the CaseNode MCP tools.

## Inputs

- The document text. If it has not been pasted, ask for it or for the file path.
- The court the document is addressed to, if known, to pass as `target_court`.

## Steps

1. Call `validate_document` with the full document text and `target_court` if
   known. This extracts every citation, resolves each against the corpus, and
   returns the per-citation status, the resolved authority, the citing count, any
   negative treatment, and binding status in one call. Prefer this over calling
   `extract_citations` and `resolve_citation` in a loop.
2. If the user only wants the raw list of citations and their canonical ids
   without resolution, call `extract_citations` instead.
3. For any citation reported as ambiguous, call `resolve_citation` with the raw
   text to show the candidate authorities so the user can pick the intended one.

## Output

```
Citation validation: 8 citations found.
Verified: 6. Not in corpus: 1. Ambiguous: 1. Negative treatment: 1.

| Citation           | Status        | Authority                          | Treatment     | Binding   |
|--------------------|---------------|------------------------------------|---------------|-----------|
| [2019] UKSC 38     | Verified      | resolved title, UKSC, 2019         | None          | Binding   |
| [2004] UKHL 30     | Verified      | resolved title, UKHL, 2004         | Distinguished | Binding   |
| Smith v Jones 2020 | Not in corpus | -                                  | -             | -         |
| [2011] EWCA 1      | Ambiguous     | 2 candidates, see below            | -             | -         |
```

List the ambiguous candidates and the not-in-corpus citations explicitly under
the table. State clearly that this pass checked existence and treatment only, and
that confirming each authority supports its point requires the cite-check skill.
Close with one line that CaseNode answers from the corpus and is a research aid,
not legal advice.
