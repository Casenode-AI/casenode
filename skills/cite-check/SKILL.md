---
name: cite-check
description: Cite-check a legal draft against the CaseNode corpus. Use when the user asks to check, verify, or cite-check the citations in a skeleton argument, advice, witness statement, or any document, and wants to know not only whether each authority is real but whether it actually supports the point it is cited for.
---

# Cite-check a draft

Check every citation in a draft in two passes. The first pass confirms the
authority exists and reports how later courts have treated it. The second pass
confirms the authority actually supports the proposition it is cited for. Do not
stop after the first pass. A citation can be real and still be cited for a point
the case does not support, and that is the failure this skill exists to catch.

All work uses the CaseNode MCP tools. Every finding must name a real authority by
its neutral citation so the user can open and confirm it.

## Inputs

- The draft text. If the user has not pasted it, ask for it or for the file path.
- The court the draft is addressed to, if known. Pass it as `target_court` so the
  report includes binding or persuasive status.

## Pass 1: existence and treatment

1. Call `validate_document` with the full draft text and `target_court` if known.
   This single call extracts every citation, resolves each against the corpus,
   and returns a per-citation status (verified, not in corpus, or ambiguous), the
   resolved authority, the number of cases that cite it, any negative treatment
   (for example overruled, doubted, or distinguished), and binding status.
2. Record, for each citation: the raw text as written, the resolved authority and
   its canonical id, the status, the treatment, and the binding status.
3. For anything not verified, say so plainly. An ambiguous citation needs the
   correct authority chosen before its usage can be checked. A citation that is
   not in the corpus cannot be confirmed, so report it as unverified rather than
   wrong.

## Pass 2: usage

Run this for every citation that resolved in Pass 1. The question is narrow: does
this authority support the specific proposition the draft cites it for?

1. Read the draft sentence around the citation and state, in one line, the
   proposition the draft is using the authority to support.
2. Read the case itself. Call `get_authority_detail` with the canonical id for the
   headline, then `read_judgment_text` (page through with `offset`) or
   `get_authority_paragraphs` to find where the case decides the point. Decide
   whether the case supports the proposition, supports it only in part, or does
   not support it.
3. Corroborate against how the authority is actually used. Call
   `get_citation_network` with `direction="cited_by"` to see which later cases
   cite it and at which paragraphs, and call `search_extracts` with the key terms
   of the proposition to see which authorities the courts reach for when deciding
   that point. If the authority never appears among them and its own text does not
   establish the proposition, treat that as a strong signal of misuse.
4. Give each citation a usage verdict: Supports, Partially supports, Does not
   support, or Unclear. When the verdict is Does not support or Partially
   supports, explain what the case actually decided and, if `search_extracts` or
   the citation network surfaced a better authority, name it.

## Reporting a citation that is real but misused

Do not collapse existence and usage into one tick. State both. A citation that
resolves cleanly but does not support its proposition should read as: real
authority, usage not supported, with one or two lines on what the case actually
held and what the draft claims. This is the most valuable output of the skill, so
make it prominent rather than burying it in a table cell.

## Output

Lead with a short summary line: how many citations were found, how many are
verified, how many carry negative treatment, and how many failed the usage check.
Then a table, one row per citation:

```
Cite-check: 6 citations found. 5 verified, 1 not in corpus.
Treatment flags: 1. Usage concerns: 2.

| Citation            | Exists      | Treatment        | Cited for (draft)              | Usage            |
|---------------------|-------------|------------------|--------------------------------|------------------|
| [2019] UKSC 38      | Verified    | None             | Proportionality is for the court | Supports        |
| [2004] UKHL 30      | Verified    | Distinguished    | Public authority always liable | Does not support |
| [2011] EWCA Civ 1   | Verified    | None             | Test is purely objective       | Partially supports |
| Smith v Jones 2020  | Not in corpus | -              | Limitation runs from discovery | Unverified       |
```

Follow the table with the usage concerns in full. For example:

```
Usage concerns

[2004] UKHL 30 - real authority, usage not supported.
  Draft cites it for: a public authority is always liable for the acts of its officers.
  The case actually held: liability turned on whether the officer acted within the
  scope of the function, and the House distinguished the earlier line the draft relies on.
  Courts deciding this point more often cite [2016] UKSC 11 (surfaced via search_extracts).

[2011] EWCA Civ 1 - real authority, usage partially supported.
  Draft cites it for: the test is purely objective.
  The case supports an objective core but expressly leaves room for a subjective
  element at paragraph 47, so the draft overstates it.
```

Close with one line reminding the user that CaseNode confirms what the corpus
holds and is a research aid, not legal advice, and that any authority not in the
corpus was reported as unverified rather than judged.
