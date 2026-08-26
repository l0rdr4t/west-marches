# Write the spec

Type: task
Status: open
Blocked by: 03, 04, 05, 06, 07, 08, 09, 10, 11, 14, 15

## Question

The destination. With every decision above resolved, write
`.scratch/west-marches-platform/spec.md`: complete enough that implementation sessions can
build from it without making further design decisions.

It needs to carry:

- The domain model and its lifecycle, in the vocabulary of `CONTEXT.md`
- The data model: entities, their fields, and the states they move through
- Screen by screen, what each one shows and who can reach it
- The notification matrix from ticket `09`
- What the fork changes in Campfire, and what it leaves alone
- Slices, ordered, so implementation has a first thing to build and a defensible stopping
  point short of the whole
- The non-goals from the map's Out of scope section, so nobody relitigates them

Fold the prototypes' outcomes in and then delete them. Update `CONTEXT.md` with any term
the spec needed and the glossary lacked.
