# ADR-0017: Uniform Price Bond Auction

**Status:** Accepted

## Context

The government finances deficits by issuing bonds to banks (ADR-0003). The auction mechanism determines how bond prices (and thus yields) are set when multiple banks bid for government debt. Two standard auction types exist in sovereign debt markets, with meaningfully different implications for bank behavior, government borrowing costs, and yield dynamics.

In the MVP, there is a single aggregate bank, so the auction degenerates to a single-bid case. However, the full game envisions multiple competing banks, at which point the auction type becomes consequential.

## Decision

**Uniform price (Dutch) auction.** All winning bidders pay the same price — the highest yield (lowest price) at which the entire issue is filled. The government announces a bond quantity; banks submit bids (quantity, yield); bids are ranked from lowest yield to highest; all winners pay the marginal (highest accepted) yield.

**MVP simplification:** With a single aggregate bank, the auction reduces to a take-it-or-leave-it offer. The bank bids for the full quantity at a yield reflecting its opportunity cost (lending rate minus risk adjustment). The government accepts. No ranking or clearing logic is needed — the uniform-price mechanism is implemented but trivially exercised.

**Rationale — why uniform price over discriminatory:**

- **US Treasury precedent:** The MVP country is the USA. The US Treasury switched from discriminatory to uniform price in 1998 and has used it since. Using the same mechanism grounds the simulation in real institutional practice.
- **Simpler bidding strategy:** In a uniform auction, the dominant strategy is to bid your true valuation — there is no incentive to shade your bid downward. This simplifies AI bank bidding logic significantly. In a discriminatory auction, optimal bidding requires banks to estimate the clearing price and bid just above it, which requires a more sophisticated strategic model.
- **Winner's curse mitigation:** Uniform pricing eliminates the winner's curse (overpaying relative to other bidders). This encourages more aggressive participation and tends to produce lower, more stable yields for the government — a desirable property for a game where the player controls fiscal policy.
- **Transparency:** The single clearing yield is easy to display and reason about in the UI. Players see one bond yield, not a distribution of prices paid by different banks.
- **Empirical evidence:** Goldreich (2007) and Brenner et al. (2009) find that uniform auctions tend to produce slightly lower borrowing costs for the issuer and higher participation rates, consistent with theoretical predictions.

**Alternatives rejected:**

- **Discriminatory (pay-your-bid) auction:** Each bank pays its own bid price. Used historically by the US Treasury (pre-1998) and still used by some sovereigns. Rejected because (a) it requires banks to solve a non-trivial bidding optimization, adding AI complexity with little gameplay benefit, (b) it tends to produce higher and more variable yields, and (c) it is not the current US institutional practice.
- **Fixed-rate tender:** Government sets the yield and banks choose quantities. Simpler than both auction types but removes yield determination from the market entirely, eliminating an interesting emergent dynamic (bond yields responding to fiscal conditions).
- **No auction (direct CB financing):** The central bank directly credits the government's account. While this is how sovereign money creation actually works in MMT's description of the consolidated government sector, the game deliberately separates treasury and central bank operations for pedagogical reasons (players should see the bond issuance → bank purchase → reserve drain flow). This separation is already established in the architecture.

## Consequences

- MVP implementation is trivial: single bank, single bid, accepted at the bid yield.
- Post-MVP multi-bank implementation needs: bid collection, sorting by yield, quantity allocation at the marginal yield, and pro-rata allocation if the marginal bid is partially filled.
- Bond yields emerge from bank behavior rather than being administratively set. Banks that expect higher returns from lending will demand higher yields on bonds, creating a realistic opportunity-cost dynamic.
- The UI displays a single clearing yield per auction, which is clean and interpretable for players.
- Bank bidding AI (post-MVP) can use a simple true-valuation strategy rather than requiring game-theoretic bid shading.

**Affected documents:** PRD (bond auction mechanics), ARCHITECTURE.md (bond market), PARAMETERS-COMMENTARY.md (auction parameters).
