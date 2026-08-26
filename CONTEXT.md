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
A person who runs quests. A campaign has several, and any of them may offer times
and confirm quests into them. It is a standing property of a person, granted and
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
admits a character to the campaign.
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
there. It is proposed by a player, discussed, scheduled into one slot, played once,
and then done. Unfinished business in the world comes back as a *new* quest, which may
say which quest it *follows from* — a pointer for people to read, not a structure the
system traverses. Nothing is inherited along it.
_Avoid_: Session, adventure, event, game, mission, module

**Slot**:
A dated window a Game Master has offered to run in. Exclusive: once a quest is
confirmed into it, it is spent.
_Avoid_: Availability, time, date, session slot, booking

**Interest**:
A player's declaration, with a named character, that they want to go on a quest,
together with which of its candidate slots they could make. Not yet a seat.
_Avoid_: Signup, RSVP, application, vote

**Standing**:
Where a player sits in the order the party would be cut in, for one candidate slot —
"4th of 6, seated" or "6th of 8, first on the waitlist". Read off who has played least
recently, and provisional: it moves as others declare and means nothing until
confirmation. There is one per candidate slot, not one per quest, because a character
who cannot make a date is not in the running for it. Names the player's position, not
the rule that produces it.
_Avoid_: Rank, priority, score, place in line, queue position

**Party**:
The characters holding a seat on a confirmed quest. Who was *given* a seat, not who
turned up — see attendance.
_Avoid_: Group, team, roster, attendees

**Attendance**:
Who actually went, as corrected by a Game Master after play. It may add a waitlist
character who came along anyway, and drop someone who fell ill. It decides who may
edit the report, and it is what "played least recently" counts, so it must be true
rather than convenient.
_Avoid_: Turnout, present, roll call, the party

**Waitlist**:
The characters who declared interest in a confirmed quest but did not make the cut,
kept in priority order in case a seat frees up.
_Avoid_: Queue, standby, overflow

**Confirmation**:
The Game Master's act of binding a quest to one of its candidate slots. It is the
single moment a quest becomes real: it spends the slot, cuts the party from the
interested characters, creates the waitlist, and names the scribe.
_Avoid_: Approval, booking, scheduling, locking

**Level range**:
The band of character levels a quest is pitched at. Advisory: an out-of-range
character is flagged, never refused.
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
The one character named at confirmation who owes a quest's report. Named as a
character because that is what a party is made of, but the debt belongs to the
player behind it and outlives that character. Others may edit the report; only the
scribe is asked for it.
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
The single campaign-wide room, where the campaign talks when no quest is at stake.
The only room with no quest behind it, and so the only one that never freezes. There
is exactly one.
_Avoid_: General, lobby, off-topic, All Talk
