# tiny-chess-01 entry

A compact, deterministic chess engine in a single Python file (`main.py`,
4,096 bytes as committed). It is an original implementation: a 0x88 board,
negamax with alpha-beta pruning, iterative deepening to a fixed node budget
(100,000 nodes per move — no clock, no randomness, so replays play the same
moves), quiescence search, a transposition table, killer-move ordering,
null-move pruning (R=2, middlegame only), late move reductions (LMR, quiet
moves after the third), and an incremental material + piece-square
evaluation (tapered king table, bishop-pair bonus).

`main.py` is the engine source packed with `makefun compress` (lzma) to fit
the 4,096-byte cap; at run time it decompresses and runs the same code.
The engine talks to the competition environment at `MAKEFUN_ENV_URL` using
the protocol from the kit's starter (`POST /game`, `POST /move`).

## Run

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:18080 python3 main.py
```

Requires Python 3.13 (standard library only), one CPU, at most 10 seconds
per move (it uses ~2 seconds per move at the 100,000-node budget).

## Local results (practice trials, 1 CPU, fixed 100,000-node budget)

v2 (null-move pruning + LMR) — the version in this commit:

| trial | W | D | L | points | Elo |
|-------|---|---|---|--------|-----|
| trial-0 | 5 | 1 | 2 | 5.5/8 | 1598 |
| trial-1 | 5 | 1 | 2 | 5.5/8 | 1598 |
| trial-2 | 5 | 1 | 2 | 5.5/8 | 1598 |
| combined (16 games) | 10 | 2 | 4 | 11.0/16 | 1598 |
| combined (24 games) | 15 | 3 | 6 | 16.5/24 | 1598 |

v1 for comparison (same methodology): trial-0 1530, trial-1 1396,
trial-2 1173, combined 16-game 1464 (9.0/16), combined 24-game 1373
(11.5/24). v2 gains +134 Elo over 16 games and +225 over 24 games.
Notably, v2 wins the trial-0 Berlin (game 4, vs nodes-100k) that v1 lost,
and draws the trial-2 game 8 (vs nodes-2M) that v1 lost.

Average move time ~2.1 s at full CPU speed (slowest observed 3.9 s,
limit 10 s); under the 3-trials-in-parallel test load the slowest
observed move was 7.6 s. A full 8-game trial takes ~18 minutes
(limit 30 minutes). Determinism verified: the same position searched
twice returns the same move. Perft passes on six standard positions
(start, kiwipete, and four tactical positions).

### What was tried and measured (all on the same methodology)

- **Null-move pruning alone (v2a):** 16-game Elo 1430 (8.5/16),
  24-game 1396 (12.0/24) — no clear gain over v1 (1464 / 1373), so
  not kept on its own.
- **Null-move + LMR (v2b, this version):** 1598 on both 16 and 24
  games — a clear, large gain; kept.
- **150,000-node budget on top of v2b (v2c):** trial-0 4.0/8,
  Elo 1396 — worse than v2b's 5.5/1598 on the same trial, and the
  extra nodes shrink the time margin, so the 100,000-node budget
  is kept.

The house baseline (sunfish's 4 KB entry) scores ~1325 Elo on the same
practice trials; the practice floor to enter the final is 1154.
