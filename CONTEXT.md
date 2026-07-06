# Context and delegated-agent brief — td-infeasible-search

This file is the entry point for the (Fable) agent delegated to this research stream. Read it fully, then follow the protocol in [TODO.md](TODO.md). Keep session logs and design memos in `reports/` (create `reports/sessions/` and `reports/design/`, mirroring the tdvrptw-workspace conventions).

## Where this sits in the programme

Onyr (Florian Rascoussier, 3rd-year PhD in OR under Romain Billot, Christine Solnon, Lina Fahed) develops **KAYROS** (`/home/onyr/code/phd/tdvrptw-workspace/kayros`, PyPI `kayros`), an anytime solver for duration-minimization TDVRPTW/TDVRP on the canonical MAMUT-routing TD benchmark families (Dabia2013, Ari2018, Vu2020, Rifki2020 — 1352 instances, BKS store maintained by the same programme).

Non-negotiable architectural contract ("the checker is the referee"):

- Travel is modeled by per-arc **NDCPWLF arrival-time functions** (non-decreasing continuous piecewise-linear); route duration is computed by exact composition (`θ_j ∘ α_ij ∘ acc` per leg) with **exact doubles, zero epsilons**.
- The reference checker `mamut_routing_lib.td.check_td_solution` *defines* the objective. Solvers propose routes, the checker prices them. Nothing enters the BKS store or a report without checker validation.
- KAYROS's TD-LS (module `cpp/ls/`) evaluates moves with LCA-BST route trees (Blauth et al. 2024 style) but **commits under the repricing rule**: float composition is not association-stable (fold-order experiment, td-route-trees P4.1), so *trees rank moves; accepted moves are rebuilt and repriced by the sequential checker fold and reverted unless strictly better*.
- One run is one thread. Seeded determinism everywhere.

Sibling study that sets the working style: `/home/onyr/code/phd/tdvrptw-workspace/td-route-trees` (protocol P4.0–P4.5: conventions memo → prototype → gates → microbench → decision memo accepted into kayros). This repo should follow the same discipline.

Stream 7 of plan 2 (`tdvrptw-workspace/reports/plans/2-bpc-website-td-ils-plan.md`) is building a **feasible-only TD-ILS** in kayros (granular neighbourhoods, ruin-and-recreate perturbation, LAHC acceptance), modeled on modern PyVRP (which, as of v0.14, removed HGS entirely for a single-trajectory ILS). That feasible-only ILS is the **baseline this stream must beat**.

## The research question

Classic VRPTW local search gains enormously from penalized infeasibility: Nagata's **time-warp** relaxation (arrive late ⇒ "jump back in time" and pay the warp), load excess for capacity, dynamic penalty weights (PyVRP's `PenaltyManager`: increase ×1.5 / decrease ×0.9 targeting ~65% feasible moves/solutions). Onyr's position: careful infeasible exploration will improve TD resolution, *especially anytime* resolution. But nothing of this exists in the exact-TD world of KAYROS:

1. **What is a TD time-warp?** Define a warp-augmented route evaluation on NDCPWLF compositions that (a) **reduces exactly (bitwise) to the checker fold on feasible routes** — no epsilons, no drift; (b) yields a principled (warp, duration) pair on infeasible routes. Candidate direction: clamp-at-deadline with warp accumulation, i.e. extend the leg composition so that when the arrival function exceeds a TW deadline the arrival is clamped and the excess accumulates as warp (the Nagata construction lifted from scalars to NDCPWLFs). Watch the interaction with duration-minimization semantics (departure-time optimization: warp may depend on the departure argmin — is the warp-augmented objective still an NDCPWLF/pair of NDCPWLFs in departure time?).
2. **Composability.** Is warp-augmented evaluation associative enough for tree evaluation (Visser-style segment concatenation extends to (duration, warp) pairs in classic VRPTW — does it extend on NDCPWLF segments)? If not, do we evaluate penalized moves sequentially only, or with a dedicated structure? Note the repricing rule already tolerates ranking-only structures.
3. **Capacity** is the easy half (prefix loads); include it for completeness.
4. **Penalty management** for TD: PyVRP's dynamic scheme as the baseline; what target-feasible ratio, what update cadence, how do penalties interact with LAHC acceptance?
5. **The payoff question (the actual deliverable):** does penalized infeasible exploration measurably improve *anytime* TD resolution (primal-integral-style metrics, time-to-target-gap) over the feasible-only ILS at equal budgets, on the MAMUT families? If yes, by how much and where (tight-TW families vs loose)?

## Constraints and cautions

- Exact arithmetic discipline: any new fold must have gates proving bitwise identity with the checker fold on feasible routes, across all four families (Rifki2020's envelope ATFs with genuine vertical steps are the stress case; duration plateaus make argmins association-fragile — assert on durations, not argmins; see td-route-trees P4.1 and kayros `tests/test_ls.py` for the precedent).
- Infeasible routes never reach the BKS store or any report; the penalized objective is a *search* device only.
- Prototype here (this repo), following td-route-trees practice: frozen copy of the kayros `cpp/pwlf/` engine as the arithmetic base, Python bindings for experiments, pytest gates, microbenches on MAMUT instances. Integration into kayros happens only via a decision memo accepted by Onyr, and only after Stream 7's feasible-only ILS baseline exists (kayros 0.3.0).
- Benchmark data: MAMUT-routing satellites (opt-in submodules); the experiment machinery precedent is `tdvrptw-workspace/src/experiments/`.

## Initial agent brief (the prompt Onyr will delegate with)

> You are starting the td-infeasible-search research stream (Stream 8 of plan 2 in tdvrptw-workspace). Read CONTEXT.md and TODO.md of this repo, then the referenced background: the td-route-trees conventions memo and P4.1 fold-order experiment, the kayros `cpp/ls/` layer and its gates, PyVRP's `PenaltyManager` + ILS loop (`/home/onyr/code/phd/PyVRP`), and the TD benchmark standard (`tdvrptw-workspace/reports/design/td-benchmark-standard.md`). Work the TODO protocol phases in order (P8.0 design memo first — get it accepted by Onyr before prototyping). Keep session logs in `reports/sessions/`, hypotheses and dead ends included. The exact-arithmetic contract and the repricing rule are non-negotiable; the deliverable of the stream is evidence, not code: does careful infeasible exploration improve anytime TD resolution, and if so, what design should kayros adopt?
