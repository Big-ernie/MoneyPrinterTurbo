# Day Trader Skill, Codex compatibility kit

This converter brings the original [Jake Schincariol day-trader-skill](https://github.com/Jakeschincariol/day-trader-skill) to OpenAI Codex without changing its original Python trading tools. All 11 skills are preserved and their `SKILL.md` instructions are adapted to Codex's Agent Skills conventions.

## One-command install on macOS

1. Extract this ZIP and open Terminal in the extracted folder, or double-click `install.command` on macOS. 
2. If using Terminal, run:

   ```bash
   python3 convert_day_trader.py --mode global
   ```

The converter checks out the original upstream commit (`3d0f4ef...`, dated October 20, 2026) over HTTPS with Git, then installs to `${CODEX_HOME:-$HOME/.codex}/skills/trader-*`. It does not run trading scripts or request broker credentials. If Git reports a commit mismatch after upstream updates, examine upstream changes and run with `--ref main` to explicitly opt in to the latest source.

For an existing working directory or local clone (no GitHub network download necessary):

```bash
python3 convert_day_trader.py --source /absolute/path/to/day-trader-skill --mode global
```

### Install for one Codex project instead

```bash
python3 convert_day_trader.py --mode project --project /absolute/path/to/my-project
```

This installs into your project's `.agents/skills/trader-*`. Project-local skills work only when Codex is working in that project.

### Generate a standalone Codex plugin

```bash
python3 convert_day_trader.py --mode plugin --out ./day-trader-codex
```

This writes the portable OpenAI plugin layout:

```
day-trader-codex/
  plugin.json
  README.md
  skills/
    trader-scanner/SKILL.md + scan.py
    trader-news/SKILL.md + news.py
    trader-entry/SKILL.md + entry.py
    trader-exit/SKILL.md + exit.py
    trader-timemachine/SKILL.md + timemachine.py
    trader-budget/SKILL.md + size.py
    trader-risk/SKILL.md + check.py
    trader-broker/SKILL.md + ticket.py
    trader-alerts/SKILL.md + alerts.py
    trader-analyst/SKILL.md + facts.py
    trader-pelosi/SKILL.md + ptr.py
    UPSTREAM_LICENSE.txt
```

To use the generated plugin in Codex, add it to your local marketplace or move the eleven subdirectories into `${CODEX_HOME:-$HOME/.codex}/skills/`. The converter does not install marketplace metadata automatically or publish any plugin.

## Verify

After installing, restart Codex so it discovers the new skill metadata. In a Codex conversation, ask: **"Use $trader-scanner to scan a small watchlist of US stocks"**. The skills invoke Python only when requested and when a shell environment permits it. Some data tools need internet access and may be rate limited.

To run the original repository's offline unit tests before conversion, add `--verify-upstream`. That is an explicit opt-in because it runs downloaded third-party Python code. This kit's own adapter tests can be run using `python3 -m unittest discover -s tests -v` in the extracted folder.

## Trading safety and limitations

- **Paper and analysis only** by default. The converter does not connect a broker or enable live orders.
- The original `ticket.py` creates/records local ticket state, it does not execute orders. An MCP broker integration must be separately set up, verified as a paper account and subject to approval on every state-changing call.
- Exact orders must pass `trader-risk`, be freshly confirmed in the current conversation and then receive separate per-call tool approval. Without that approval mechanism, the Codex broker adapter stops at the proposed ticket.
- Nothing in this kit runs background market monitoring by itself. External scheduling is a separate choice, never a means to place unattended orders.
- Most day traders lose money. A simulation is not a forecast. Market data may be delayed or rate limited, and stop-loss execution prices are not guaranteed.
- The upstream toolkit targets US stocks and needs additional engineering for XAUUSD, MetaTrader 5 or other gold-trading instruments. It does not automatically integrate with Gold Quant AI.

## Source, license and compatibility

Original project: https://github.com/Jakeschincariol/day-trader-skill, original license: MIT. The converter retains a copy of the MIT license in generated plugin bundles, preserves original source scripts unmodified, and keeps original broker instructions as a historical reference. The Codex adaptation is not endorsed by the original author.

Codex skill layout is based on OpenAI's Agent Skills docs and local project `.agents/skills` structure. Python 3.8 or newer is needed for the original tools; the converter itself also targets Python 3.8+. Git and network access are needed for downloading from GitHub unless a local clone is provided.

## Test results and limitations (November 2, 2026)

The compatibility adapter was exercised using a harmless eleven-skill fixture:
`python3 -m unittest discover -s tests -v`. The 11 passing tests cover global, project,
and plugin installations, metadata and Python-script preservation, path
rewriting, isolated smoke invocation, avoiding execution at install time,
no-overwrite and `--force` safety, input validation, and rollback behavior.
An initial test found two failures: invalid upstream content could leave a
partial install and a failed `--force` attempt could erase existing skills.
Both were fixed by staging/validating all skills before swapping them into
place, then retested successfully.

**Not yet tested:** The actual GitHub repository's Python unit tests, a real
Codex CLI skill-discovery session, live market-data access, or brokerage
integration. The testing environment cannot resolve GitHub for command-line
cloning and does not have Codex CLI installed. These remain necessary checks
on a network-connected machine with Codex. The converter itself never places
orders; keep all execution on paper and require per-call approvals.

After installing with a network connection and Codex available, run:

```bash
# Explicitly execute the upstream offline test suite before conversion.
python3 convert_day_trader.py --mode project --project /path/to/project --verify-upstream
# Open the project in Codex and ask it to use $trader-scanner on a small watchlist.
```