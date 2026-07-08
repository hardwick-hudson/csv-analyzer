# csv-analyzer

**[Live demo](https://hardwick-hudson.github.io/csv-analyzer/)** - Drop a CSV on it; analysis happens in your browser.

A browser-based CSV analyzer: drop in a file, get a plain-English summary,
per-column statistics, and flags for common data-quality problems.

**Your data stays in your browser by default.** Parsing, statistics, and anomaly
detection all run client-side. There's no upload, no server, and no account
required in the static demo. The planned AI summary feature will be opt-in and
explicit: it will send column statistics, anomaly flags and at most 
five sample rows.

## What it does

- **Summary** - row/column counts and a plain-English description of each
  column, generated deterministically from the statistics.
- **Per-column stats** - inferred type, blank count, distinct count, and
  min/max/mean for numeric columns.
- **Anomaly flags** - non-numeric values in mostly numeric columns, statistical
  outliers (IQR method), mostly-blank columns, constant columns, duplicate
  rows, and parser-level errors.
- **Optional AI assessment** - the AI summary button is present in the static
  demo, but it currently shows an unavailable-state message instead of running
  the assessment. The planned Claude-hosted version will send a compact data
  profile (column statistics, anomaly flags, and at most five sample rows),
  and return an interpretive summary. 

## Design notes

- **Type inference by voting.** CSV has no schema, so a column is inferred
  numeric when at least 80% of its non-blank values parse as numbers. The
  threshold is deliberately below 100% so a stray `abc` in an age column
  should be flagged as an anomaly, not flip the column to text.
- **Outlier detection is quiet on small samples.** The IQR method needs
  enough data to actually mean anything, so it only runs on columns with 8+
  numeric values rather than producing confident nonsense on 4.
- **Sample rows are randomly selected with a fixed cap.** The planned AI feature
  sends at most 5 rows, chosen by Fisher-Yates shuffle rather than taken from the
  top (the first rows of sorted data generally don't represent much). The cap
  is fixed rather than proportional: sampling a percentage of a file would scale 
  the privacy leak with file size. In the case of files with five rows or fewer, 
  the sample necessarily includes every row; surfacing this in the UI before 
  sending is planned.
- **No framework, no build step.** Just an HTML file; the browser does the file 
  intake, parsing hooks, and rendering. The single dependency is 
  [Papa Parse](https://www.papaparse.com/) for CSV parsing. 
- **Privacy as architecture.** Because analysis is entirely client-side, the 
  optional AI feature can be strictly additive: nothing is sent anywhere unless 
  the button is clicked, and what's sent is bounded and described above.

## Running it

Open `index.html` in a browser, or use the
[live demo](https://hardwick-hudson.github.io/csv-analyzer/).

That's the entire deployment.

## Related

[csv-validator](https://github.com/hardwick-hudson/csv-validator) - the
sibling project: a Java CLI that checks CSVs against rules you declare.
This tool finds problems without being told the rules.
