# Management Context

Conway's Law is a tool you use to pick the right architecture, not a topic to lecture on. The shape of the team is the shape of the system. When you propose modules, services, or boundaries, reason about org structure first — because that's what the architecture will look like in production whether anyone planned it or not.

This rule is about **separation of concerns at the team scale**: the team boundary *is* the concern boundary, and the architecture that survives is the one that matches the org chart.

## Conway's Law

> *Any organization that designs a system will inevitably produce a design whose structure is a copy of the organization's communication structure.* — Melvin Conway, 1967

If two teams have to coordinate to ship a feature, the feature ships as two coupled modules. If one team owns it end-to-end, the feature ships as a single coherent module. The org chart leaks into the architecture diagram every time.

## Apply this to every architecture proposal

Before proposing a new service, a module split, an API boundary, or a refactor:

- **Who owns this code in six months?** One person, one team, multiple teams?
- **Where do team boundaries fall?** That's where the seams will form whether you wanted them or not.
- **Does the proposed shape match the team shape?** Microservices for a one-person project = abandonware. A shared monolith owned by three independent teams = constant merge conflict.
- **Where would a handoff happen?** That handoff *is* the architecture. Make it explicit and stable rather than implicit and fragile.

## Heuristics

- **Solo / small team:** default to a monolith with clean internal modules. Don't propose service splits unless there's a real driver — independent scaling, a hard latency boundary, regulatory isolation, a security domain.
- **Multiple independent teams:** propose shapes that let each team ship without waiting on others — services, hard interfaces, owned-end-to-end modules.
- **Single owner doing everything:** that's an architecture risk, not just a personnel risk. Flag it. Propose pairing, shared ownership, or breaking the silo before the architecture breaks.
- **Cross-team friction in a feature:** look for the team boundary inside the feature. If you find one, the API belongs there.

## Separation of concerns, at the team scale

The most useful question for *"where should this seam go?"* is *"which team will own each side?"*

- If two teams own opposite sides, the seam belongs there. Full stop.
- If one team owns both sides, the seam is internal — make it a module boundary, not a service boundary, until org reality changes.
- If "we should split this" comes up, ask: is that a real domain boundary, or is it a team boundary in disguise?
- If "this is so hard to change" comes up, count how many teams have to agree before it ships. That number is the architecture.

## Inverse Conway Maneuver

If the user wants a certain architecture, the team has to be structured to produce it. If you see a mismatch — *"we want microservices"* from a three-person team, or *"we want a unified domain"* from five teams that already touch the code — surface it. The org shape will reassert itself; better to name it now.

The right move is often **org change before code change**.

## Ask when you don't know

If you're proposing structure and don't know the team, ask one sentence:

> Before I propose a structure: who owns this code now, and who will own it in six months — one person, one team, multiple teams?

The answer reshapes the proposal. Don't skip the question to look decisive.

## Real-world example: Databricks connectors

A connector that took Databricks engineering three quarters got built in a couple of days as an experiment. The first round of "how do we go faster" got the answer down to nine weeks — still too slow. The fundamental question was: *why does it take this long at all?*

**Before:** 1 month on-site discovery. 1 person, working alone, in a silo.
**After (required a re-org):** 1 week discovery, 7 people on 7 connectors as a team — sharing work, solving the bus problem. One quarter per connector.

The re-org wasn't optional. The architecture they wanted (parallel, fast, resilient) required the team structure that produces it. Code change followed org change, not the other way around.

## What not to do

- Don't propose microservices because they sound modern.
- Don't propose a monolith because it sounds simple.
- Don't propose a refactor without asking who's going to own the result.
- Don't treat *"what should the architecture be?"* as purely a code question. It's an org question with a code answer.
