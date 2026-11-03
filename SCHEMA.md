# NFL app weekly feed

Canonical files:
- `latest.json` — current week (always the active slate)
- `week-NN.json` — per-week snapshot (kept forever in the store and on GitHub)

Written Tue / Thu / Sun / Sat (overwrite current week). After Monday grades: fill `final_score`, leave `details` intact.

## GitHub (TheBini02/nfl-desk-app, main)
On every feed write, push **both** `latest.json` and the current `week-NN.json`.
Never delete or overwrite prior `week-MM.json` archives when rolling `latest.json` to a new week.

Exact shape:

```json
{
  "week": 8,
  "generated_at": "ISO date/time",
  "games": [
    {
      "away": "CAR",
      "home": "GB",
      "kickoff": "Thu Oct 29 7:15 PM CT",
      "spread": "GB -3.5",
      "pick": "GB",
      "why": "one-line reason from Pool",
      "type": "BASE",
      "money_flag": false,
      "summary": "Writer's full per-game take (natural voice; winner + ATS + why woven in)",
      "final_score": null,
      "details": {
        "scheme": "Tape’s short reusable matchup note",
        "injuries": "starter QB/WR1 status or UNKNOWN",
        "edges": "any EPA, structure, or key mismatch notes",
        "sources": ["named sources from Tape/Scout"]
      }
    }
  ]
}
```

Rules:
- Always include `details` (UNKNOWN or [] if missing). Keep short.
- scheme = Tape; injuries = Scout QB/WR1; edges = mismatch/structure; sources = named array.
- pick/why null until Pool Saturday lock.
- money_flag true only for $ spread beaters.
- summary = Writer's full take for that game (not a compressed one-liner).

## details.tables (evidence)
- `details.tables.matchup`: `{columns: ["Matchup","Offense","Defense allowed","Edge vs avg"], rows: string[][]}` — 8 rows per game (pass, rush, 3rd down, protection for each side). EPA values signed to 3 decimals; sack rates as one-decimal percents.
- `details.tables.injuries`: `{columns: ["Team","Player","Status","Injury","Replacement drop-off"], rows: string[][]}` — Status is Out/Doubtful/Questionable; drop-off is an EPA delta or "Qualitative only".
- `details.sources`: public, named sources only (e.g. nflverse, ESPN, rbsdm.com, edraft.com). Never list internal desk/bot names; `why`, `summary`, and `scheme` must also be free of them.

## picker (backend / Grok Build only)
- Top-level `picker: {winner, ats, why}` on every game (sibling of `pick`/`why`/`summary`). `winner` = team abbrev; `ats` = side + number, ASCII hyphen (e.g. `BUF -3`); `why` = one short line, no desk/bot names.
- UI may ignore it. `pick`/`why`/`type` remain the posted DRAFT card; `summary` is the prose take (aligned with `picker`, not necessarily `pick`).
- Every Tue/Thu/Sat/Sun write of latest.json/week-NN.json must include `picker` once Picker has posted.
