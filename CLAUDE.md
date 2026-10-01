# CLAUDE.md: Zoox Safety Validation Analysis

## Project
- Title: "Prioritizing Safety Validation for Zoox's Robotaxi Expansion: Which Crash Scenarios Need Attention First?"
- Notebook: notebooks/zoox_safety_validation_priorities.ipynb
- Repo name: zoox-safety-validation-analysis
- Business problem: Zoox is expanding driverless rider service into new cities. Its safety and validation teams have limited testing capacity and need to know which real-world crash scenarios to prioritize in simulation and on-road validation.
- Key questions:
  1. Which crash scenarios are over-represented in Zoox's crashes vs. peers (especially Waymo)?
  2. Do test-fleet patterns carry over to the driverless robotaxi?
  3. Do scenarios differ by city (SF vs. Las Vegas metro)?

## Data
- Source: NHTSA Standing General Order (SGO) crash reports, ADS only.
- Files (local, never committed): data/raw/ads_archive.csv (2021 to mid-2025), data/raw/ads_current.csv (June 2025 onward, 3rd Amendment schema).
- Keep the path in a single DATA_DIR variable (will move to S3 later).
- Read CSVs with low_memory=False.

## Cleaning rules (do not change without asking)
1. Dedupe on Report ID, keeping the latest Report Version.
2. Map renamed fields (SV Was Vehicle Towed?, SV Any Air Bags Deployed?, SV Were All Passengers Belted?), then combine on common columns with a data_source tag.
3. Strip whitespace on all text columns; parse dates at month granularity.
4. Dedupe cross-company duplicates on Same Incident ID, preferring the row where Reporting Entity matches Operating Entity.
5. Replace the CBI redaction marker with "Redacted" as its own category, never a null.
6. Log every dropped column and row count; never drop silently.

## Zoox vehicle labeling (mandatory)
- Classify by Make: Toyota → test fleet, Zoox → robotaxi.
  - Model "sedan" + Make Zoox = robotaxi (older label).
  - Model "111324" + Make Toyota = Highlander (data entry error).
  - There is no "Other/unclear" group. Verify with a crosstab of vehicle type × Driver / Operator Type.
- Labels:
  - Full: "Highlander test fleet (Zoox self-driving software, safety operator onboard)" and "Zoox robotaxi (driverless, no operator)"
  - Short (chart legends): "Highlander (Zoox software + safety operator)" and "Zoox robotaxi (driverless)"
- Always state that NHTSA requires Highlander crashes to be reported because Zoox's automated driving system was engaged.

## Analysis rules
- Never claim one company is "safer". The data is not normalized by miles driven. Compare shares within each company's own crashes.
- Use 95% Wilson CIs for proportions. Use Fisher's exact test for 2x2 comparisons (chi-square only when all expected counts are ≥5).
- Every Zoox vs. Waymo test goes into the multiple-comparisons table with Holm correction.
- Only report scenario-level rates with at least 5 Zoox crashes; otherwise label the result "too few to tell".
- Split Zoox results by vehicle type wherever it changes the conclusion.
- Narrative keyword tagging is keyword matching only; say so.

## Style rules
- Business-language section titles (e.g. "Scenario: Struck from behind while stopped"), not "% of crashes" titles.
- Numbered sections; all "see Section X" references must be correct.
- A short "Observation" markdown cell after every chart.
- Zoox highlighted in purple (#8E44AD), peers in gray (#B0B0B0), Highlander in light purple (#B497C7), consistent across all charts.
- Sorted bars, labeled axes, clear titles.
- No meta text about how the notebook was written or reviewed.

## Workflow rules
- Work one section at a time. Run it, show the output, and wait for my go-ahead before the next section.
- After any change, re-run the notebook top to bottom.
- Every number in markdown must match its cell output; check this before saying a task is done.
- Do not change analysis logic I didn't ask about.
- At the end of each task, summarize what changed and which findings got stronger, weaker, or flipped.
- Keep README.md in sync with the notebook (title, business problem, key questions, executive summary, how to run, limitations).
