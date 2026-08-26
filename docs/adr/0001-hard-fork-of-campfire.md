# Hard fork of Campfire, kept single-tenant

This platform is a hard fork of Basecamp's Campfire rather than a fresh Rails app,
because the genuinely expensive parts of the product — real-time rooms, mentions,
attachments, search, Web Push, and a working Docker deployment — already exist there
and are exactly what a campaign needs for the arguing that surrounds an expedition.

We will not track upstream. Quests, characters and scheduling need to reach into
`Room`, `User` and the navigation chrome, and contorting the design to stay mergeable
costs more than upstream chat fixes are worth.

We also keep Campfire's single-tenant shape: the `singleton_guard` on `Account` means
one deployment is one campaign. Multi-tenancy would mean rebuilding the account, auth
and room-scoping layers before a single TTRPG feature existed.

## Consequences

- Campfire's vocabulary leaks in and has to be actively redefined. See `CONTEXT.md`:
  its `Account` is our Campaign, its `Room` is a quest's room, and its `Session` is an
  auth session and nothing to do with playing.
- A second campaign means a second deployment. Accepted.
