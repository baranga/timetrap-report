# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single bash script, `timetrap-report`, that generates a CSV time report from
[timetrap](https://github.com/samg/timetrap) sheet data. There is no build
system, package manifest, or test suite — the script is the only functional
file in the repo (alongside README.md, LICENSE, and this file).

## Dependencies

The script requires both on `PATH`:
- `timetrap` — the time tracking CLI whose `display --format json` output is
  the data source.
- `jq` — used to reshape/aggregate that JSON into CSV.

Both are checked via `require_commands` at startup and the script exits with
an error if either is missing.

## Running / manually verifying changes

There's no test suite, so validate changes by running the script directly
against real (or a scratch) timetrap data:

```
./timetrap-report --sheet <sheet> --start 2026-01-01 --end 2026-01-31
```

- With no `--sheet`, it falls back to `$TIMETRAP_SHEET`, then to the contents
  of a `.timetrap-sheet` file found by walking up from `$PWD` (see
  `find_sheet_file`).
- With no `--start`/`--end`, it defaults to the first/last day of *last
  month* relative to the system clock (`date --date="last month" ...`),
  computed once before the arg-parsing loop into `first_of_last_month`/
  `last_of_last_month` and reused by both the default and the `--last-month`
  flag.
- The other time frame flags (`--this-month`, `--last-week`, `--this-week`,
  `--yesterday`) each set `date_start`/`date_end` inline inside the
  arg-parsing loop, computed via `date` at parse time. Weeks are
  Monday-Sunday, computed from `date +%u` (ISO day-of-week) rather than GNU
  date's `"monday this week"`-style phrases, which are inconsistent about
  which week they resolve to — verify with `date -d ... +%A` before changing
  this logic. Because flags are processed left-to-right in the same loop as
  `--start`/`--end`, a `--start`/`--end` given after a time frame flag
  overrides just that end of the range.
- Progress/config info (`generate report ... for sheet ...`) goes to stderr;
  only the CSV (header `date,duration,notes` plus rows) goes to stdout — this
  split matters if you change output and want to keep it pipeable.

## Architecture: the jq pipeline

The core logic is the `JQ_FILTER` string built at the top of the script. It
transforms timetrap's JSON array of time entries into CSV rows, one per
distinct date, in four stages:

1. **Normalize** each entry to `{date, start, end, note}`, deriving `date`
   from the first 10 chars of `start` and coercing `start`/`end` into
   ISO 8601 UTC strings (`...T...Z`) that `fromdateiso8601` can parse.
2. **Compute duration** per entry in seconds (`end - start` as epoch
   seconds).
3. **Reduce/group by date** into `{date: {date, duration, notes[]}}`,
   summing durations and collecting notes for entries sharing a date.
4. **Format for CSV**: duration is converted from seconds to hours rounded
   down to the nearest 0.01 (`(duration / 36) | floor | . / 100`, i.e.
   seconds → hundredths-of-an-hour → hours), notes are deduplicated and
   joined with `, `, and each row is manually quoted/escaped into a CSV
   string.

When editing the filter, keep in mind it's embedded as a shell
double-quoted string, so `jq`'s own double quotes and backslashes are
already escaped (`\"`, `\\\"`) — test filter changes directly with `jq`
against sample JSON before re-embedding them.
