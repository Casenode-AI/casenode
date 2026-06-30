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
   it was last cited. Then take the five most recent citing judgments and, for
   each, pull the passage that cites this case using `search_extracts` or
   `read_judgment_text` on that citer, and keep one short verbatim quote with its
   pinpoint, so the user sees how the case is used in practice and not just that it
   was cited. Quote it word for word, including the citation exactly as the later
   court wrote it, so the reader can see the precise reference used (later courts
   often cite the same case by different reports, for example a Court of Appeal
   citation rather than the Supreme Court neutral citation). Lead with a one line characterisation of the treatment overall
   (followed, applied, distinguished, or qualified). Name every citing case by its
   canonical reference: its neutral citation, or, for a tribunal decision that has
   none, its tribunal case number. If the network does not make the treatment
   clear, say so rather than inferring approval or criticism.

## Output

Render the summary as clean markdown so it is easy to scan. Use a heading for the
case, a single compact metadata line, bold section labels, and bullets for the
reasoning and the cited authorities. Follow this shape:

```markdown
## [2019] UKSC 38  Resolved Case Title

UK Supreme Court · 20 June 2019 · on appeal from [2018] EWCA Civ 100
**Panel:** Lead judge (lead), with the other members of the court.
**Parties:** claimant side v respondent side.

**Issue**

The question the court had to decide, in one or two lines.

**Decision**

What the court held and the disposition, with the paragraph reference for the
holding.

**Reasoning**

- The first step in the court's reasoning, with a paragraph pin ([20]).

- The next step, quoting a precise phrase verbatim where it matters ([24]).

**Test elements**

- Each element of the legal test in turn, and where the burden lies.

**Statutes and authorities cited**

- *Statute or Regulations name*, section or regulation, on what it is cited for.

- *Leading case name* [citation], on the point it is relied on for.

**Treatment**

Cited N times. Last cited on [date] in [citation]. One line on what later courts
cite it for and whether it has been followed, applied, or qualified.

Five most recent citations:

- *[citing case]* [neutral citation or tribunal case number] ([court], [date]) at
  [pinpoint]:

  > A short verbatim quote showing how it was used.

- *[next citing case]* [neutral citation or tribunal case number] ([court], [date])
  at [pinpoint]:

  > Another short verbatim quote.

[Read the judgment](source-url)
```

Formatting rules:

- Put each section label on its own line as a bold header, with the content on the
  line beneath it, not on the same line. Leave a blank line above each header.
- Separate bullets with a blank line so the list is not cramped.
- Only include a section when it has content. Drop **Test elements** when the
  authority carries no test, and drop **Panel** when the bench is not in the
  record. Do not print an empty header.
- Keep the metadata line to one row, separated by a middot. Do not use em or en
  dashes anywhere in the output.
- Quote the judgment verbatim where a precise phrase matters, in quotation marks,
  and attribute every quote to its paragraph.
- If the treatment is not clear from the citation network, say it is not
  characterised in the corpus rather than inferring approval or criticism.
- Close with one quiet line that CaseNode answers from the corpus and is a
  research aid, not legal advice.
