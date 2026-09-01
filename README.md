# timetrap-report

Generate a CSV time report from a [timetrap](https://github.com/samg/timetrap)
sheet, grouped by date.

## Requirements

- [`timetrap`](https://github.com/samg/timetrap) — the time tracking CLI this
  script reports on.
- [`jq`](https://jqlang.org/) — used to aggregate the report data.

## Usage

```
./timetrap-report [OPTIONS]
```

Options:

| Flag              | Description |
| ----------------- | ----------- |
| `-s, --sheet SHEET` | Timetrap sheet to report on. Defaults to `$TIMETRAP_SHEET`, then to the contents of a `.timetrap-sheet` file found in the current or a parent folder. |
| `--start DATE`      | Start date (default: first day of last month). Can also be set via `$DATE_START`. |
| `--end DATE`         | End date (default: last day of last month). Can also be set via `$DATE_END`. |
| `-h, --help`         | Show the help message and exit. |

Environment variables (overridden by the corresponding flag if given):

- `TIMETRAP_SHEET` — sheet to report on.
- `DATE_START` — start date.
- `DATE_END` — end date.

## Output

Progress info is printed to stderr. A CSV is printed to stdout with the
header `date,duration,notes`, one row per date in the range:

- `date` — the day, e.g. `2026-01-15`.
- `duration` — total hours tracked that day, rounded down to the nearest
  0.01.
- `notes` — the unique notes logged that day, joined with `, `.

## Example

```
./timetrap-report --sheet work --start 2026-01-01 --end 2026-01-31 > jan.csv
```
