# How West Marches groups coordinate today, and what breaks

Research for issue `03-how-west-marches-groups-coordinate-today`. Every claim carries a
URL. This is web research into real practice, not codebase research.

**Vocabulary note.** Real groups almost never use our words. The mapping I found:

| `CONTEXT.md` | What practice calls it |
| --- | --- |
| Campaign | "the server", "the community", "the table", "the game" |
| Quest | adventure, mission, expedition, sortie, job, job notice, delve, "session" |
| Slot | "available date", "play date", "job notice date", part of an "adventuring block" |
| Interest | sign-up, RSVP, reaction, "dibs", "priority", "wait-list" |
| Party | party (this word survives intact) |
| Waitlist | wait-list, "alternate", "standby" |
| Confirmation | "confirmed", "claimed", GM "crossing it off the list" |
| Report | game summary, session summary, session log, recap, quest summary, **the Chronicle** |
| Room | a per-adventure Discord channel |
| Level range | "recommended level", "level range" |
| Least recently played | "priority", "recency", **"regency"** (sic), "rotation", "the Waiting List" |

The single most useful vocabulary finding: **there is no established term for
"cut the party by who played least recently."** The three real implementations I found
call it, respectively, "prioritize players that have played less frequently"
(Colvillian), "the Waiting List" (Alexandrian), and a literal `PriorityQueue`
(SeedRoll bot). If we ship this we are naming it.

Confidence labels used throughout:
- **[FOUNDING]** — Ben Robbins' own posts, the format's source text.
- **[CONVERGENT]** — several independent Game Masters report the same thing.
- **[SINGLE]** — one group or one person; treat as an existence proof, not a norm.

---

## 1. The founding text says something different from what we assumed

Ben Robbins ran West Marches ~2001-2003 and wrote it up in 2007. The primary posts
are on ars ludi (the live site is behind a Cloudflare bot challenge; I read them
through the Wayback Machine, canonical URLs below).

**[FOUNDING]** The three defining rules
([Grand Experiments: West Marches](https://arsludi.lamemage.com/index.php/78/grand-experiments-west-marches/)):

> - There was no regular time: every session was scheduled by the players on the fly.
> - There was no regular party: each game had different players drawn from a pool of around 10-14 people.
> - There was no regular plot.

**[FOUNDING]** Scheduling is a chat negotiation, not a poll. Same post, section
"Scheduling: Players Are In Control":

> Players send emails to the list saying when they want to play and what they want to
> do. A normal scheduling email would be something like "I'd like to play Tuesday. I
> want to go back and look for that ruined monastery we heard out about past the Golden
> Hills. I know Mike wants to play, but we could use one or two more. Who's interested?"
> Interested players chime in and negotiation ensues. Players may suggest alternate
> dates, different places to explore [...] whatever — **it's a chaotic process, and the
> details sort themselves out accordingly.**

Note the shape: **one message carries the destination AND the date AND the recruitment
call, all at once.** There is no separate "propose a quest" step, no separate "offer a
slot" step, and no separate "declare interest" step. Robbins states only two hard rules:
the GM must be free that day, and the players must tell the GM the destination far
enough in advance to prep. "All other decisions are up to the players — they fight it
out among themselves, sometimes literally."

**[FOUNDING]** There is no party-cutting rule in the founding text at all. Who goes is
settled socially, by who answered the email. Robbins' only fairness levers appear later,
in [West Marches: Running Your Own](https://arsludi.lamemage.com/index.php/94/west-marches-running-your-own/):

> require scheduling on the mailing list — It doesn't matter whether a bunch of players
> agreed to go on an adventure when they were out bowling, they have to announce it on
> the mailing list or web forum [...] This prevents the game from splintering into
> multiple separate games. **If you notice cliques forming you can make a rule requiring
> parties to mix after two adventures. Conversely if you notice players being dropped
> from follow-up sorties too often just because some people can't wait to play, you can
> require parties to stay together for two adventures.** That forces a little more long
> time strategy in party selection, less greedy opportunism. Season to taste.

Both levers are **party-composition rules, not seat-allocation rules**, and they point
in opposite directions depending on which failure you have.

**[FOUNDING]** And the warning that everything in this ticket orbits, same post:

> **fear the social monster** — [...] West Marches is a swirling vortex of ambition and
> insecurity. How come no one replied when I tried to get a group together last week?
> Why didn't anybody invite me to raid the ogre cave? [...] The thrilling success or
> catastrophic failure of your West Marches game will largely hinge on the confidence or
> insecurity of your player pool. Buckle up.

**[FOUNDING]** On reports, Robbins is unambiguous that the Game Master must not write
them (same post):

> **let the players take over** — Don't write game summaries, don't clean up the shared
> map. You want the players to do all those things. If you do it, you'll just train them
> not to.

And on what reports became
([part 2, Sharing Info](https://arsludi.lamemage.com/index.php/79/grand-experiments-west-marches-part-2-sharing-info/)):

> Because there was a large pool of players, the average person was in about a third of
> the games — or to look it the other way, **each player missed two-thirds of the games.**
> [...] What started off as humble anecdotes evolved into elaborate game summaries [...]
> and eventually became a creative outlet in their own right.

**[FOUNDING]** On multi-session expeditions, Robbins in his own comments
(same URL as "Running Your Own", comment thread):

> Sometimes parties stayed in the wilds for more than one session, but they were of
> course required to get another game scheduled with the same people and couldn't join
> other expeditions (or report their adventures to the group) until they got back.

and

> the scheduling lock-in alone was a pretty strong incentive to get back.

This validates our "a quest is played once; unfinished business comes back as a new
quest" rule. See §6 for the Alexandrian's harder-won version of the same lesson.

---

## 2. THE FAIRNESS PROBLEM

### 2a. The traditional answer really is first-come — with a *carry-over* twist

The most complete published treatment of open-table seat allocation is Justin
Alexander's *Open Table Manifesto*, [Part 3: Organizing Your Open
Table](https://thealexandrian.net/wordpress/38669/roleplaying-games/open-table-manifesto-part-3-organizing-your-open-table).
**[SINGLE, but the most-cited how-to in the hobby]**:

> My first forays with an open table featured a simple, "We'll play with whoever shows
> up!" ethos. As the popularity of the open table grew, however, this quickly became
> non-viable: I had one session in which I GMed for ten players running something like
> twenty-two total PCs and hirelings.
>
> Then I imposed table caps.
>
> The system is pretty simple: I cap the number of players at a certain number. These
> slots are filled on a **first-come, first-serve basis.** Once the cap is hit, that's it
> for that session.
>
> **To prevent highly active players from monopolizing the slots at every session, I also
> instituted the concept of the Waiting List: If you signed up for a session, but
> couldn't play because of the session cap, then you were given preferential placement
> for the next session.**

This is the important finding for us. The real-world fairness mechanism is
**first-come plus waitlist carry-over**, not recency. It has three properties our
"played least recently" rule does not:

1. It is *earned by a specific disappointment*, not by absence. You get priority because
   you tried and were bumped — not because you were quiet for three weeks.
2. It is legible: "I was on the waitlist last time, so I'm in this time" needs no
   database and no explanation.
3. It doesn't reward lurking. Under a pure recency rule, the player who has never
   declared interest sorts to the top of every list.

The same post also gives table-cap sizing: "I know that I can run 5 players comfortably,
6 players are usually manageable, and with 7+ players the quality of the session will
usually begin to decline. So I set my table cap at 5."

### 2b. Recency priority does exist in the wild, and here is exactly how it's written

**[SINGLE, well documented]** The Colvillian West Marches (a large Discord game out of
the Matt Colville community, player's guide dated 2017) is the clearest published
recency rule I found. From the
[player's guide README](https://github.com/NinjaBob117/Colvillian-West-Marches/blob/master/README.md)
(also published as a
[GM Binder document](https://www.gmbinder.com/share/-LAmPCMCp-4L2HdO0mUT)):

Quest post format — note it carries **one** date:

```
Name: <Quest Name>
Time and Date: <Time Zone / Time>
Run time estimation: ...   Type: <Exploration/RP/Town/Raid>
Recommended Level: <Level Range: i.e: Level 13-24>
Slot: <Insert Number>
Signup Deadline: <date>
Signup: Open/Closed (DMs List the players once the application is closed)
```

The rule:

> **DMs are to prioritize players that have played less frequently. They should also not
> list players that have signed up until the close date for signups, so that players are
> not discouraged from signing up. When you message a DM to request joining their Quest,
> include the date of the last game you played.**

And restated in their DM Guidelines:

> 6. Accept players based on regency, the more recent they've played, the lower their
>    priority to join the quest.

("regency" is their spelling of recency.) Four design details worth stealing:

- **A signup deadline exists.** Seats are not allocated until signups close. This is what
  makes recency priority possible at all — you cannot rank a set you are filling
  first-come.
- **The interest list is hidden until the deadline**, explicitly "so that players are not
  discouraged from signing up." A visible "4/4 seats taken" kills declarations.
- **Recency is self-reported**: "include the date of the last game you played." Nobody had
  the data, so they asked the player for it. A platform that already knows this is a real
  advantage.
- **Player-proposed quests are exempt.** Same guide:

  > Post your request in the #player-quest-posting channel. [...] For Player Quests, you
  > may collect the other players you would like to join in your quest or take people who
  > request to join. **Player Quests do not require prioritizing people who have played
  > less frequently** [the GM Binder version adds: "though it's kind to do so"].

  That is a real, load-bearing carve-out: when a *player* proposes the expedition, that
  player picks their own party. Our design applies the same cut to every quest regardless
  of who proposed it.

Colvillian also shows the reward for reports: "Every player that posts a quest summary
receives [DM] Inspiration that can be used in their next quest to reroll any Saving
throw, Ability Check, Attack roll or Skill check. Session summaries are posted in the
#session-summaries channel."

### 2c. Other named fairness devices in use

**[SINGLE]** *"Dibs."* Michael Pfaff, running a ~40-player ACKS open table in Louisville
KY, in the comments of Chris Kutalik's
[Whither the West Marches?](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html):

> All of our scheduling is done via forum as well and we've implemented a **"dibs" system
> to keep the social monster at bay** (although we've had a couple situations we've had to
> deal with).

The forum where the dibs process was documented (louisvillednd.com/forum) is dead and not
archived, so I cannot report what dibs actually did — only that a large campaign named
its seat-allocation rule and that it did not fully prevent incidents.

**[SINGLE]** *No back-to-back sign-ups.* Shiroiken, on EN World's
[Setting up and running open-table sandboxes and West Marches campaigns](https://www.enworld.org/threads/setting-up-and-running-open-table-sandboxes-and-west-marches-campaigns.681883/)
(Aug 2021):

> If you have more players than you want to run in a given session, set up a web signup
> for the game. Set the number of player slots, and let people sign up. **You can't sign
> up for a session beyond the next one you're in, forcing some level of rotation.** You
> might want to set up an alternate spot, in case of last minute cancellations. **Assuming
> the alternate doesn't get the play, they automatically take the #1 slot of the next game
> they want to sign up for.** If someone cancels too often, you might want to remove them
> from the game.

Two mechanisms in one paragraph: a *lookahead cap* (you may hold at most one future seat)
and the Alexandrian's waitlist carry-over, arrived at independently.

**[SINGLE]** *Skip a turn.* Jason Lutes (designer of *The Perilous Wilds*) ran a
15-player West Marches over Zoom and wrote it up as
[Western Wilds: an Expedition into the West Marches](https://indiegamereadingclub.com/indie-game-reading-club/western-wilds-an-expedition-into-the-west-marches/).
His fix, stated as a plan for next time rather than a tried result:

> I'm going to implement a top-of-session "situation report" as described above, and
> **ask players to avoid signing up for two consecutive missions in hopes of rotating more
> people through.**

Note "ask", not "enforce". Every recency-ish rule I found in the wild is either a
*social norm the GM applies by hand* or a *constraint on signing up*, never an automated
cut.

**[SINGLE]** *An actual least-recently-played priority queue exists as code.* The Discord
bot [zacherbt/SeedRoll](https://github.com/zacherbt/SeedRoll) is described as "A simple
Discord bot for managing a priority queue for a West Marches TTRPG Campaign". Its
`Database.java` implements exactly our rule:

```java
public void progressTime() {
    for (Rollplayer nonPlayer : database) {
        if (!nonPlayer.playedLastWeek()) { nonPlayer.decreasePriority(); }
        else                              { nonPlayer.increasePriority(); }
        nonPlayer.setParticipation(false);
    }
    playingThisWeek.clear();
}
```

Two stars, no docs, no release. Read it as evidence that the need is felt and that people
hand-roll it because nothing off-the-shelf does it — not as evidence that it worked.

### 2d. The distribution the fairness rule is fighting is steep, and it is documented

**[CONVERGENT]** Four independent campaigns report the same power-law shape:

- Robbins: pool of 10-14; "the average person was in about a third of the games."
  ([part 2](https://arsludi.lamemage.com/index.php/79/grand-experiments-west-marches-part-2-sharing-info/))
- Jason Lutes: Discord had ~35 members; **15 people ever played**, splitting into
  "casual (7 people, each playing 2-3 sessions), engaged (5 people, each playing 9-16
  sessions), and dedicated (3 people, each playing 27-34 sessions)."
  ([Western Wilds](https://indiegamereadingclub.com/indie-game-reading-club/western-wilds-an-expedition-into-the-west-marches/))
- Mors Immortalis: "We had a total of 19 players participate, with a core group of about
  9 people who played regularly" over 61 sessions.
  ([West Marches Debrief](https://mors-immortalis.ca/game%20summaries/2019/07/21/west-marches-debrief/))
- Louisville D&D: "nearly 40 people play at least one session in the campaign and have
  around 15 or so 'regulars' who play weekly or bi-weekly."
  ([Hill Cantons comments](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html))

So roughly: **half the roster never really plays, and a third of the roster takes most of
the seats.** That is the problem our recency cut is aimed at, and it is real.

### 2e. Did recency/rotation rules cause new problems? What I could and could not find

I looked hard for first-hand accounts of a rotation rule backfiring and **found none**.
That absence is itself a finding — nobody publishes "our priority system caused a fight" —
so treat this section as the weakest in the document. What I did find:

- **The problem the fairness rule is actually solving may be misdiagnosed.** Lutes'
  campaign found the pain was not "I didn't get a seat" but "I've fallen behind."
  He quotes one of his own players:

  > [...] once you "fall behind" in the lore, your engagement drops even further. I mean,
  > I was the 4th most game-playing player with 16 games, and yet **I often felt like I was
  > missing a bunch of context since I did miss more than 50% of the games.**

  His remedies are a top-of-session recap and rotation — but the recap is listed first.
  Forcing an out-of-the-loop player into a seat does not fix being out of the loop.

- **Levels diverge, and modern players read that as unfair.** On EN World's
  [How do you run an open table game in D&D '24?](https://www.enworld.org/threads/how-do-you-run-an-open-table-game-in-d-d-24.715388/),
  the OP's whole blocker is that "everyone I've talked to is 100% adamant that you can
  NEVER have varying levels within the same party." Uneven attendance produces uneven
  levels, which produces a *second* fairness argument downstream of the first. Robbins
  himself punted on it: "Honestly that's up to the players to sort out. Picking a team is
  their job, not yours." ([comment #311 on Running Your
  Own](https://arsludi.lamemage.com/index.php/94/west-marches-running-your-own/))

- **The counter-lever exists and is equally real.** Robbins' "you can require parties to
  stay together for two adventures" is a rule *against* churn — for groups where players
  are being dropped from follow-ups. A platform that only knows how to rotate people out
  cannot express this.

---

## 3. SCHEDULING MECHANICS

### 3a. Who proposes times? In practice, overwhelmingly the Game Master

**[CONVERGENT — this is the strongest finding in the document]** The founding text says
players schedule. Almost every GM who has actually run one inverts it: **the GM posts
dates they are free, and players claim one.**

- Jason Lutes: "Every month or two I posted a new slate of dates when I would be
  available to play. [...] Once a party was assembled, the leader let me know which date
  they wanted, I put it on the calendar, and crossed it off the list of available dates.
  **This part was great from the GM/Judge perspective, because all I had to do was post my
  availability and then update the session calendar once a slot was claimed.**"
  ([Western Wilds](https://indiegamereadingclub.com/indie-game-reading-club/western-wilds-an-expedition-into-the-west-marches/))
- Stuart Watkinson: "A regular West Marches game works by the players deciding where they
  want to go and when they're available and the DM works around that. Me, the
  father-husband-teacher-podcaster-writer, does not have that sort of flexibility. So, my
  method is slightly different. I say, 'Hey, I'm available this Friday. Who's keen for a
  game?'"
  ([West Marches 1](https://largshirebulletin.substack.com/p/west-marches-1))
- Hipsters & Dragons: "I would post in the group some upcoming times / dates when I could
  play, and as long as four players signed up it was game on!"
  ([Running A West Marches Campaign in the City](https://www.hipstersanddragons.com/west-marches-urban-campaign/))
- Michael Ghelfi Studios: "The GM decides the time(s) they can play. Then, the players
  sign up for a session they can attend."
  ([West Marches Games: An Introduction](https://michaelghelfistudios.com/west-marches-games-intro/))
- Mors Immortalis: started with a Google Group where "I would keep a post updated with my
  availability and players would post to organize games"; and the top tip is "Be clear
  from the start that players schedule the games. **Keep your availability up-to-date**."
  ([Debrief](https://mors-immortalis.ca/game%20summaries/2019/07/21/west-marches-debrief/))
- The Alexandrian's "OPEN CALLS: For an open call, the GM simply announces the date and
  time of the game. Players then RSVP."
  ([Part 3](https://thealexandrian.net/wordpress/38669/roleplaying-games/open-table-manifesto-part-3-organizing-your-open-table))

So our **Slot** concept — a dated window a Game Master offers, spent when a quest is
confirmed into it — is the right primitive, and it matches practice better than the
founding text does. Lutes literally describes crossing the date off a list.

**But** note the one important inversion. The Alexandrian's *second* scheduling mode
runs the other way:

> SPONSORED SESSIONS: Alternatively, specific players (or groups of players) can "sponsor"
> a session by approaching the GM and requesting it. **I generally tell players that they
> should offer me a range of dates and then I can figure out which one works for me.** It's
> up to them whether they want the sponsored session to also include an open call to fill
> any empty seats or if it's an "exclusive" event just for them.

Here the *players* supply the candidate slots and the GM picks — the mirror image of our
model. A real campaign needs both directions.

### 3b. Does availability-polling across candidate slots happen? Mostly no

**[CONVERGENT]** In the accounts I read, the candidate-slot poll collapses in one of two
directions:

1. **Down to a single date.** The Colvillian quest template has one "Time and Date" field.
   Rpg-schedule, Sesh events, and westmarches.games adventures are all single-datetime
   objects with RSVP. The GM picks the date; players say yes or no.
2. **Up to a batch of dates offered at once**, with claiming rather than voting — Lutes'
   "slate of dates", Ghelfi's "the time(s) they can play".

The one genuine multi-slot declaration model I found is EricVulgaris' **adventuring
blocks** **[SINGLE, but exactly our shape]**. Described in
[West Marches Organization - Discord](https://www.patreon.com/posts/west-marches-48889263)
and [West Marches — Adventuring Blocks](https://www.patreon.com/posts/48890666) (both
Patreon posts are Cloudflare-gated to direct fetching; I have their content only via
search-index extracts, so quote them cautiously). The scheme:

- Rather than each game being spontaneous, the GM offers **5-12 sessions inside a
  "block"**, publishing length, start date and start time and leaving the rest to players.
- Each session is posted with the **Sesh** bot with reaction-role sign-ups:
  **Confirmed** (up to max group size, commonly 5), **Wait-List**, **DM**, and
  **Priority/Only-Time** — the last being "a visible way to say 'this is the only session
  I can go to'".
- **"No one confirms any sign-up until the conclusion and retrospective of each adventuring
  block, but are free to priority/wait-list themselves."**
- Between blocks there is GM downtime and a **retrospective**, explicitly because "the
  primary cause of death for west marches campaigns is DM burnout."

That is our design, arrived at independently: declare interest across many candidate
slots, mark which ones you can actually make, and confirm only at the end. It is the best
evidence I found that our model is buildable. It is also **one person's system**, and it
required a batch-scheduling ("block") concept we do not have.

**[SINGLE]** Generic availability pollers do get used, but as a one-off setup step rather
than per-session: Lutes used a Google poll to shape the setting and "After polling the
players to figure out which weeknights were most convenient for the most people
(prioritizing my own schedule), I settled on alternating Tuesdays and Thursdays."
The poll fixed the *cadence* once; individual sessions were then claimed off a list.

### 3c. What roster size makes it break — and how it breaks

**[CONVERGENT]** Scheduling breaks *at the bottom*, not at the top. Nobody reports "too
many people to schedule." Everybody reports "nobody would start the thread."

- Norman J. Harman Jr., on
  [Whither the West Marches?](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html):

  > **Player scheduling was FAIL. Unless you have confident, organized, assertive players
  > they won't schedule. No one wanted to be the guy who said fuck it, we're playing Sat**
  > It doesn't matter "bob" can't make it that day. Nothing every happened unless I forced
  > it. Never enough players.

- "Blue" on the EN World thread, describing 3 DMs and 12+ players:

  > **It died by scheduling.** The players were completely foreign to the idea of "get a
  > group together and let a DM know" and on the flip side the DMs posted some availability
  > but asked gamers weren't coming together. Or worse one DM needed 24 hour notice
  > (self employed) and a party would come together like 45 minutes before start time and
  > try to get him. In the end I think we half less than half a dozen sessions and then it
  > just faded out with barely a whimper.
  ([EN World](https://www.enworld.org/threads/setting-up-and-running-open-table-sandboxes-and-west-marches-campaigns.681883/))

- Mors Immortalis: "**I had difficulty getting the players to take an active role in
  scheduling. This only really started to happen about 5 months in** when we had a good
  group of core players who were motivated to play again. Before that I was mostly
  scheduling the games." And: "Once we had a solid group of ~9 players, scheduling was
  easy."
  ([Debrief](https://mors-immortalis.ca/game%20summaries/2019/07/21/west-marches-debrief/))

The threshold that recurs is **around 9-10 genuinely engaged players.** Below it the
player-driven loop does not self-start and the GM ends up prodding. Robbins' own floor is
the same: "if you don't have enough players for multiple groups, a lot of the logic of
west marches simply won't apply."
([comment #311](https://arsludi.lamemage.com/index.php/94/west-marches-running-your-own/))

Party size caps, **[CONVERGENT]**: 3-5 players plus the GM (Mors Immortalis), "up to 4
(occasionally 5, but level of engagement over zoom starts to suffer at 5 players)"
(Lutes), cap of 5 (Alexandrian), max group size 5 (EricVulgaris/Sesh), minimum 4 to run
(Hipsters & Dragons).

**[CONVERGENT]** The other thing that reliably breaks is **the sequel session**. The
Alexandrian's account is the sharpest warning our design will face:

> There was one time when I set up a sequel session and several of the players had to
> cancel on it. Trying to reschedule proved challenging and we ended up bouncing it around
> for two or three months before I finally wrote it off and released the lockdowns. In the
> interim, however, all of the momentum in that section of the campaign had been lost:
> **I would have been far better off immediately writing off the sequel session and allowing
> subsequent expeditions to be scheduled to pursue the loose ends which had been left.**
> ([Part 3](https://thealexandrian.net/wordpress/38669/roleplaying-games/open-table-manifesto-part-3-organizing-your-open-table))

He also documents the bookkeeping this forces: a "campaign status sheet" listing every
active PC, which are **locked down** in an outstanding expedition, and which locations are
locked down with them. Our "one quest, played once, unfinished business returns as a new
quest" is exactly the discipline both he and Robbins converged on the hard way.

---

## 4. SESSION REPORTS

### 4a. Who writes them, and what the conventions are

**[FOUNDING]** Players write them; the GM must not
([Running Your Own](https://arsludi.lamemage.com/index.php/94/west-marches-running-your-own/)):
"Don't write game summaries, don't clean up the shared map. [...] If you do it, you'll just
train them not to."

**[CONVERGENT]** The convention that actually works is **a named per-session role,
assigned before play starts**, not "whoever feels like it afterwards":

- Mors Immortalis: "PCs choose roles: Navigator, Scout, **Scribe**, Quartermaster, and
  Cartographer. [...] **The Scribe wrote the session summary in return for an extra downtime
  action next time they played.** The Cartographer tracked any areas added to the map and
  made the edits to the player map after the game."
  ([Debrief](https://mors-immortalis.ca/game%20summaries/2019/07/21/west-marches-debrief/))
- Louisville D&D: "**at the beginning of each session we have volunteers who offer to be the
  Mapper and Notetaker** (a lot of times, these are the more dedicated and reliable people in
  the campaign). **Sometimes session notes go missing, but most of the time they get up** and
  we link it all back together."
  ([Hill Cantons comments](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html))
- An Arden Vul open table publishes reports with an explicit "notes taken by:" scribe
  attribution, a player roster line, region, narrative, a numbered rumours list, gold/XP
  totals and magic items.
  ([Arden Vul Open Table: Session 1](https://eviltables.com/2025/01/13/arden-vul-open-table-session-1/))

The recurring report *structure* across groups is: who was there → where they went →
what they found → rumours/leads → what they took. The "rumours/leads" section is the part
that feeds the next quest, and it is the part a plain chat log never produces.

### 4b. What makes people actually write them

**[CONVERGENT]** Every group that sustained reports paid for them with an in-game reward,
and the reward was small and immediate:

- Colvillian: one Inspiration (a re-roll) per quest summary posted.
  ([player's guide](https://github.com/NinjaBob117/Colvillian-West-Marches/blob/master/README.md))
- Mors Immortalis: one extra downtime action next session, for the Scribe.
- Stuart Watkinson: "**Players are rewarded for entering information into The Chronicle.**
  [...] The Chronicle is the record of the players adventures and it is then used for
  adventure preparation for the GM."
  ([West Marches 4 — Organisation](https://largshirebulletin.substack.com/p/west-marches-4-organisation))
- The generic advice, from Domain of Many Things: "players will need to be highly
  incentivised to do this, as most are used to being passive consumers of content" —
  suggesting "meta currency, or even XP as incentives."
  ([The Seven Elements of West Marches Play](https://www.domainofmanythings.com/blog/west-marches))

**[SINGLE]** The second mechanism that makes reports get written is **the GM visibly
consuming them**. Watkinson: the per-adventure channels "became the tool I used most when
preparing for adventures." Colvillian pays the Inspiration out on *your next quest*, so
the loop closes where you can feel it.

**[SINGLE]** A caution from Domain of Many Things: the GM "must never correct players'
interpretations in these reports or update the world to match them" — players should
discover their own mistakes in play. Our Report is described as "the campaign's only
durable record of the world"; practice treats it as *the players' record of what they
think they saw*, which is not the same object.

### 4c. Report rot: the risk is real, and it is specifically wiki rot

**[CONVERGENT]** The failure is not "no one writes anything." It is "the durable shared
record decays while the chat keeps going."

- Brendan (Necropraxis), on
  [Whither the West Marches?](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html):

  > **I've tried using wikis, and they tend to rot and not be read by players.** I'm
  > currently using G+ threads which are nice for low friction, but they are a nightmare to
  > search [...]

- The Alexandrian, [Part 3](https://thealexandrian.net/wordpress/38669/roleplaying-games/open-table-manifesto-part-3-organizing-your-open-table):

  > **My group experimented with a wiki for awhile, but the participation rate was low** and
  > it was ill-suited for event announcements.

- Mors Immortalis, whose Scribe role *did* keep summaries coming, still reports the deeper
  failure:

  > **Most players didn't actively share information. They would write session summaries and
  > add things to the map, but that was about it.** This led to some unneeded information
  > asymmetries and time-wasting efforts to rediscover the same piece of information. [...]
  > **A campaign wiki could help this, but again I'm not sure players would want to maintain
  > it.**

  With a worked example: a party set off for a far dungeon needing a key held by a player
  who wasn't present, and only avoided the wasted session by changing their mind.

- Lutes' player, again: context "was recorded on the Discord, but it could be hard to
  track."
  ([Western Wilds](https://indiegamereadingclub.com/indie-game-reading-club/western-wilds-an-expedition-into-the-west-marches/))

The pattern across all four: **per-session reports survive when they're paid for; the
cross-session synthesis (the wiki, the lore index, "what do we know about the Black
Door") is what dies.** Nobody in these accounts reports a group that sustained a wiki.

**[SINGLE]** One structural fix is well attested. Watkinson moved from a channel per
*location* to a channel per *adventure*:

> Logistically, we started with using one channel for each location but they quickly became
> hard to follow. The Goblin Cave, for example, had four adventures in it. Each one vastly
> different from the one before which made for an extremely convoluted and confusing to
> follow. **So, we changed to each adventure got it's own channel and this worked much
> better for the players, but also for me.**
> ([West Marches 4](https://largshirebulletin.substack.com/p/west-marches-4-organisation))

That is our Room-per-quest, validated against the obvious alternative, by someone who
tried the alternative first.

---

## 5. TOOLS: what people actually use, and where each stops

### 5a. Discord is the substrate, and its channel list is the data model

**[CONVERGENT]** Every post-2016 account uses Discord. The channel sets are strikingly
consistent and are essentially a schema:

- Mors Immortalis: `#general`, `#scheduling`, `#mission-planning`, `#achievements`.
- Colvillian: `#dm-quest-posting`, `#player-quest-request`, `#request-discussions`,
  `#session-summaries`, `#map-requests`, `#ask-dm-questions`.
- Watkinson: `#start-here`, `#the-mess-hall` (in-character), `#the-map-room`, `#the-temple`,
  `#rules-questions`, `#character-options-unlocked`, plus one channel per adventure. He
  publishes [a Discord template](https://largshirebulletin.substack.com/p/west-marches-4-organisation).
- Lutes: setting guide, char-gen, "town opportunities" (the job board), "mission planning",
  "town activity", "tavern talk".
- Louisville used a Simple Machines forum with sub-forums Session Planning / Campaign
  Discussion / Tavern Talk / The Marketplace, plus MediaWiki
  ([Hill Cantons comments](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html)).

Note that **two separate proposal channels** — one for GM-posted quests and one for
player-requested quests — recurs (Colvillian explicitly, Lutes implicitly via the "town
opportunities" board vs player-initiated missions).

**[SINGLE, but a striking number]** Lutes reports the split: "about 70% of the missions came
from posted opportunities. The rest were player-initiated."
([Western Wilds](https://indiegamereadingclub.com/indie-game-reading-club/western-wilds-an-expedition-into-the-west-marches/))

**[CONVERGENT]** And the cost: "**Managing a Discord is a lot.** One of the theoretical goals
of a West Marches campaign is to reduce the GM's workload by shifting scheduling
responsibilities onto the players; that part worked great. [...] But maintaining the Discord
took more effort than I anticipated, at times amounting to the equivalent of 3-5 simultaneous
play-by-post subgames." (Lutes)

### 5b. Generic scheduling bots

- **Sesh** ([sesh.fyi](https://sesh.fyi/), [manual](https://sesh.fyi/manual/)) — events with
  RSVP, polls, reminders, and a "time finder" availability poll. Critically, **attendee
  limits and waitlists are premium-only**: "When an option reaches its limit, additional
  responses are automatically added to a waitlist. If someone removes their RSVP, the first
  person on the waitlist receives a DM and is automatically moved up." Note that is
  strict FIFO. The time-finder and the event are separate objects; there is no "poll
  candidate slots then confirm one into a capped event" flow.
- **RPG Schedule** ([github](https://github.com/sillvva/rpg-schedule)) — "Automated or manual
  sign ups and automated waitlisting", reaction sign-ups, reminders, auto-reschedule. Again
  single-datetime, first-come, FIFO waitlist. Games are auto-pruned 48-72h after the
  scheduled time — i.e. **the tool deliberately forgets**, which is the opposite of what a
  campaign record needs.
- **Apollo** ([apollo.fyi](https://apollo.fyi/)) — same category.
- **When2meet / Doodle / Rallly** — availability pollers, used (if at all) to fix the
  cadence once, not per session. Their known weakness is directly ours: no follow-up, no
  confirmation notification, no chasing non-responders, and no link between the poll and
  the event that results.

**[SINGLE]** The gap is stated out loud on EN World. Yora, replying to Shiroiken's rotation
scheme: "I was thinking of that. **Does anyone know of good tools for managing this? Ideally,
it should be something where all players can see the whole schedule at any time, and not just
the GMs with admin access.**" Nobody in the thread had an answer.
([EN World](https://www.enworld.org/threads/setting-up-and-running-open-table-sandboxes-and-west-marches-campaigns.681883/))
Mors Immortalis, two years earlier: "I tried to find a scheduling/calendar bot, but none of
them seemed like they would work for us."

### 5c. Purpose-built West Marches tools, and where they stop

- **[WestMarches.games](https://www.westmarches.games/)** is the serious incumbent. Discord
  login and role sync, adventure creation with player limits and level requirements, signup
  with character selection, waitlist, XP/gold/reward distribution, character hub, built-in
  world wiki, hexcrawl maps, adventure history. Free tier plus a "Community Boost" premium.
  The site claims 590 communities, 8,047 characters, 10,028 adventures, and a 2026 ENNIE
  nomination.

  Where it stops, per [its own Adventures docs](https://www.westmarches.games/guide/adventures):
  an adventure has **one** date and time; signups are either auto-approved or manually
  reviewed by the GM; there is a waitlist but **no documented selection criterion beyond
  level requirements** — no recency, no rotation, no candidate-slot polling. Its
  [practical guide](https://www.westmarches.games/guide/running-west-marches) advises
  fairness only as social norms: "Take turns giving everyone a chance to go first" and
  "Host mixer sessions where groups are chosen at random."

- **Home-grown Discord bots** are everywhere and mostly abandoned: a search of GitHub returns
  a long tail including [steven-hines/West-Marches-Discord-Bot](https://github.com/steven-hines/West-Marches-Discord-Bot)
  (built for a 200+ player, multi-DM game; its README says "unlike in home campaigns with
  people you know and trust, **some measure of accountability and fairness would need to be
  core components**" — but the automation described is dice/marketplace, not seat
  allocation), [zacherbt/SeedRoll](https://github.com/zacherbt/SeedRoll) (the recency
  priority queue in §2c), [lore-league/daggerbot](https://github.com/lore-league/daggerbot),
  and a dozen others with 0-2 stars.

- **Foundry VTT** modules exist for multi-party campaigns —
  [Oronder](https://github.com/oronder/Oronder) ("designed to meet the needs of running a
  complex Westmarches game with multiple DMs and players", schedules sessions and awards XP
  from Discord), [Party Operations](https://foundryvtt.com/packages/party-operations),
  [Intoterica Campaign Manager](https://foundryvtt.com/packages/intoterica) — but Foundry is
  the *play* layer. Nobody in the first-hand accounts coordinates in Foundry.

- **Wikis** (MediaWiki, Fandom, LegendKeeper, Obsidian) are where the world record is meant
  to live and where it rots (§4c). LegendKeeper's own case study says the players "were
  responsible for all bookkeeping, including session summaries, treasure, items, etc."
  ([LegendKeeper in Action](https://www.legendkeeper.com/how-to-run-a-west-marches-campaign/)).

- **Spreadsheets** persist under everything. Mors Immortalis tracked "character info, quest
  board, session summaries, shop contents in town, and town upgrades" in Google Sheets, with
  players able to edit most tabs. Colvillian ran a "DM Tile Claim Spreadsheet" so two GMs
  didn't prep the same hex.

- **Published guidance** worth knowing exists: *Izirion's Enchiridion of the West Marches*
  (Dom Liotti & Sam Sorensen), a paid toolkit that Lutes read while designing his campaign
  ([DMs Guild](https://www.dmsguild.com/en/product/333956/izirion-s-enchiridion-of-the-west-marches)).
  I could not read it (paywalled), so I cannot say what it prescribes for scheduling; Lutes
  quotes it only on the "no adventure in town" dictum, which he then ignored.

### 5d. The named coordination pattern we don't have: the block

**[SINGLE]** EricVulgaris' "adventuring block" — a discrete real-world *and* in-world chunk
of time containing 5-12 sessions, followed by GM downtime and a retrospective — is
explicitly an agile-sprint analogy, and it is aimed at the failure mode everyone agrees
kills these campaigns: "the primary cause of death for west marches campaigns is DM
burnout."
([Adventuring Blocks](https://www.patreon.com/posts/48890666))
Our model has no unit above the individual Slot.

---

## 6. Other things that will bite us

**[CONVERGENT]** *Multi-GM continuity.* Louisville ran two DMs "who share the same world
(updating each other on their happenings in the games they run so we're fairly coherent when
we touch on the same parts of the world)"; Colvillian gave DMs a tile-claim spreadsheet;
Mors Immortalis added a co-DM at 12 months to avoid burnout. Our Campaign has several Game
Masters and no notion of who has claimed which part of the world.

**[SINGLE]** *Locking.* The Alexandrian locks both characters *and locations* while an
expedition is outstanding, and keeps a "campaign status sheet" of both
([Part 3](https://thealexandrian.net/wordpress/38669/roleplaying-games/open-table-manifesto-part-3-organizing-your-open-table)).
Colvillian instead explicitly permits overlap: "You may always participate in a quest. If
your character is involved in a multisession quest, they may still play in other quests
in-between sessions." Both are coherent; ours implies neither.

**[CONVERGENT]** *Character stables.* Multiple characters per player is near-universal in
large rosters (Alexandrian: "each player having multiple characters in the campaign world";
"Blue"'s dead campaign allowed "a stable of up to three characters"; Colvillian allows a
secondary that "cannot be active in quests until your primary character has died or
permanently retires"). Our Character model allows several, which is right; the constraint
rules differ wildly between campaigns.

**[SINGLE]** *Recruiting and onboarding are load-bearing.* Michael Pfaff gates forum access
until a new player has read the Pitch and Rules; Watkinson has a whole `#start-here` section;
Chgowiz-era advice is "advertise your game well and you will find those assertive players."
And a 2-year-veteran GM in Robbins' comments: "**keeping a steady influx of new players has
been key**."
([Running Your Own comments](https://arsludi.lamemage.com/index.php/94/west-marches-running-your-own/))

**[CONVERGENT]** *These campaigns die.* Chris Kutalik, surveying the 2008 cohort in 2012:
"looking around today at all those related experiments I am struck by the fact that 3-5 years
later **all of the ones I am familiar with are gone**."
([Whither the West Marches?](https://hillcantons.blogspot.com/2012/09/whither-west-marches.html))
Cause of death is GM burnout or scheduling failure, not lack of a tool. That should temper
how much we expect any feature to fix.

---

## Where this contradicts our design

Ordered by how exposed I think each assumption is.

**1. "A player proposes a quest" is the minority path.** Our model is player-proposes,
GM-offers-slots. In practice the GM posts a job board and players pick off it: Lutes measured
**70% GM-posted, 30% player-initiated**; Colvillian runs two separate channels
(`#dm-quest-posting` and `#player-quest-request`) and treats them as *different objects with
different rules*; Ghelfi and Hipsters & Dragons describe GM-posted missions only. A design
where only players can propose quests will have Game Masters working around it on day one.
Sources: [Western Wilds](https://indiegamereadingclub.com/indie-game-reading-club/western-wilds-an-expedition-into-the-west-marches/),
[Colvillian](https://github.com/NinjaBob117/Colvillian-West-Marches/blob/master/README.md).

**2. Cutting the party by recency on *every* quest contradicts the one documented recency
rule.** Colvillian — the only campaign I found that wrote a recency rule down — explicitly
exempts player-proposed quests: "**Player Quests do not require prioritizing people who have
played less frequently.**" Their reasoning is implicit but obvious: the person who organised
the expedition picks who comes. Robbins says the same thing about party selection generally:
"Picking a team is their job, not yours." Our Confirmation performs an automatic cut that
overrides the proposer, on quests the proposer created. That is the sharpest conflict in this
document.
Sources: [Colvillian](https://github.com/NinjaBob117/Colvillian-West-Marches/blob/master/README.md),
[GM Binder](https://www.gmbinder.com/share/-LAmPCMCp-4L2HdO0mUT),
[ars ludi comment #311](https://arsludi.lamemage.com/index.php/94/west-marches-running-your-own/).

**3. "Played least recently" is not what the traditional fix optimises for, and it rewards
lurking.** The attested mechanism is *first-come plus waitlist carry-over*: you get priority
because you tried and got bumped, not because you were absent. The Alexandrian: "To prevent
highly active players from monopolizing the slots at every session, I also instituted the
concept of the Waiting List." Shiroiken reinvents the same thing. Under pure recency, a
player who has never declared interest outranks a player who has been bumped twice. We
should at minimum consider ranking by *unrewarded interest* (declared and not seated) before
raw recency.
Source: [Open Table Manifesto Part 3](https://thealexandrian.net/wordpress/38669/roleplaying-games/open-table-manifesto-part-3-organizing-your-open-table),
[EN World](https://www.enworld.org/threads/setting-up-and-running-open-table-sandboxes-and-west-marches-campaigns.681883/).

**4. Our Interest list is presumably visible; the one recency implementation hides it.**
Colvillian: DMs "should **not list players that have signed up until the close date for
signups, so that players are not discouraged from signing up**." If our quest page shows
"6 characters interested, 4 seats", the seventh player stops declaring — and a recency cut
only works if people keep declaring. Visibility of Interest before Confirmation is a real
design decision we have not made.

**5. Confirmation as a single instant assumes a deadline we haven't specified.** Every
recency-or-rotation system in the wild depends on a *signup deadline* — Colvillian has an
explicit `Signup Deadline` field; EricVulgaris forbids confirming "until the conclusion and
retrospective of each adventuring block." Without a deadline, seats fill first-come and the
recency rule has nothing to rank. Our Confirmation is described as a Game Master action at
an arbitrary time.

**6. Availability-polling across candidate slots is rarer than we assume, and the direction
sometimes reverses.** Practice mostly collapses to a single-date post plus RSVP (all three
major bots; westmarches.games' Adventures are single-datetime), or to a GM-published slate
of dates that parties *claim*. The Alexandrian's "sponsored sessions" run the opposite way:
players "offer me a range of dates and then I can figure out which one works for me." Our
one-directional model (GM offers slots → players declare which they can make) covers only
one of the two real flows.

**7. The Waitlist "in priority order" is FIFO everywhere else.** Sesh, rpg-schedule and
westmarches.games all promote strict first-in-line. If our waitlist is ordered by recency
too, that is a second novel rule people will have to be taught — and one that makes
"someone dropped out, who gets the seat?" non-obvious in the moment.

**8. Scheduling breaks below ~9 engaged players, not above one table.** The ticket asks what
breaks "at roster sizes above one table." The evidence says the opposite: the failure is
under-supply of initiative. "Player scheduling was FAIL... No one wanted to be the guy who
said fuck it, we're playing Sat." "It died by scheduling." Mors Immortalis had to schedule
personally for five months. A tool that only makes coordination *tidier* does not fix this;
what fixed it in every surviving account was the GM publishing dates unprompted. Our Slot
does that — but we should treat "the GM must keep slots on the board" as the critical path,
not a nicety.

**9. "The Report is the campaign's only durable record of the world" is the riskiest single
sentence in CONTEXT.md.** Per-session reports do survive when paid for (Inspiration, a
downtime action, "the Chronicle"), and the mechanism that works is a **named Scribe chosen
before play**, not open authorship afterwards. But what dies in every account is exactly what
we are asking Reports to be: the cross-session synthesis. "Wikis... tend to rot and not be
read by players" (Necropraxis). "The participation rate was low" (Alexandrian). "I'm not sure
players would want to maintain it" (Mors Immortalis). Our Report has no reward attached, no
assigned author, and carries more weight than any wiki that has ever survived one of these
campaigns.

**10. "Written and edited by anyone who was there" is not how it works.** Louisville assigns
a Mapper and a Notetaker at the *start* of each session; Mors Immortalis assigns a Scribe as
a character role with a mechanical payoff; Arden Vul reports carry a scribe byline. Diffuse
responsibility is the documented failure mode. If everyone may write, no one does.

**11. We have no unit above the Slot, and burnout is the documented cause of death.**
Adventuring blocks with a retrospective and enforced GM downtime, two-GM rotation, "prep only
the next mission" — every long-lived campaign in this research has some pacing device for the
Game Master. Ours has none.

**12. Level range as purely advisory is right, but it inherits a fight.** Uneven attendance
produces level spread, and modern D&D players read level spread as unfairness — an entire EN
World thread stalls on it. The countermeasure real groups use is character stables (bring the
character closest in level), which makes *which* character you bring part of the seat
decision, not just flavour. Our Interest already names a character; the cut should probably
see it.
Source: [EN World, How do you run an open table game in D&D '24?](https://www.enworld.org/threads/how-do-you-run-an-open-table-game-in-d-d-24.715388/).

---

### Confidence caveats

- I could not read r/westmarches or r/DMAcademy directly: Reddit blocks both WebFetch and
  scripted access from this environment, and the search index returned almost nothing from
  them. **There is a whole tier of first-hand accounts I have not seen.** If any claim here
  needs a second opinion, that is where to look.
- EricVulgaris' two Patreon posts (adventuring blocks, Sesh reaction roles) are
  Cloudflare-gated; I have their content only through search-index extracts and could not
  verify wording against the page. Everything attributed to them is flagged **[SINGLE]** and
  should be re-checked before it drives a decision.
- ars ludi is also Cloudflare-gated; all Robbins quotes were read via the Wayback Machine
  and are cited to canonical URLs.
- I found **no** first-hand account of a recency/rotation rule causing new problems. That is
  an absence of evidence, not evidence of absence — negative experiences of this kind are
  rarely written up.
