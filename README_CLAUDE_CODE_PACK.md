# Claude Code 30 MB Pack — Intuit SMB Hackathon

This pack is intentionally below 30 MB. It omits full `train.csv` because that file alone is ~36 MB.

## Use this pack for
- validating the current fallback submission,
- threshold sensitivity using `validation.csv` + fallback A predictions,
- updating A decisions only,
- regenerating B using validation/test applications + derived train timing curves,
- checking C diagnostics against data dictionary / intervention queries,
- preserving the current writeup unless outputs change materially.

## Do not use this pack for
- full model retraining from raw train.csv.

If full retraining is needed, run it outside Claude Code or upload `train.csv` alone in a separate workflow.
