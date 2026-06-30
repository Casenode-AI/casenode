---
name: summarise-case
description: Summarise a single UK case from the CaseNode corpus. Use when the user knows the title or citation of a case and wants a structured summary of what it decided, the parties, the court, the issue, the outcome, the reasoning, the legislation and authorities it relies on, and how later cases have treated it and when it was last cited.
---

# Summarise a case

Given a case the user can name by title or citation, produce a structured summary
of what it decided. All work uses the CaseNode MCP tools, and the summary is built
from the authority record and judgment text in the corpus, not from memory.

## Inputs

- A case title or citation. If the reference is partial or vague, confirm the
  exact authority before summarising.

## Steps

1. Resolve the case. Call `resolve_citation` with the citation, or
   `autocomplete_citations` with a partial title or reference, to get the
   canonical id. If the result is ambiguous, show the candidates and confirm which
   one the user means before continuing. If it does not resolve, say it is not in
   the corpus rather than summarising from memory.
2. Get the record. Call `get_authority_detail` with the canonical id for the
   citation, title, parties, court, date, jurisdiction, and any legal test
   elements.
3. Read enough of the judgment to summarise it accurately. Call
   `read_judgment_text` and page through with `offset`, or `get_authority_paragraphs`,
   to find the issue, the decision, and the core reasoning. Do not summarise from
   the headline alone if the judgment text is available.
4. Identify the key legislation and authorities the case relies on. Call
   `get_citation_network` with `direction="cites"` and pick out the statutes,
   procedural rules, and leading cases it leans on most. If a statute is named in
   the judgment text but is not held as its own authority, use `extract_citations`
   on the relevant passage to capture the reference. Do not list every citation,
   only the ones that carry the reasoning.
5. Establish how the case has been treated since. Call `get_citation_network` with
   `direction="cited_by"` for the count, the most recent citing case, and the date
   it was last cited. For the leading citing cases, call `search_extracts` to see
   the proposition later courts cite it for, so the summary reports treatment
   rather than a bare count. If the network does not make the treatment clear, say
   so rather than inferring approval or criticism.

## Output

```
[2019] UKSC 38 - resolved title
Court: UK Supreme Court. Date: judgment date. Jurisdiction: United Kingdom.
Parties: claimant side v respondent side.

Issue
  The question the court had to decide, in one or two lines.

Decision
  What the court held and the disposition.

Reasoning
  The core of the court's reasoning, with paragraph references where useful.

Test elements
  If the authority carries a legal test, each element in turn and the burden.

Statutes and authorities cited
  The key legislation, procedural rules, and leading cases the judgment relies on,
  each by its citation, with one phrase on what it is cited for.

Treatment
  Cited by N cases in the corpus, last cited [citation] on [date]. One or two lines
  on what later courts cite it for and whether it has been followed, applied, or
  qualified, drawn from the citing cases and their extracts. If that is not clear
  from the network, say the treatment is not characterised in the corpus.

Source
  The source URL from the authority record, so the user can open the judgment.
```

Quote the judgment verbatim where a precise phrase matters, and attribute every
quote to its paragraph. Close with one line that CaseNode answers from the corpus
and is a research aid, not legal advice.
