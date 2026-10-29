# AI PRD · Juno

_Module 3. AI PRD for Juno's front door: internal product and tech Q&A in Slack._

## Problem & user

Sales and CS reps at RocketShip ask product and tech questions in Slack and wait on the only PM for answers that are usually already documented. Juno answers from documented sources, or says it cannot and tags the PM in the same thread. It works inside limits a human set on purpose and defends every call it makes.

## The harness

| Surface | Specification |
|---|---|
| 01 Context | Reads product docs and catalog, the Product Quarterly Roadmap, the Engineering Capacity Note, the Sales Pipeline Snapshot (account, blocker and status only), designated product and incident Slack channels (including #backend-monitoring) and the production branch of the code (read-only, no secrets, no customer data), fetched on demand. Roadmap and Engineering Note are read live; if a source is unreachable, Juno says so and tags the PM. |
| 02 Tools | Eight tools: six reads (search_product_docs, search_roadmap, read_engineering_note, read_pipeline_snapshot, search_slack_channel, search_code) and two sends (post_reply_in_thread, tag_pm_in_thread). No edit, DM or promise-date tool. |
| 03 Loop | Ceiling of 6 steps per question (up to 4 reads, 1 reply, 1 PM tag). Hard timeout of 60 seconds, a safety stop and not a performance target. Stops and tags the PM when the same tool fails twice in a row, after two searches with no usable result, or on conflicting sources. |
| 04 Memory | Team-level log of questions and answers, one entry per thread. A stored answer is trusted only while its cited sources are unchanged. PM corrections outrank everything. |
| 05 Permissions | read auto · draft n/a · write blocked on systems of record (team log and thread reply allowed) · send auto (internal threads, sourced content only). |
| 06 Verification | Every status claim must match a status field in a document retrieved this session. On failure: "I could not confirm this", a PM tag and a "do not promise" label. What Juno posts in each situation is defined in the Situation Table of the Agent Control Panel. |

## 01 Context · Data Requirements

**Required sources, and exclusions**

Reads: product docs and catalog, Product Quarterly Roadmap, Engineering Capacity Note, Sales Pipeline Snapshot (account, blocker and status only, no ARR or contract values), designated product and incident Slack channels (#backend-monitoring for known incidents), and the production branch of the code (read-only, no secrets, no customer data).

Context method: the role and rules are put in the prompt (the skill file). The sources above are fetched on demand through the read tools, so Juno only sees what the question needs. There is no retrieve-and-inject index, and no loading of whole documents by default.

Left out on purpose: user interview transcripts and support tickets (too noisy, risk of exposing confidential detail to the wrong audience); customer data; CEO email and press claims (unscoped, not a source of truth); staffing details such as leave (personal, irrelevant to the question); the Product Strategy one-pager (Juno does not prioritize, the PM does).

Principle: Juno answers status and documented-behavior questions, so it only reads sources that carry a status or a documented fact. Sources that carry opinions, wishes or private customer detail are excluded here. Synthesizing them into insights is out of scope for this version.

**How fresh must it be**

Roadmap and Engineering Note are read live at question time. If a source cannot be reached, Juno does not answer from memory: it says so and tags the PM, because a stale "yes" can end up repeated to a customer.

## 02 Tools · System Capabilities

**The verb list**

- search_product_docs (read)
- search_roadmap (read)
- read_engineering_note (read)
- read_pipeline_snapshot (read)
- search_slack_channel (read)
- search_code (read, production branch only)
- post_reply_in_thread (send)
- tag_pm_in_thread (send)

Not given on purpose: editing the roadmap or tickets, DMs, posting in customer-facing channels, anything that states a date as a promise.

**Deliberate omissions**

- No edit_ticket or edit_roadmap: changing a priority or status changes what engineering and customers expect, and is not undone by deleting a message.
- No DM tool: a private answer hides the answer from the next person, which defeats the public-thread principle.
- No promise_date: dates are PM decisions. Juno may show a date exactly as found in the tracker, labelled "do not promise".

## 03 Loop · AI Costs & Latency

**Step ceiling and escalation**

Ceiling: 6 steps per question (up to 4 reads, 1 reply, 1 PM tag). Stops and tags the PM if the same tool fails twice in a row, if two searches find nothing reliable, or if sources conflict. If the ceiling or the 60-second timeout is hit, Juno posts "I could not finish checking this" with what it found, labels the thread "do not promise" and tags the PM, never a silent cut-off.

**Latency and cost target**

Not defined in this version. We do not yet know the real tokens or seconds per question, so any number would be a guess. The step ceiling and the 60-second safety timeout are the only controls for now: each question is capped, so the cost of one question cannot grow without limit. Next step: run a sample of real questions, record the steps and time per question, then set latency and cost targets from those measurements.

## 04 Memory · Data Requirements

**What persists, at what scope**

Scope: team level, one entry per question thread. Each entry stores the question, thread link, sources cited, date answered and the answer, so a repeated question redirects to the existing thread. PM corrections persist with it. Never persists: customer-confidential details across users.

**Expiry and write rules**

The entry is kept as an index (12 months, adjustable). A stored answer is only trusted while its cited sources are unchanged. On a repeated question Juno re-reads the sources: unchanged, it redirects to the old thread; changed, it posts an update in that thread and says what changed. Status, date and pipeline blocker answers are never reused without a re-check. A PM correction outranks everything, and a newer source outranks an older stored answer. A replaced answer is kept and labelled superseded. Only Juno writes entries; only the PM can override them.

## 05 Permissions · AI Risks & Mitigations

**The permission tier**

| Side-effect class | Tier |
|---|---|
| read | auto |
| draft | n/a in this version |
| write | blocked on systems of record (Jira, roadmap). Allowed: the reply and PM tag in the asking thread, and team log entries |
| send | auto, internal threads only |

**Justification**

Reads auto: blast radius is wasted tokens. Drafts n/a: Juno has no draft stage. It posts a reply or a hand-off directly in the thread, and anything uncertain is handed off to the PM, not drafted. Writes blocked on systems of record: Juno has no tool that edits tickets or the roadmap, a decision on the record. The only writes allowed are the reply and PM tag in the thread where the question was asked, and entries in the team log. Send auto, but only inside internal threads and only with sourced content: a factual answer whose cited source shows the status as shipped, or a hand-off labelled "do not promise" that tags the PM. Customer-facing channels stay blocked. Worst case: a CSM repeats a wrong "yes" to a customer, which cannot be undone once said, hence the shipped-only rule.

## 06 Verification · AI Testing & Measurement

**The check before output ships**

Every status claim in a reply must match a status field in a document retrieved this session, and the reply names that document. A paraphrased or inferred status fails. Bar: 100% of status claims verified before posting. Only a status of shipped allows Juno to answer on its own; planned, in progress or not committed items are handed off with the status shown as found and labelled "do not promise".

**Failure behaviour**

If any claim fails, or sources conflict or are missing, Juno posts "I could not confirm this", tags the PM and labels the thread "do not promise". A Medium-confidence result is posted as links found and what is missing, with no conclusion. It never drops the answer silently and never rewrites a claim to pass the check.

## Eval plan

_Stub. Module 6 fills this in: golden set, pass thresholds, and regression cadence for the verification gate above. The scoring rubric is defined in Module 5._

## Out of scope

Any capability not in the verb list above, and any action tiered `blocked`. Both are decisions on the record, not omissions. Also out of scope in this version: synthesizing signals into insights, drafting specs, ranking risks, and logging or routing requests.
