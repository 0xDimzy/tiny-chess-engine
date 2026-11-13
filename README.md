# Tiny Chess — v3 entry (0x88 mailbox, PVS + TT + quiescence)

Engine: iterative deepening, fixed 70,000-node budget (deterministic, never a
clock), PVS, 2^17-slot transposition table, 2 killers/ply, butterfly history,
null-move pruning (R=2/3), LMR, quiescence (captures/promotions only in quiet
positions; all evasions in check), check extension (+1 ply in check),
repetition/50-move draws with contempt. Eval: material + Michniewski PSTs
(÷5, flat list), mg/eg taper, bishop-pair term (+30), game-history repetition
avoidance.

v3 changes vs the submitted v2: node budget 50,000 → 70,000; check extension
added; quiescence generates captures/promotions directly instead of
generating all moves then filtering (1.09x speedup, zero behaviour change).
Perft-clean (startpos d4 = 197,281). Slowest move ~2.9 s (limit 10 s).

## Local practice trials (kit, 8 games each, same 8 house engines)

| engine | t0 | t1 | t2 | mean |
|---|---|---|---|---|
| v1 shipped (4,084 B) | 1,530 (5-0-3) | 1,464 (3-3-2) | 1,326 (2-3-3) | 1,440 |
| v2 submitted (3,980 B) | 1,464 (4-1-3) | 1,598 (5-1-2) | 1,464 (4-1-3) | 1,509 |
| v3 ND=80000 (3,970 B) | 1,748 (6-1-1) | 1,530 (5-0-3) | 1,396 (4-0-4) | 1,558 |
| **v3 ND=70000 (3,975 B, this entry)** | **1,669 (5-2-1)** | **1,464 (4-1-3)** | **1,598 (4-3-1)** | **1,577** |

v3-70K is +68 Elo over v2 (same 3 practice instances), record 13W-6D-5L vs
v2's 13W-3D-8L — 3 fewer losses. Point estimate, not proof: per-trial Elo
swings ±100–180, paired difference +68 ± ~103. Components validated in
self-play (80 games each, ND=3000): check extension +35 ± 77 (kept),
node doubling +163 ± 85 (kept); repetition fix −17 ± 76 (dropped); TT 2^18
untested (dropped). 70,000 chosen over 80,000: same practice score (1,577 vs
1,558) with more wall-time headroom. Still below the ~1,600 top-50 bar
(rank 100 = 1,530).

## Wall time (trial cap 1,800 s)

v3-70K walls: 1,429 s (1,042 plies), 1,518 s (1,147 plies), 1,455 s
(1,046 plies). v3 plays longer games than v2 (mean ~1,078 plies vs ~849), so
wall time is driven by game length more than node budget. A ~1,300-ply trial
would land near 1,730 s at this budget; observed max over 7 v3 practice
trials is 1,147 plies. ND=80000 was rejected: its trial 1 hit 1,660 s
(995 plies) and a 1,150-ply trial would time out there. ND=60000 would buy
~150 s for ~9 Elo; not taken.

## Opening book — tested and REJECTED (v2 work, unchanged)

A 7-entry Stockfish depth-24 book was built, packed to 4,089 bytes, and run
through all 3 practice trials on v2. Measured: 1,441 mean, −68 vs v2 without
the book. The Stockfish-value model does not transfer to this engine, so no
book is in this entry either. Book version preserved at
`dev/chess/main_v2_book.py` (4,089 B).

## Files

- `main.py` — packed entry (3,975 bytes, `makefun compress` of `engine.py`)
- `engine.py` — readable source (gitignored, kept for the bundle)
- `LICENSE` — MIT (SPDX line required by the `license_osi` check)
- `.makefun/owner.json` — payout wallet
