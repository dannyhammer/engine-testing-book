# Overview of Engine Testing

I'll say it up front:

> **You MUST thoroughly test _every_ change you make to your engine, regardless of the complexity.**

Why?
Let's say you have a new engine with only a handful of basic features implemented, such as a iterative deepening with an alpha-beta and quiescence search, transposition tables, move ordering through hash move and MVV-LVA, and an evaluation based on piece-square tables.
You hacked these features together by reading various pages on Wikipedia, the Chess Programming Wiki, Stack Overflow, and other websites until your engine seemed to grow stronger.
You then add another basic feature, like aspiration windows, to your engine, so you write a few dozen lines of code and call it good.
But how can you be sure this feature actually _works_?
For all you know, it could be a total improvement right up until your engine finds a mate-in-`n` and then suddenly everything breaks because of a bug in your transposition table.

Engines are complicated.
Especially the search.
Every feature you add interacts with the other features of the engine, so it is almost impossible to predict whether a change you made will actually be an improvement without properly testing during you development.
This is particularly true if you have largely copy-and-pasted feature implementations from various websites without fully understanding the logic behind them.
For example, does your search use a "fail soft" or a "fail hard" framework?
Are you properly modifying mate-in-`n` scores as they bubble up the search stack?
When performing transposition table cutoffs, are you ensuring that you're not in a PV node?

If you _don't_ know the answers to those questions, or if you don't fully understand them, then chances are high that your implementation(s) are bugged.
If you don't fix these bugs quickly, they will propagate to other parts of the engine later and your development will get sloppier and more frustrating.

## Minimum Viable Product

Properly testing your engine involves having the old version of the engine play against the new version, so your engine must be able to play a complete game of chess without crashing or playing illegal moves.

The _first_ test that you run should be a "sanity check" of your baseline engine vs itself.
This is intended to ensure that your testing framework works _before_ you start testing actual changes to the engine.

Your "baseline engine" should be a UCI-compliant random-mover that supports being benchmarked[^baseline-bench].

### UCI

The [Universal Chess Interface (UCI)](https://backscattering.de/chess/uci/) is the standard communication protocol that chess engines and chess GUIs use.
UCI has many specifications, but only a [subset of them](https://www.chessprogramming.org/Sequential_Probability_Ratio_Test#Minimum_UCI_Requirements) are required for testing:

```
go wtime <> btime <> winc <> binc <>
position startpos
position fen <fen>
quit
stop
uci
ucinewgame
isready
```

The details on how to make your engine UCI-compliant will be specific to your engine and are out of the scope of this guide.
Please see the relevant links to UCI documentation for more information.

### Random Moves

Your baseline should return a random [_legal_](https://www.chessprogramming.org/Legal_Move) move when asked to search on a position.
This should be trivial to implement, assuming you have (well-tested) code in place to generate moves for a given position and check the legality of those moves.

### Benchmarking

This is somewhat of a [OpenBench-specific](https://github.com/AndyGrant/OpenBench/wiki/Requirements-For-Public-Engines#basic-requirements) requirement, meaning that you do not need this if you intend to use `fastchess` or `cutechess` directly.
However, it is _highly recommended_ as it will be a useful aid in development.

A "benchmark" is simply a fixed-depth search on a pre-defined set of positions.
It is used to determine if a change to your engine has affected the search in a significant way, as well as serving as a "fingerprint" for each commit you make during development.
You can get use an existing bench suite from a reputable engine such as [Stormphrax](https://github.com/Ciekce/Stormphrax/blob/main/src/bench.cpp#L29), [Viridithas](https://github.com/cosmobobak/viridithas/blob/master/src/bench.rs#L1), or [Stockfish](https://github.com/official-stockfish/Stockfish/blob/master/src/benchmark.cpp#L30).
I would _personally_ recommend including ["edge-case" positions](https://github.com/dannyhammer/toad/blob/a5b4a5a5300a15e30e582217b5694f5fa6f2276e/src/utils.rs#L58) in your benchmark suite, such as positions with only 1 legal move available, drawn/checkmated positions, [zugzwang](https://en.wikipedia.org/wiki/Zugzwang) positions, etc. as they can help detect non-functional changes for things that can be annoying to debug if implemented improperly.

Your engine needs to support being benched from the command line:

```bash
./YOUR-ENGINE bench
```

And produce an output containing the number of nodes search and the nodes-per-second (`nps`) metric.

```
8763483 nodes / 3.426305007s := 2557706 nps
```

---

[^baseline-bench]: For the baseline engine, you _can_ just hardcode the `bench` command to display a fixed number for the `nodes` and `nps` values. You _will_ need to change this once you implement a real search, however.
