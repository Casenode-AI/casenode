---
name: trace-treatment
description: Trace how a UK authority has been treated by later courts over the CaseNode corpus. Use when the user wants to know whether a case is still good law, who has cited it, and whether later courts have followed, applied, distinguished, doubted, or overruled it.
---

# Trace how a case has been treated

Given one authority, show how the later case law has treated it: who cites it,
for what, and whether it has been followed, applied, distinguished, doubted, or
overruled. The goal is a clear answer to "is this still good law, and how is it
used". All work uses the CaseNode MCP tools, and every citing case is named by
its neutral citation.

## Inputs

- A case title or citation. If the user gives a vague reference, confirm the
  exact authority first.

## Steps

1. Resolve the case. Call `resolve_citation` with the citation, or
   `autocomplete_citations` with a partial name or reference, to get the canonical
   id. If the result is ambiguous, show the candidates and confirm which one the
   user means before continuing.
2. Get the headline with `get_authority_detail` so you can state what the case
   itself decided before describing how others have treated it.
3. Trace who cites it. Call `get_citation_network` with `direction="cited_by"`.
   The result lists the citing cases with their court, year, and the paragraph
   references where they cite the authority. Sort your reading by court seniority
   and recency, because a later, higher court matters most.
4. Read the actual treatment. The network gives you who and where, not the words.
   For the most significant citing cases, call `get_authority_paragraphs` or
   `read_judgment_text` on the citing case at the cited paragraph reference, and
   read how that court used the authority. Classify each as followed, applied,
   distinguished, doubted, or overruled, based on what the citing paragraph says.
5. Note what the authority relies on. Call `get_citation_network` with
   `direction="cites"` if the user also wants to see the authorities this case
   was built on, and `direction="co_cited"` to see what is commonly cited
   alongside it.

## Output

```
Treatment of [2004] UKHL 30
What it decided: one or two lines from get_authority_detail.

Standing now: still good law, with one significant distinction (see below).
Cited by 37 cases in the corpus, most recently [2023] UKSC 4.

How later courts have treated it
  Followed
    [2009] EWCA Civ 55 at [22] - applied the test without qualification.
  Distinguished
    [2016] UKSC 11 at [40] - confined it to its facts where the officer acted
    outside the scope of the function.
  Doubted
    [2018] EWHC 90 (QB) at [61] - questioned whether the reasoning survives the
    later Supreme Court line.

Relied on (cites)
  The principal authorities this case was built on, if the user asked for them.
```

If the authority has no citing cases in the corpus, say so plainly rather than
implying it is untreated for certain, since absence in the corpus is not the same
as never cited. Close with one line that CaseNode answers from the corpus and is
a research aid, not legal advice.
