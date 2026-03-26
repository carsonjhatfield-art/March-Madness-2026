# March Madness Analysis — Quick Start Guide

## Project Overview
Analysis of the **BUSTin Bracketz 2026** ESPN Tournament Challenge group.

- **Group URL**: https://fantasy.espn.com/games/tournament-challenge-bracket-2026/group?id=e4a3cda3-7dc9-4b35-a1d5-d3b3196fb502
- **Group Size**: 21 brackets
- **Event Dates**: 03/19/2026 – 04/06/2026
- **Group Password**: Buster

## File Structure

```
March Madness Analysis/
├── QUICK_START_GUIDE.md              ← You are here
├── BUSTin_Bracketz_Elimination_Analysis.xlsx  ← Main analysis workbook (4 sheets)
├── update_game_overview.py           ← Regeneration script (updates index.html with game results)
├── march-madness-2026/
│   └── index.html                    ← Live visualization — two tabs: Bracket View + Game Overview
└── data/
    ├── group_standings_2026-03-26.csv         ← Raw scraped standings data
    ├── bracket_picks_2026-03-26.csv           ← All 21 brackets' S16/E8/F4 picks
    ├── tournament_metadata_2026-03-26.json    ← S16 matchup slots, scoring, team mappings
    ├── permutation_summary_2026-03-26.json    ← Win scenario counts per bracket
    ├── permutation_results_2026-03-26.csv     ← Same as above in CSV format
    ├── winning_paths_2026-03-26.json          ← Full winning path details per bracket (for visualization)
    └── tiebreaker_scores_2026-03-26.csv       ← ESPN tiebreaker predictions (total championship game pts) per bracket
```

## Key Files

| File | Description |
|------|-------------|
| `BUSTin_Bracketz_Elimination_Analysis.xlsx` | Four-sheet workbook: Group Standings, Permutation Analysis, Elimination Summary, Key Stats |
| `update_game_overview.py` | Regeneration script — pass winning team abbreviations as args to lock in results and regenerate `index.html` with updated percentages for both tabs. Usage: `python3 update_game_overview.py TEX IOWA ARIZ` |
| `march-madness-2026/index.html` | Combined interactive visualization with two tabs: **Bracket View** (per-bracket path to victory with S16 elimination analysis) and **Game Overview** (per-game matchup showing which brackets prefer each outcome). Deployed via GitHub Pages. |
| `data/group_standings_2026-03-26.csv` | Raw CSV of group standings scraped from ESPN (for reuse without re-scraping) |
| `data/bracket_picks_2026-03-26.csv` | All 21 brackets' picks for S16 (8 slots), E8 (4 slots), F4 (2 slots) + champion pick, current PTS, elimination status |
| `data/tournament_metadata_2026-03-26.json` | Sweet 16 matchup slot mapping, scoring rules (40/80/160/320), and team abbreviations |
| `data/permutation_summary_2026-03-26.json` | Summary stats: outright wins, tiebreak wins, total scenarios, win % for each alive bracket |
| `data/winning_paths_2026-03-26.json` | Detailed winning paths for each bracket — every permutation of tournament outcomes that leads to a group win, with game-by-game results and scores. Primary data source for the HTML visualization. |
| `data/tiebreaker_scores_2026-03-26.csv` | ESPN tiebreaker predictions (total championship game pts) per bracket |

## Analysis Results Summary (as of 2026-03-26)

- **5 brackets eliminated** by ESPN Analytics (0% win probability)
- **3 additional brackets** have zero winning scenarios per permutation analysis (Choice, Gabys Picks, tyoung2848)
- **1 bracket** (Niclad) can only win via tiebreaker — never has the sole lead
- **Top contenders**: AndrewMCorral (21.3%), Sampsonites (13.1%), Mick is the Pick (12.8%), Mobleys (11.8%), Eleff18 (10.9%)
- **Carson's bracket**: 512 winning scenarios (1.56% outright win rate)

## Scoring System
ESPN standard bracket scoring:
- Round of 64: 10 pts | Round of 32: 20 pts | Sweet 16: 40 pts
- Elite 8: 80 pts | Final Four: 160 pts | Championship: 320 pts

## Permutation Engine Details
- 15 remaining games (8 S16 + 4 E8 + 2 F4 + 1 Championship) = 2^15 = 32,768 possible tournament outcomes
- Each outcome is scored against all 16 alive brackets
- Winner = highest total score (current PTS + points earned from correct remaining picks)
- Tiebreaker scenarios flagged separately (ESPN uses a tiebreaker score, not modeled here)

## HTML Visualization — Data Pipeline

The interactive visualization (`march-madness-2026/index.html`) is a self-contained HTML file with two tabs — **Bracket View** and **Game Overview** — with all data embedded directly in JavaScript. No external dependencies or API calls. Deployed via GitHub Pages from the `march-madness-2026/` directory (which is its own git repo). Both tabs are generated together by `update_game_overview.py`. Here's how data flows from source to visualization:

**Source data files → HTML generation:**

1. **`data/winning_paths_2026-03-26.json`** — Primary source. Contains every permutation of remaining tournament outcomes that results in each bracket winning the group. Each path includes S16/E8/F4/Championship winners, the bracket's total score under that outcome, and whether it's an outright or tiebreak win. The HTML aggregates these paths to compute per-game team win percentages (e.g., "Arizona wins West S16 in 100% of Carson's winning scenarios").

2. **`data/permutation_summary_2026-03-26.json`** — Supplies the stats bar values: outright wins, tiebreak wins, total winning scenarios, and win percentage for each bracket. Also includes owner name, champion pick, current PTS, and tiebreaker score.

3. **`data/bracket_picks_2026-03-26.csv`** — Used to mark which team in each game node is the bracket's actual pick (shown with a blue star in the UI).

4. **`data/tournament_metadata_2026-03-26.json`** — Provides the bracket structure: which teams play in each S16 slot, how winners feed into E8/F4/Championship matchups, and team abbreviation mappings.

**How the HTML is built:**

`update_game_overview.py` reads all four data files above, accepts optional game results as command-line arguments, then for each non-eliminated bracket:
- Filters winning paths to only those consistent with locked-in results
- Computes the percentage of remaining winning scenarios where each team wins each game
- Identifies "locked" requirements (100% of paths) vs "flexible" outcomes
- Determines S16 elimination risks (which game outcomes would reduce a bracket to 0 winning paths)
- For the Game Overview tab, categorizes bracket owners by which team they prefer in each matchup
- Embeds all aggregated stats as JavaScript objects directly in the HTML

**To regenerate or update the visualization after a game result:**
```
python3 update_game_overview.py TEX                     # After one game
python3 update_game_overview.py TEX IOWA ARIZ HOU       # After four games
python3 update_game_overview.py                          # Reset to pre-results state
```
- Pass the WINNING team abbreviation for each completed S16 game (order doesn't matter)
- Valid teams: DUKE, SJU, MSU, CONN, IOWA, NEB, ILL, HOU, ARIZ, ARK, TEX, PUR, MICH, ALA, TENN, ISU
- Both tabs (Bracket View + Game Overview) are regenerated together in one run
- Output goes to `march-madness-2026/index.html` — this is the only file to update
- Push changes to the `march-madness-2026` GitHub repo to redeploy via GitHub Pages
- The HTML file is fully self-contained — also works locally in any browser

## Project Rules (from user)
- Always ask clarifying questions before executing
- Provide a plan summary before executing
- Save all scraped data in organized file structure to avoid re-scraping
- Maintain this quick start guide as documents are created
