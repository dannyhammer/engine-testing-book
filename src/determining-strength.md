# Determining Engine Strength

The [Computer Chess Rating Lists (CCRL)](https://computerchess.org.uk/ccrl/) is a website dedicated to testing and rating chess engines.
Engines, just like humans have their strength based in the [Elo rating system](https://en.wikipedia.org/wiki/Elo_rating_system).

If you want to determine how strong _your_ engine is, you must test it against an engine with a known strength.
Within the chess programming community, the engine [Stash](https://github.com/mhouppin/stash-bot) is a popular choice for this task.
Stash has a [well-documented development history](https://gitlab.com/mhouppin/stash-bot/-/releases) with [plenty of releases](http://computerchess.org.uk/ccrl/4040/cgi/compare_engines.cgi?family=Stash&print=Rating+list&print=Results+table&print=LOS+table&print=Ponder+hit+table&print=Eval+difference+table&print=Comopp+gamenum+table&print=Overlap+table&print=Score+with+common+opponents) that range from 1000 Elo to 3300 Elo and beyond[^stash-discord-message]:

```
Blitz Rating (* Not ranked by CCRL, only estimates)

v36     3399
v35     3358
v34     3328
v33     3286
v32     3252
v31     3220
v30     3166
v29     3137
v28     3092
v27     3057
v26     3000*
v25     2937
v24     2880*
v23     2830*
v22     2770*
v21     2714
v20     2509
v19     2473
v18     2390*
v17     2298
v16     2220*
v15     2140*
v14     2060
v13     1972
v12     1886
v11     1690
v10     1620*
v9      1275
v8      1090*
```

So, to find your engine's strength, just pick a version of Stash, download and compile it, `bench` it, and queue up a test between your engine and that version of Stash.
Rather than run an SPRT, just run a fixed-games test of 1000-5000, depending on how accurate of an Elo estimate you want.

## Example

Consider [the following test](https://pyronomy.pythonanywhere.com/test/775/) of [Yukari](https://github.com/yukarichess/yukari/tree/8b0deea65507cbf621a5a8959b0a222906ae3c2d) vs [Stash v30.0](https://gitlab.com/mhouppin/stash-bot/-/releases/v30.0):

```
Elo   | 20.82 +- 9.90 (95%)
Conf  | 8.0+0.08s Threads=1 Hash=16MB
Games | N: 3008 W: 1175 L: 995 D: 838
Penta | [128, 285, 565, 331, 195]
```

After 3000 games, Yukari was an estimated ~10-~30 Elo stronger than Stash v30.0.
Since v30.0 has a [CCRL Blitz rating of 3130](http://computerchess.org.uk/ccrl/4040/cgi/engine_details.cgi?match_length=30&each_game=0&print=Details&each_game=0&eng=Stash%2030.0%2064-bit#Stash_30_0_64-bit), it can be concluded that Yukari's strength is somewhere around ~3140 Elo.

---

[^stash-discord-message]: See [this message](https://discord.com/channels/435943710472011776/882956631514689597/1176195814171889715) in the Stockfish Discord server for exact ratings.
