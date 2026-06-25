# Door Condition Monitoring: Anomaly Detection Pipeline

## What this is

Railway doors are instrumented with sensors that record each open and close cycle.
This project takes that raw sensor data and, for every door operation, determines
whether it is normal or faulty and, if faulty, which fault type it most resembles.

## The data

Thousands of door cycles supplied as nested JSON files, grouped into folders by
condition: normal operation plus ten fault types (for example pressure roller too
tight, stop-pin interference, centering anomaly). Each cycle contains motor current,
position and speed traces, along with timing and digital switch signals.

## Approach (in brief)

1. **Extract:** parse the nested folder-and-JSON files into one labelled table.
2. **Clean:** drop a small number of corrupt/incomplete records, split the data into
   open and close operations, which behave very differently and are modelled separately.
3. **Feature engineering:** reduce each variable-length cycle to a fixed set of
   descriptive numbers (peak current, work done, cycle duration, reversal behaviour,
   per-phase summaries, switch angles).
4. **Model:** a Random Forest classifier per direction, returning the most likely
   fault plus a probability for every fault type, so a maintenance team sees both the
   diagnosis and the model's confidence.

## Key findings

- The model classifies faults well, with macro-F1 around 0.92 (open) and 0.95 (close).
- It is strongest on faults like interference, tightness, and wear, and weaker on faults like centering, lower-arm-roller, which have both small sample sizes and fainter signals. Confident predictions on the first
  group are reliable, while predictions on the second should prompt a closer look.

## Important caveat

The main limitation is in the data, not the model. Normal and fault cycles were
recorded in separate time windows, and some sensor readings drift over time, so part
of the model's apparent skill could reflect *when* a cycle was recorded rather than
*how the door behaved*. We tested this by removing the time-correlated features and
re-training: performance fell only modestly (open 0.92 to 0.85, close 0.96 to 0.94),
which indicates the model is mostly using genuine fault signal. The headline scores
should still be read as somewhat optimistic.

## Recommendations for a production system

- Collect normal and faulty cycles over the same time periods to remove the confound.
- Gather more examples of the rare geometry faults.
- Add shape-based features comparing each cycle's full current curve against a normal
  reference to improve detection of the subtle faults.

## Contents of this submission

- `notebook.ipynb` — full analysis, run in order with outputs saved. Markdown cells
  explain each step in plain language; readers can follow the narrative without code.
- `requirements.txt` — Python dependencies and versions.
- `README.md` — this file.

## How to run

1. Install dependencies: `pip install -r requirements.txt`
2. Place the data folders alongside the notebook (the loader expects one folder per
   condition, each containing the JSON files).
3. Open the notebook and run the cells in order.

Standard-library modules (json, ast, glob, os) need no installation. Core libraries:
pandas, numpy, matplotlib, scikit-learn.
