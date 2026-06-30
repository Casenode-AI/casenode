# CaseNode plugin for Claude Code

The CaseNode plugin bundles the CaseNode MCP connection with a set of skills for
researching UK case law. The skills are short playbooks that teach the assistant
how to cite-check a draft, research an area of law, trace how a case has been
treated, validate the citations in a document, and summarise a case, so you get
grounded, cited results without prompting for each step.

Every result is anchored to a real authority in the CaseNode corpus with its
neutral citation, so you can open and confirm it. CaseNode is a research aid, not
legal advice.

## Install

```
/plugin marketplace add casenode-ai/casenode
/plugin install casenode
```

On install you will be asked for your CaseNode API key. Create one at casenode.ai
under Settings, then API keys. The key is stored securely by Claude Code and sent
as the `X-API-Key` header on every request to the CaseNode MCP server at
`https://api.casenode.ai/mcp`. Keep it secret: anyone with the key can query the
corpus on your behalf.

## Verify your setup

After installing and entering your key, confirm the connection works before you
rely on the skills. Ask the assistant to resolve a known citation, for example:

```
Use CaseNode to resolve [2019] UKSC 38
```

If it returns the authority with its title and court, the key is bound correctly
and every skill is ready to use. If it reports an authentication error, re-enter
your key with `/plugin` and check that it was copied in full.

## Skills

Skills are namespaced under the plugin. You can invoke each one directly, or
describe what you need and let the assistant pick.

| Skill | Invoke | What it does |
|---|---|---|
| Cite-check a draft | `/casenode:cite-check` | Runs two passes over every citation in a draft: an existence and treatment pass, then a usage pass that checks whether each authority actually supports the proposition it is cited for. |
| Research an area of law | `/casenode:research-area` | Scopes a research pack with real filters (court, domain, topic, year), then gathers the governing authorities, statutes, test elements, and judicial reasoning. |
| Trace case treatment | `/casenode:trace-treatment` | Given one authority, traces how later courts have treated it through the citation network and verbatim extracts. |
| Validate citations | `/casenode:validate-citations` | Bulk-resolves every citation in a document and reports which are real, which are not in the corpus, and which carry negative treatment. Existence only, not usage. |
| Summarise a case | `/casenode:summarise-case` | Given a case title or citation, resolves it and produces a structured summary of what the case decided. |

## What the corpus covers

The CaseNode corpus holds UK authorities (cases, statutes, and procedural rules)
with citation edges between them and paragraph level text for search. The skills
answer from that corpus rather than from what the model remembers.

## Requirements

- Claude Code with plugin support.
- A CaseNode API key.

## Licence

MIT. See [LICENSE](LICENSE).
