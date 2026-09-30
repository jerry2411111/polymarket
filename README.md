# Sports Prediction App

Use the app to view upcoming NBA and MLB games, compare model win probabilities
with Polymarket prices, and record the money you have spent on a game.

## Start the app

On the existing prepared project, open PowerShell in the project directory and run:

```powershell
.\sports_app\launch.ps1
```

Open http://127.0.0.1:8766. Keep the server running for automatic schedule,
lineup and price updates. To stop it when running directly in a terminal,
press Ctrl+C.

For a new Python installation, use Python 3.14, matching the current runtime:

```powershell
py -3.14 -m venv final_models\.venv
& .\final_models\.venv\Scripts\python.exe -m pip install -r requirements.txt -r final_models\requirements.txt
.\prepare_storage.ps1
```

Before requesting predictions, prepare the dated game, player and team histories
and the model inputs described in `final_models/README.md`. The app needs those
local histories to calculate probabilities. Creating empty storage alone does
not train the models. Data acquisition and date-query instructions are in
`data collecting/README.md`.

To create the NBA estimator configuration when initializing a new project:

```powershell
& .\final_models\.venv\Scripts\python.exe .\final_models\nba\create_template.py
```

This creates an unfitted estimator configuration. Historical feature data is
still needed for training. Then launch the app using the first command above.
For a visible server terminal, use:

```powershell
& .\final_models\.venv\Scripts\python.exe .\sports_app\server.py --port 8766
```

## Use the dashboard

1. Choose NBA, MLB or all games. Use the search box for a team, the date window
   for upcoming games, and the price filter to find games with market quotes.
2. Check the teams, start time and hours remaining. Schedules and final results
   come from official league sources. The app displays Polymarket international.
3. Read **Model Pick** and each team's predicted win probability. Decimal odds
   corresponding to a probability are `1 / probability`.
4. Follow the sport-specific guidance below. The Polymarket link opens the
   market where you can place your own order.
5. After buying, check **Bought this game**, enter your total spent in USD, and
   save it. The purchase history retains your entry after the game starts.

The purchase control is a manual record. It does not submit an order, check a
wallet balance or automatically size an order for your trading account.

## Refreshes and waiting messages

**Refresh fixtures & odds** updates schedules, predictions and quotes. The
optional five-minute refresh switch requests full refreshes while the page is
open. The server separately checks MLB lineups and prices every five minutes
inside the decision window and refreshes schedules daily while running.
The page reads the latest saved dashboard every minute, so newly ready MLB
forecasts appear even when the full-refresh switch is off. It avoids disturbing
an active purchase-entry form.

A quote older than two minutes is stale: refresh before acting on it.
MLB needs both starting pitchers and both complete nine-player batting orders
from the official pregame feed. It also uses a fixed 12-hour selection window.
A missing lineup, an unannounced start time, or a game outside that window can
legitimately show a waiting message. Check the displayed reason and source health.
An off-season league without scheduled games is shown accordingly.

## NBA model and how to bet

NBA uses rolling logistic regression with 27 raw features and ten interaction
terms. Features describe earlier team and player performance. Scaling, missing
value handling and calibration use earlier observations, with the existing
12-hours-before-start forecast cutoff. Historical training begins in 2021.

The buy split displayed under **Model Pick** equals the two model probabilities.
Choose a total budget for that match, then multiply it by each probability.
For example, with 40% away and 60% home, a $100 match budget becomes $40 on the
away team and $60 on the home team. These are cash amounts, not numbers of shares,
and the percentages describe the match budget, not the entire account.

At a purchase price of `q` dollars per share, an amount `A` buys `A / q` shares.
The winning contract normally settles at $1 per share. Holding both sides does
not guarantee a profit: the prices paid determine the final payout relative to
the combined cost. This split does not apply MLB's entry filter.

## MLB model and how to bet

MLB uses 53 baseball features: team form, starting-pitcher history, announced
batters, platoon matchup, recent player form, contact quality and bullpen form.
The model averages calibrated logistic regression and CatBoost probabilities
equally. Preprocessing uses training-only scaling, selected log transformations,
clipping and missing-value indicators. Market prices are not predictive inputs.

The fit updates at the UTC daily boundary using earlier known results; the latest
300 eligible earlier games are reserved for calibration. Features use only
information available by the forecast time. The original team inputs retain their
pre-12-hour cutoff, while announced-player inputs use the verified pregame report.

1. Wait for the official pitchers and complete batting orders within the game's
   fixed 12-hour selection window.
2. Select the model's higher-probability team.
3. Buy only at a price no higher than `model probability - 0.05`.
   The app shows that maximum price and its equivalent minimum decimal odds.
4. Use 1% of current total account equity for the normal stake. Keep total open
   purchase cost at or below 20% of account equity. Reduce new stakes if cash or
   remaining capacity is insufficient.
5. Buy once per game and hold to the result. Record the purchase in the app.

Example: a 60% model probability gives a maximum buy price of $0.55 per share
and minimum decimal odds of about 1.819. With $1,000 account equity, the normal
stake is $10, buying about 18.18 shares at $0.55. A win returns about $18.18
before costs; a loss returns zero. This is a price threshold, not a guarantee.
Displayed limits round conservatively. The app flags stale or unavailable quotes
and marks purchases you already recorded.

The current MLB rule buys the qualifying predicted winner. Its two-side
probability-allocation comparison remains research and is not the active rule.
The market-aware reference model is also not the active predictor.

## Maintain the app

Keep historical model inputs current using the data collection and feature
construction scripts. Refreshing fixtures and quotes does not itself extend the
historical training tables. The models use games from 2021 onward, subject to
actual collected coverage; date-specific training must not use later results.

Run the data-independent checks from the project directory:

```powershell
& .\final_models\.venv\Scripts\python.exe -m unittest discover -s tests -p test_quote_selection.py
& .\final_models\.venv\Scripts\python.exe -m unittest discover -s tests -p test_shared_http_limits.py
```

With the prepared historical data, run `sports_app/verify_final_models.py` to
replay model inputs and predictions. This also tests forecasts whose game IDs
do not exist in the completed-game history. Browser checks require Playwright's
Chromium installation. Source timestamps, model errors and missing-data reasons
should be checked before interpreting a displayed prediction.
