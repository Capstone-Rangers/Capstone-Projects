# Task Brief — Clinical Coding & Reimbursement Chain

**Project:** Regional Hospital Spatial Management (NM rural acute-care capstone)
**Owner:** Joseph Ikogho
**Assisting:** Abduwakeel
**Deliverable:** Secondary Colab notebook — `cloud_based_diagnostic_tool.ipynb`

---

## 1. What this is

The main capstone found that county-level need drivers do **not** predict acute-care
hospital bed provision in New Mexico (null result, confirmed across four model families,
two sample sizes, leave-one-state-out validation, and two independently constructed
targets). Spatial analysis then identified candidate sites for a lightly-staffed triage
and diagnostic unit.

The immediate objection to any rural facility proposal is not whether need exists — it is
whether the facility can stay solvent. This task builds the evidence layer for that
question: given a patient's recorded conditions, what reimbursement weight would the
encounter carry?

A working proof-of-concept triage tool already exists (Streamlit app, real Synthea FHIR R4
ingestion, rule-based safety flags, 5-patient demonstration cohort). This task produces a
**function** that plugs into it.

---

## 2. Scope boundary — read before starting

**This is a sustainability demonstration, not a revenue optimizer.**

The capstone's central finding is that provision *should* track need rather than payer
mix. A tool that recommends chasing profitable patients would contradict the project's own
argument. The correct framing throughout:

> "Can a lightly-staffed rural triage unit generate enough billable encounter volume to
> remain viable?"

Not: "Which patients are most profitable?"

**All data is synthetic.** Synthea-generated FHIR R4 records. Nothing in this work is for
clinical use, and every output surface must carry that label.

---

## 3. The chain

| Step | Input | Output | Source |
|---|---|---|---|
| — | Synthea FHIR bundle | Condition resources (SNOMED CT) | already built |
| 1 | SNOMED CT code | ICD-10-CM code | NLM UMLS map |
| 2 | ICD-10-CM code | validated code | CDC/NCHS ICD-10-CM 2027 |
| 3 | ICD-10-CM code | DRG + MDC | NBER crosswalk |
| 4 | DRG | relative weight | NBER DRG weight file |

### Sources

- **NLM SNOMED CT → ICD-10-CM map**
  https://www.nlm.nih.gov/research/umls/mapping_projects/snomedct_to_icd10cm.html
- **CDC/NCHS ICD-10-CM 2027**
  https://ftp.cdc.gov/pub/Health_Statistics/NCHS/Publications/ICD10CM/2027/
- **NBER DRG–MDC crosswalk and DRG weights**
  https://www.nber.org/research/data/diagnosis-related-group-major-diagnostic-category-crosswalk

---

## 4. Step-by-step notes

### Step 0 — UMLS registration (do this first)

The NLM map is free but license-gated. Register for a UMLS account and accept the license
before writing any code. Downloads require credentials, so this cannot be a plain
`requests.get()` the way the AHRF and CMS pulls were. Budget a day for account approval.

### Step 1 — SNOMED → ICD-10-CM (the hard part)

This is **not** a lookup table. The NLM map is rule-based:

- One SNOMED concept may map to several ICD-10-CM codes
- Maps are organised into **map groups** with **priorities**
- Rules are conditional — e.g. `IFA 248152002 | Female |`, or `OTHERWISE TRUE` fallbacks
- Some concepts have no valid target and map to a "no map" flag

Expect most of the task's effort here. Deliverable for this step: a function that takes a
SNOMED code (plus patient age and sex, since rules need them) and returns the mapped
ICD-10-CM code, or `None` with a reason.

Document the proportion of the cohort's SNOMED codes that map cleanly, that map with
rules, and that fail to map. That percentage is itself a finding worth reporting.

### Step 2 — ICD-10-CM validation

Confirm the codes produced in Step 1 exist in the 2027 code set. Cheap to implement,
prevents invalid codes propagating silently into Step 3.

Note: the CDC files are fixed-width government text, not clean CSV. Inspect the layout
before parsing — do not assume delimiters.

### Step 3 — ICD-10 → DRG (state the limitation)

**Important:** DRGs are assigned per *inpatient stay*, not per diagnosis. Real DRG
assignment (a "grouper") considers principal diagnosis, secondary diagnoses, procedures,
complications/comorbidities, discharge status, and sometimes age. The NBER crosswalk gives
DRG→MDC mapping and DRG weights; it does not perform grouping.

Going from a patient's ICD-10 list to a single DRG is therefore an **approximation**. Two
acceptable approaches for a proof of concept:

1. Use the patient's most severe/primary condition as the principal diagnosis proxy
2. Report the DRG range the patient's conditions could fall into, rather than one value

Either is fine. What is **not** fine is presenting the result as an exact DRG assignment.
State the limitation in the notebook, in plain language, where the result is produced.

### Step 4 — Weight to estimate

DRG relative weight × a base rate = estimated payment. Use a documented CMS base rate and
cite it. Simple arithmetic once the DRG is assigned; the honesty is in labelling it an
estimate.

---

## 5. Interface — what to hand back

A single function with a defined signature, so it can be wired into the existing triage
app without refactoring:

```python
def estimate_encounter_value(snomed_codes, age, sex):
    """
    Map SNOMED conditions through ICD-10-CM to a DRG weight estimate.

    Returns dict:
      {
        'icd10_codes':   [...],          # successfully mapped
        'unmapped':      [...],          # SNOMED codes with no valid target
        'drg':           str or None,    # approximated principal DRG
        'mdc':           str or None,
        'weight':        float or None,  # DRG relative weight
        'estimate_usd':  float or None,  # weight x base rate
        'caveats':       [...],          # e.g. 'DRG approximated from primary condition'
      }
    """
```

Abduwakeel handles wiring this into the Streamlit app and adding the sustainability panel.

---

## 6. Documentation expectations

The notebook should follow the same standards as the main capstone:

- **Plain-English explanation before code**, not after
- **Missingness handled by mechanism** — state whether an unmapped code is structurally
  absent (no valid ICD-10 target exists) or a data gap; do not silently drop
- **Restart & Run All certification** — the notebook must execute top to bottom from a
  cold runtime before it is considered done
- **Limitations stated where they occur**, not only in a closing section
- Cite every source with a URL and access date

---

## 7. Definition of done

- [ ] UMLS account approved, map file downloaded
- [ ] SNOMED → ICD-10-CM function working, with rule handling
- [ ] Mapping coverage reported (clean / rule-based / unmapped percentages)
- [ ] ICD-10-CM codes validated against CDC 2027
- [ ] DRG and weight joined from NBER crosswalk
- [ ] `estimate_encounter_value()` returns the documented structure
- [ ] Runs on all five cohort patients without error
- [ ] Limitations documented, especially DRG approximation
- [ ] Restart & Run All passes clean
- [ ] Committed to the project repo

---

## 8. If it turns out to be too large

This is a legitimate outcome, not a failure. If Step 1's rule handling proves larger than
the timeline allows, the fallback deliverable is a **designed integration**: the sources
identified, the chain documented, the technical obstacles named (rule-based mapping, DRG
grouping approximation), and a worked example on one or two patients rather than a general
function.

Naming the scope honestly is worth more than a half-wired feature. Flag it early if you
get there.
