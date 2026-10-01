   **[▶ View the interactive playbook](https://geethasagarb.github.io/zoox-safety-validation-analysis/)**

   **(https://jyothirmayibavireddy.github.io/zoox-robotaxi-crash-scenario-validation/)**

# Prioritizing Safety Validation for Zoox's Robotaxi Expansion: Which Crash Scenarios Need Attention First?

**Business problem:** Zoox is expanding driverless robotaxi service into new cities. Its safety and validation teams have limited testing capacity, so they need to know which real-world crash scenarios to prioritize in simulation and on-road validation work.

**Key questions:**
1. Which crash scenarios are over-represented in Zoox's crashes compared to peers like Waymo?
2. Do patterns seen in the test fleet carry over to the driverless robotaxi?
3. Do scenarios differ by city (San Francisco vs. Las Vegas), which matters for planning new-city launches?

**Notebook:** [`notebooks/zoox_safety_validation_priorities.ipynb`](notebooks/zoox_safety_validation_priorities.ipynb)

## Data source

All crash data comes from NHTSA's **[Standing General Order (SGO) Crash Reporting](https://www.nhtsa.gov/laws-regulations/standing-general-order-crash-reporting)** program — public, downloadable CSVs that manufacturers and operators of ADAS/ADS-equipped vehicles are legally required to file. This project uses two of NHTSA's exports:
- `ads_current.csv` — reports filed under the 3rd Amendment reporting fields (June 2025 onward)
- `ads_archive.csv` — reports filed under the earlier reporting fields (2021 through mid-2025)

Data files are not included in this repo (see `.gitignore`) — download them from the NHTSA page above and place them in `data/raw/`. NHTSA's own field-level documentation for each schema is reproduced for reference in [`docs/`](docs/).

## How to run it

1. Download `ads_current.csv` and `ads_archive.csv` from the [NHTSA SGO page](https://www.nhtsa.gov/laws-regulations/standing-general-order-crash-reporting) and place them in `data/raw/`.
2. Create a virtual environment and install dependencies:
   ```
   python -m venv .venv
   .venv\Scripts\pip install -r requirements.txt
   ```
   > **Note:** if your `.venv` lives under a folder subject to a Windows Application Control / WDAC policy (e.g. some corporate `Documents` folders), pandas's compiled DLLs may be blocked from loading. If you hit `ImportError: DLL load failed ... Application Control policy`, create the venv in an unrestricted location (e.g. `C:\Users\<you>\venvs\`) instead and point your Jupyter kernel there.
3. Register the kernel and launch Jupyter:
   ```
   python -m ipykernel install --user --name zoox-av-eda --display-name "zoox-av-eda"
   jupyter notebook notebooks/zoox_safety_validation_priorities.ipynb
   ```
4. Run all cells top to bottom. `DATA_DIR` in the first code cell is the single place to change if you move the data (e.g. to an S3 path later).

## Executive summary

- **Scenario ranking (which scenarios are over-represented?):** two scenarios are both large and statistically significant against Waymo, even after correcting for running 13 tests: right turns into a passenger car (6.3% of Zoox's crashes vs. 0.65% of Waymo's — a 9.7x ratio) and being rear-ended by a passenger car while stopped (13.3% vs. 6.1% — a 2.2x ratio). These are the two concrete scenarios we'd prioritize for validation, not a vague "Zoox crashes differently."
- **Robotaxi carryover (does the pattern transfer?):** the rear-end scenario carries over to the robotaxi (10.4% vs. Waymo's 6.1%); the right-turn scenario has too few robotaxi crashes yet (1) to say either way. Separately, and importantly: the much-discussed "Zoox has a higher injury rate than Waymo" finding (20.9% vs. 8.8% in aggregate) does **not** carry over to the robotaxi — it's a **Highlander test-fleet effect** (25.5%, still significant after correction); the robotaxi's own injury rate (10.4%, n=48) is statistically indistinguishable from Waymo's.
- **City differences (does it vary by city?):** in this still-small sample, Las Vegas shows nearly double San Francisco's right-turn crash share (17.7% vs. 9.5%, n=62 and 84), suggesting road-network design — not just the vehicle — shapes the scenario mix. That matters for predicting a new city's profile before it has any crash history of its own.
- Zoox's reported crashes split 110 **Highlander test fleet** (Zoox self-driving software, safety operator onboard) / 48 **Zoox robotaxi** (driverless, no operator), and the robotaxi has outnumbered the Highlander fleet in monthly crash volume since May 2024, though the test fleet remains active in parallel.
- None of this is a safety ranking — the dataset has no mileage or exposure data — but it points to two specific, statistically-grounded scenarios worth prioritizing in simulation and on-road validation ahead of new-city launches, rather than a generic "investigate everything."

## Limitations (important — read before citing any number from this project)

- **Not normalized by miles driven.** Nothing here ranks companies by safety — we have no exposure data.
- **Most Zoox crashes come from the Highlander test fleet, not the robotaxi** (110 of 158, vs. 48 robotaxi), so any unsplit "Zoox" comparison mixes two very different vehicle programs (supervised test-fleet driving vs. autonomous rider-only service).
- **The `crash_scenario` field is one reasonable construction, not the only one** — a different combination of movement/partner/speed could rank scenarios differently.
- **The robotaxi carryover labels depend on a chosen threshold** (≥1.5x Waymo's share to call it "persists," minimum 5 robotaxi crashes to say anything at all) — reasonable, stated cutoffs, not the only valid ones.
- **Narrative keyword tagging is literal keyword matching, not NLP** — it undercounts anything phrased differently than the literal pattern and is not manually verified.
- **The San Francisco vs. Las Vegas comparison is descriptive, not a statistical test** — 62 and 84 crashes respectively is enough to describe a pattern but too few to confirm one.
- **Self-reported, unverified.** Companies file these reports themselves; there is no independent verification layer.
- **Schema changed mid-dataset** (June 2025, 3rd Amendment) — some fields (`Lighting`, `Posted Speed Limit`, `Mileage`) exist only in the older schema and are noted as such wherever used.
- **Cross-company duplicate resolution is a best-effort heuristic** (keep the row whose `Reporting Entity` matches its `Operating Entity`, with a fuzzy fallback for inconsistently formatted names) — not manually verified row by row.
- **Redacted fields are informative, not missing** — e.g., `Within ODD?` and `Narrative` are redacted for confidential business information and kept as an explicit "Redacted" category.
- **No ground-truth fault field** — any "who struck whom" framing in this notebook is an inference from contact-area and movement fields, not a stated fact in the data.
- **Exact crash coordinates are redacted** as potential PII for every company — the geospatial section approximates using public city-centroid coordinates, not true crash-level geocoordinates.
- **Small samples for further slicing.** The robotaxi-only group (n=48) and the right-turn detail breakdown (n=20) are both small enough that individual-category counts should be read as illustrative, not precise rates.
