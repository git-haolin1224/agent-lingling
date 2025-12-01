### Problem Statement 

China concept stocks listed on the NYSE and Nasdaq (BABA, PDD, JD, etc.) are exposed to far more than company fundamentals. Their prices swing with:

- U.S.–China foreign policy (tariffs, export controls, sanctions, delisting risks)

- Global liquidity and risk appetite (rates, DXY, VIX, EM flows)

- China’s domestic economy (growth, property crisis, consumption, tech regulation)

- Real-time social sentiment on platforms like X, where a single post from a high-profile politician can move markets

Retail investors have almost no chance of tracking all of this in real time. They see price volatility but not the causal story behind it. As a result, they often react to noise, miss key policy developments, or underestimate structural risks such as delisting or VIE structures.

The problem I’m solving is: how to give retail investors a structured, multi-angle explanation of what’s driving a China concept stock, including policy risk, macro/liquidity, fundamentals, technicals, and X/Twitter sentiment, without turning it into a black-box “magic signal” or trading bot.

### Why agents?

This problem is naturally multi-disciplinary:

- A policy analyst mindset for U.S.–China relations

- A macro & liquidity perspective

- A fundamental analyst for company metrics

- A technician for price action and volatility

- A social media analyst for X sentiment and narrative shifts

Trying to bake all of this into a single monolithic prompt or model makes it messy and fragile. Each “expert” needs different tools, context, and instructions.

Agents are a good fit because they let me:

- Decompose the task into specialist sub-agents (policy, macro, liquidity, fundamentals, technicals, X sentiment, risk/scenarios, report writer).

- Attach the right tools to the right agent (e.g., X API only for the sentiment agent, macro API for the macro agent, price history for technicals).

- Orchestrate them with a supervisor agent/function that decides the sequence, aggregates outputs, and enforces a stable report structure.

Google ADK provides a nice way to define these LLM Agents, register tools, and run them with a Runner and session service, without building my own orchestration framework from scratch.

### What you created 

At a high level, my system is a Google ADK multi-agent app called china_concept_stock_agent. Given a ticker (e.g. BABA) and a horizon (e.g. 6–12 months), it produces a structured report for retail investors.

Core components:

1. Orchestrator (Supervisor)

    -  Python function analyze_china_concept_stock(ticker, horizon_months, extra_focus)

    - Calls each sub-agent via ADK’s Runner and aggregates their outputs.

2.  Sub-agents (LlmAgents):

    - universe_data_planner — Plans which indices, ETFs, macro series, and X queries/accounts to use for the ticker.

    - geopolitics_policy_analyst — Summarizes U.S.–China policy and regulatory risks for that stock (delisting, export controls, sanctions, etc.).

    - china_macro_sector_analyst — Interprets China’s domestic macro data and sector-specific conditions.

    - global_liquidity_analyst — Assesses global risk sentiment and liquidity using rates, DXY, VIX, EM/China ETFs.

    - fundamental_valuation_analyst — Explains business model, growth, profitability, and valuation vs peers/history.

    - technical_flow_analyst — Analyzes trend regime, volatility, and relative performance (vs KWEB, FXI, SPY, etc.).

    - social_sentiment_analyst — Uses X posts to score China-wide and ticker-specific sentiment and extract key narratives (including posts from accounts like high-ranking politicians and other high-influence users).

    - risk_scenario_checker — Combines all expert views into bull/base/bear scenarios and a list of key risks and signposts.

    - china_equity_report_writer — Writes the final, human-readable report in sections for retail investors, explicitly avoiding trading recommendations.

3.  Shared tools (Python functions registered as ADK tools):

    - fetch_price_history — Price/volume history via yfinance or similar.

    - fetch_macro_series — Macro series such as US10Y, DXY, VIX, China PMI.

    - fetch_news — Policy and company news.

    - fetch_x_posts — X/Twitter posts filtered by ticker, China keywords, and a whitelist of impactful accounts.

The orchestrator passes the ticker and a “universe plan” to the agents, collects their structured JSON/text, feeds it to the risk/scenario agent, then calls the report writer to produce the final output.

### Demo 

1. A typical demo flow looks like this:

    - User input:

    - Ticker: BABA

    - Horizon: 12 months

    - Focus: “Explain volatility and policy risk.”

2.  Universe plan:

    - Plan includes: KWEB, FXI, MCHI as benchmarks, macro series like US10Y, DXY, VIX, CN_PMI, plus X queries such as "$BABA OR Alibaba", "China tariffs", "export controls", with priority accounts (key US officials, major journalists).

3. Specialist outputs:

    - Policy agent: explains HFCAA, audit access, data security concerns, and receives US comments about China tech platforms.

    - Macro agent: explains China’s weak property sector but relatively resilient online consumption.

    - Liquidity agent: interprets the current rate cycle and USD strength as mildly negative for EM / China.

    - Fundamentals/technicals: shows BABA’s revenue trend, valuation vs history, and recent downtrend vs KWEB.

    - X sentiment: surfaces a few influential posts about tariffs, TikTok/China tech, and a sentiment score that’s currently negative.

4. Final report:

    - Sections: Executive summary; US–China policy; China macro; global liquidity; fundamentals & valuation; technical picture; social sentiment; scenarios & what to watch.

    - Tone: descriptive, transparent, focused on drivers and monitoring points, not buy/sell calls.

### The Build

1. Stack & tools:

    - Google ADK for defining LlmAgents, tools, and running them via Runner + InMemorySessionService.

    - Gemini models (e.g., gemini-2.0-flash) as the underlying LLMs for all agents.

    - Python for the orchestrator and tools layer.

    - yfinance / HTTP APIs for price history and macro/news data (pluggable).

    - A placeholder fetch_x_posts tool that can later be wired to an X/Twitter API or custom scraper.

2. Development steps:

    - Wrote the problem statement and constraints: retail-friendly, no trading advice, multi-angle explanation.

    - Designed the agent decomposition (policy, macro, liquidity, fundamentals, technicals, sentiment, risk, writer), inspired by multi-agent ADK examples.

    - Implemented tools as plain Python functions, then wrapped them as ADK tools.

    - Implemented each sub-agent as an LLM Agent with focused instructions and attached tools.

    - Built a manual orchestrator function that simulates a “supervisor” by calling each LLM Agent in sequence and passing their outputs as context.

    - Tested the pipeline on a few tickers (e.g., BABA, PDD) and iterated on prompts so outputs are structured and non-overlapping.

### If I had more time, this is what I'd do

- Real X integration: Replace the placeholder fetch_x_posts with a production-grade X API backend, including engagement statistics and better author metadata.

- Structured JSON everywhere: Parse and validate JSON outputs from each agent, then use them programmatically (charts, dashboards, alerts) instead of just text.

- Continuous monitoring mode: Turn it into a “watchlist agent” that periodically re-runs agents and highlights new policy events or sentiment shifts for tracked tickers.

- Better fundamentals data: Plug into a richer financials API (or a local database) and add simple factor or risk-model analysis.

- UI & explainability: Build a small frontend where users can click into each section, expand the underlying data, and see how scores were derived.
