---
name: crosspollinate
description: Find solutions to a problem by borrowing from distant fields. Abstracts the problem, picks 4-6 far-away domains, searches papers, code, discussions and encyclopedic sources in parallel, ranks the analogies, and writes a one-page crosspollinate-report.md. Use this whenever the user is stuck, out of ideas, or wants fresh angles on a technical, product or research problem, and whenever they ask "who else has solved something like this?", "how would nature (or another industry) handle this?", or mention analogies, biomimicry, TRIZ or lateral thinking, even if they don't name the skill.
compatibility: Requires Task, WebFetch and WebSearch tools; network access to no-key APIs.
---

# crosspollinate

Turn one problem into ranked, sourced ideas from other fields.

The best outside ideas come from fields that met the same *abstract* pattern under a different name. An ambulance dispatcher and a serverless engineer both decide how much idle capacity to keep warm, but they'd never search for each other's words. So the skill first strips the problem down to its pattern, then sends searchers to fields that live with that pattern, and only then ranks what comes back. For a finished output, see `examples/cold-start-report.md`.

## 1. Get the problem

If the user hasn't described the problem yet, ask for it in one or two sentences, including any constraints (budget, scale, hard limits). Don't ask anything else. Everything after this point runs without the user, so one short question is all the friction this step should add.

## 2. Abstract it and pick fields

Work through these steps silently, then show the result to the user:

1. **Strip the jargon.** Restate the problem without domain words, as a verb plus an object plus a condition (for example "keep many independent parties in agreement despite unreliable messages"). Domain words only find the user's own field, so any left in the restatement will pull the searches back home.
2. **Name the contradiction.** "Improving X makes Y worse." If there's more than one, keep the sharpest. This is the thing a good idea has to break, and the ranker scores fit against it.
3. **Write 2-3 abstract search phrases** made of generic terms that someone in an unrelated field would use for the same pattern.
4. **Pick 4-6 distant fields.** For each, imagine an expert who meets this abstract pattern every day. Give them a one-line persona ("a hospital triage nurse who has to rank patients with incomplete information"). The persona tells the agents what that expert would actually know to look for. Rules:
   - At least one field from biology or ecology, one from physical engineering or the physical sciences, and one from a human system (economics, logistics, law, sports, military, or the arts). Mixing the three kinds stops the run from collapsing into a single kind of answer.
   - Leave out the user's own field and its close neighbours. The user already knows those ideas.
   - Prefer fields that have *already solved* the pattern, not fields that just resemble it. A solved field leaves behind papers, patents and code to cite.

Show the user a short block with the abstract problem, the contradiction, and the fields with their personas. Continue straight away unless they object; don't wait for approval. The searches are the slow part, and the user can still redirect you while they run.

## 3. Fan out (parallel)

Read the four prompt files in this skill's `agents/` folder: `papers.md`, `code.md`, `discussion.md` and `fields.md`. Launch **all four in a single message** as parallel Task tool calls (subagent_type `general-purpose`). Calls made in separate messages run one after another, which roughly quadruples the wait. Each Task prompt is the file's contents followed by this context block:

```
PROBLEM: <user's problem, verbatim>
ABSTRACT: <abstract restatement>
CONTRADICTION: <contradiction>
SEARCH PHRASES: <the 2-3 abstract phrases>
FIELDS:
- <field>: <persona>
...
```

If an agent fails or comes back empty, carry on with the others. Don't retry more than once. The other three sources usually cover the gap, and a second failure normally means the source is rate-limited or down.

## 4. Check URLs, then rank

Merge every item the agents returned into one list. Open each `source_url` with WebFetch. The ranker can't check URLs itself, and search summaries sometimes attach a claim to the wrong page:
- The page loads and supports the item: keep it.
- 404, a dead domain, or a page that doesn't support the claim: drop the item.
- A DOI that resolves (30x) to a publisher page that returns 403 or a cookie wall: keep it, and append `resolves, blocked for bots` to its `evidence`. The paper exists; the publisher just blocks automated readers.

Then launch one more Task with the contents of `agents/ranker.md` followed by:

```
PROBLEM: <user's problem>
CONTRADICTION: <contradiction>
ITEMS:
<all items, as returned>
```

## 5. Write the report

Write `crosspollinate-report.md` to the user's **current working directory**, not the skill folder. Keep it to about one page (roughly 500-700 words), so the user can read it in one sitting and pick an idea to try. Use this format:

```markdown
# Crosspollinate: <short problem title>

**Problem (abstracted):** <one line>
**Core contradiction:** <one line>
**Fields searched:** <comma list>

## Top ideas

### 1. <solution name> — from <field>  (fit 5 · novelty 4 · credibility 4)
<2-3 sentences: what the other field does, and how it maps onto the user's problem.>
Source: <url>

... (up to ~8)

## Try first
<1-2 sentences: the single highest-leverage experiment, built from the top idea.>

---
<footer lines, only those that apply:>
Data from Semantic Scholar.
Quoted text from Wikipedia: "<article title>" (<url>), CC BY-SA 4.0.
```

The footer comes from the source terms. Semantic Scholar's API license requires attribution, and Wikipedia text is CC BY-SA 4.0, which needs a link and the license name wherever it's quoted. Footer rules:
- Add `Data from Semantic Scholar.` if any idea in the report has `via` set to semantic scholar.
- For each Wikipedia sentence quoted in the report, add one line with the article link and `CC BY-SA 4.0`. Paraphrased Wikipedia content needs only the normal `Source:` link.
- If neither applies, leave out the footer entirely.

Then tell the user the file path and name the top three ideas in one line each.
