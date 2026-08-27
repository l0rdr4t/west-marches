# Which events reach whom, through which channel?

Type: grilling
Status: resolved
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

## Answer

**ADR 0007** — *Nothing is only a notification*. It carries the surfaces, the constraint, the
clock, the composition rule and the full matrix.

### The facts decided most of it

Read before anything was asked, and three of them are load-bearing:

- **The only producer of a push in this fork is a message in a room.** `Room::MessagePusher`
  fires from `Room#receive`, reaches only *disconnected* players, never the author, and an
  `@`-mention pushes a player even at the inherited `mentions` default. That last one is a
  targeted-push mechanism that already exists and costs nothing to build.
- **There is no email.** No `app/mailers`, no `ActionMailer` configuration, no SMTP.
- **There is no clock.** Resque is the queue; there is no `resque-scheduler`, no cron, no
  `recurring.yml`. Nothing in this application happens because time has passed.

An open room also grants a `Membership` to every active user
(`app/models/rooms/open.rb`), so every quest room is a membership row per player, each with
its own involvement, unread flag and sidebar entry.

### Three surfaces, no new system

**The Tavern** for the campaign's news, **the quest's room** for that quest's life, **The
Herald** — a per-player direct room from a bot — for your own. A `Notification` model was
rejected: it needs an inbox, read state, a delivery path and a mute story, every one of which
a room already has. Putting personal news in the quest's room was rejected too — it lands in
front of forty people and detonates the unread badge `01` found is what actually breaks.

### The constraint

Push is unreliable and there is no fallback, so **no notification may be the only place a fact
lives**. Every event is also legible as state on a screen. This is a rule for the whole spec,
not just this ticket, and it is what makes "we never override a player's preference" safe to
say — decisively, because a refused browser permission is outside our reach anyway, so
"unmutable" would be a promise we could not keep.

### What is deliberately silent

Interest declarations. Standing moving before confirmation — `15` established that it moves
after you have seen it, and `10` asked outright whether losing a provisional seat is a
notification. **It is a silence.** ADR 0005 already made standing meaningless until
confirmation, so a message would assert a significance the design denies, and on a busy quest
it is a ping to everyone displaced every time anyone declares. Also silent: a date lost while
you still hold a live one, a seat given back, a quest becoming played, attendance corrections,
and the *first* declaration of interest on your own quest — a taker-count is on the page, and
the failure worth an event is not "nobody declared today" but "nobody ever declared", which
abandonment already covers.

### The three `06` handed here

- **A slot withdrawn that was your only live date** — told, in The Herald. ADR 0006 called
  silence here indefensible and it was right. The same rule covers a slot *spent* by another
  quest's confirmation, which is worth more than the precision of splitting the two causes.
- **A cancelled quest** — told, to the party **and** the waitlist, because ADR 0006 keeps
  interest intact and puts the quest back on the board, so the second half of the message is
  addressed exactly to the people who might still go.
- **Promotion off the waitlist** — told, loudly, and it is the event the whole design is
  weakest at: an hour before play, with push as the only channel. The constraint is the honest
  answer, not a fix.

### The event nobody had listed

**A slot's deadline passes and its Game Master is not told.** ADR 0005 gates confirmation on
that deadline; ADR 0006 abandons quests fourteen days later. Between them they created a
failure with no owner: nobody looks, nothing is confirmed, the clock kills it. The Herald now
tells the slot's Game Master, naming how many quests want the date — the only cell in the
matrix aimed at a Game Master *as* a Game Master, and the one that gives every confirmation in
the campaign a trigger.

### The report, and the tombstone

`07` posts one message into the quest's room when a report is published — but `08` freezes that
room at the same moment and drops it from every sidebar, so the message is a tombstone nobody
reads. Under ADR 0004 the reports *are* the campaign's memory and retrieval is our whole answer
to "what do we know about X", which only works on something you know exists. So a published
report is the **third** thing The Tavern carries, alongside a new quest and a new slot. `07`'s
room message stands as the link out `08` wanted; edits still do not re-post.

### The Scribe, chased twice

`07` set the state and the deadline and left the chase here. **Two private messages in The
Herald** — one when the quest becomes played, with the template, and one at day 11 naming the
three days left — and then silence, because the obligation lapses and `07` already ruled that
the debt does not spread. Nothing public: `07` rejected coercive instruments, and a public
chase is shaming rather than the reward the research says actually works.

### One message per act

A confirmation can hit one player four ways at once. Four pushes in a second read as a fault,
so The Herald sends **one message per act, per player**, composed from every consequence. Copy
becomes a function of `(act, player)` rather than a template per event.

### Involvement follows interest

Quest rooms keep the inherited `mentions` default; **declaring interest raises your own
membership to `everything`**. The platform only ever raises an involvement, never lowers it —
a player who set the bell by hand has said something it must not quietly reverse.

### `04`'s question, closed

Whether notifications name times at all: **they do**, rendered server-side in
`config.time_zone`. `04` already assumed exactly this for push copy when it ruled out
per-player zones. `local_time_controller.js` cannot reach a push payload or a message composed
by a job, and in a one-zone campaign it does not need to.

### Handed on

- **`12` — the front page carries the constraint.** Every silence in the matrix is a fact that
  has to be visible somewhere, and most of them land there. The map already had the front page
  unblocked; this makes it load-bearing rather than nice.
- **`12` — player-to-player direct rooms have never been decided.** The Herald is a
  `Rooms::Direct`, so the *model* must survive the fork even if Campfire's DM feature does not.
  This is a genuine gap, not a deferral: no ticket ever owned it.
- **`12` — a scheduler enters the `Gemfile`.** Owed to ADR 0006 and `07` as much as to this
  ticket; `09` is merely the third consumer.
- **The bot user needs a scope audit.** `without_bots` must keep The Herald's author out of the
  roster, and it must never hold a character, declare interest, or appear on a rota.

### Vocabulary

Added **The Herald**. Amended **The Tavern**, which claimed to be the only room with no quest
behind it.
