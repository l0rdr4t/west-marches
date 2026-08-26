# Report lifecycle: prompting, unreported quests, and editing

Type: grilling
Status: resolved
Blocked by: -

## Question

The report is the campaign's only durable record of the world (ADR 0002), which makes "did
anyone write it" a load-bearing question rather than a nicety.

- When is a report solicited, and from whom? Everyone in the party, or one nominated person?
- Is a played quest without a report a distinct state the interface surfaces, or just an
  absence?
- Anyone who was there can write and edit it (Q23). Does that include the Game Master? What
  about someone who was on the waitlist and turned up anyway?
- Is there one body of text with a shared edit history, or does it accumulate contributions
  attributed to their authors?
- Does the report post into the quest's room, mirror there, or stay separate? What happens
  when it's edited afterwards?
- Can a report be edited forever, or does it settle?
- Does anything depend on a report existing — can a quest be considered finished without one?

## Answer

**One named Scribe owes each report, and the campaign's memory is allowed to have holes.**
Confirmation names a Scribe alongside cutting the party. The quest becomes played when its
slot passes, which starts a 14-day clock. If a report lands, the quest is over; if the clock
runs out, the quest is over anyway and reads "no report" for good.

This overturns Q23. The research (`research/03-practice.md` §4a, contradiction 10) found that
"written and edited by anyone who was there" is the documented failure mode — *"if everyone
may write, no one does"* — and that every campaign which sustained reports used **a named
role assigned before play**: Mors Immortalis' Scribe, Louisville's Notetaker, Arden Vul's
"notes taken by:" byline. Open editing survives as a *fallback*, not as the mechanism.

### Who owes it

- **Confirmation names a Scribe** from the **Party**. It is one more field in the act that
  already cuts the party, so ticket `11` should prototype it there.
- **Named as a Character; owed by the Player behind it.** The party is made of characters, so
  that is what a Game Master picks from at confirmation. But the debt and the reports-written
  count follow the player — characters die and retire, and the obligation must not die with
  them.
- **A Game Master may not write or edit a report, and the system enforces it.** Robbins is
  blunt: *"Don't write game summaries... If you do it, you'll just train them not to."* Domain
  of Many Things goes further — the GM *"must never correct players' interpretations in these
  reports or update the world to match them."* A GM may name or rename the Scribe, and chase
  one. That is all.
- **The Game Master amends attendance after play**, and the amended list is who may edit the
  report. **Party** is who held a seat at confirmation; attendance is who turned up — a
  **Waitlist** character who came along, minus whoever fell ill. Merging the two either locks
  out someone who was there or feeds a session nobody played into "played least recently",
  and the second corrupts ADR 0003's fairness mechanism.
- **If attendance removes the Scribe**, the system asks the Game Master to name a new one
  rather than leaving the debt with someone who wasn't there.

### When it is asked for, and when the quest is over

- **A confirmed quest becomes played automatically when its Slot passes.** Not a Game Master
  action: a GM who forgets to mark it played silently stops the campaign's memory.
- **The quest becomes terminal — and its Room freezes — when the report is published, or 14
  days after the slot ends, whichever is first.** This is the trigger ticket `08` deferred
  here. Terminal-on-report-alone is a trap: an unwritten report would keep the room live
  forever, which is the exact accumulation `08`'s sidebar decision depends on avoiding, and
  the research is explicit that *"sometimes session notes go missing."*
- **A report can still be written after the room freezes.** It does not live in the room.
- **At the deadline the obligation lapses; it does not spread to the party.** Spreading it
  recreates the diffuse responsibility this ticket exists to fix, just later. Anyone who was
  there may still write it, bylined to whoever actually did.
- **"Played, not reported" is a first-class state**, shown on the quest, on the campaign's
  front page and to the Scribe, with the deadline attached so the ask is specific and finite.
  Only played quests can owe a report — declined and abandoned ones never do.
- **14 days is a hard-coded constant, used for both clocks.** Ticket `04` set the precedent:
  one campaign, no settings page. It is explainable in one sentence — *two weeks to write it,
  two weeks to fix it.*

### Writing and editing

- **One body of rich text**, not accumulated attributed contributions. `has_rich_text :body`,
  the same pattern as `Message` (`app/models/message.rb:9`), behind the existing filter chain
  (`app/helpers/content_filters.rb:2`), so rich text and image attachments come free.
  Accumulated contributions *are* a chat log, and the quest already has one next to it.
- **A Scribe byline and a "last edited by" line. No version history.** There is no
  `paper_trail` in the `Gemfile` and no consumer for one.
- **Pre-filled with a heading template**: who was there, where they went, what they found,
  rumours and leads, what they took. This is the recurring structure across unrelated groups,
  and the research makes a specific claim for it — *"the 'rumours/leads' section is the part
  that feeds the next quest, and it is the part a plain chat log never produces."* A Scribe
  given a blank box writes a paragraph. Attendance renders itself from the amended list.
  Headings, not separate fields: a form would fight every quest that doesn't fit.
- **Settles 14 days after publication**, with an administrator able to reopen it for a genuine
  correction. A record anyone can silently rewrite is not durable — the same argument `08`
  made for freezing rooms. There is a domain reason too: when a later quest shows the party
  read something wrong, the correction belongs in the *new* report. The old one should keep
  saying what they thought at the time.

### Where it lives, and who reads it

- **Separate, on the quest page** — with **one message posted into the Room when it is first
  published**. That message is the last thing in the room before it freezes, so the discussion
  ends pointing at the durable record and `08`'s link back out closes the loop from both ends.
- **Edits do not re-post.** The room is frozen by then, and mirroring an edited document into
  a chat log produces neither a good log nor a good document.
- **Readable by the whole campaign**, including players who weren't there and players who join
  later. Mors Immortalis names the alternative as a documented harm: *"unneeded information
  asymmetries and time-wasting efforts to rediscover the same piece of information."* It is
  also the only setting coherent with `08`, which already hands a mid-campaign joiner every
  frozen room — gating the reports while the raw chat stays open would be backwards.
  The counter-instinct, that discovered knowledge is something the party won, is real but is
  a choice about what the Scribe writes down, not a permission the platform enforces.
- **`has_one :report, dependent: :destroy`.** The report dies with its quest, as the room does.
  This makes quest deletion the one irreversible way to lose campaign memory, so its
  confirmation must name the report and the room it will take.

### What this deliberately does not do

The research is unambiguous that every campaign which sustained reports **paid** for them —
an Inspiration, a downtime action, an entry in "the Chronicle" (§4b). We cannot pay any of
those: rewards at the table are out of scope on the map.

So the platform **records the debt and lets a Game Master pay it at the table**: "report owed"
on the quest and the player, and a reports-written count on the profile. Rejected: blocking an
indebted Scribe from declaring **Interest** (coercive, and it punishes the people who
volunteered), and letting reports written improve priority in the cut (a second hidden currency
inside the fairness rule ADR 0003 spent a ticket making legible).

This is the honest limit of the design. The mechanism that demonstrably works is out of scope,
and what remains is a named person and a visible deadline.

### Handed on

- **`06` must define "it didn't happen"** — a confirmed quest cancelled on the day. Without it
  the system demands a report from a party that never met, and feeds a phantom session into
  "played least recently". `07` depends on that path existing exactly as `08` depends on
  abandonment.
- **`11` should prototype naming the Scribe** as part of the confirmation screen.
- **`09` owns when and how the Scribe is actually chased.** This ticket sets the state and the
  deadline, not the notifications.
- **`13` challenges ADR 0002.** A report the Game Master may not write or correct is the
  players' fallible account, not a record of the world — and reports have documented gaps.
  Both `07` and `08` lean on the ADR's current wording.
