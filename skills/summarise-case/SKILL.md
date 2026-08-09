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

   Read for two different things, because the summary needs both.

   First, the **facts and the disposal**: who the parties were, what the dispute
   was actually about, what happened below, and how this court disposed of it.
   This is what the **In short** section is written from, and it is usually in the
   opening paragraphs and the closing ones rather than the middle.

   Second, the **issues**, plural. A judgment normally decides several distinct
   points, and each gets its own entry with its own answer and its own pinpoint.
   Do not collapse them into a single sentence. Take care with two traps: a
   section headed "The Law" is the court restating settled principles, not an
   issue it decided; and on a long judgment the real issues are spread throughout,
   so if every pinpoint you have is in the opening paragraphs you have summarised
   the preamble. Check the paragraph count and read to the end.
4. Identify the key legislation and authorities the case relies on. Call
   `get_citation_network` with `direction="cites"` and pick out the statutes,
   procedural rules, and leading cases it leans on most. If a statute is named in
   the judgment text but is not held as its own authority, use `extract_citations`
   on the relevant passage to capture the reference. Do not list every citation,
   only the ones that carry the reasoning.
5. Establish how the case has been treated since. Call `get_citation_network` with
   `direction="cited_by"` for the count, the most recent citing case, and the date
   it was last cited.

   **Rank the citers by court seniority, not by recency.** `get_citation_network`
   already returns the most senior court first, so take them in the order given.
   Recency is the wrong axis on a well-known case: *Daejan Investments Ltd v
   Benson* [2013] UKSC 14 has 1,631 distinct citers, of which 1,599 are
   First-tier Tribunal (Property Chamber) dispensation applications, so the five
   most recent are a wall of first-instance boilerplate and the Court of Appeal
   treatment never appears at all. On a heavily cited authority, pass the `court`
   filter to pull the appellate treatment directly.

   For each of the citers you keep, get the passage that cites this case with
   `get_citation_edge_paragraphs(citing_authority_id, cited_authority_id)`, which
   returns the citing paragraphs for that edge without a text search. Keep one
   short verbatim quote with its pinpoint, so the user sees how the case is used
   in practice and not just that it was cited. Quote it word for word, including
   the citation exactly as the later court wrote it, so the reader can see the
   precise reference used (later courts often cite the same case by different
   reports, for example a Court of Appeal citation rather than the Supreme Court
   neutral citation).

   Lead with a one line characterisation of the treatment overall (followed,
   applied, distinguished, or qualified). Name every citing case by its canonical
   reference: its neutral citation, or, for a tribunal decision that has none, its
   tribunal case number. If the network does not make the treatment clear, say so
   rather than inferring approval or criticism.

## Output

Render the summary as clean markdown so it is easy to scan. Use a heading for the
case, a single compact metadata line, bold section labels, and bullets for the
reasoning and the cited authorities. Follow this shape:

```markdown
## [2019] UKSC 38  Resolved Case Title

UK Supreme Court · 20 June 2019 · on appeal from [2018] EWCA Civ 100
**Panel:** Lead judge (lead), with the other members of the court.
**Parties:** claimant side v respondent side.

**In short**

Two or three short paragraphs of plain English. What the dispute was actually
about, how it reached this court, and how it came out. Written for someone who
will read nothing else, so no citations and no paragraph numbers here.

**Issues**

- The first question the court had to decide, phrased as a question. The answer
  it gave, in a sentence ([44]).

- The next question, and its answer ([61]).

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

How the senior courts have used it:

- *[citing case]* [neutral citation or tribunal case number] ([court], [date]) at
  [pinpoint]:

  > A short verbatim quote showing how it was used.

- *[next citing case]* [neutral citation or tribunal case number] ([court], [date])
  at [pinpoint]:

  > Another short verbatim quote.

[Read the judgment](source-url)
```

Formatting rules:

- **Lead with In short, always.** It is the section most readers will read, and
  the only one written for someone who does not already know the case. Never open
  with the issue, the panel, or the procedural history.
- **Order the citing cases by court seniority**, highest first, and say which
  court each one is. A reader wants to know what the Court of Appeal did with a
  case, not what the most recent tribunal did.
- Give each issue its own bullet with its own pinpoint. A case that decided four
  things should show four.
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
