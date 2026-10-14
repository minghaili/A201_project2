# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
I picked 4 out of 5 tries because my search is a plain keyword match and some phrasings could miss.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
I set 5 out of 5 tries here because anytime if a query that matches no listings, the agent
should stop rather than calling the next tool

---

## 3. State: every tool reads its input out of the session

Two things about `session`, both checked on every run.

**The item survives the hand-off.** On a matching query,
`session["selected_item"]["id"]` equals `session["search_results"][0]["id"]`,
and the item printed on the `select_item` trace line is the same item printed
on the `suggest_outfit` input line — same title, same price, same platform.
5 of 5 runs.

**The session stops where the loop stops.** On the impossible query, the run
ends with `session["search_results"] == []`, `session["error"]` a non-empty
string, and `selected_item`, `outfit_suggestion` and `fit_card` all still
`None` — none of them half-written. 5 of 5 runs.

**Why this target:**
5 of 5 on both, because neither has anything in it that could vary. Selecting
the item is one list index and one assignment, and `suggest_outfit` is handed
`session["selected_item"]` directly — no model call, no ranking, no
randomness sits between the two points I'm comparing. A target of 4 of 5
would be me reserving room for a bug.

Both halves are here because state failure doesn't announce itself. If a
value ever reached `suggest_outfit` by some route other than the session — a
local variable still in scope, say — the run would still finish and still
print a fit card, just one about a different garment than the one the trace
says was selected. And if the early-return path wrote a field before
returning, the error branch would look identical from the outside while
leaving a `selected_item` behind that no tool ever used. The session is only
a useful record of the run if a field being set means the tool that sets it
actually ran, and these are the two ways that stops being true.

This is also the criterion the design in `new_session()` exists to make
checkable. Passing values straight from one call to the next would work and
would be untestable — I can't inspect a variable I've already overwritten.
Routing everything through one dict is what turns "the right item got styled"
from an opinion into a field I can read.



---

## 4. The fit card carries the price and the platform, and doesn't repeat itself

Run the same matching query 5 times. Each fit card names the item's price and
the platform it's listed on, and runs to 4 sentences or fewer — at least 4 of
5 cards meet both. Across those same 5 cards, no two open with the same
sentence — 5 of 5.

**Why this target:**
Two different numbers because they're two different kinds of failure.

4 of 5 on the content, because `TEMPERATURE` is 0.9 in `config.py` — I'm
deliberately paying for varied wording, and a model free to rephrase will
occasionally write "under twenty-five dollars" instead of "$24.00", or let the
platform drop out of a sentence it rewrote. Both are named as explicit
requirements in the prompt, so one miss in five is a wording slip; two would
mean the prompt isn't carrying its instructions and the prompt is what I'd
need to fix.

5 of 5 on the distinctness, because that one isn't about wording quality at
all. `run_eval.py` turns `CACHE_ENABLED` off, so five runs are five real model
calls. Five cards opening identically wouldn't be the model being boring — it
would mean the cache didn't actually get disabled, and that's a configuration
fault with no acceptable failure rate.

I picked price and platform over "sounds like a real post" because I can count
them. Whether a caption reads naturally is an opinion I'd be grading myself on
after the fact.



---

## 5. A price ceiling in the query is respected by every result

When the query names a price ceiling, the ceiling in `session["parsed"]`
matches the number the user typed, and every listing in
`session["search_results"]` has a `price` at or below it. Checked across three
ceilings — $20, $30 and $50 — 5 of 5 runs each.

**Why this target:**
This is arithmetic on a number already in the listing data. No model call, no
keyword scoring, nothing that could go either way on a given run, so anything
short of 5 of 5 would be me writing in room for a bug.

The three ceilings are $20, $30 and $50 because the 40 listings run from $12
to $75, median $28. All three cut the catalogue somewhere real. A $100 ceiling
would sit above every listing in the data and pass without the filter ever
excluding anything — a test that can't fail isn't evidence.

I chose it because it's the failure in this agent that looks most like
success. `parse_query` is a regex over a fixed set of phrasings — "under",
"below", "less than", "max", "up to" — so "nothing over thirty dollars"
matches none of them and `max_price` comes back `None`. No error is raised,
and the run completes normally with a fit card for a $75 jacket the user said
they couldn't afford. Checking the parsed ceiling and the returned prices
separately is what tells the two failures apart: a ceiling that was never
parsed is a parsing bug, and a ceiling that was parsed but exceeded is a
filter bug.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
