# Can a Game Master propose a quest?

Type: grilling
Status: open
Blocked by: -

## Question

`CONTEXT.md` defines a **quest** as "proposed by a player", and ADR 0003 builds the whole
three-act flow on it: a player proposes, Game Masters offer **slots**, players declare
**interest**, a Game Master **confirms**. Ticket `03` found that this is the minority path in
every campaign it could observe.

- Lutes measured the split at **70% Game-Master-posted, 30% player-initiated**.
- Colvillian runs `#dm-quest-posting` and `#player-quest-request` as **two separate channels
  with different rules** — they are different objects, not one object with a different author.
- Lutes' "town opportunities" is a job board the Game Master stocks; Ghelfi and
  Hipsters & Dragons describe Game-Master-posted missions only.
- The founding text is the other way: Robbins' players email the list with destination, date
  and recruitment call together. But Robbins had 10-14 players and no job board.

The questions:

- Can a Game Master propose a quest at all? If not, what stops them from proposing one through
  a player, which is what a workaround looks like on day one?
- Is a Game-Master-proposed quest the *same object* with a different proposer, or a different
  object with different rules? Colvillian's answer is the second, and ticket `15` depends on
  which.
- A player-proposed quest needs a Game Master to pick it up and offer slots for it. A
  Game-Master-proposed quest arrives with its slots already attached — does that collapse
  proposal and slot-offering into one act, and is that a different lifecycle?
- Does a Game-Master-proposed quest still need **interest** declared against candidate slots,
  or does it carry one date and take signups? All three major bots and westmarches.games use
  single-datetime-plus-RSVP (`03` §5b, §5c).
- If both paths exist, does the campaign's front page show one list or two?
- Does anything change about who may edit or withdraw a quest that a Game Master proposed?
  (Overlaps the roster ticket `05`.)

Read `research/03-practice.md` §5a and contradiction 1 first.

**Coupled to `15`.** Colvillian's carve-out — that player-proposed quests are exempt from the
recency cut — keys off exactly the distinction this ticket draws. The two are answerable in
either order, but probably want taking in one sitting.
