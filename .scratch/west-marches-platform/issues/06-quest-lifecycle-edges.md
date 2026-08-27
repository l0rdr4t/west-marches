# Quest lifecycle: cancellation, drop-outs, waitlist promotion, and spent slots

Type: grilling
Status: open
Blocked by: -

## Question

The happy path is settled (ADR 0003). The edges are not, and they are where the design
either holds or falls over.

- What are a quest's exact states, and which transitions are legal? Proposed, confirmed,
  played, abandoned, declined, stale — is that the set?
- What is the party size on a quest: a maximum only, or a minimum too? Can a Game Master
  confirm a quest that hasn't attracted enough interest?
- A confirmed party member drops out. Does the waitlist auto-promote the next character,
  or does someone decide? What if it's an hour before the quest starts?
- A Game Master cancels a confirmed quest. Is the slot returned to the pool? Does the
  quest go back to proposed with its interest intact, or die? Do the players who lost that
  slot for their own quests get it back?
- A Game Master withdraws a slot that nothing has been confirmed into. What happens to the
  availability players declared against it?
- A quest is confirmed and the day passes with nobody marking anything. Does it become
  played automatically, or sit waiting?
- Does being in a party that never actually met count toward "played least recently"?
- What does "stale" mean concretely — how long, and does staleness hide a quest or just
  sort it down? (Q21 settled: no auto-deletion.)
- Surfaced by `04`: does a slot carry a duration or end time, or only a start? Players need to know
  whether they are committing to three hours or six before they declare interest.
- Surfaced by `15`, and now load-bearing for fairness rather than tidiness: **a cancelled
  quest must not spend anybody's standing.** ADR 0005 makes the *seat* the currency, not
  attendance, so a Game Master calling off a session would otherwise silently cost five
  people their turn on the rota. (This also answers the older bullet above — "does being in a
  party that never actually met count toward played least recently" — in the affirmative
  unless this ticket says otherwise, which is precisely the danger.)
- Surfaced by `15`: **stale interest on distant slots.** Deadlines are per slot, so someone
  can tick a date seven weeks out and be seated on it having long since forgotten. Withdrawing
  before that slot's deadline is free and costs no standing; what happens after it is this
  ticket's.
- Surfaced by `15`: **confirming an under-subscribed slot** — `10` found this is the common
  case, not an edge, with two of four candidate slots producing a party of four against five
  seats. Related to the party-size bullet above, and now with a deadline in play: the Rota
  cannot rank a set it has not finished collecting.
