<!-- solve-problem:ee79797c-6de5-4546-817f-dd0132c70ab1 -->
# Coalition bounds on clock distortion

A [Tangle Solve problem](https://solve.tangle.tools/problems/ee79797c-6de5-4546-817f-dd0132c70ab1).

## What is the problem?

In PoW blockchains (Bitcoin, Ethereum pre-Merge), each block carries a miner-supplied timestamp. Consensus rules constrain these: Bitcoin requires a block's timestamp to exceed the median of the previous 11 blocks (MTP-11) and be below node-adjusted time plus a slack (future-time limit, ~2 hours); Ethereum had analogous GHOST/clique constraints. These timestamps feed difficulty adjustment (difficulty retargeting uses block-time differences), block validity, and smart-contract deadlines. The problem: **quantify how much a coalition of hash-power fraction α can sustainably distort the effective chain clock — stretching apparent time (to lower difficulty) or squeezing it (to speed retargeting or manipulate time-sensitive contracts) — when the coalition membership is time-varying (rented hash, changing alliances), and when the coalition's manipulation may trigger races (competing chains near the tip, orphan risk).**

Formally: let T_n be the honest arrival process of blocks (Poisson with rate proportional to total hash rate) and t_n the timestamps on the main chain. Define distortion D(n) = t_n − T_n (or the induced bias in difficulty d_n). The open question is a tight characterization of max sustainable |D| or cumulative bias as a function of α, coalition duration distribution, MTP window size, and future-time slack, including the cost (orphan/reorg probability) of forcing timestamps in races at the chain tip. A changing coalition means the attacker's hash share is a stochastic process α_t, not a constant — most existing analysis assumes a fixed α.

## Why does it matters?

Timestamps directly determine difficulty, hence mining revenue per unit hash, and gate time-locked contracts and protocol deadlines (e.g., Bitcoin's testnet, timelocks, fork activation). A tight bound tells protocol designers how robust MTP-style rules are, guides parameter choice (median window, future slack), and prices the attack for exchanges and contract parties relying on chain time. Without it, security arguments for time-dependent consensus features rest on unquantified folklore.

## What's already known?

- **Yaish, Stern, Zohar (2022), "Blockchain Stretching & Squeezing":** demonstrated difficulty-manipulation strategies — "stretching" (inflating apparent block times to drive difficulty down) and "squeezing" — showing concrete profitable attacks against Bitcoin-style difficulty adjustment; quantified profit for specific fixed-α strategies but did not give general sustained-distortion bounds.
- **Yaish, Stern, Zohar (2023), "Uncle Maker":** showed timestamp manipulation in Ethereum (pre-Merge) that pushes a block's timestamp into a new epoch to trigger an artificial difficulty drop ("difficulty bomb" timing), exploiting MTP at epoch boundaries; includes race conditions at the tip where competitors can snipe the cheap epoch, i.e., the "cost of inducing races."
- Classical results: MTP median windows are robust to <50% Byzantine timestamps in any single evaluation, but this pointwise robustness does **not** translate to bounds on sustained drift or difficulty bias, because consecutive manipulated timestamps compound.
- Byzantine clock synchronization literature (Dolev–Strong–... style, Sundaram–Wattenhofer) bounds clock agreement under f < n/3 faults in message-passing; it does not model hash-power-weighted block production, difficulty feedback, or rational coalition churn — hence not directly transferable.

Gap: no closed-form bound on sustained distortion as a function of (α_t process, window size, future limit), and no analysis incorporating coalition turnover or race-induced orphan costs.

## What would count as solving it?

1. **Upper bound:** a theorem giving, for any coalition strategy with hash fraction process bounded by α_t, the maximum sustainable clock distortion / difficulty bias as a function of median window w and future slack s (e.g., "sustained |D| ≤ f(α, w, s), with mixing time of difficulty back to honest level ≤ g(α)").
2. **Lower bound / matching construction:** a coalition strategy (with changing membership, i.e., α_t adapted to observed chain state) achieving distortion matching the upper bound up to constants, including the equilibrium cost of races (probability of losing the epoch-boundary race times reward).
3. **Game-theoretic variant:** characterize when manipulating is a (Nash or strong Nash) equilibrium for rational coalitions of given size — profitability threshold α*.
4. **Simulation validation** on realistic difficulty-adjustment code (Bitcoin Core, Ethereum fork config) confirming tightness.

## How to start

1. Read Yaish et al. (2022) Sections on difficulty retargeting and the stretching/squeezing constructions — extract the exact per-epoch manipulation constraints imposed by MTP-11 and the 2-hour rule. Then Uncle Maker (2023) for the epoch-boundary race model.
2. Model as a controlled random walk: block arrivals are Poisson; the coalition chooses t_n within [MTP, wall clock + s]. Derive the one-step feasible distortion set, then the drift of difficulty as a function of policy — this reduces to a Markov decision process; solve for optimal policy at fixed α first (known territory), then extend to α_t drawn from a distribution each epoch (changing coalitions).
3. Main obstacles: (a) the feedback loop (distorted timestamps alter difficulty, which alters arrival rates, which alters MTP); (b) races near epoch boundaries require analyzing simultaneous-block forks — use uncle/orphan-rate formulas (e.g., from GHOST literature) rather than re-deriving.
4. Sanity-check any bound against the pointwise MTP robustness claim (median needs >w/2 bad timestamps) — a correct bound must degenerate to that at α→0.

## Key references

- Yaish, G., Stern, H., Zohar, A. (2022). *Blockchain Stretching & Squeezing: Positively Correlated Difficulty Manipulations*. https://doi.org/10.1145/3490486.3538250 — Exhibits concrete profitable attacks on Bitcoin-style difficulty adjustment by shifting timestamps to inflate/deflate measured block times.
- Yaish, G., Stern, H., Zohar, A. (2023). *Uncle Maker: (Time)Stamping Out The Competition in Ethereum*. https://doi.org/10.1145/3576915.3616674 — Demonstrates Ethereum timestamp manipulation at epoch boundaries to force difficulty drops and disadvantage competitors, including the race dynamics at the tip.
- (verify) General Byzantine clock-synchronization surveys, e.g., work following Srikanth & Toueg on bounded-clock drift, for background robustness techniques.

## Warnings

- The census record is thin; the "changing coalitions" and "cost of races" formalizations above are the most reasonable reading but are not canonical — pin down definitions before proving anything.
- Don't assume constant α: the entire novelty is coalition churn; fixed-α results overlap with what Yaish et al. already did.
- Median robustness (≤w/2 corrupt timestamps per window) is about *instantaneous* timestamps, not sustained drift — conflating these is the most common error.
- Ethereum's relevant rules are pre-Merge PoW; check whether the target includes PoS timestamp games (post-Merge validators also propose timestamps) before scoping.

---
*`BCGT-000243` · flagged for review*

## Acceptance guidance

1. **Upper bound:** a theorem giving, for any coalition strategy with hash fraction process bounded by α_t, the maximum sustainable clock distortion / difficulty bias as a function of median window w and future slack s (e.g., "sustained |D| ≤ f(α, w, s), with mixing time of difficulty back to honest level ≤ g(α)").
2. **Lower bound / matching construction:** a coalition strategy (with changing membership, i.e., α_t adapted to observed chain state) achieving distortion matching the upper bound up to constants, including the equilibrium cost of races (probability of losing the epoch-boundary race times reward).
3. **Game-theoretic variant:** characterize when manipulating is a (Nash or strong Nash) equilibrium for rational coalitions of given size — profitability threshold α*.
4. **Simulation validation** on realistic difficulty-adjustment code (Bitcoin Core, Ethereum fork config) confirming tightness.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/ee79797c-6de5-4546-817f-dd0132c70ab1?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000243.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000243)
