# West Marches

A coordination tool for a single West Marches tabletop RPG campaign: a large roster
of players with no fixed party, who propose expeditions and self-organise who goes
and when, around the availability of one or more Game Masters. It covers the time
*between* sessions. Play itself happens elsewhere, at a table or on a VTT.

Built on a hard fork of Campfire, so some inherited vocabulary is deliberately
redefined below.

## The campaign

**Campaign**:
The single, whole shared world this deployment serves. One deployment is one
campaign; there is no notion of a second.
_Avoid_: Account, tenant, workspace, server

**Game Master**:
A person who runs quests. A campaign has several, and any of them may propose a
quest, offer times, and confirm quests into them. It is a standing property of a person, granted and
revoked by an administrator — not a role held for one quest, and not something that
makes them any less a player. The Game Master *of* a quest is never stored: it is
whoever owns the slot the quest was confirmed into.
_Avoid_: GM as a possessive ("the GM"), DM, referee, owner

**Player**:
A person who proposes quests and goes on them. Game Masters are also players when
they aren't running, with their own characters and no special restrictions — save
that a character is never a candidate for a slot its own player offered.
_Avoid_: Member, user, participant

**Character**:
A persona belonging to one player, with a name and a level. A player may have
several, creates them freely, and they are what actually goes on a quest. Nobody
admits a character to the campaign. The level is a note for people to read: the
platform records it and never reasons about it.
_Avoid_: PC, avatar, hero, sheet

**Roster**:
Everyone on the campaign, which is simply everyone with an account — there is no
membership record to hold and no pending state. A person joins with the campaign's
one shared join code, which only administrators can see. Leaving is normally silent:
someone who declares no interest is already absent from every party.
_Avoid_: Members, signups, the server, the party

**Administrator**:
Whoever runs the deployment — the join code, the campaign rooms, deactivation. A
standing property separate from Game Master, held by one or two people, and neither
implies the other. An administrator's mistakes are operational; a Game Master's are
social.
_Avoid_: Owner, moderator, staff, superuser

## Organising an expedition

**Quest**:
A single proposed expedition: where the party is going and what they mean to do
there. Proposed by a player, or by a Game Master — whose proposal arrives with its
slots already attached — then discussed, scheduled into one slot, played once,
and then done. Unfinished business in the world comes back as a *new* quest, which may
say which quest it *follows from* — a pointer for people to read, not a structure the
system traverses. Nothing is inherited along it. Its life is six states: **proposed**,
**confirmed**, **played** (its slot has passed), **closed** (reported, or 14 days on),
and from proposed the two dead ends **withdrawn** and **abandoned**. Cancelling is not a
state — a cancelled quest goes back to proposed with its interest intact.
_Avoid_: Session, adventure, event, game, mission, module

**Slot**:
A dated window a Game Master has offered to run in, with a start and an end. Offered
**unattached**: it names no quest, and quests are pointed at it — by their proposer or
by a Game Master — which makes it one of their candidate slots. Several quests may
target the same slot. Exclusive: once a quest is confirmed into it, it is spent, for
every other quest too. Carries its own deadline — interest in it closes 72 hours before
it starts, set by the Game Master who offered it, and it cannot be confirmed until then.
_Avoid_: Availability, time, date, session slot, booking

**Interest**:
A player's declaration, with a named character, that they want to go on a quest,
together with which of its candidate slots they could make. Names **at least one**
slot — an interest with no date cannot be ranked or seated, so it is not one, and
withdrawing the last date withdraws the interest. Not yet a seat.
_Avoid_: Signup, RSVP, application, vote

**Rota**:
The order the party is cut in. Ranks the interested players by bumps since their last
seat, then by how long since that seat — never seated sorts as longest — then by who
declared first. Computed per candidate slot, applied at confirmation, and binding: a
Game Master picks the slot, never the party.
_Avoid_: Priority queue, rotation, algorithm, the list, least recently played

**Bump**:
What you carry away when the slot you declared for is confirmed and you were not
seated. The unit the rota counts, and the thing that earns your next seat. A quest
that dies unconfirmed bumps nobody, and a slot you could not have made cannot bump
you — you were never in the running for it.
_Avoid_: Miss, rejection, strike, snub, loss

**Standing**:
Where a player sits in the order the party would be cut in, for one candidate slot —
"4th of 6, seated" or "6th of 8, first on the waitlist". Read off the rota, and
provisional: it moves as others declare and means nothing until confirmation. There is one per candidate slot, not one per quest, because a
character who cannot make a date is not in the running for it. Names the player's
position, not the rule that produces it — that is the rota.
_Avoid_: Rank, priority, score, place in line, queue position

**Party**:
The characters holding a seat on a confirmed quest. Who was *given* a seat, not who
turned up — see attendance. A seat may be given back free at any time before the quest
starts, which promotes the waitlist; keeping it and not appearing spends your standing.
_Avoid_: Group, team, roster, attendees

**Attendance**:
Who actually went, as corrected by a Game Master after play. It may add a waitlist
character who came along anyway, and drop someone who fell ill. It decides who may
edit the report. It does *not* feed the rota — a seat spends your standing whether or
not you walk in — so it is a record of what happened and nothing hangs on it but the
report.
_Avoid_: Turnout, present, roll call, the party

**Waitlist**:
The characters who declared interest in a confirmed quest but did not make the cut.
The rota continuing past the cut line, and frozen at confirmation: the order does not
re-sort afterwards, so "first seat to free up is yours" stays true. Promotion is
automatic and needs nobody's decision, at any hour.
_Avoid_: Queue, standby, overflow

**Confirmation**:
The Game Master's act of binding a quest to one of its candidate slots. Where several
quests target the same slot it chooses between them too, so it settles which quest and
which date at once. Possible only once that slot's deadline has passed. It is the single moment a quest becomes real: it
spends the slot, cuts the party from the interested characters by the rota, creates the
waitlist, and derives the scribe. The Game Master chooses the slot and nothing else — the
cut is the rota's and cannot be overridden, and neither is the scribe theirs to name.
_Avoid_: Approval, booking, scheduling, locking

**Level range**:
The band of character levels a quest is pitched at, written on the quest for people
to read. The platform never compares it to a character's level: nothing is flagged and
nothing is refused.
_Avoid_: Requirement, tier gate, restriction

## Afterwards

**Report**:
The account of what happened on a quest. One per quest, owed by its scribe and
editable by anyone in its attendance. It is the players' account of what they
believe they saw, not an authoritative record of the world: a Game Master may
neither write one nor correct one. Quests do not persist, so the reports are what
the campaign remembers — with holes where nobody wrote one.
_Avoid_: Recap, log, summary, minutes, journal

**Scribe**:
The one character in a party who owes a quest's report. **Derived at confirmation, not
chosen**: it falls to whoever in the party has gone longest without writing one,
never-scribed first, ties broken by rota position. A Game Master no more picks the scribe
than they pick the party. Derived as a character because that is what a party is made of,
but the debt belongs to the player behind it and outlives that character. Others may edit
the report; only the scribe is asked for it.
_Avoid_: Author, note-taker, recorder, secretary

**Room**:
The chat room attached to a quest, inherited from Campfire, where the expedition is
argued over before it happens and after. Open to the whole campaign, not just the
party, for its whole life — it is never narrowed to the party when the quest is
confirmed. A quest owns its room: the room is created with the quest and destroyed
with it, and cannot be deleted on its own.
_Avoid_: Channel, thread, chat, discussion

**Frozen**:
What a room becomes when its quest is over. It accepts no further writes — no
messages, no boosts, no edits to what is already there — leaves the sidebar, and is
read by following a link from its quest. Frozen is not a state a room holds; it is
read off the quest, so reopening the quest reopens the room. Everything stays
readable and searchable.
_Avoid_: Archived, closed, locked, read-only

**The Tavern**:
The single campaign-wide room, where the campaign talks when no quest is at stake. The
only *open* room with no quest behind it, and one of the two that never freeze — see the
Herald for the other. There is exactly one. The platform speaks here only about what the
whole campaign should notice: a new quest, a new slot, a new report.
_Avoid_: General, lobby, off-topic, All Talk

**The Herald**:
The room where the campaign tells you what happened to *you* — that you have a seat, that
you were promoted, that your interest ended when its last date went. One per player,
written only by the platform, and a player cannot write back into it. It never freezes,
and nothing that happens to somebody else appears in it. What happened to a *quest* is
said in that quest's room instead, and what the whole campaign should notice is said in
the Tavern.
_Avoid_: Notifications, alerts, inbox, feed, DM
