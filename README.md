# Tiny Chess — v2 entry (0x88 mailbox, PVS + TT + quiescence)

Engine: iterative deepening, fixed 50,000-node budget (deterministic, never a
clock), PVS, 2^17-slot transposition table, 2 killers/ply, butterfly history,
null-move pruning (R=2/3), LMR, quiescence (captures/promotions; all evasions
in check), repetition/50-move draws with contempt. Eval: material + Michniewski
PSTs (÷5, flat list), mg/eg taper, bishop-pair term (+30), game-history
repetition avoidance.

v2 changes vs the submitted v1: bishop-pair term re-added, repetition
avoidance via game history (the 6 threefold draws in v1 trials t1/t2 — all in
better positions — became 4 wins, 2 losses, 0 draws), code golfed to free
room. Perft-clean (startpos d4 = 197,281; Kiwipete d3 = 97,862). Slowest move
2.09 s (limit 10 s).

## Local practice trials (kit, 8 games each, same 8 house engines)

| engine | t0 | t1 | t2 | mean |
|---|---|---|---|---|
| v1 shipped (4,084 B) | 1,530 (5-0-3) | 1,464 (3-3-2) | 1,326 (2-3-3) | 1,440 |
| **v2 (3,980 B)** | **1,464 (4-1-3)** | **1,598 (5-1-2)** | **1,464 (4-1-3)** | **1,509** |
| v2 + opening book (4,089 B) | 1,530 (4-2-2) | 1,396 (4-0-4) | 1,396 (4-0-4) | 1,441 |

v2 is +69 Elo over v1. Still below the ~1,600 top-50 bar (rank 100 = 1,530).

## Opening book — tested and REJECTED

A 7-entry Stockfish depth-24 book (dict literal keyed on the low 20 bits of the
engine's Zobrist hash, legality-checked) was built, packed to 4,089 bytes, and
run through all 3 practice trials. Measured: 1,441 mean, −68 vs v2 without the
book, +1 vs v1. The three entries that fired: slav a2a4 (loss→draw), catalan
g1f3 (win kept), english-reversed-sicilian f1g2 (win→loss — Stockfish says
+139 cp, our engine loses the resulting position while winning from its own
choice). The three highest-value entries (alekhine BREP +223, kings-indian W11
+163, alekhine W11 +147) never fire in practice trials 0–2 (those openings do
not appear); only scored trials with fresh seeds could reach them. The
Stockfish-value model does not transfer to this engine, so the book is NOT in
this entry. Book version preserved at `dev/chess/main_v2_book.py` (4,089 B).

## Files

- `main.py` — packed entry (3,980 bytes, `makefun compress` of `engine.py`)
- `engine.py` — readable source (gitignored, kept for the bundle)
- `LICENSE` — MIT (SPDX line required by the `license_osi` check)
- `.makefun/owner.json` — payout wallet
