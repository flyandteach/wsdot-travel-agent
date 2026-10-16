# WSDOT Travel Form Agent (standalone)

A small, local Python tool for WSDOT Aviation travel paperwork — no Streamlit,
no server, no browser. It intakes trip data through a guided terminal
questionnaire and generates:

1. a completed Aviation Travel Request PDF;
2. a completed 133-103 Travel Expense Voucher XLSX; and
3. a route map PDF for POV trips.

It does not approve travel, apply signatures, or make reimbursement
eligibility decisions. Meals, mileage rates, lodging, registration, and
charge-code splits should be reviewed by the traveler before routing.

## Files included

- `wizard.py` – interactive guided intake (the main interface).
- `travel_agent/` – core engine: rates/object codes (`rates.py`), driving
  distance + route maps (`geo.py`), PDF filler (`fill_pdf.py`), Excel filler
  (`fill_excel.py`), data model (`models.py`), profile storage (`profile.py`).
- `travel_agent/cli.py` – non-interactive filler: JSON in, forms out.
- `templates/` – blank Travel Request PDF and 133-103 XLSX templates.
- `examples/bfi_travel_intake.json` – sample trip intake.

## Install (one time)

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

## Run the wizard

```bash
.venv/bin/python wizard.py
# forms land in ./outputs/  (override with --out-dir)
```

Walk through the 7 sections — most prompts have sensible defaults (press
Enter to accept). Your traveler profile and charge codes save to
`~/.wsdot_travel_profile.json` and are reused next time.

What the wizard does automatically:

- Resolves destination city → county → OFM meal tier (with optional
  geocoding confirmation); GSA per-diem lookup for out-of-state travel
- Computes days/nights from dates; assigns Chapter 5 object codes
  (GA01/GA02, GF05/GF02, GC01/GC02, EG01/EG02/EG94, GD01/GD03, GB01/GG05/GG06)
- Calculates driving miles via OSRM (or enter manually) and applies the
  date-aware POV rate ($0.725/mi before 7/1/2026, $0.76/mi on/after)
- Per-day meal claiming with provided-meal deductions; same-day taxable warning
- Generates the route map PDF for POV trips

Or skip the questionnaire and fill forms straight from JSON:

```bash
.venv/bin/python -m travel_agent.cli --input examples/bfi_travel_intake.json --out-dir outputs
```

## Updating rates each January

Edit the rate block at the top of `travel_agent/rates.py` (POV rates and the
meal-tier B/L/D splits, per the OFM per diem map). Nothing else needs to
change. Current revision: OFM per diem map rev. 10/1/2026.

## Form limits

The 133-103 account-coding block fits 5 lines and the receipts block fits 6.
If a trip produces more, the agent keeps the first lines that fit and prints a
warning naming what was omitted.

## Review checklist before routing

- Verify travel dates, meeting times, and destination.
- Confirm meal eligibility and any provided meals.
- Confirm mileage rates and POV reason codes.
- Confirm registration or other receipt amounts.
- Confirm charge codes and account-code split amounts.
- Attach supporting documentation before routing for signature.
