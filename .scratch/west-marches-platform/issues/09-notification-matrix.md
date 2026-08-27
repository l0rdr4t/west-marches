# Which events reach whom, through which channel?

Type: grilling
Status: open
Blocked by: -

## Question

Campfire ships Web Push and per-room involvement levels. A coordination tool lives or dies
on whether people find out things in time, and equally on whether it becomes noise they
mute.

Work through the events and decide, for each, who hears about it and how loudly:

- A quest is proposed
- A Game Master offers a new slot
- Someone declares interest in your quest
- Your quest is confirmed into a slot
- You made the party
- You were waitlisted, and where you stand
- You were promoted off the waitlist
- A slot you were counting on was spent by another quest
- A quest you're on was cancelled, or its party changed
- A quest you were on is played and nobody has written the report
- ~~Your quest went stale~~ — **struck by `06`**: there is no stale state. Your quest was
  **abandoned** by the clock, 14 days after the last moment anything could confirm it. Whether
  a proposer hears that, or whether it is a silence, is this ticket's.

Then: which of these are push, which are in-app only, which are digests, and which are
nothing at all? Does a player get any say in it? Does the "you made the party" moment need
to be louder than everything else?

Ticket `06` is resolved, and the transitions are settled — ADR 0006 has the state set. It
hands three events here, one of them the sharpest in the set:

- **A Game Master withdrew a slot, and it was your only live date on a quest.** Your interest
  in that quest is over, by ADR 0003's rule that an interest naming no live date is not one.
  That is somebody else's act silently removing you from something you had signed up for, and
  it is the one event in this list where saying nothing is indefensible.
- **A quest you are on was cancelled.** The party already has the date in a diary, and under
  ADR 0006 the quest goes back on the board rather than dying, so the message has two halves:
  it is off, and it is still going to happen sometime.
- **You were promoted off the waitlist**, possibly an hour before play. ADR 0006 made promotion
  automatic at any hour specifically so no human has to decide — which only works if the person
  who just got a seat finds out in time to use it.

Also settled since this ticket was written, and bearing on it: `15` established that **standing
moves after you have seen it** — someone else declaring can push a player below the line with
no event they would notice — and `10` asked outright whether losing a provisional seat is a
notification or a deliberate silence. That question is still open and it is this ticket's.

Read ADR 0005 and ADR 0006 for the transitions, and `07` for the report clock — it sets the
state and the deadline but explicitly leaves "when and how the Scribe is actually chased" here.
