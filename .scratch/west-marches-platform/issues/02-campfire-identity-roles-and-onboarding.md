# Campfire's identity, roles and onboarding

Type: research
Status: resolved
Blocked by: -

## Question

We need a Game Master role and a player roster, and Campfire already has opinions about
users and access. Answer, from this codebase:

- What roles exist today? (`User` roles, admin, bot, the `Ban` model.) What can an admin do
  that a member cannot?
- How does someone join? Trace the join code on `Account`, the first-run wizard
  (`FirstRun`, `FirstRunsController`), and the invitation or QR flows
  (`QrCodeController`, `Users::` controllers).
- How is authentication handled (`Session`, `SessionsController`), and is there anything
  that assumes one identity per person?
- What happens when a user is deactivated or banned — what is preserved, what is severed?
- Where would a third role slot in with the least disruption: a new column on `User`, a
  role model, or a per-campaign membership?

Deliverable: findings written to `.scratch/west-marches-platform/research/02-identity.md`,
with file and line references.

## Answer

Findings: [`research/02-identity.md`](../research/02-identity.md), with a full inventory of
every authorization site.

**The authorization model is two methods.** `enum :role, %i[ member administrator bot ]`
(`app/models/user/role.rb:5`) and `can_administer?` (`app/models/user/role.rb:8-10`), enforced
by one `before_action` (`app/controllers/concerns/authorization.rb:3-5`). No policies, no
capabilities. Roughly 30 call sites, inventoried in the findings.

**Game Master must not be a fourth `role` value.** `role` is single-valued, so `game_master`
would exclude being a `member` — contradicting the glossary's "Game Masters are also players
when they aren't running." The recommended shape is a separate boolean or orthogonal enum on
`users`, following the precedent Campfire already set with `status`. That leaves every existing
authorization site untouched. Campfire has itself *removed* a third human role before
(`db/migrate/20240115124901_remove_owner_role.rb` collapsed `owner` into `administrator`), and
its actual third role — bot — is routed *around* the two-role UI rather than accommodated by it.

**Per-campaign membership is ruled out.** `Account` has no `users` association; "in the
campaign" just means a `User` row exists, and `Membership` is room-scoped. Building it means
inventing the layer ADR 0001 says we aren't building.

**There are no invitations.** One shared account-wide join code (`app/models/account/joinable.rb:5`),
visible and shareable by every signed-in user; only regeneration is admin-gated. No per-person
tokens, no email, no approval. Joining always yields `member`.

**Deactivate and ban differ sharply.** Deactivate keeps every message; ban destroys every
message the user ever wrote (`app/models/user/bannable.rb:23-28`). Neither is reversible in the UI.

Three traps carried forward to `05`:
- A deactivated administrator keeps `role: administrator`, and `User.administrator.first` does not
  filter by status — "the Game Masters" must scope `active` explicitly.
- A returning player comes back as a **new** `User` and would lose their characters.
- Ban must never be the mechanism for "this player left", if reports are message- or user-backed.
- `app/controllers/accounts/users_controller.rb:24` silently downgrades an unrecognised role to
  `member` rather than rejecting it.
