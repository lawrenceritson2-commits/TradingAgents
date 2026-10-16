# CLAUDE.md

Fork of TauricResearch/TradingAgents (v0.6.0, Apache-2.0). Multi-agent LLM
trading research framework on LangGraph. Research/paper-trading only.

## Hard rules
- Paper trading and backtesting only. Never add live order execution or code
  that touches broker credentials.
- Never hardcode, print, or log API keys. Keys come from env vars / `.env`
  (gitignored). Don't commit `.env*` files other than the `.example` ones.
- Keep fork changes modular so upstream stays mergeable: put new code in new
  modules (`tradingagents/learning/`, `tests/learning/`); edit upstream files
  only for minimal hook points, and list each such edit in `FORK_NOTES.md`.
- Preserve Apache-2.0 notices; mark modified upstream files (see FORK_NOTES.md).

## Commands
- Install: `pip install -e ".[dev]"` (Python >=3.11)
- Tests: `pytest -q` (integration tests excluded by default; `-m integration` opts in)
- Lint: `ruff check .` (select E,W,F,I,B,UP,C4,SIM; E501 ignored; no repo-wide `ruff format`)
- CLI: `tradingagents` (interactive), `tradingagents backtest T1,T2 --start --end --every`
- Library: `TradingAgentsGraph(config=...).propagate(ticker, "YYYY-MM-DD")` -> (state, rating)

## Architecture
- `tradingagents/graph/`: LangGraph wiring. `setup.py` builds the graph:
  4 analysts + Memory Log node in parallel -> Bull/Bear debate -> Research
  Manager -> Trader -> Aggressive/Conservative/Neutral risk debate -> Portfolio
  Manager. `trading_graph.py` is the entry point (propagate, checkpointing,
  record_decision, settle_*).
- `tradingagents/agents/`: agent factories (`create_*`), `state.py` (AgentState),
  `schemas.py` (pydantic structured outputs), `rating.py` (5-tier parser),
  `tools.py` (analyst tools; `trade_date` injected from state and clamped).
- `tradingagents/dataflows/`: only layer allowed to import vendor libs (enforced
  by tests/test_layering.py). `router.py` maps tool -> vendor via config.
  `date_window.py` holds the point-in-time rules.
- `tradingagents/memory/`: existing markdown memory log, settlement (returns vs
  benchmark after `holding_period_days`), Reflector. Only the Portfolio Manager
  reads `past_context`.
- `tradingagents/backtest.py`: grid evaluation of decision quality, NOT a
  portfolio simulator by design.
- `tradingagents/llm_clients/`: provider factory; model catalog in `model_catalog.py`.
- `default_config.py`: all config; `TRADINGAGENTS_*` env vars override (see `_ENV_OVERRIDES`).
  Fork default: `max_debate_rounds` = 3 (upstream: 1).

## Conventions
- Point-in-time integrity is a core invariant: any dated path must never serve
  data after the run's `trade_date`. Add a lookahead test for any new data path.
- Ratings: Buy/Overweight/Hold/Underweight/Sell, or `REVIEW` when unparseable;
  never coerce REVIEW to Hold.
- LLM output is nondeterministic; tests must mock LLMs and network. Use fixtures
  in tests/conftest.py. Tests are marked unit/integration/smoke.
- CI runs py3.11-3.14 plus a New York TZ run: avoid UTC-only date logic.
- Line length 100. Match surrounding style; reference issues like `(#1251)` in comments only when upstream does.

## Gotchas
- Memory log is keyed by (ticker, date); store_decision is idempotent.
- Backtests write to `results_dir/backtest/<run_id>/`, never the live memory log.
- Settlement entry price = first close on/after trade date (optimistic fill).
- News/social feeds are not archived; historical news/sentiment is often "unavailable".
