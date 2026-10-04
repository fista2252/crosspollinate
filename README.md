# crosspollinate

A Claude Code skill that solves a problem by borrowing from distant fields. It restates your problem in abstract terms, names the core contradiction and picks 4-6 unrelated fields (biology, engineering, economics and so on). It then searches papers, code, discussions and Wikipedia in parallel, ranks the analogies it finds and writes a one-page `crosspollinate-report.md` to your current directory.

## Install

```sh
cp -r crosspollinate ~/.claude/skills/
```

**Required tools:** Task (for the parallel agents and the ranker), WebFetch and WebSearch. You also need outbound network access.

## Usage

In Claude Code:

```
/crosspollinate How do I reduce cold-start latency of serverless functions?
```

You can also just ask "who else has solved something like this?" The skill asks for the problem if you haven't given it.

## Sources

None of these need an API key. Each agent keeps its result counts small and skips any source that errors or rate-limits.

| Agent | Source | Rate limit (no key) |
|---|---|---|
| papers | arXiv API | 1 req / 3 s |
| papers | Semantic Scholar | Pool of ~5,000 req / 5 min shared by all anonymous users, so expect 429s. Skipped after one retry on a 429 |
| papers | Crossref | ~50 req/s if you send a mailto (polite pool) |
| papers | Europe PMC | No published limit; stay around 10 req/s |
| code | GitHub repo search | 10 search req/min and 60 req/hr per IP |
| discussion | HN Algolia | ~10,000 req/hr per IP |
| discussion | Lobsters tag feed (`/t/<tag>.json`), optional | No stated limit; stay around 1 req/s |
| fields | Wikipedia search + summaries | No hard limit for serial requests |

The code agent uses GitHub repo search only, because code search rejects requests that have no token. Rate limits are approximate and may change.

## Example

[examples/cold-start-report.md](examples/cold-start-report.md)

## Evals

Test prompts: [evals/evals.json](evals/evals.json)

## License and credits

License: MIT.

All prompts are original. No code or text was copied from other projects.

**Credits/ideas:** [STORM](https://github.com/stanford-oval/storm) (Stanford OVAL), [TRIZ Agents](https://arxiv.org/abs/2506.18783) (arXiv 2506.18783) and [AutoTRIZ](https://arxiv.org/abs/2403.13002) (arXiv 2403.13002). These were inspiration only.

**Data:** search results come from [Semantic Scholar](https://www.semanticscholar.org/) and the other sources above. Wikipedia content is available under [CC BY-SA](https://creativecommons.org/licenses/by-sa/4.0/).
