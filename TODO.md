# TODO — td-infeasible-search protocol

Phases mirror the td-route-trees discipline (design memo → prototype → gates → benchmark → decision memo). Detailed context and the agent brief: [CONTEXT.md](CONTEXT.md).

- [ ] **P8.0 — Design memo: TD time-warp on NDCPWLF.** Literature pass (Nagata time-warp, Vidal penalized HGS, PyVRP PenaltyManager/ILS) + the formal construction: warp-augmented leg composition, exact-reduction-to-checker property, departure-time-optimization interaction, composability analysis (does (duration, warp) segment concatenation survive on NDCPWLF segments / trees?). **Accepted by Onyr before any prototyping.**
- [ ] **P8.1 — Prototype the warp-augmented fold.** Scaffold (scikit-build-core, frozen kayros `cpp/pwlf/` engine, pybind11, pytest, CI, nix-dev flake extending base — td-route-trees is the template). Gates: bitwise identity with the checker fold on feasible routes across all four families (Rifki vertical-step stress; duration-not-argmin assertions); controlled warp values on hand-built infeasible routes.
- [ ] **P8.2 — Penalized move evaluation.** Warp/load-penalized deltas for the kayros operator set (relocate/swap/2-opt*, splice-style); sequential vs structure-assisted evaluation (per P8.0's composability verdict); microbench cost vs the feasible-only evaluators.
- [ ] **P8.3 — Penalty management + ILS-with-infeasibility prototype.** Dynamic weights (PyVRP baseline: ×1.5/×0.9, target-feasible ratio), interaction with LAHC; a minimal penalized ILS loop over the prototype evaluators.
- [ ] **P8.4 — The payoff experiment.** Penalized vs feasible-only ILS at equal budgets on MAMUT subsets (all four families, sizes stratified), anytime metrics (primal-integral-style, time-to-target-gap), seeds × instances protocol as in `tdvrptw-workspace/src/experiments/`. Honest verdict either way.
- [ ] **P8.5 — Decision memo for kayros integration** (only if P8.4 is positive): what kayros adopts, API/params, migration plan on top of the Stream-7 ILS. Accepted by Onyr → work moves to kayros.

## Gating

- Integration into kayros is gated on Stream 7 (feasible-only TD-ILS, kayros 0.3.0) being complete — this stream is parallel research until then.
- Every phase produces a report under `reports/`; session logs in `reports/sessions/` (create on first session).
