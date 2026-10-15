# Prototype · Juno

> Module 2 · Prototype. The working build that tests your idea, made with Lovable.

## Prototype link
https://crisp-visual-sync.lovable.app

All tickets, customers, dates and sources in the prototype are simulated.

## What it demonstrates

One flow: a CSM or Sales rep asks a product or tech question in a Slack-style channel between two calls. Juno either answers with sources and a confidence level, or says clearly that it cannot answer and hands the question to the PM in the same public thread.

**Hypothesis tested:** a stranger can tell within 30 seconds whether they can answer their customer or need to wait.

**Four scripted outcomes:**
- **Answered:** a feature question answered from three agreeing sources.
- **Status:** a customer setup status, with the next step on the customer's side.
- **Nothing found:** Juno does not guess, explains what it checked, and offers a "Tag the PM on this thread" button.
- **Date without commitment:** Juno shows the date it found in the tracker, refuses to promise it, flags the conflicting signals, and recommends tagging the PM.

**Skill file constraints visible on screen:** no commitment on dates, scope or priority; no guessing when sources are missing; every reply shows sources, a confidence level with its reason, a next step and a status line.

**Deliberately not built:** "Request" mode (visible but disabled), voice dictation (labeled placeholder), and real connections to Jira, Notion and Slack.

## Debrief

- **What worked:** Scoping to one screen and one flow kept the build focused. Showing sources, a confidence badge and a status line in every reply makes Juno's judgment readable at a glance. Having four distinct outcomes shows where Juno stops, which a single canned answer could not. Preselecting "Question" mode removed a click before the user can ask anything. Using a button that tags the PM inside the thread keeps questions public, so the next person finds the answer instead of asking again.
- **What broke / felt like a toy:** The first build gave the same "I can't commit to dates or scope" reply to every question, so Juno looked like a bot with one answer. It also needed two clicks before a question could be asked, and its first handoff button sent people into a private message, which goes against the goal of keeping questions in the open. All the data is simulated, so the prototype proves the screen is understood, not that Juno's answers would be reliable.
- **What I'd change next pass:**
  1. Connect real sources (Jira, Notion, Slack) so answers are live and checkable.
  2. Build "Request" mode: log the request, categorize it, and route it to a PM review queue.
  3. Detect repeated questions and redirect them to the existing thread.
  4. Test voice with real CSMs and Sales before building it: would they actually speak to Juno between calls?
  5. Run it with real users and measure the share of questions resolved without tagging the PM.
