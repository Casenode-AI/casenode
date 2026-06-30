---
name: search-cases
description: Search across the CaseNode corpus for UK cases on a topic, by party, or by court. Use when the user wants to find cases that match a concept, a party name, a court, or a date range, ranked by influence, rather than researching a whole area or looking up one known case.
---

# Search across cases

Find the cases that match what the user is looking for and present them ranked by
influence. All work uses the CaseNode MCP tools, and every result is named by its
neutral citation.

Unlike `research_pack`, the `search_cases` tool genuinely searches on its `query`,
so plain language search terms work here. Use this skill for "find cases on X",
"cases involving party Y", or "Supreme Court cases about Z". For a structured map
of an area of law, use the research-area skill instead. For one known case, use
the summarise-case skill.

## Inputs

- The search terms. These can be a legal concept, a party name, or a phrase.
- Any narrowing the user gives: court, legal domain or topic, year range, party
  side, or a minimum citation count for influential cases only.

## Steps

1. If the user named a court, a domain, or a topic, confirm the valid value before
   filtering. Use `list_courts` for court codes, `list_topics` or `browse_taxonomy`
   for domain and topic slugs, and `autocomplete` to resolve a party name to its
   full form. Do not guess codes or slugs.
2. Call `search_cases` with the user's terms as `query`, plus any of `court`,
   `domain`, `topic`, `year_from`, `year_to`, `claimant`, `respondent`, or
   `cited_by_min`. Leave `mode` as the default hybrid unless the user wants exact
   keyword or purely semantic matching.
3. Present the top results ranked by influence, each with its neutral citation,
   court, year, and citation count. Offer to go deeper on any one result with the
   summarise-case or trace-treatment skill.

## Output

```
Search: vicarious liability, Supreme Court

Top matches, ranked by influence:
  1. Catholic Child Welfare Society [2012] UKSC 56 (UKSC, 2012) - cited 53 times
  2. Cox v Ministry of Justice [2016] UKSC 10 (UKSC, 2016) - cited 33 times
  3. WM Morrison Supermarkets [2020] UKSC 12 (UKSC, 2020) - cited 23 times

Want one of these in detail, or a wider net across the Court of Appeal?
```

If the search returns nothing, say so and suggest loosening a filter or trying
broader terms rather than inventing results. Close with one line that CaseNode
answers from the corpus and is a research aid, not legal advice.
