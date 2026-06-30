---
name: research-area
description: Research an area of UK law over the CaseNode corpus. Use when the user wants to map an unfamiliar area, find the governing authorities and statutes on a topic, understand a legal test, or pull together the leading cases and the reasoning behind them.
---

# Research an area of law

Build a grounded picture of an area of law: the governing test, the leading
authorities, the relevant statutes, and how the courts apply the test in
practice. All work uses the CaseNode MCP tools, and every authority is named by
its neutral citation.

## The scoping rule you must follow

`research_pack` scopes the corpus by its filter fields only. Its `query` field is
informational and is echoed back. It does not narrow the corpus. Passing a bare
`query` with no filters returns the whole-corpus ranking, which is not topical
research. Always scope `research_pack` with real filters: `domain`, `court`,
`topic`, `year_from`, `year_to`, `jurisdiction`, or `disposition`.

To find valid filter values before you call `research_pack`, do not guess. Use:

- `browse_taxonomy` with no argument for the top level domains, then with a
  `domain_slug` to drill in.
- `list_topics`, optionally scoped to a domain, for valid topic slugs.
- `list_courts` for valid court codes and the court hierarchy.

Where you need free text searching rather than filtering, use `search_cases` and
`search_extracts`. In those two tools the query genuinely searches, so they are
the right tools for "find cases that mention X" and "find paragraphs that discuss
X". `research_pack` is for the structured pack once you have a filtered scope.

## Steps

1. Turn the user's question into a scope. Identify the domain and, if relevant,
   the court and a year range. Confirm the domain slug with `browse_taxonomy` and
   any topic slugs with `list_topics`. Confirm court codes with `list_courts`.
2. Call `research_pack` with those filters (for example `domain="landlord-tenant"`,
   `topic=["right-to-manage"]`, `court="UKSC"`). Set `target_court` to the user's
   court if they have one, so each authority is marked binding or persuasive. The
   pack returns the governing authorities, governing statutes, procedural rules,
   legal test elements, and usage examples for the filtered scope.
3. For the top governing authorities, call `get_citation_network` with
   `direction="cited_by"` to see how influential each one is and how it has been
   applied.
4. For the key reasoning, call `search_extracts` with the legal concept (for
   example `term="proportionality test"`) scoped by the same domain or court, to
   pull the actual paragraphs where courts discuss the point.
5. For any authority that carries test elements, explain each element in turn,
   including which party bears the burden.

## Output

Produce a short research memo, not a raw tool dump:

```
Research: the right to manage under the Commonhold and Leasehold Reform Act 2002
Scope: domain landlord-tenant, topic right-to-manage, 312 cases in scope, 2003 to 2024.

Governing test
  The statutory conditions for a valid claim, drawn from [authority], with each
  element set out and the burden noted.

Leading authorities
  1. [2019] UKSC 38 - what it establishes, cited by 42 cases.
  2. [2014] EWCA Civ 12 - what it establishes, cited by 18 cases.

Governing statutes
  Commonhold and Leasehold Reform Act 2002, with the relevant provisions.

How the courts apply it
  Two or three verbatim extracts from search_extracts, each attributed to its
  case and paragraph, showing the test in practice.
```

If the scope returns very few cases, say so and widen the filters rather than
falling back to an unscoped query. Close with one line that CaseNode answers from
the corpus and is a research aid, not legal advice.
