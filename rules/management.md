# Management

Engineering decisions don't live in a vacuum — they live in an org chart. The shape of the team is the shape of the system.

## Conway's Law

> *Any organization that designs a system will inevitably produce a design whose structure is a copy of the organization's communication structure.* — Melvin Conway, 1967

If two teams have to coordinate to ship a feature, the feature ships as two coupled modules. If one team owns it end-to-end, the feature ships as a single coherent module. The org chart leaks into the architecture diagram every time, whether you noticed or not.

- **The architecture you ship is the team structure you have.** If you don't like the architecture, the org chart is upstream — that's where the change lives.
- **Cross-team handoffs become API boundaries.** Whether you wanted them or not.
- **Silos become modules.** One person owning one thing in isolation produces code only they can touch.
- **Communication overhead is architectural overhead.** Two teams with weekly syncs integrate weekly. Two teams in adjacent rooms integrate constantly.

## Inverse Conway Maneuver

If you want a certain architecture, structure the team to produce it.

- Want microservices? Independent teams owning independent deliverables.
- Want a unified domain? One team, end-to-end ownership.
- Want a shared platform? A platform team that *serves* product teams, not gates them.

You can't fight Conway. You can use it.

## Bus factor — solve the silo

One person on one thing is an architecture risk, not just a personnel risk.

- Single owners produce single-owner code: assumptions in their head, abstractions only they understand, decisions only they remember.
- Pair, mob, rotate, share — anything that puts a second pair of eyes on the work.
- "What happens if X gets hit by a bus?" is the wrong question. The right one is: "Why is X the only person who can do this?"

## Real-world example: Databricks connectors

Built a connector in a couple of days as an experiment. The engineering team's normal time for the same connector: three quarters. They asked why.

The first answer: *"we can get it down to nine weeks."* Not good enough.

The fundamental question: *why does it take this long at all?*

**Before:**
- 1 month on-site discovery.
- 1 person, working alone, in a silo.

**After (required a re-org):**
- 1 week discovery, iterating fast.
- 7 people on 7 connectors as a team — sharing work, solving the bus problem.
- One quarter per connector.

The re-org wasn't optional. The architecture they wanted (parallel, fast, resilient) required the team structure that produces it. That's Conway in action.

## How to spot Conway's Law shaping your decisions

- *"We need a new API between X and Y."* — Is that an actual technical boundary, or a team boundary in disguise?
- *"This service should be owned by team Z."* — Is that a real domain, or a coordination cost you're paying for?
- *"Why is this so hard to change?"* — Look at how many teams have to agree before it ships.
- *"Why is this so easy to change?"* — Usually one team owns it end-to-end.

When in doubt, draw the team org chart and the system architecture diagram side by side. They will look the same. The question is whether you designed it that way on purpose.
