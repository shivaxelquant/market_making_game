# Market of Cards

A group market-making card game you play in the browser against Bayesian bots. Every card on the table adds to one hidden total. You quote it, trade it, and read the other players' markets before the centre cards turn.

Open `index.html` in any modern browser. There is no build step. To host it, turn on GitHub Pages for this repository and serve from the root of `main`.

## Rules

- **The contract.** Everyone trades one number: the sum of every card in play. That is each player's two private cards plus three face-down centre cards.
- **Card values.**
  - 2–10 count ten times their rank (20–100) in any suit.
  - Red J, Q and K are +110, +120 and +130; black ones are −110, −120 and −130.
  - Aces are ±140 by colour, or +10 each with the low-ace setting.
  - The average card is worth 2,160 / 52 ≈ 41.5.
- **Rounds.**
  - There are four rounds. In each round every player makes one two-way market (bid @ ask) within the max spread, and the starting seat rotates.
  - When a market goes up, every other player answers at once: buy at the ask, sell at the bid, or pass.
  - An answer can be 1 lot up to the Max lots setting. The cap is the same for everyone.
  - One centre card turns after each of rounds 1–3.
- **Settlement.** Buying at p earns `total − p` per lot, and selling earns `p − total`. With a position limit on, orders are cut down to fit, or cancelled if nothing fits.
- **Shot clock (optional).** A fixed number of seconds per decision. If time runs out while you are quoting, your market is skipped; if it runs out while you are answering, you pass. Decision times are logged so you can compare timed and untimed play.

## How the bots think

- **Fair value.** Each bot computes `own hand + centre shown + Σ E[other hands] + E[hidden centre]` exactly from the unseen deck. The last term allows for card removal: if other hands look rich, the centre cards that remain are expected to be poorer.
- **Reading markets.** Given public information, a quoter's fair value is linear in its hidden hand sum `s`, so `mid ≈ A + B·s`. Bots invert that line and update a full distribution over each hand by Bayes' rule, over every two-card pair from the unseen deck.
- **Reading you.** In Mixed and Fallible play the bots weigh five models of you:
  - full-information, tight
  - full-information, loose
  - own-cards, tight
  - own-cards, loose
  - guarded quotes

  The style odds are learned from your revealed hands across games.
- **Answering and sizing.** Bots are risk-neutral. They trade when the market clears their reservation price `r = FV − κ·position`, and keep adding lots while each extra lot still clears the bar.

## Bot policies and handicaps

| Setting | What it does |
|---|---|
| Exact | Every bot quotes its true fair value. Readable, and beatable. |
| Mixed (default) | Each deal, every bot secretly quotes open (60%) or guarded (40%). A guarded quote reveals nothing about the bot's hand. |
| Fallible | Mixed, plus one random bot quotes and answers with about ±25 points of error. |
| Secret bot team | 2 or 3 bots share their hands, never trade with each other, and are revealed at settlement. Team bots quote as if they knew only their own hand, and use the pooled cards when answering. |
| Bots see your cards | Every bot sees one or both of your cards. The reading model allows for this when anyone reads a bot's market. |

## Can a perfect player beat them?

The page includes a benchmark that plays hundreds of games with a scripted player in your seat:

- **Concealer:** reveals nothing in its quotes.
- **Honest:** quotes and answers at its full-information value.
- **Own-cards:** uses only its own hand and the centre.

Measured results, with 4 bots, spread 20, limit ±5, and the 95% range on the per-game edge:

| Scripted player | Exact (1 lot) | Mixed (1 lot) | Exact (5 lots) | Mixed (5 lots) |
|---|---|---|---|---|
| Concealer | +114 to +140 ± 56 | −12 ± 45 | +74 ± 53 | +46 ± 52 |
| Honest | +48 ± 50 | −22 ± 43 | +36 ± 63 | −35 ± 55 |

| Handicap (Mixed, 5 lots) | Concealer | Honest |
|---|---|---|
| Secret team of 2 | −93 ± 114 | −82 ± 102 |
| Bots see 1 card | −61 ± 82 | −131 ± 77 |
| Bots see both cards | −197 ± 76 | −119 ± 71 |
| Team of 2 + bots see both | −310 ± 98 | −228 ± 83 |

Why these results come out this way:

- **Random price noise makes bots weaker.** By Jensen's inequality, an informed trader's take `E[(|X − η| − h)⁺]` only grows with noise η.
- **Randomising which information a quote uses (the Mixed policy) works.** A guarded quote's gap from the bot's own value comes only from cards nobody else can see, so nobody can trade against it.
- **No strategy is guaranteed an edge.** The game is zero-sum and symmetric, so in a symmetric equilibrium every seat averages 0, and no single player can do better on their own.

## After each game

The settlement view shows:

- the final total
- a hindsight fair value for each round, with every hand known
- every market midpoint against that fair value
- how far off the bots' read of your hand was
- your quote error, right answers, decision times and timeouts

Finished games are kept in the browser's local storage and can be exported as CSV.
