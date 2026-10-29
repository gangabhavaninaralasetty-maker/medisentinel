# MediGuard AI — Medication Safety Second Pair of Eyes

**Status: runnable educational prototype using a user-supplied Kaggle dataset.** The dataset has not been independently clinically validated. Its rows are screening signals only, not medical advice. Never use this prototype to make treatment decisions or enter real patient information.

## Features
- Public landing page and patient medication checker.
- Name normalisation, exact matching, and fuzzy candidate matching with uncertainty labels. Only exact matches trigger interaction checks.
- Multi-medicine interaction screening for up to 20 entries per check (up to 190 unique pairs for 20 different medicines).
- General context prompts for under-18 and 65+ patients, supplied eGFR, allergies, health conditions, and pregnancy/breastfeeding context. These are workflow reminders only, not medicine-specific clinical rules or dose calculations.
- Exact repeated medicine entries trigger a duplicate-entry review. Therapeutic-class duplication is not validated because class mapping is not included.
- Basic medicine lookup shows only fields present in the supplied dataset (name, database ID, target, enzyme); it does not provide indications, dosage, side effects, or patient-specific treatment advice.
- Medicine-specific renal dosing, validated pediatric/older-adult rules, and contraindication checks are not established by this Kaggle dataset.
- Prioritised alerts with reason, suggested review action, confidence, limitations, and clinician disclaimer.
- Technical page, schema-validation CSV import endpoint, demo evaluation endpoint, and pytest tests.
- Browser voice input for questions and medication names (supported browsers only; verify every transcript).
- English/Telugu response-language option for local Ollama explanations. Telugu wording can still be imperfect; verify medicine names and claims.
- Downloadable PDF screening summary, generated on request and not saved by the server.
- Password-protected local admin dashboard with model/data status and in-memory usage counters. Counters reset when the app restarts; no patient lists are stored.
- No claim of official benchmark accuracy. Official near-miss recall and serious-issue recall remain unavailable until official records, reference tables, hidden labels, and the benchmark evaluation contract are provided.

## Requirements
Python 3.10+ recommended. Works on macOS with VS Code.

## Dataset files
The `data/` folder contains `drug_interaction.csv` and `drug_information_1258.csv` from the ZIP supplied by the user. The app loads these CSVs at startup. `drug_interaction.csv` has `drugA`, `drugB`, `mechanism`, and `action`; because it has no validated severity field, every match is labelled `REVIEW`. `drug_information_1258.csv` provides names and identifiers, not complete brand/generic aliases or renal dosing rules.

## Ollama hybrid AI integration (local only)
The `/assistant` page uses your locally running Ollama API. The existing `/api/analyze` safety checker remains separate and uses only the bundled, unvalidated dataset/rules. The AI can explain the exact alerts returned by the checker but must not add new interaction claims, doses, or safety conclusions.

Defaults: `OLLAMA_BASE_URL=http://127.0.0.1:11434` and `OLLAMA_MODEL=llama3.2:3b`. Set these environment variables only if your local Ollama uses a different URL or model name. Keep Ollama running while using the assistant. No cloud AI service is called by this integration.

## Run on MacBook
Open Terminal and copy these commands one at a time:

```bash
mkdir -p ~/Desktop/mediguard-ai
cd ~/Desktop/mediguard-ai
```

Download/unzip this project into that folder, then:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
PORT=5010 python app.py
```

Open http://127.0.0.1:5010. First, make sure the Ollama app is open and `ollama list` shows `llama3.2:3b`. Use **AI explainer** for general educational questions. Choose English or Telugu, or use voice input in a supported browser. Run the checker and select **Explain these results with AI** or **Download PDF report**.

To run tests, open a second terminal in the project folder, activate the environment, then:

```bash
source .venv/bin/activate
pytest -q
```

## Demo flow
1. Open Home → Try the checker.
2. Enter up to 20 medicine entries that exist in `data/drug_information_1258.csv`, one per line. The checker examines every unique pair among exact matches; fuzzy matches require manual verification. Repeated entries are retained for duplicate-entry review.
3. For a sample, try `Apixaban` and `Bivalirudin`; the app will show matching dataset rows as `REVIEW` signals.
4. Under-18 and 65+ ages, a supplied eGFR, allergies, health conditions, or pregnancy/breastfeeding context create general review prompts. These are not medication-specific decision rules; the app does not calculate doses or infer renal safety.
5. Open Basic medicine guide to search the bundled reference by name and review general medicine-safety reminders. The lookup only shows dataset metadata and cannot recommend a drug or dose.
5. Open Technical Dashboard → Demo evaluation. The tiny fixture metrics are a software smoke test only, not the challenge score.


## Voice, Telugu, PDF, and admin dashboard
- Voice input uses the browser SpeechRecognition API and may require permission/network access depending on the browser. It is not supported in every browser. Always review transcripts; medication names can be misheard.
- Select Telugu in the AI explainer or the checker result language menu. The model may make language errors, and Telugu output is not validated medical translation.
- PDF export creates a report from the current browser result and returns it as a download. The server does not save the report. Treat it as a prototype summary, not a clinical record.
- Admin login is at `/admin/login`. For local use, set credentials when starting Flask. Example (choose your own password):

```bash
ADMIN_USERNAME=admin ADMIN_PASSWORD='choose-a-long-password' MEDIGUARD_SECRET_KEY='replace-with-a-long-random-secret' PORT=5010 python app.py
```

The dashboard is intended for local demos only. Its usage counters are in memory and reset on restart; it does not provide production-grade user management, audit logging, or multi-user security. Do not deploy it publicly without a security review.

## CSV schema validation
`POST /api/import-csv` expects multipart form fields:
- `kind`: `medicines`, `interactions`, `patients`, or `labels`
- `file`: UTF-8 CSV file

Minimum required columns:
- medicines: `generic_name`
- interactions: `drug_a`, `drug_b`, `severity`
- patients: `patient_id`, `age`, `medications`
- labels: `patient_id`, `issue_type`, `drug_or_pair`

This endpoint validates structure only. It deliberately does not activate uploaded medical data. Map and review the official schema before integrating it.

## Official dataset integration checklist
- Verify the dataset source, licence, coverage, and clinical validity before any use beyond a hackathon prototype.
- Preserve exact dataset identifiers and schema; document all mappings.
- Implement renal dosing only where the reference provides explicit criteria and the patient's dose data supports a valid check.
- Implement contraindications only from explicit reference rules.
- Match the benchmark `evaluate.py` input/output contract exactly.
- Keep hidden labels separate from prediction code and load them only inside the authorised evaluation workflow.
- Report brand/generic accuracy on labelled misspellings, serious-issue recall, 300 near-miss recall, precision, false positives, and low-severity alert volume.
- Never publish placeholder/demo metrics as benchmark results.

## Safety, privacy, and production limits
- The medication list can be incomplete; absence of an alert is not proof of safety.
- Fuzzy candidates are not confirmed medicine identities.
- Allergy input is only a context prompt plus exact text comparison; it is not a validated allergy checker and cannot assess cross-reactivity.
- No real clinical database or validated medical reference is bundled.
- Do not upload real patient information to this prototype.
- A qualified clinician must verify alerts and make decisions.
- Production deployment requires clinical validation, security review, authentication, audit logs, privacy controls, and jurisdiction-specific review.

## Deployment
For a hackathon preview, deploy to a Python web host that supports Flask and the provided `requirements.txt`. Set the start command to `gunicorn app:app` only after adding `gunicorn` to `requirements.txt`. Use a production WSGI server; do not enable Flask debug mode publicly. SQLite is suitable for a local demo, not a multi-user clinical production service. Keep real patient data out of the demo deployment.
# medisentinel
