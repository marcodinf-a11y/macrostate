# ADR-0014: Inter-Sector Labor Mobility with On-the-Job Search

**Status:** Accepted

## Context

FR-LBR-002 specified that "workers must be able to move between sectors over time" but defined no mechanism, friction, or transition cost. The initial proposal was that only unemployed workers could move — employed workers would have to be fired before switching sectors.

This was challenged by a policy scenario: if the government boosts spending in a specific sector (e.g., renewable energy), in reality:

1. The booming sector offers higher wages to attract workers
2. Employed workers in other sectors quit and switch — they don't become unemployed first
3. Sectors losing workers must raise wages to retain them, creating economy-wide wage pressure
4. This cross-sector wage competition is a key inflation transmission channel

An "unemployed only" model misses all of this. It artificially constrains sectoral expansion, hides the wage-competition dynamic, and understates inflationary pressure from fiscal policy — undermining the MMT insight that the real constraint on government spending is real resource competition, not money.

## Decision

**Combined approach** drawing from four sources:

**1. On-the-job search (Caiani et al. 2016):**
Employed workers search for better-paying jobs with some probability each tick. If they find a higher-wage offer in another sector, they can switch. This creates the wage competition dynamic that drives cross-sector inflationary pressure.

- `jobSearchProbability`: per-household-class parameter (higher for low-income workers who have more to gain from switching)

**2. Sector-pair mobility matrix (Artuc, Chaudhuri & McLaren 2010):**
Not all sector transitions are equally easy. A sector-pair mobility matrix captures skills transferability:

- Manufacturing -> Construction: high mobility (overlapping skills)
- Agriculture -> Services: low mobility (different skill sets)
- Mobility parameters loaded from data files, moddable

The mobility value scales the probability of a successful sector switch. A worker searching in a low-mobility sector pair is less likely to receive/accept an offer.

**3. Sector-specific human capital (Dix-Carneiro 2014):**
Workers accumulate sector experience. Switching sectors means losing some sector-specific productivity. This creates a natural cost of switching that workers weigh against the wage differential — even without an explicit "moving cost" parameter.

**4. Transition productivity penalty (Dawid et al. 2019, Eurace@Unibi):**
Workers who switch sectors experience a productivity penalty during a retraining period of 1-2 ticks. During this period they work at reduced effectiveness. The penalty magnitude depends on the sector-pair mobility value (low mobility = larger penalty).

**Unemployed workers** remain available to any sector immediately with no transition delay, but still subject to the sector-pair mobility matrix for matching probability.

**Rationale — why a combined approach:**

No single paper provides a complete inter-sector mobility model suitable for a multi-sector SFC simulation game. The post-Keynesian/SFC tradition (Godley & Lavoie 2012) does not model inter-sector mobility at all. The four sources are complementary:

- Caiani et al. provides the search mechanism (how workers find opportunities)
- Artuc et al. provides the friction structure (why some transitions are harder)
- Dix-Carneiro provides the theoretical justification (sector-specific human capital)
- Dawid et al. provides the implementation pattern (productivity penalty during transition)

**Alternatives rejected:**

- **Unemployed-only mobility:** Too restrictive. Misses job-to-job transitions, which account for roughly half of all labor reallocation (Fallick & Fleischman 2004). Artificially constrains sectoral expansion and hides wage competition dynamics.
- **Frictionless mobility (any worker, any sector, instantly):** Too permissive. Would equalize wages across sectors immediately, eliminating sector wage differentials that are empirically persistent and important for policy analysis.
- **Dosi et al. (2015) approach:** Workers enter general unemployment pool, can be hired by any sector. No on-the-job search, no sector-pair friction. Simpler but misses the employed-to-employed transition channel.

## Consequences

- Data files gain: `jobSearchProbability` per household class, a sector-pair mobility matrix, and a `transitionProductivityPenalty` base value.
- The labor market matching algorithm becomes sector-aware: offers are weighted by mobility parameters, not just wage levels.
- Fiscal policy that creates demand in a specific sector will draw workers from other sectors, creating cross-sector wage pressure — a realistic and visible consequence that demonstrates the MMT "real resource constraint" principle.
- Government wage anchoring (ADR-0013) interacts with mobility: if public sector wages are attractive, workers flow toward government employment, creating private sector labor scarcity. This makes the fiscal policy → labor market → inflation transmission chain visible to the player.

**Affected documents:** PRD (FR-LBR-002), ECONOMIC-MODEL.md (Labor Market), ARCHITECTURE.md (Labor Market matching).
