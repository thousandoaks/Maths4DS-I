# Outpatient Appointments Dataset — Data Dictionary

**Riverbend Community Hospital (fictional) · Outpatient clinics · Calendar year 2025**
Master in Healthcare Analytics · Dataset 1 of 5 · Version 1.0 (September 2026)

## 1. What this dataset is

This dataset contains every outpatient appointment scheduled during 2025 at the four outpatient clinics of Riverbend Community Hospital, a fictional 300-bed community hospital in a mid-sized US city. It is **fully synthetic**: no row describes a real person, provider or visit. It was generated in Python from an explicit statistical model so that the patterns it contains are known, realistic in size and direction, and recoverable with the methods taught in this programme (exploratory analysis, contingency tables, group comparisons, and logistic regression).

The central question the data supports is **why patients miss appointments (no-shows)**, and what a clinic could do about it. The data are deliberately clean: there are no missing values, no duplicate rows, no malformed dates, and no double bookings. The analytical difficulty lies in the relationships between variables, not in cleaning.

| File | Rows | Grain | Purpose |
|---|---|---|---|
| `appointments.csv` | 79,253 | One row per scheduled appointment | Main analysis table (flat; patient attributes already joined) |
| `patients.csv` | 11,523 | One row per patient | Reference table; lets students practise joins and patient-level analyses |
| `providers.csv` | 25 | One row per clinician | Reference table for provider-level comparisons |

Patients are identified by pseudonymous IDs only. There are no names, dates of birth, addresses or ZIP codes; distance to the clinic and an urban/suburban/rural label stand in for geography.

## 2. The setting

Four clinics share one scheduling system: **Family Medicine** (8 providers), **Internal Medicine** (7), **Cardiology** (5) and **Orthopedics** (5). Clinics operate Monday to Friday, 08:00–16:30, in 30-minute slots with no appointments at 12:00 or 12:30 (lunch) and none on US federal holidays. Appointments are booked in advance; the gap between booking and appointment date is the **lead time**. On the day, each appointment ends in one of four statuses: the patient **arrived**, the patient did not attend and did not cancel (**no-show**), or the appointment was **cancelled** in advance by the patient or by the clinic.

Overall, 77.4% of appointments were attended, 13.5% were no-shows, 7.1% were cancelled by the patient and 2.0% by the clinic. Among appointments that were not cancelled, the **no-show rate is 14.9%**, which is in the typical US ambulatory range.

## 3. Column reference — `appointments.csv`

| Column | Type | Values / range | Description |
|---|---|---|---|
| `appointment_id` | string | `A000001` … | Unique appointment identifier, assigned in date order. Primary key. |
| `patient_id` | string | `P000001` … | Pseudonymous patient identifier. Foreign key to `patients.csv`. A patient has 1–45 appointments in the year (median 6). |
| `provider_id` | string | `DR001` … `DR025` | Clinician who was scheduled to see the patient. Foreign key to `providers.csv`. |
| `clinic` | category | Family Medicine, Internal Medicine, Cardiology, Orthopedics | Clinic where the appointment was scheduled. |
| `visit_type` | category | Established, New patient, Annual wellness, Telehealth | Type of visit. *Established* = follow-up with a known patient (55%); *New patient* = first visit to that clinic (20%); *Annual wellness* = preventive visit (12%); *Telehealth* = video visit (13%; more common in primary care). |
| `booking_date` | date (ISO) | 2024-09-04 … 2025-12-31 | Date the appointment was booked. |
| `appointment_date` | date (ISO) | 2025-01-02 … 2025-12-31 | Scheduled date of the appointment. Weekdays only, no federal holidays. |
| `appointment_time` | time (HH:MM) | 08:00 … 16:30 | Scheduled slot start, 30-minute slots, no 12:00/12:30. |
| `weekday` | category | Monday … Friday | Day of week of `appointment_date` (derived; provided for convenience). |
| `slot_period` | category | Early morning, Mid-day, Late afternoon | Time band: Early morning = 08:00–09:30; Mid-day = 10:00–14:30; Late afternoon = 15:00–16:30 (derived). |
| `lead_time_days` | integer | 0 … 120 | Days between booking and appointment (`appointment_date − booking_date`). Median 11, mean 15.7, strongly right-skewed. Longer for new-patient and specialty visits. |
| `prior_no_shows` | integer | 0 … 38 | Number of no-shows by this patient in the **365 days before** this appointment (includes late 2024, which is not in the file). 48% of rows are 0; 6% are 5 or more. |
| `sex` | category | F, M | Patient sex as recorded in registration (54% F). |
| `age` | integer | 18 … 95 | Patient age in years on 1 January 2025. Adults only. Mean 52.6. |
| `primary_payer` | category | Commercial, Medicare, Medicaid, Self-pay | Primary insurance coverage. Medicare is concentrated in patients aged 65+. Shares: 52%, 23%, 18%, 7%. |
| `residence_type` | category | Urban, Suburban, Rural | Type of area where the patient lives (46%, 37%, 17% of rows). |
| `distance_miles` | float | 0.5 … 80.0 | Straight-line distance from the patient's home to the clinic, in miles. Median 5.7, right-skewed; rural patients live much farther. |
| `status` | category | Arrived, No-show, Cancelled by patient, Cancelled by clinic | Outcome of the appointment. **The no-show rate should be computed as No-show ÷ (Arrived + No-show)**, excluding cancellations, because a cancelled slot can be refilled and a no-show cannot. |

The patient-level columns (`sex`, `age`, `primary_payer`, `residence_type`, `distance_miles`) are repeated on every appointment row so that the table can be analysed without joins. They are constant within a patient and identical to `patients.csv`.

## 4. Column reference — reference tables

**`patients.csv`** — `patient_id`, `sex`, `age`, `primary_payer`, `residence_type`, `distance_miles` (as above), plus `home_clinic` (category: the clinic the patient most often attends; about 78% of a patient's appointments are at their home clinic).

**`providers.csv`** — `provider_id`; `clinic` (the provider works in exactly one clinic); `role` (MD, DO, NP, PA); `fte` (1.0, 0.8 or 0.6; providers with higher FTE carry proportionally more appointments); `years_in_practice` (integer, 2–29). Provider-level no-show rates range from about 10% to 26%.

## 5. Questions this dataset can answer

The dataset was built around the no-show problem, and the variables were chosen because each is a plausible driver in the scheduling literature or a plausible red herring. Questions worth pursuing, roughly in order of difficulty: How does the no-show rate vary with lead time, and is the relationship linear? Do patients with a history of missed appointments keep missing them? Are telehealth visits kept more reliably than in-person visits, and does that hold within each clinic? Do early-morning or late-afternoon slots differ, and is Friday really worse? Do clinics differ because of who they see or because of how they schedule? Do providers within the same clinic differ by more than chance would produce? Does payer predict no-shows once lead time, distance and history are accounted for, or is the raw payer gap explained by access? Is there a seasonal pattern, or only the appearance of one? Which of these factors could a clinic actually change?

Not every variable in the table carries a real effect, and at least one strong raw association weakens substantially once other variables are controlled. Reaching a defensible conclusion requires moving from the crosstab to an adjusted model.

## 6. Suggested analysis sequence

1. **EDA.** Distribution of `lead_time_days` (why the median, not the mean); no-show rate by weekday, slot period, visit type, clinic and lead-time band; appointments per patient.
2. **Contingency tables and chi-square.** `visit_type × status`, `primary_payer × status`, with standardised residuals; two-proportion tests between clinics or providers.
3. **Group comparisons.** Lead time by clinic and visit type (Kruskal–Wallis, since lead time is skewed); provider no-show rates within a clinic.
4. **Logistic regression.** `no_show ~ lead_time_days + prior_no_shows + visit_type + slot_period + weekday + age + distance_miles + primary_payer + clinic`. Interpret odds ratios; compare the crude and adjusted payer effects; check calibration by decile.
5. **Decision.** Choose a probability threshold for an enhanced reminder call and estimate how many no-shows it would prevent under an assumed reminder effect; simulate a simple risk-based overbooking rule.

## 7. Provenance and reproducibility

Generated with `generate_appointments.py` (Python 3, numpy, pandas), random seed 2025. The generator draws a patient population, simulates 27 months of bookings so that `prior_no_shows` is well defined from January, computes each appointment's no-show probability from a logistic model with fixed coefficients plus a small provider effect, and keeps only 2025 appointments. Re-running with a different seed produces a new dataset with the same underlying truth, which allows a fresh dataset per cohort. The true coefficients are held in the instructor answer key and are not part of student materials.
