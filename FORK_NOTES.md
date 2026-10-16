# Fork notes

This fork is based on TauricResearch/TradingAgents v0.6.0 (Apache-2.0). Upstream
files modified here are listed below so the changes stay visible (Apache-2.0
§4b) and upstream merges stay easy. New functionality lives in new modules.

## Modified upstream files

| File | Change | Why |
|------|--------|-----|
| `tradingagents/default_config.py` | `max_debate_rounds` default 1 -> 3 | Longer bull/bear debate (raises LLM cost per run: 6 researcher turns instead of 2). |
| `tests/test_env_overrides.py` | Default-value assertions updated 1 -> 3 | Follows the config default change. |
