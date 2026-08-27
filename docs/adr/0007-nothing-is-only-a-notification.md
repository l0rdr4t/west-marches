# Nothing is only a notification

Ticket `09` asked which events reach whom, through which channel. The channels turned out to
settle most of the answer, so this ADR takes them first.

**The fork has one channel and no clock.** The only thing in Campfire that produces a push is
a message in a room (`app/models/room/message_pusher.rb`); it reaches only players who are
*disconnected*, never the author; and an `@`-mention pushes a player even at the inherited
default involvement of `mentions`. There is no `ActionMailer`, no mailer, and no SMTP
configuration anywhere in the repo. There is no scheduler either: Resque is the queue
(`config/environments/production.rb:95`), and nothing in this application currently happens
because time has passed.

**We build neither email nor a notification system.** Email would put deliverability, bounce
handling and an unsubscribe surface into a self-hosted single-campaign deployment, and ticket
`04` already set the precedent that this platform does not grow settings. A `Notification`
model would need its own inbox, read state, delivery path and mute story — all of which a room
already has.

## Three surfaces, and what each one is for

- **The Tavern** carries what the whole campaign should notice: a **quest** is proposed, a
  **slot** is offered, a **report** is published. At the inherited `mentions` default this is
  silent-but-visible — an unread on the room everyone reads, pushing nobody. `03`'s
  contradiction 8 says campaigns die of *under-supply of initiative*, which makes the
  visibility of a new proposal load-bearing rather than decorative.
- **The quest's room** carries that quest's own life: **confirmed**, cancelled, and the
  message `07` already posts when a **report** is published. **System messages mention
  nobody** — under this split they are about the quest, never about a person — so `@`-mentions
  stay purely the human affordance they were inherited as.
- **The Herald** carries what happened to *you*. One `Rooms::Direct` per player, written by a
  bot user, which a player cannot write into. Direct rooms default to `everything`
  (`app/models/rooms/direct.rb:19`), so it pushes; unread state, history and a mute control
  all come free.

The rule is short enough to hold in the head: **The Tavern for the campaign's news, the
quest's room for that quest's life, The Herald for your own.**

## The constraint that makes it honest

Push is not reliable and we have no fallback. iOS requires the PWA to be installed, permission
is routinely refused, and subscriptions expire without telling anyone.

So: **no notification may be the only place a fact lives.** Every event below is also legible
as state on a screen someone can walk up to — a **standing** on the quest page, "report owed,
6 days left" on the campaign's front page. If nothing is ever delivered, the campaign runs
slower and still runs. This is the same limit `07` named when it recorded the debt and let a
Game Master pay it at the table, and it binds every screen in the spec, not just this ADR.

It follows that **the platform never overrides a player's own preference.** A coordination tool
that cannot be turned down is one people abandon, and — decisively — a denied browser
permission is outside our reach anyway, so "unmutable" would be a promise we could not keep.
Loudness is set by *defaults*, not by overrides.

## The clock tells, it does not decide

Four clocks are already committed: the 72-hour per-slot deadline (ADR 0005), played-when-the
-slot-passes and the 14-day report lapse (`07`), and abandonment 14 days after the last
confirmable moment (ADR 0006).

**State is derived from time; a recurring job only sends messages.** `played?` is a comparison
against the slot's end, not a column some worker was supposed to set. The job's whole
responsibility is to notice what has newly become true and say so, recording that it did. A
dead or late worker therefore loses messages and never corrupts data — the cheap failure to
have. This puts a scheduler in the `Gemfile`, incurred on behalf of ADR 0006 and `07` as much
as this ADR.

## One message per act, not per event

A single **confirmation** can hit one player four ways at once: seated here, **bumped** on the
quest that lost the date, and — by ADR 0006 — their **interest** ended on a third quest whose
only live date was just spent. Four pushes in one second read as a broken application and bury
the line that mattered.

**The Herald sends one message per act, per player**, composed from every consequence that act
had for them: *"Tuesday 7pm went to The Sunken Mill — you're seated, with Vess. Your interest
in Blackfen ended when that date went."* The cost is that copy becomes a function of
`(act, player)` rather than a template per event type.

## The matrix

| Event | Who hears | Surface | Pushes |
|---|---|---|---|
| A quest is proposed | the campaign | The Tavern | no |
| A slot is offered | the campaign | The Tavern | no |
| Interest is declared | — | silence | — |
| A quest is pointed at your slot | — | silence | — |
| A slot's deadline passes | the slot's Game Master | The Herald | yes |
| A quest is confirmed | the campaign | the quest's room | interested only |
| → you are in the party | you | The Herald | yes |
| → you are on the waitlist, at *n* | you | The Herald | yes |
| → your last live date on another quest went | you | The Herald | yes |
| → you still have live dates there | — | silence | — |
| A slot is withdrawn, and it was your last live date | you | The Herald | yes |
| A seat is given back; the party changes | — | silence | — |
| You are promoted off the waitlist | you | The Herald | yes |
| A confirmed quest is cancelled | the campaign | the quest's room | interested only |
| A confirmed quest is cancelled | the party **and** the waitlist | The Herald | yes |
| A quest becomes played | — | silence | — |
| The scribe is asked, and reminded at day 11 | the scribe | The Herald | yes |
| A report is published | the campaign | The Tavern + the quest's room | no |
| A quest is withdrawn or abandoned | the proposer and everyone who declared interest | The Herald | yes |
| Attendance is corrected | — | silence | — |

Everything marked *silence* remains visible as state, per the constraint above.

Four notes on cells that were arguments rather than derivations:

- **A date disappearing is only an event if it ends your interest.** One rule covers a slot
  spent by a confirmation and a slot withdrawn outright. If you still hold a live date on that
  quest, your **standing** merely moved — and ADR 0005 already made standing provisional and
  meaningless until confirmation, so a message would assert a significance the design denies.
  This is the same reasoning that keeps the whole of pre-confirmation drift silent: on a busy
  quest, telling people would be a message to everyone displaced every time anyone declares,
  and prototype `10` asked for *a forecast, not a verdict*.
- **A cancellation reaches the waitlist too.** ADR 0006 keeps interest intact and returns the
  quest to **proposed**, so the second half of the message — it is off, and it is going back on
  the board — is addressed precisely to the people who might still go.
- **A quest that dies unconfirmed reaches everyone who declared.** A quest with interest on it
  that no Game Master confirmed is the campaign's sharpest signal of trouble, and the people
  best placed to act on it are the ones who wanted to go.
- **The ripe slot is an event `09` never listed.** ADR 0005 gates confirmation until a slot's
  deadline has passed and nothing tells the Game Master when it does. Left silent, quests sit
  and the clock abandons them — a failure produced by ADR 0005 and ADR 0006 together, with no
  owner. This is the only cell aimed at a Game Master *as* a Game Master, and it is what gives
  every confirmation in the campaign a trigger instead of relying on somebody remembering to
  look.

## Involvement follows interest

Every quest room is `Rooms::Open`, so every active player holds a `Membership` on it at the
room's default of `mentions` — which would leave the room where a quest is argued into shape
silent for everybody. `everything` is the opposite failure: forty players pushed for every
message on every live quest, which is the noise-then-mute path `03` documents.

**Declaring interest in a quest raises your own membership on its room to `everything`.** The
strongest signal a player can give that a quest is theirs to follow, and the inherited bell is
always one click from overruling it. **The platform only ever raises an involvement**, never
lowers it: a player who set `everything` by hand and later withdrew interest has said something
the platform must not quietly reverse.

## Notifications name times, in the campaign's zone

`04` handed one question here: whether notifications name times at all. **They do** — a
message that says a quest was confirmed and does not say when is useless — and they are
rendered server-side in `config.time_zone`, which is exactly the assumption `04` made when it
ruled out a per-player zone. The browser-side `local_time_controller.js` cannot reach a push
payload or a message composed by a job, and it does not need to: this is a one-zone campaign
by ADR decision, so a server-rendered time is the right time for everybody.

## Consequences

- **A scheduler enters the `Gemfile`.** There is no clock today. ADR 0006 and `07` needed one
  regardless; this ADR is the third consumer.
- **A room now refuses writes for two reasons, not one.** `08` derives freezing from quest
  state. The Herald refuses always — and unlike a frozen room it keeps its place in the
  sidebar, so only the write-refusal half of `08` is borrowed.
- **The platform gains a bot user.** A `User` with `role: :bot`; the webhook and HTTP posting
  paths are not needed, since the platform writes server-side. The existing `without_bots`
  scope must keep it out of the roster, and it must never hold a character, declare interest,
  or appear on a rota.
- **The Tavern is no longer the only room with no quest behind it.** `CONTEXT.md` is amended.
- **Player-to-player direct rooms have never been decided.** The Herald is a `Rooms::Direct`,
  so the *model* must survive the fork even if the feature does not. Nobody has ruled on
  Campfire's DMs; this ADR now depends on that question having an answer.
- **The campaign's front page carries the weight of the constraint.** Every silence in the
  matrix is a fact that has to be visible somewhere, and most of them land there.
