# Game Masters offer slots and players declare availability across them

**Retitled by ADR 0005.** This ADR was originally titled "...and the cut favours whoever
played least recently". That clause named a rule ADR 0005 replaces; the three acts below
are the decision this ADR actually made, and they stand.

A quest becomes real through three acts. A Game Master offers dated **slots** they
could run in. Interested players declare, per quest, which of its candidate slots they
could make — an availability matrix, not a single pick. The Game Master then **confirms**
the quest into one slot, which spends that slot, and the system cuts the party from the
interested characters, ranked by **who played least recently** (never-played first, ties
broken by who declared earliest). The rest become the waitlist in the same order.

**Amended by ADR 0005.** The three acts stand. The ranking does not: the cut is now made by
**the Rota** — bumps first, then how long since your last seat — it is **binding, with no
Game Master override**, and confirmation is gated on the slot's own deadline rather than
being available at any moment.

Two rejected alternatives are worth remembering. **The proposer picks one time**: simpler,
but it relocates the coordination problem back into chat ("can anyone else do Saturday?"),
which is the exact thing this platform exists to fix. **First-come, first-served on seats**:
the traditional West Marches answer, and it rewards whoever is fastest to refresh rather
than whoever has been waiting longest. In a campaign with a roster larger than a party,
first-come reliably produces a clique.

## Consequences

- Players must be able to see their own standing before confirmation ("3rd of 8, party of
  5"), or the cut reads as arbitrary. **Amended by ticket `10`:** there is no single number,
  and "3rd of 8, party of 5" cannot be written. **Standing is per candidate slot**, because
  the party depends on which slot is confirmed and a character who cannot make Saturday is
  not in the running for Saturday. On a plausible roster the same player, unchanged, is at
  once 3rd of 4 and seated on one date, 6th of 8 and first reserve on another, and absent
  from a third. So the requirement is that a player can see, **per candidate slot**, whether
  they would be seated and where they stand — one answer per slot, not one per quest. ADR
  0005's Rota is built to that shape: bumps, recency and the cut line are all per slot.
- **Added by ticket `10`:** an interest that names no slot is not an interest. Declaring
  interest and naming at least one slot you could make are **one act**, not two: an interest
  with no date cannot be ranked and cannot be seated, so withdrawing your last date withdraws
  you from the quest.
- Confirming one quest into a slot invalidates that slot for every other quest targeting
  it, and those proposers have to be told.
- "Played least recently" needs a definition that survives a player's first quest and a
  player who was on the waitlist. Both sort as never-played. **Answered by ADR 0005:** under
  the Rota they no longer sort alike — the waitlisted player carries a bump and the
  first-timer does not. That symptom is what pointed at the wrong ranking in the first place.
