# Engine Strength Testing

I'll begin with a non-negotiable commandment:

> **You must _thoroughly_ test every change you make to your engine, regardless of the complexity.**

Why?
Let's say you have a new engine with only a handful of basic features implemented.
You hacked these features together by reading various pages on Wikipedia, the Chess Programming Wiki, Stack Overflow, and other websites until your engine seemed to grow stronger.
You then decide to add another feature to your engine, such as a modification to the search algorithm or a tweak to the evaluation function.
So you write a few dozen lines of code and call it good.
But how can you be sure this feature actually _works_?
For all you know, it could be a total improvement on the few positions you unleashed the new version of your engine on, but fail catastrophically on many other positions, leading to an overall decrease in strength.
This could happen because you have a sneaky bug in your implementation, or because the feature you added just doesn't help your engine.

Engines are complicated.
Especially the search.
Every feature you add interacts with the other features of the engine, so it is almost impossible to predict whether a change you made will actually be an improvement without properly testing during you development.
This is particularly true if you have largely copy-and-pasted feature implementations from various websites without fully understanding the logic behind them.
For example, does your search use a "fail soft" or a "fail hard" framework?
Are you properly modifying mate-in-`n` scores as they bubble up the search stack?
When performing transposition table cutoffs, are you ensuring that you're not in a PV node?

If you _don't_ know the answers to those questions, or if you don't fully understand them, then chances are high that your implementation(s) are bugged.
If you don't fix these bugs quickly, they will propagate to other parts of the engine later and your development will get sloppier and more frustrating.

## Different Kinds of Tests

This guide is focusing on strength testing, wherein you are trying to determine if the changes you made to your engine have altered its strength ([Elo](https://en.wikipedia.org/wiki/Elo_rating_system)) in a measurable way.
This is not the only kind of testing you should be doing during development.
You should familiarize yourself with the concepts of [unit testing](https://en.wikipedia.org/wiki/Unit_testing) and [integration testing](https://en.wikipedia.org/wiki/Integration_testing) (if you want bonus points, look into [mutation testing](https://en.wikipedia.org/wiki/Mutation_testing), too).
These kinds of tests will help you detect if the _implementations_ of features in your engine are correct, whereas strength testing will **only** tell you whether or not the feature gains Elo.

You can spend hours researching the proper ways of unit/integration tests, so I will not go into extreme detail here.
I will simply say that you should write unit tests for every logical function in your engine, and integration tests for every case wherein these functions interact with each other.
If your functions are too complicated and cannot be tested easily, you should refactor your code into more manageable (and testable) components.

I _highly_ encourage you to use the [test suites](https://github.com/TerjeKir/EngineTests) compiled by other engine devs.

Just for fun, here are some issues I have seen arise due to poorly written and untested code:

-   Evaluation functions that return mate scores
-   Hash collisions causing the engine to play illegal moves
-   Engine crashed because it was told to search for a negative amount of time
-   Search returned nonsensical values because it timed out and overwrote a valid hash table entry
-   I/O being read/written out-of-order
-   Integer overflow. Enough said.

## Minimum Viable Product

Properly testing your engine involves having the old version of the engine play against the new version, so your engine must be able to play a complete game of chess without crashing or playing illegal moves.
If you do not have your move generation working, or if it has not passed a [perft suite](https://www.chessprogramming.org/Perft), you should _not_ proceed to strength testing yet.

The _first_ test that you run should be a "sanity check" of your baseline engine vs itself.
This is intended to ensure that your testing framework works _before_ you start testing actual changes to the engine.

For testing with OpenBench, your "baseline engine" should be UCI-compliant, play random legal moves, support being benchmarked[^baseline-bench] and must be compilable with a `Makefile`.

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

A "benchmark" is simply a search on a pre-defined set of positions.
It is used to determine if a change to your engine has affected the search in a significant way, as well as serving as a "fingerprint" for each commit you make during development.
If the changes you've made to the engine do not cause the benchmark output to change, then the changes are likely to be non-functional.
[Non-regression tests](sprt.md#non-regression-bounds) should be used for testing non-functional changes.

You can get use an existing bench suite from a reputable engine such as [Stormphrax](https://github.com/Ciekce/Stormphrax/blob/main/src/bench.cpp#L29), [Viridithas](https://github.com/cosmobobak/viridithas/blob/master/src/bench.rs#L1), or [Stockfish](https://github.com/official-stockfish/Stockfish/blob/master/src/benchmark.cpp#L30).
I would _personally_ recommend including ["edge-case" positions](https://github.com/dannyhammer/toad/blob/a5b4a5a5300a15e30e582217b5694f5fa6f2276e/src/utils.rs#L58) in your benchmark suite, such as positions with only 1 legal move available, drawn/checkmated positions, [zugzwang](https://en.wikipedia.org/wiki/Zugzwang) positions, etc. as they can help detect non-functional changes for things that can be annoying to debug if implemented improperly.

For OpenBench, your engine must support being benched from the command line:

```bash
./YOUR-ENGINE bench
```

The output must contain the number of nodes search and the nodes-per-second (`nps`) metric:

```
8763483 nodes / 3.426305007s := 2557706 nps
```

### `Makefile`

This is another [OpenBench-specific](https://github.com/AndyGrant/OpenBench/wiki/Requirements-For-Public-Engines#basic-requirements) requirement.
OpenBench will build your engine by running `make EXE=<ENGINE_NAME>-<commit hash>`, so your engine must support being built via [`Makefile`](https://makefiletutorial.com/).

Below is an example `Makefile` for an engine written in Rust.
Change the compilation commands to whatever compiler your engine uses.

```Makefile
ifndef EXE
	EXE := YOUR-ENGINE
endif

openbench:
	@echo Compiling $(EXE) for OpenBench
	cargo rustc --release --bin YOUR-ENGINE -- -C target-cpu=native --emit link=$(EXE)
```

---

[^baseline-bench]: For the baseline engine, you _can_ just hardcode the `bench` command to display a fixed number for the `nodes` and `nps` values. You _will_ need to change this once you implement a real search, however.
