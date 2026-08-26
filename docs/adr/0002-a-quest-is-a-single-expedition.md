# A quest is a single expedition, not a durable place

A quest is proposed, scheduled into one slot, played once, and finished. It is one
model with a lifecycle, not a durable "quest" paired one-to-one with a "session".
When a party runs out of time or leaves something undone, that comes back as a new
quest, proposed afresh.

We considered the durable shape — a quest that persists and accumulates many sessions
over the campaign — which models revisiting a location more faithfully. We rejected it
because it buys structure the campaign does not need: a 1:1 pair is a sync bug waiting
to happen, and a many-sessions quest raises questions (whose party? which level range
now?) that the ephemeral shape simply does not ask.

## Consequences

- Quests carry no history. **Amended by ADR 0004:** the report is *not* the campaign's
  durable record of the world — it is one party's account of what they believe they saw,
  with permanent holes where nobody wrote one. The campaign has no world model.
- "Same place, next week" has no first-class expression. **Amended by ADR 0004:** it
  chafed. A quest may now name the quest it follows from — a pointer for people to read,
  which nothing in the system traverses.
