# Tiny tapered alpha-beta engine (4 KB)

A complete chess engine in a single `main.py` of **4,087 bytes** (cap: 4,096).
The file holds an lzma-compressed, base85-encoded payload of the readable
engine source (~7.2 KB) plus a one-line unpacker that `exec`s it. Python
standard library only; no network except the game-environment protocol the
harness defines.

## Design

- **Board**: 64-square mailbox array, signed piece codes, fully incremental
  state — tapered midgame/endgame scores, phase counter, Zobrist key,
  castling rights, en-passant and halfmove clock are all maintained by
  make/unmake (no re-scanning), with a single undo stack that restores every
  field from a snapshot.
- **Move generation**: pseudo-legal generation (pawns with promotions and
  en-passant, unified stepper/slider ray loop, castling with transit checks);
  legality is enforced by testing the own king after each make.
- **Search**: negamax alpha-beta with PVS, a transposition table (depth,
  score, bound flag, best move), quiescence search over captures and
  promotions with full check evasions when in check, check extensions,
  null-move pruning, late-move reductions, killer moves, MVV-LVA and
  transposition-move ordering, exact mate/stalemate scoring at every node
  (a mated side scores −mate adjusted by distance; a stalemate scores 0),
  and draw handling by repetition against the game history inside the
  search.
- **Evaluation**: tapered material plus formula piece-square terms —
  pawn advancement by rank and file, knight/bishop centrality, rook on the
  seventh rank, and a king that hides in the middlegame and centralises in
  the endgame.
- **Determinism**: the root searches iterative deepening under a fixed
  80,000-node budget — never a clock — so every move is a pure function of
  the position and replays reproduce exactly. Root state (history stack,
  transposition table, repetition set seeded from the game's own moves) is
  rebuilt from the harness position on every turn.
- **Protocol**: each turn the driver rebuilds the game from `start_fen` and
  the move list, asks the root for a move, validates it against the legal
  moves the environment reports, and falls back to those legal moves in
  order if a submission is ever refused. It never resigns.
