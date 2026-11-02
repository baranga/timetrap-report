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

| Flag                 | Description |
| -------------------- | ----------- |
| `-s, --sheet SHEET`  | Timetrap sheet to report on. Defaults to `$TIMETRAP_SHEET`, then to the contents of a `.timetrap-sheet` file found in the current or a parent folder. |
| `--start DATE`       | Start date (default: first day of last month). Can also be set via `$DATE_START`. |
| `--end DATE`         | End date (default: last day of last month). Can also be set via `$DATE_END`. |
| `--last-month`       | Report on last month. (default) |
| `--this-month`       | Report on this month. |
| `-m, --month MONTH`  | Report on the given month, as `YYYY-MM`. |
| `--last-week`        | Report on last week (Monday-Sunday). |
| `--this-week`        | Report on this week (Monday-Sunday). |
| `--yesterday`        | Report on yesterday. |
| `-h, --help`         | Show the help message and exit. |

The time frame flags (`--last-month`, `--this-month`, `--month`,
`--last-week`, `--this-week`, `--yesterday`) each set both `--start` and
`--end`. Flags are processed in order, so a `--start`/`--end` given after one
of them overrides just that end of the range, e.g.
`--this-week --end 2026-09-02`.

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
$ ./timetrap-report --sheet acme --last-month
generate report 2026-08-01 - 2026-08-31 for sheet acme
date,duration,notes
"2026-08-05","7.25","ACME-123: try to reproduce bug, ACME-456: fixing type issue"
"2026-08-06","2.66","ACME-123: fix issue, ACME-789: review PR"
"2026-08-08","6.5","ACME-123: review on staging, ACME-456: review on staging, Release of version 1.2.3"
```

Redirect stdout to save the CSV, since the progress line above is written to
stderr:

```
./timetrap-report --sheet acme --last-month > 2026-08.csv
```

## License

GPL-3.0-or-later. See [LICENSE](LICENSE) for the full text.
