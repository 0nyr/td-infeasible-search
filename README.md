# td-infeasible-search

Research study repo: **careful infeasible search-space exploration for time-dependent vehicle routing (TDVRPTW/TDVRP) under exact arithmetic**.

Part of Onyr's PhD KAYROS programme (Stream 8 of [plan 2](https://github.com/0nyr/tdvrptw-workspace) — `reports/plans/2-bpc-website-td-ils-plan.md`). Sibling of [td-route-trees](https://github.com/0nyr/td-route-trees) (the TD local-search structures study that produced the KAYROS TD-LS layer) and of [kayros](https://github.com/0nyr/kayros) itself.

## The question

Modern LS-based VRP solvers (PyVRP's ILS, HGS lineage) owe a large part of their strength to **penalized exploration of the infeasible space**: time-warp for time-window violations, load excess for capacity, with dynamically managed penalty weights. KAYROS is time-dependent and **exact by construction** — route durations come from NDCPWLF arrival-time-function composition with exact doubles, no epsilons, refereed by the `mamut-routing-lib` Duration checker, which has *no violation semantics*. This repo studies how to define, evaluate, and manage TD infeasibility so that anytime resolution improves, without giving up the exact-arithmetic contract.

Full context, research questions, protocol, and the delegated-agent brief: [CONTEXT.md](CONTEXT.md). Work tracking: [TODO.md](TODO.md).

## Status

Scaffold only (created 2026-07-06). Research not started.

## License

[MIT](LICENSE).
