# Tiny Chess — 4KB Python engine

Compact chess engine in at most 4,096 bytes of Python (stdlib only), for the
make.fun `tiny-chess-01` bounty. The shipped file is `main.py`: an LZMA-packed
payload from `makefun compress` applied to the readable source `engine.py`
(kept in the working directory for the transcript bundle, not committed, to
stay inside the 8,192-byte tree cap).

## Approach

- **Board:** 0x88 mailbox, white pawns move +16.
- **Eval:** material (P100 N320 B330 R500 Q900) + Michniewski piece-square
  tables with a midgame/endgame taper + bishop pair +30.
- **Search:** iterative deepening, fixed 50,000 nodes/move (deterministic,
  never a clock), PVS, transposition table (2^17 slots), 2 killers/ply,
  butterfly history, null-move pruning (R=2/3), LMR, quiescence
  (captures/promotions; all evasions in check), repetition/50-move draws
  with contempt.
- **Move gen:** pseudo-legal + make/unmake king-safety check; en passant,
  castling, promotions. Perft-clean (startpos d4 = 197,281; Kiwipete
  d3 = 97,862).
- **Protocol:** HTTP to the kit env (`/game`, `/move`), position rebuilt
  from the FEN each turn.

## Local results (kit practice trials)

Dev engine: trial 0 — 4-0-4, Elo 1,396; trial 1 — 5-0-3, Elo 1,530
(mean 1,463).

Shipped `main.py` (4,084 bytes) is the same engine with two size-driven
cuts: the bishop-pair term is removed and the game-history replay for
3-fold repetition detection is dropped (repetition is still detected
inside the search tree). Validated on all 3 practice trials:

- Trial 0: **5-0-3, Elo 1,530** (7 games by checkmate; game 8 cut short by
  a service restart, scored as an abandoned loss — the engine did not
  crash). Wins vs depth-2/3/4 and nodes-100k/250k; losses to
  nodes-500k/1M/2M.
- Trial 1: **3-3-2, Elo 1,464** (3 draws by threefold repetition).
- Trial 2: **2-3-3, Elo 1,326** (3 draws by threefold repetition).
- **Mean Elo 1,440** — above the 1,154 practice floor and the ~1,325
  house-baseline mean. The repetition draws are the cost of the dropped
  game-history detection; the cuts cost very little strength.

Time per move at 50,000 nodes: ~1.0–1.8 s (startpos 1.21 s, Italian
middlegame 1.77 s, endgame 1.04 s), inside the 10 s/move limit.

## Size budget (4,096 bytes)

`main.py`: **4,084 bytes** (12 bytes of headroom). The LZMA payload is
dominated by search and move-generation logic; the piece-square tables are
stored divided by 5 as a flat list to keep their packed cost low.

## Licence

MIT — see `LICENSE`.
