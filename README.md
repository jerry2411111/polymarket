# Sports Prediction App and Copy Trading App

This repository contains two separate local applications. The Sports Prediction
App provides NBA, MLB, NHL and soccer forecasts and controlled limit-order
execution on **Polymarket US**. The Copy Trading App screens macro/geopolitical
traders and follows their positions on **Polymarket International** in sandbox or
real mode. Each app has its own environment, account controls and stored data.

## Find the right app

| App | Project folder | Local address | Venue and workflow |
| --- | --- | --- | --- |
| [Sports Prediction App](#sports-prediction-app) | `sports_prediction_app` | http://127.0.0.1:8766; execution at `/trading` | Polymarket US; four-sport forecasts, automatic collection, dry run and LIVE limit orders |
| [Copy Trading App](#copy-trading-app) | `Copy Trading App` | http://127.0.0.1:8790 | Polymarket International; account screening, allocation and position copying |

Use Windows and Python 3.14. Run each command from the app directory named in its
section. Keep this README at the repository root beside `sports_prediction_app/`
and `Copy Trading App/` so its documentation links resolve. The sports ZIP contains
the `sports_prediction_app/` folder and its own project README; it does not contain
the separate Copy Trading App. The different ports allow both apps to run at once.

## Sports Prediction App

**Command directory: `sports_prediction_app`.** Updated October 6, 2026.

The app combines NBA, MLB, NHL and soccer forecasts, automatic data collection,
and a shared **Polymarket US** execution engine. The prediction dashboard is at
http://127.0.0.1:8766; account and trading controls are at
http://127.0.0.1:8766/trading.

Soccer coverage includes Premier League, La Liga, Bundesliga, Serie A, Ligue 1
and UEFA Champions League, with separate away, home and regulation-draw
probabilities. UEFA Nations League research is not an enabled live route.
An official fixture still needs a verified matching US market before execution.

### Package layout and data

The source ZIP retains the `sports_prediction_app/` layout, including
`data collecting/`, `collected data/raw/`, `collected data/result/`,
`final_models/`, `reports/`, `sports_app/` and `tests/`. It includes 682 historical
collection/reconstruction scripts, the current live collectors, setup helpers,
model code and tests. Placeholder READMEs retain the data directories.

Acquired datasets, fitted model artifacts, virtual environments, API credentials,
account preferences, purchases and execution ledgers are excluded. A fresh source
checkout needs the retained historical/model assets or their reconstruction;
creating empty storage does not prepare forecasts.

Use the [data requirements](<sports_prediction_app/DATA_REQUIREMENTS.md>),
[collection guide](<sports_prediction_app/data collecting/README.md>),
[model recipes](<sports_prediction_app/final_models/README.md>) and
[package layout](<sports_prediction_app/PACKAGE_LAYOUT.md>) for the corresponding paths.

### Install and start

On an already prepared installation, double-click **Open Sports App.cmd**, or run:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\sports_app\launch.ps1
```

For a new installation, use Windows and Python 3.14. From the extracted
`sports_prediction_app` directory:

```powershell
py -3.14 -m venv final_models\.venv
& .\final_models\.venv\Scripts\python.exe -m pip install -r requirements.txt -r final_models\requirements.txt
& .\final_models\.venv\Scripts\python.exe -m pip install -r sports_app\requirements-live.txt
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\prepare_storage.ps1
```

The storage helper creates the local directories and the `data` junction to
`collected data`. It does not download history and refuses to replace an unrelated
existing `data` path. Restore or prepare the dated feature tables and model assets
listed in the data requirements before expecting forecasts.

If the NBA estimator configuration has not been restored, create it with:

```powershell
& .\final_models\.venv\Scripts\python.exe -B .\final_models\nba\create_template.py
```

This creates the unfitted recipe and refuses to overwrite an existing template.
It does not supply training data. Then use the launcher above. To run the server
in a visible terminal instead:

```powershell
& .\final_models\.venv\Scripts\python.exe -B .\sports_app\server.py --port 8766
```

Keep the server running for collection and execution; closing the browser does
not stop either. Before intentionally shutting down an enabled instance, use
**STOP NEW TRADING** and wait for the cancellation/reconciliation result. Filled
positions remain held. A foreground server can then be stopped with Ctrl+C.

### Automatic collection and model updates

Collection starts with the app, including when no trading account is connected
and execution is OFF. NBA/MLB completed-game collection feeds dated player/team
history and rolling training. The separate NHL/soccer worker catches up calendars,
completed reports, validated features and selected monthly model preparation.
Missed work is retried after downtime. The collection status endpoint is
`/api/collection/status`.

The main collection entry points are `sports_app/lifecycle.py` for NBA/MLB and
`sports_app/live_worker.py` for NHL/soccer, with their schedule, detail, feature
and model modules. Historical discovery, acquisition, parsing, normalization and
date-query scripts are in `data collecting/`; current NHL/soccer support also
uses `nhl_soccer_iteration/` and `soccer_nhl_granular/` within that folder.
Run historical stages in their documented dependency order.

Sports history remains usable until new results, input revisions or rescheduling
require an update. It does not expire merely because a few minutes passed.
Account state, market books and official event status still need current checks.
Missing historical inputs, unconfirmed lineups, unavailable US markets and source
failures have separate readiness messages. Collection cannot make an unavailable
input or exchange listing immediately available.

NBA retains its rolling logistic model with 27 raw features, ten interactions
and the established T-12h feature cutoff. MLB retains the 53-input daily
logistic/CatBoost ensemble with dated calibration and confirmed official lineups;
its history-only preliminary forecast is labelled separately. NHL/soccer use
their selected monthly sports models, with soccer's separate confidence model.
Training and calibration preserve observation cutoffs, and started-game forecasts
stay frozen.

### Dashboard and account controls

Use sport/league filters, team search and the date window to inspect official
fixtures, forecasts, model readiness and US quotes. **Bought this game** remains
a manual purchase record; it does not send an order. Automatic orders are managed
on the separate execution page.

To start execution on a new installation:

1. Open `/trading`, enter the Polymarket US API key ID and secret, and use
   **Connect & save**. Review the account-check result beside the controls.
2. Select **DRY RUN** or **LIVE**, enter the bot allocation, and confirm it for
   that mode. Each mode retains its own confirmed budget.
3. Edit the sport amounts and the soccer/NHL dollars-per-point values as needed,
   then use **Confirm sport budgets**.
4. Use **Start dry run** for simulated execution. For real orders, use
   **Enable LIVE trading** and the separate on-page confirmation.

Dry run uses live account/book inputs and the same decision rules, with simulated
fills. It does not establish real queue priority, fillability or exchange fees.
The match list explains whether execution is working, holding or waiting.

Sign-in is encrypted with Windows DPAPI for the current Windows user. On restart,
the app restores the saved account, mode and allocation. Previously enabled
execution resumes only after the saved identity and mode match and account,
storage and allocation checks succeed. **Stop** clears the saved execution intent.
**Disconnect** retains the saved sign-in but disables automatic reconnection;
**Forget saved sign-in** removes it. A new source checkout has no account or
enabled trading preference.

Account actions show queued, running, succeeded or failed status. A local
connection interruption retains the last displayed account state and retries.
An uncertain account check delays new orders while verification retries; the
displayed reason identifies the unresolved step.

### Allocation and current sport rules

Confirmed allocation **A** may be up to **10 times account value** for strategy
sizing: an account value of $100 permits an allocation of $1,000. This does not
create extra cash or borrowing capacity; actual orders remain limited by available
cash after reservations and existing commitments.

Automatic starting sport targets are soccer 35%, NHL 30%, NBA 20% and MLB 15%.
The displayed dollar amounts are editable directly, with a confirmation button.
Confirmed custom sport budgets become the limits for new matches and must total
no more than A. Existing match budgets remain locked when allocation changes.

Here `p` is the selected sports-model probability, `ask` is the economic price of
the selected outcome, and `c` is that sport's editable dollars-per-point value
(initially $5 for soccer and NHL).

| Sport | Entry timing | Current qualification | Requested match budget before cash/sport limits |
| --- | --- | --- | --- |
| NBA | From T-3h until start | Selected forecast winner; the current execution rule has no probability-discount filter | 3% of A below $10,000; 2% of A at or above $10,000 |
| MLB | Pregame, as soon as confirmed inputs are ready | Selected winner's ask <= p - 0.05 | 3% of A below $10,000; 2% of A at or above $10,000 |
| Soccer | From T-12h until start | p > 51%, confidence > 55%, selected outcome's YES contract | c × max(100p - 51, 0) |
| NHL | From T-12h until start | p > 60%, ask > $0.50; first two matches per UTC day in each floor((p - ask) / 0.04) band | c × max(100p - 51, 0) |

The NBA T-3h order-entry window is separate from its unchanged T-12h model cutoff.
MLB has no additional fixed 12-hour gate in the execution engine; confirmed-model
readiness still applies. Historical backtest sizing and entry settings are not
the current live execution rules. The current rules are implemented in
`sports_app/trading_rules.py` and `sports_app/trading_engine.py`.

### Limit orders and reconciliation

The engine uses **limit orders (LMT)**. Subject to exchange increments, minimum
size and available cash, it divides remaining locked match dollars between a
midpoint limit and an ask-price limit. Working orders are reviewed on the normal
five-minute refresh cycle, with cancellation confirmed before replacement.
Pregame ordering ends at the scheduled start; filled positions are held through
settlement. Manual or externally managed positions are protected from automatic
changes.

For a NO purchase, the connector converts the Polymarket US complementary-YES
quote convention to the economic NO bid, ask and order price before qualification,
midpoint and sizing calculations. YES-side quotes are not used as NO purchase
prices without this conversion.

An uncertain submission response triggers checks of open orders, recent fills,
position, available balance and relevant market/order state. The engine adopts
a verified existing order, reconciles fills, or recomputes the remaining action.
It does not blindly resubmit an ambiguous attempt or permanently stop solely
because a submission response was uncertain.

### Account values and execution costs

| Field | Meaning |
| --- | --- |
| Estimated portfolio value | Cash and marked positions with complementary/short obligations accounted for; the estimate can differ from another screen's prices or observation time |
| Available cash after reservations | Exchange-reported cash currently available after exchange reservations |
| Confirmed allocation A | The confirmed strategy sizing budget |
| Capital invested | Purchase principal of confirmed app fills on unsettled matches, before fees |
| Fees paid (net rebates) | Verified exchange commissions, keeping rebates negative |
| Spread/slippage (estimate) | Filled purchase principal minus filled quantity times the economic midpoint recorded before each order |
| Execution cost (estimate) | Net fees plus the spread/slippage estimate |
| App unfilled order reserves | Principal reserved for the unfilled remainder of active or unresolved app orders |
| Available bot capital | Uncommitted capacity for new matches, limited by allocation, sport budgets, existing commitments and available cash |
| Manual/external cost basis | Exchange-reported cost basis of positions treated as manual or externally managed; it is separate from app execution expenses |

The account-level app cost totals cover orders on unsettled matches. Invested
principal is separate from execution expenses. Spread/slippage is already embedded
in the purchase price and is **not an additional cash charge**. Negative estimates
can represent price improvement. Missing verified fees or pre-order midpoint
evidence display as unavailable rather than zero.

An unfilled order is a reservation, not a paid execution expense. A canceled
remainder releases its reservation; any earlier partial fill and its actual fees
remain recorded. Locked capital includes the match budget already committed to
filled contracts, working orders and intended future fills.

### Validation

The October 6 application revision passed 147 regression tests plus browser and
live account-reconciliation checks. Packaging also verified the retained folder
layout, source hashes, storage initialization and NBA template reproduction.
These checks do not claim new model backtest results or guarantee future fills.

Run the packaged execution/accounting checks with simulated exchanges:

```powershell
& .\final_models\.venv\Scripts\python.exe -B .\tools\run_execution_checks.py
```

Data-dependent tests and `sports_app/verify_final_models.py` require the retained
official history and model assets. The source archive's `FILE_MANIFEST.json`
records per-file SHA-256 values; the adjacent ZIP checksum verifies the archive.

## Copy Trading App

**Command directory: `Copy Trading App`.**

A local Polymarket International copy-trading app and rolling research workflow.
It screens the observed macro/geopolitical books of other accounts, allocates to
every qualifying account using inverse volatility, and follows changes in their
relative macro-position values. Sandbox fills are assumed at the current
bid/ask midpoint; real trading uses midpoint limit orders and confirmed fills.
This implementation connects to Polymarket International, not Polymarket US.

### Install and prepare

Use Windows, Python **3.14**, an internet connection, and a current browser.
Python 3.14 is needed by collectors that use `Executor.map(buffersize=...)`.
PDF generation also uses the standard Windows Arial and Arial Bold fonts at
`C:/Windows/Fonts/arial.ttf` and `arialbd.ttf`.

Extract into a writable directory. Open PowerShell in `Copy Trading App`:

```powershell
py -3.14 -m venv .venv
& .\.venv\Scripts\python.exe -m pip install -r requirements.txt
& .\.venv\Scripts\python.exe prepare_project.py
& .\.venv\Scripts\python.exe data_retrieval\prepare_history.py init
```

`prepare_project.py` assembles the source into `data_storage/runtime`, preserving
the original sibling-module imports and the app's relative research-data paths.
It does not modify trading logic. Run it before startup, not while that generated
instance is running. Re-running it updates source copies and retains stored data.

**Historical data preparation is required before the current app can start.**
Its configuration selector, replay, and history reuse read prepared research
outputs. Empty storage is not a completed simulation. Follow
[Data retrieval](<Copy Trading App/data_retrieval/README.md>) in order: discover markets, collect
their trade histories, prefilter activity/frequency, complete candidate histories,
collect inventory actions and historical prices, reconstruct books, then simulate.
Use [Final simulation](<Copy Trading App/final_simulation/README.md>) for the screening formulas,
parameter grid, report commands, and data-dependent checks.

Preparation can be substantial: the research uses a year of warm-up and a year
of replay. Public API coverage, retention, rate limits and available historical
prices determine what can be reconstructed. New downloads need not reproduce
the original market universe or exact published numbers.

```powershell
& .\.venv\Scripts\python.exe prepare_project.py --check-ready
```

This read-only check must pass before startup. Do not create fictitious account
histories, selected-account files or equity curves to satisfy it.

### Start and stop

After preparation, double-click **Start Copy Trading App.cmd**, or run:

```powershell
& .\.venv\Scripts\python.exe data_storage\runtime\polymarket_mock_app\server.py --port 8790 --open
```

Open http://127.0.0.1:8790. Keep the server and computer running for forward
monitoring. The root launcher keeps a server console open; press **Ctrl+C** there
to stop it. Pause copying first, and for real trading let outstanding order
cancellations/fills reconcile before stopping. Closing only the browser does not
stop the server. If port 8790 is occupied, use a different `--port` and its matching
browser URL rather than starting a second process against the same ledger.

The retained app-level `launch.ps1` is a background launcher. Use it only in
the generated runtime app directory, with the intended Python environment on
PATH. It writes `data/server.pid`; closing its app window leaves its server
running. The root launcher above is the simpler foreground startup/shutdown path.

### Use the app

1. **Strategy settings:** create a named sandbox or real-money portfolio, choose
   a prepared configuration, and enter its capital budget. Portfolios have
   separate ledgers; the portfolio selector changes the view, not the mode of an
   existing portfolio and not whether another portfolio keeps running.
2. **Start copying:** discover current macro/geopolitical participants and screen
   their preceding history. Allocation starts when a complete selection snapshot
   is ready. The historical followed set is not automatically selected today.
3. **Portfolio:** view NAV, compounded return, daily strategy Sharpe, allocation,
   current status, next reset and the execution journal.
4. **Followed accounts:** inspect every selected wallet/name, historical metrics,
   inverse-volatility weight, reset budget, invested value, cash and current
   contribution. Account detail shows portfolio performance and, when separately
   prepared, the account's historical macro-book chart.
5. **Positions held:** view current shares and market values, largest first, and
   filter by followed account. **Transaction history** links our fills to source
   accounts and position-change decisions; exports provide fills, selections and
   NAV. An observed source trade is attribution evidence, not a promise of a
   one-for-one copied execution.
6. **Discovery & screening:** monitor collection, activity checks, reconstruction,
   qualification and unresolved history. Pause/resume discovery here.
7. **Execution:** inspect order states; connect, enable or disable real routing;
   reconcile an uncertain submission using its actual exchange order ID.
8. **Historical replay:** explore the prepared configuration's research curve
   and account sets. This is separate from current sandbox/live performance.

**Pause copying** retains holdings and continues marking them; it cancels resting
limits and reconciles already submitted real orders. Manual account exclusion in
sandbox requests an exit/reallocation and persists for future selections in that
portfolio. On restart, sandbox restores its saved holdings and catches up to the
latest observed source book. It does not backdate executions during downtime.
Real portfolios restart paused, with credentials disconnected.

### Account selection

The default defined in the active code is **Effective Sharpe in (3, 4], a 75-day
account reset, and a 365-day maximum volatility window**. Other prepared cases
have different thresholds, ranges, reset periods and volatility windows; the
selected configuration controls them. There is no top-N selection or return rank.

At each reset, discovery uses market participants from the classified macro/geo
catalog, including newly discovered markets. The classifier covers economic
indicators, central banks, macro assets, geopolitical events, national government,
elections and policy. It uses category metadata plus text rules and explicit
subject exclusions; it is not a perfect human taxonomy.

The current activity/style rules are:

- At least **30 distinct active UTC macro/geo trading days in the preceding 40**.
  An active day contains a macro/geo BUY or SELL.
- Average **strictly fewer than 50 macro/geo trades per calendar day**, over
  available preceding history, capped at 365 days. Inactive calendar days remain
  in the denominator. Trade fills are grouped using transaction/token/side/time.
- At most **10% of closed shares held less than one hour**, at most **20% of
  token/hour groups with both BUY and SELL**, and at most **20% of market/hour
  groups involving both outcome tokens**, using the preceding-year window.
- The chosen Raw or Effective Sharpe cutoff/range must pass, and the lagged
  allocation volatility must be positive and computable.

Activity/frequency gates precede expensive portfolio reconstruction, with exact
checks again on reconstructed inputs. No macro-participation percentage, median
holding-time band, minimum completed-market count, P&L ranking or minimum selected
account count is applied. Missing/reconciling histories are identified separately.

Screening returns describe **the trader's active macro/geopolitical book,
conditional on being invested**, not their whole-account returns. Earlier marked
position values weight subsequent price returns. Absolute book-size increases
are not gains, and no unobservable category cash is inferred. Unpriced positions
enter the return book only when historically observed prices become available.
Available invested observations within the preceding 365 calendar days determine
the sample; a full year is not fabricated. See the formulas in
[Final simulation](<Copy Trading App/final_simulation/README.md>).

### Allocate between accounts

Every qualifying account receives a capital weight proportional to `1 / sigma`,
normalized across that selected set. `sigma` is the sample standard deviation
of its macro-book daily returns, using the configuration's lagged window. The
return ending at the reset boundary is omitted from allocation volatility.

For example, volatilities of 1% and 2% produce account weights of 66.67% and 33.33%.
At reset, those weights multiply **current portfolio NAV**, including prior gains
and losses. This compounds; it is not a repeated fixed-dollar investment.
Between account resets, slices gain/lose value and their portfolio percentages
drift. One trader changing positions does not trigger a transfer of another
trader's allocation. Cross-account target budgets are recalculated at resets or
an explicit manual removal/reallocation.

### Copy positions, purchases, sales and cash

The app follows the **relative marked values of the source's macro positions**,
including positions already open when the source is first selected. It does not
observe or reproduce that trader's total-account cash percentage.

Within a followed account's slice, the target cash value for a position is the
slice's current equity multiplied by the source position's share of the macro
book; target shares divide that value by the observed price. The executor trades
the difference between target and already-held shares, selling reductions before
funding purchases. Each slice is limited to its own available cash and shares.

- **A purchase or addition** changes relative source weights and may require
  buying that position and reducing another position within the same slice.
- **A partial sale** reduces that source position's relative weight. Because the
  remaining book is normalized again, proceeds may be redistributed to its other
  positions. A SELL does not necessarily leave permanent cash.
- **A full exit from one market** targets zero shares there and may increase the
  remaining market weights. If the source exits the entire macro book, its slice
  stays in cash until a later observed position change.
- **Absolute scaling without changing relative weights** need not generate any
  trade. The app copies allocation proportions, not absolute source share counts.
- **Cash also remains** when quotes are unavailable, purchases lack slice cash,
  real limits are unfilled, or resolved/redeemable source value reserves cash.
  The target denominator includes resolved source value, but resolved targets are
  not newly purchased.
- **At account reselection**, continuing accounts keep their existing shares
  subject to the new targets; removed accounts are targeted flat and new accounts
  receive their selected allocations. Multi-order execution is not instantaneous
  or atomic in real mode.

Example: an allocated trader initially has equal-value B and C positions and then
sells C, leaving only B. The app targets the available slice into B, so it can sell
C and buy more B. It does not reproduce an unobservable source cash balance.

### Position monitoring versus account resets

Source positions/activity refresh continuously, targeting **five seconds after
each completed account check**; the portfolio worker also wakes approximately
every five seconds. API latency and indexing delays make actual intervals longer.
Public order books update independently through the WebSocket and REST checks.
An observed inventory change triggers new targets; price movement alone does not
continually rebalance unchanged source quantities.

Account reselection uses the chosen **reset_days** (75 by default). The first
screen uses the current UTC date boundary and available preceding history; later
resets repeat discovery, filters and inverse-volatility weights. A late screen
executes at then-current prices. The UI displays Chicago time, while historical
screening boundaries and stored epoch timestamps use UTC.

### Sandbox, real execution and historical simulation

| Mode | Position adjustment | Fill/return assumption |
| --- | --- | --- |
| Sandbox app | Observed source-book changes between resets | Immediate BUY/SELL at exact `(bid + ask) / 2`; no depth, trade-print, queue, fee or venue-minimum requirement; valid quotes and slice cash/shares still required |
| Real app | Same source targets and account allocations | Post-only GTC midpoint limits for both sides; BUY rounds down and SELL up to the valid tick; check pending limits every 60 seconds and replace an unfilled remainder only after cancellation; confirmed exchange fills only |
| Final historical research | Daily macro-book return changes; account weights reset by configuration | Compound each selected account's daily modeled book return; zero costs; no live order queue or limit-fill simulation |

Real execution respects venue minimum quantities/ticks, available collateral,
share balances and reservations. Partial fills remain partial; unknown submissions
block automatic resubmission until reconciled. The code records confirmed maker/
taker fills and accounts for its implemented exchange-fee calculation. It does
not assume real midpoint limits will fill. Sandbox has no spread cost at entry
by construction and does not estimate executable market impact.

Real fills can leave cash or incomplete target adjustments; historical returns
therefore are not a live execution forecast. Research is daily, whereas the app
responds to observed inventory changes throughout the day. The saved replay is
not recalculated from the app's forward fills.

Source settlement values enter historical marks only after their recorded
availability. Sandbox credits observed resolved payouts. Real holdings require
the user to redeem through Polymarket; the app verifies token extinction and a
matching collateral change before treating proceeds as spendable cash.

### Real-money credentials

Creating a real portfolio requires the existing **trading wallet address, signing
private key, CLOB API key, API secret and API passphrase**. The app checks funding
and permissions before creating it. Use the local form, then explicitly enable
real routing and start copying. Merely connecting does not submit orders.

The helper `data_storage/runtime/polymarket_mock_app/generate_clob_credentials.py`
prompts for a signing key and creates/derives international CLOB credentials. It
does not place an order. Keep its displayed secrets private. The application's
credentials remain in server memory, not its persistent ledger. `.env.example`
is a reference template only; the current app does **not** automatically load it.

Use an account dedicated to this strategy: unrelated orders and external balance
changes require reconciliation. This package's offline checks do not validate a
funded wallet, approvals, venue access or actual authenticated order execution.

### Results and validation

[Result notes](<Copy Trading App/final_simulation/RESULTS.md>) summarize the existing completed
research and its limits. They are not a new historical replay of freshly downloaded
data. [Validation](<Copy Trading App/VALIDATION.md>) distinguishes source/accounting checks from
historical-data and authenticated checks.

```powershell
& .\.venv\Scripts\python.exe prepare_project.py --syntax
& .\.venv\Scripts\python.exe run_checks.py
```

Run these after assembling the runtime. The runner omits three explicitly listed
historical-data tests; the remaining checks use local fixtures and do
not submit real orders. Do not run the separate public-feed checker or discovery
when expecting an offline test. Research/account return moments use invested days;
strategy Sharpe includes calendar cash days. Forward app Sharpe uses completed
24-hour periods since funding and is unavailable until there is enough valid
variation/history; total return includes current marked NAV.
