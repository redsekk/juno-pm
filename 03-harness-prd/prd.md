# AI PRD · Juno

_Module 3. AI PRD for Juno's front door: internal product and tech Q&A in Slack._

## Problem & user

Sales and CS reps at RocketShip ask product and tech questions in Slack and wait on the only PM for answers that are usually already documented. Juno answers from documented sources, or says it cannot and tags the PM in the same thread. It works inside limits a human set on purpose and defends every call it makes.

## The harness

| Surface | Specification |
|---|---|
| 01 Context | Reads the Product Quarterly Roadmap, Engineering Capacity Note, Sales Pipeline Snapshot (account, blocker and status only) and the #backend-monitoring thread, fetched on demand. Roadmap and Engineering Note are read live; if a source is unreachable, Juno says so and tags the PM. |
| 02 Tools | Six tools: four reads (search_roadmap, read_engineering_note, read_pipeline_snapshot, search_slack_channel) and two sends (post_reply_in_thread, tag_pm_in_thread). No edit, DM or promise-date tool. |
| 03 Loop | Ceiling of 5 tool calls per question. Stops and tags the PM after two failed searches or on conflicting sources. No latency or cost target in this version. |
| 04 Memory | Team-level log of questions and answers, one entry per thread. A stored answer is trusted only while its cited sources are unchanged. PM corrections outrank everything. |
| 05 Permissions | read auto · draft auto · write blocked · send auto (internal threads, sourced content only). |
| 06 Verification | Every status claim must match a status field in a document retrieved this session. On failure: "I could not confirm this", a PM tag and a "do not promise" label. |

## 01 Context · Data Requirements

**Required sources, and exclusions**

Reads: Product Quarterly Roadmap, Engineering Capacity Note, Sales Pipeline Snapshot (account, blocker and status only, no ARR or contract values), and the #backend-monitoring Slack thread (known incidents).

Context method: the role and rules are put in the prompt (the skill file). The four sources above are fetched on demand through the read tools, so Juno only sees what the question needs. There is no retrieve-and-inject index, and no loading of whole documents by default.

Left out on purpose: user interview transcript and support ticket (too noisy, risk of exposing confidential detail to the wrong audience); CEO email and press claims (unscoped, not a source of truth); staffing details such as leave (personal, irrelevant to the question); the Product Strategy one-pager (Juno does not prioritize, the PM does).

Principle: Juno answers status questions, so it only reads sources that carry a status. Sources that carry opinions, wishes or private customer detail are excluded here; they belong to Juno's insight-synthesis job, not this Q&A front door.

**How fresh must it be**

Roadmap and Engineering Note are read live at question time. If a source cannot be reached, Juno does not answer from memory: it says so and tags the PM, because a stale "yes" can end up repeated to a customer.

## 02 Tools · System Capabilities

**The verb list**

- search_roadmap (read)
- read_engineering_note (read)
- read_pipeline_snapshot (read)
- search_slack_channel (read)
- post_reply_in_thread (send)
- tag_pm_in_thread (send)

Not given on purpose: editing the roadmap or tickets, DMs, posting in customer-facing channels, anything that states a date as a promise.

**Deliberate omissions**

- No edit_ticket or edit_roadmap: changing a priority or status changes what engineering and customers expect, and is not undone by deleting a message.
- No DM tool: a private answer hides the answer from the next person, which defeats the public-thread principle.
- No promise_date: dates are PM decisions.

## 03 Loop · AI Costs & Latency

**Turn ceiling and escalation**

Ceiling: 5 tool calls per question (up to 3 reads, 1 reply, 1 PM tag). Stops and tags the PM if two searches find nothing reliable or if sources conflict. If the ceiling is hit, Juno returns its best answer labelled "turn limit reached" and tags the PM, never a silent cut-off.

**Latency and cost target**

Not defined in this version. We do not yet know the real tokens or seconds per question, so any number would be a guess. The turn ceiling above is the only cost control for now: each question is capped at 5 tool calls, so the cost of one question cannot grow without limit. Next step: run a sample of real questions, record the tool calls and time per question, then set latency and cost targets from those measurements.

## 04 Memory · Data Requirements

**What persists, at what scope**

Scope: team level, one entry per question thread. Each entry stores the question, thread link, sources cited, date answered and the answer, so a repeated question redirects to the existing thread. PM corrections persist with it. Never persists: customer-confidential details across users.

**Expiry and write rules**

The entry is kept as an index (12 months, adjustable). A stored answer is only trusted while its cited sources are unchanged. On a repeated question Juno re-reads the sources: unchanged, it redirects to the old thread; changed, it posts an update in that thread and says what changed. Status, date and customer-setup answers are never reused without a re-check. A PM correction outranks everything, and a newer source outranks an older stored answer. A replaced answer is kept and labelled superseded. Only Juno writes entries; only the PM can override them.

## 05 Permissions · AI Risks & Mitigations

**The permission tier**

| Side-effect class | Tier |
|---|---|
| read | auto |
| draft | auto |
| write | blocked |
| send | auto |

**Justification**

Reads auto: blast radius is wasted tokens. Drafts auto: a private draft harms nobody. Writes blocked: Juno has no tool that edits tickets or the roadmap, a decision on the record. Send auto, but only inside internal threads and only with sourced content: a factual answer whose cited source shows the status as shipped, or a hand-off labelled "do not promise" that tags the PM. Customer-facing channels stay blocked. Worst case: a CSM repeats a wrong "yes" to a customer, which cannot be undone once said, hence the shipped-only rule.

## 06 Verification · AI Testing & Measurement

**The check before output ships**

Every status claim in a reply must match a status field in a document retrieved this session, and the reply names that document. A paraphrased or inferred status fails. Bar: 100% of status claims verified before posting.

**Failure behaviour**

If any claim fails, or sources conflict or are missing, Juno posts "I could not confirm this", tags the PM and labels the thread "do not promise". It never drops the answer silently and never rewrites a claim to pass the check.

## Eval plan

_Stub. Module 6 fills this in: golden set, pass thresholds, and regression cadence for the verification gate above._

## Out of scope

Any capability not in the verb list above, and any action tiered `blocked`. Both are decisions on the record, not omissions.
