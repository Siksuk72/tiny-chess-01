# tiny-chess-01 entry

A compact, deterministic chess engine in a single Python file (`main.py`,
4,096 bytes as committed). It is an original implementation: a 0x88 board,
negamax with alpha-beta pruning, iterative deepening to a fixed node budget
(100,000 nodes per move — no clock, no randomness, so replays play the same
moves), quiescence search, a transposition table, killer-move ordering, and
an incremental material + piece-square evaluation (tapered king table,
bishop-pair bonus).

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

| trial | W | D | L | points | Elo |
|-------|---|---|---|--------|-----|
| trial-0 | 4 | 2 | 2 | 5.0/8 | 1530 |
| trial-1 | 4 | 0 | 4 | 4.0/8 | 1396 |
| combined (16 games) | 8 | 2 | 6 | 9.0/16 | 1464 |

Average move time ~2.1 s, slowest observed 3.4 s (limit 10 s); a full
8-game trial takes ~18 minutes (limit 30 minutes). Determinism verified:
the same position searched twice returns the same move. Perft passes on
six standard positions (start, kiwipete, and four tactical positions).

The house baseline (sunfish's 4 KB entry) scores ~1325 Elo on the same
practice trials; the practice floor to enter the final is 1154.
