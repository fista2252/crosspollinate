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

- **papers:** arXiv, Semantic Scholar, Crossref, Europe PMC
- **code:** GitHub repo search
- **discussion:** Hacker News, Lobsters
- **fields:** Wikipedia

No API keys or sign-ups needed. Busy or rate-limited sources are skipped automatically.

## Example

[examples/cold-start-report.md](examples/cold-start-report.md)

## Evals

Test prompts: [evals/evals.json](evals/evals.json)

## License and credits

License: MIT.

All prompts are original. No code or text was copied from other projects.

**Credits/ideas:** [STORM](https://github.com/stanford-oval/storm) (Stanford OVAL), [TRIZ Agents](https://arxiv.org/abs/2506.18783) (arXiv 2506.18783) and [AutoTRIZ](https://arxiv.org/abs/2403.13002) (arXiv 2403.13002). These were inspiration only.

**Data:** search results come from [Semantic Scholar](https://www.semanticscholar.org/) and the other sources above. Wikipedia content is available under [CC BY-SA](https://creativecommons.org/licenses/by-sa/4.0/).
