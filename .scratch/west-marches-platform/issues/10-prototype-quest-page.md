# Prototype: the quest page, as an interested player sees it

Type: prototype
Status: open
Blocked by: 01, 04

## Question

This is the screen the platform is judged on. A player arrives at a proposed quest and has
to understand, without being told: what it is, whether it's pitched at their character,
which of the Game Master's slots they could make, who else wants in, and where they stand
in the queue.

Build a rough, throwaway prototype that makes those concrete enough to react to:

- The quest itself: title, description, level range, party size, who proposed it
- The availability matrix — the candidate slots crossed with the interested characters.
  This is the hard part. It has to stay legible with eight interested players and four
  slots, on a phone.
- Declaring interest: choosing which character, then which slots you can make
- Your own standing before confirmation: "3rd of 8, party of 5". ADR 0003 flags that
  without this, the cut reads as arbitrary. Find out what makes it feel fair rather than
  discouraging.
- The out-of-range warning when your character's level doesn't match (Q16: advisory, never
  refused)
- The way into the quest's room

The prototype is disposable. Its job is to provoke "no, not like that" — not to be kept.

## Findings

Prototype: [`prototypes/10-quest-page/quest-page.html`](../prototypes/10-quest-page/quest-page.html).
One self-contained file, no build, nothing wired to Rails. Open it in a browser and use the
floating bar: `←`/`→` (or the arrow keys) cycle three variants, `you` toggles before/after
declaring, `width` swaps a 390px phone frame in. It is live — ticking a date recomputes every
party, waitlist and position on the page, which is the part worth playing with. Fake data is the
ticket's stress case: 8 interested characters, 4 candidate slots, party of 5, one character below
the level range and one above.

- **A · The Board** — the availability matrix. Rows are the interested characters in the order the
  party is cut in; columns are the slots; your own row is the declaration control.
- **B · Four Dates** — the matrix transposed into a stack of slot cards. No grid anywhere; each
  card shows the party *that date would produce*.
- **C · You First** — your declaration leads, the collective picture is reduced to counts and a
  bar per date, and who is interested sits behind a disclosure.

**The matrix problem was not the one the ticket expected.** Eight characters by four slots does
fit on a phone, without sideways scrolling, once the avatars and the player names are dropped and
the header is stacked (`Fri` / `4 Sept` / `17:00` / `4/8`). Fitting it was an hour's CSS. The
actual problem is that **a grid of ticks answers the Game Master's question, not the player's.**
"Who can make what" is a comparison across slots, which is the input to **confirmation**. The
player arriving has three questions — is this pitched at me, when might it happen, am I on it —
and none of them is a cross-tab.

The matrix only became worth its space when the rows were **sorted by the cut order and the cut
line drawn per column**. At that point it stops being a spreadsheet and starts being the argument:
you can see who is above you, why (the recency is printed on every row), and where the line falls
on each date. But note what that does — once the matrix carries the order and the cut, **the
separate "3rd of 8, party of 5" widget is redundant**, because the standing is readable off the
grid. The two do not both earn their place.

### "3rd of 8, party of 5" cannot be written

ADR 0003 asks for one number. There isn't one. **Standing is per candidate slot**, because the
party depends on which slot is confirmed and a character who can't make Saturday isn't in the
running for Saturday. In the prototype's data the same player, unchanged, is simultaneously:

| Date | | |
| --- | --- | --- |
| Fri 4 Sept | 3rd of 4 | seated — and the party is a seat short |
| Sat 12 Sept | 6th of 8 | 1st on the waitlist |
| Sun 13 Sept | 4th of 6 | seated |
| Wed 23 Sept | — | can't make it |

That is not a contrived arrangement; it falls out of a plausible roster. The consequence in ADR
0003 should be restated as *"a player must be able to see, per candidate slot, whether they would
be seated and where they stand"*, and whatever ticket `15` decides has to produce up to four
answers, not one.

### What makes standing feel fair rather than discouraging

Five things, all of which came out of writing the copy rather than out of the layout:

1. **It has to be a forecast, not a verdict.** Every number on the page is hedged — *"the party
   this date would produce"*, *"you'd be on it"*, *"the cut, if that date were confirmed today"* —
   and the page says outright that nothing is decided until a Game Master picks a date. A bare
   "6th of 8" is a door closing; "6th of 8 **so far**" is a queue you are standing in.
2. **The reason is printed on every row, not just yours.** "played 11 days ago", "never played".
   What makes a cut arbitrary is not losing, it is losing to an unexplained ordering. If the
   reason is public for everyone, the ordering is checkable by the person it disadvantages, and
   the ranking stops being the platform's opinion.
3. **The reason must not round.** First draft said "played 2 weeks ago" for two characters who
   ranked 5th and 6th. Two identical reasons in different positions reads as a bug or as favour —
   exactly the failure ADR 0003 is guarding against. Exact days under four weeks, weeks beyond.
4. **A losing position must come with a future.** The only way found to make the waitlist bearable
   was to say what the position is *worth*: *"1st on the waitlist — first seat to free up is
   yours."* This is a design request aimed at `15`: the Alexandrian's carry-over is not only the
   better fairness rule, it is the only copy that makes being cut survivable. Without it, being
   6th is information with nothing attached to it.
5. **Say the thing the player will suspect.** One line — *"which character you bring doesn't change
   your place in the order, that follows you, not them"* — because the first theory a bumped player
   forms is that the level range or the character choice was held against them.

### The recommendation

**B is the player's page. A is ticket `11`'s screen.** The per-slot card answers the player's
actual question, never needs a grid, and degrades gracefully — it is a list, so ten slots and
twenty interested characters cost scrolling, not legibility. A degrades badly: a fifth or sixth
slot column breaks the phone, and it is precisely the Game Master who needs six columns.

Two things to lift from A into B: the **numbered priority order inside each card** (already there
on the chips) and the **recency line** on each character, which B currently drops and which is
what makes the ordering checkable.

**Rejected, with reasons worth keeping:**

- **A grid that scrolls sideways on a phone.** If the cross-tab needs horizontal scrolling, the
  cross-tab is the wrong shape for that screen; scrolling hides exactly the column you are being
  asked to compare against.
- **Leading with "4 of 8 can make this."** It is the most available number and it is the Game
  Master's number. Leading with it teaches players to optimise the *quest's* schedule rather than
  state their own availability honestly.
- **Counts only, roster hidden (variant C).** It is Colvillian's position and it does protect the
  seventh declarer — but it makes the standing unverifiable, and an unverifiable rank is the
  arbitrary cut ADR 0003 was trying to avoid. C is kept in the prototype anyway because `15` has
  to decide this and it deserves to be seen, not described.
- **Interest with no dates.** Every variant ended up treating "declare interest" and "name at least
  one date you can make" as the same act — the button is disabled until a date is picked, and
  unticking your last date withdraws you. An interest that names no slot cannot be ranked and
  cannot be seated, so it is not an interest.

### Smaller things the prototype settled or exposed

- **A slot cannot be labelled without an end time.** Every variant wanted to write
  "17:00–21:00 · about 4 hours", and the version with only a start reads as an ambush. `06` asks
  this question; the prototype's answer is that a slot needs a start *and* an end.
- **Under-subscribed slots are the common case, not an edge.** Two of the four candidate slots in a
  realistic 8-interested quest produce a party of four against five seats. The page has to say
  "1 seat unfilled" as a neutral fact, not as a failure, and `06` has to say whether a Game Master
  may confirm into one.
- **The page cannot be written without a deadline.** All three variants needed an "interest closes
  …" line, because that is what makes the standing provisional rather than a verdict, and it is
  what tells the seventh declarer that declaring still matters. An `interestCloses` field was
  invented to write the prototype. It belongs to `15` and is currently unowned.
- **The proposer is in the running like everyone else.** The fake data has Marta Vance's character
  interested in Marta Vance's own quest, seated by the rule rather than by right. It is worth
  looking at that row on screen while deciding `15`'s contradiction 2.
- **Naming the mismatch in public is a decision nobody made.** Q16 says the level warning is
  advisory. The prototype shows it twice: as an inline advisory when *you* pick the character, and
  as a small pill on that character's row where *everyone* can see it. The second one is a social
  pressure the ADR never asked for, and it may be the thing that stops an out-of-range player
  declaring at all — which is a refusal by other means.
- **Times worked exactly as `04` said.** Bare UTC instants, formatted in the browser, no zone
  label; nothing in the layout wanted one. One wrinkle: at phone width the matrix header can only
  carry a start time, so the duration disappears in variant A and survives in B. One more point for
  the cards.
- **The room link.** Tried in three positions. It reads best where B has it — directly under the
  quest's description and *above* the declaration — because arguing for a date is what you do
  before you tick one, not after.
- **Chrome check.** The page is drawn inside a fake Campfire sidebar carrying `08`'s rules (The
  Tavern, then live quest rooms only, soonest date first) so the density is honest. Nothing about
  the quest page fights it.

### Handed on

- **`15` — three questions this screen cannot answer for itself.** When does interest close;
  whether waitlist position carries over to the next quest (the prototype leans on it for the copy
  that makes a cut bearable); and how much of the interest list is visible. On the last one, note
  that variant B discloses something *stronger* than a roster — it names the people who would be
  cut, per date.
- **`15` — waitlist order.** B numbers the waitlist continuing from the party (6, 7, 8), which
  asserts recency ordering rather than FIFO. Contradiction 7 says everyone else is FIFO. Whichever
  wins, the numbers on this card have to mean it.
- **`11` — start from variant A.** The matrix with rows in cut order is the Game Master's tool, and
  the column footer the prototype grew ("4 seated · 1 seat short", "5 seated · 3 waiting") is the
  actual decision aid: it prices each slot in parties rather than in headcount.
- **`11` — the matrix needs a fourth cell state.** Ticket `05` (resolved while this was being
  built) rules that a character is never a candidate for a slot offered by its own player, derived
  rather than stored. The prototype has three states — can, can't, and the seated/waitlisted split
  within "can". A Game Master who declares interest in a quest they also offered a slot for needs a
  cell that reads *"can make it, but not on this one"*, on both this page and the confirmation
  screen. It is the only cell whose meaning is not about availability.
- **`09` — standing moves after you have seen it.** Someone else declaring can push a player below
  the line with no event they would notice. Whether losing a provisional seat is a notification or
  a deliberate silence is a real question, and it only exists because this page shows standing at
  all.
- **`06`** — slot end times, and confirming an under-subscribed quest, per above.

### Vocabulary

Added **Standing** to `CONTEXT.md`. The page needed a word for "where a player sits in the cut
order for a given slot" and used it a dozen times; there was none. This deliberately does *not*
name the rule itself — `03` established that naming the least-recently-played cut is still open,
and that is `15`'s to do.
