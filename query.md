# Healthcare Ontology — Sample Graph Queries (GQL)

Copy and paste any query directly into the Microsoft Fabric graph query editor.

**Ontology entities:** `Patient` · `Provider` · `Appointment` · `Diagnosis` · `Prescription`

**Relationship types:**
| Relationship | Direction |
|---|---|
| `has_appointment` | Patient → Appointment |
| `sees` | Provider → Appointment |
| `diagnosed_with` | Patient → Diagnosis |
| `diagnoses` | Provider → Diagnosis |
| `treated_by` | Diagnosis → Prescription |
| `prescribes` | Provider → Prescription |

---

## Single-Hop Queries

Single-hop queries traverse exactly one relationship — the simplest graph patterns.

---

### Query 1 — All appointments for a specific patient

```gql
// Traverses ONE relationship: Patient -[:has_appointment]-> Appointment
// Useful for: pulling up a patient's full visit history
MATCH (pat:Patient)-[:has_appointment]->(appt:Appointment)
FILTER pat.name STARTS WITH 'Liam'
RETURN pat.name AS patient_name,
       appt.appointmentId,
       appt.scheduledTime,
       appt.type,
       appt.status
ORDER BY appt.scheduledTime ASC
```

---

### Query 2 — All diagnoses made by providers in a given specialty

```gql
// Traverses ONE relationship: Provider -[:diagnoses]-> Diagnosis
// Useful for: auditing diagnosis patterns within a clinical specialty
MATCH (prov:Provider)-[:diagnoses]->(dx:Diagnosis)
FILTER prov.specialty = 'Cardiology'
RETURN prov.name       AS provider_name,
       dx.diagnosisId,
       dx.icdCode,
       dx.description  AS condition,
       dx.severity,
       dx.diagnosedDate
ORDER BY dx.diagnosedDate ASC
```

---

### Query 3 — All prescriptions treating a specific ICD diagnosis code

```gql
// Traverses ONE relationship: Diagnosis -[:treated_by]-> Prescription
// Useful for: checking what medications are used to treat hypertension (I10)
MATCH (dx:Diagnosis)-[:treated_by]->(rx:Prescription)
FILTER dx.icdCode = 'I10'
RETURN dx.description      AS condition,
       rx.rxNumber,
       rx.medication,
       rx.dosage,
       rx.frequency,
       rx.refillsRemaining
```

---

### Query 4 — All appointments handled by providers in the Emergency department

```gql
// Traverses ONE relationship: Provider -[:sees]-> Appointment
// Useful for: reviewing emergency department visit volume
MATCH (prov:Provider)-[:sees]->(appt:Appointment)
FILTER prov.department = 'Emergency'
RETURN prov.name         AS provider_name,
       appt.appointmentId,
       appt.scheduledTime,
       appt.type,
       appt.status
ORDER BY appt.scheduledTime DESC
```

---

### Query 5 — Prescription count per provider, ranked by volume

```gql
// Traverses ONE relationship: Provider -[:prescribes]-> Prescription
// Useful for: identifying the highest-prescribing clinicians
MATCH (prov:Provider)-[:prescribes]->(rx:Prescription)
RETURN prov.name       AS provider_name,
       prov.specialty,
       count(rx)       AS total_prescriptions
GROUP BY prov.name, prov.specialty
ORDER BY total_prescriptions DESC
```

---

## Multi-Hop Queries

Multi-hop queries chain two or more relationships to reveal connections that
would require complex SQL joins — but are natural and readable in GQL.

---

### Query 6 — Which providers has each patient been seen by? (2 hops)

```gql
// Traverses TWO relationships through the Appointment as a bridge node:
//   Patient -[:has_appointment]-> Appointment <-[:sees]- Provider
// Power of graphs: no join table needed — the graph already knows this connection
MATCH (pat:Patient)-[:has_appointment]->(appt:Appointment)<-[:sees]-(prov:Provider)
RETURN DISTINCT pat.name     AS patient_name,
                prov.name    AS provider_name,
                prov.specialty
ORDER BY pat.name, prov.name
```

---

### Query 7 — Full medication trail for every patient (2 hops)

```gql
// Traverses TWO relationships:
//   Patient -[:diagnosed_with]-> Diagnosis -[:treated_by]-> Prescription
// Power of graphs: links a patient to their prescriptions through the clinical reason
MATCH (pat:Patient)-[:diagnosed_with]->(dx:Diagnosis)-[:treated_by]->(rx:Prescription)
RETURN pat.name        AS patient_name,
       dx.icdCode,
       dx.description  AS condition,
       rx.medication,
       rx.dosage,
       rx.frequency
ORDER BY pat.name, dx.icdCode
```

---

### Query 8 — Provider prescribing output ranked by diagnosis-driven prescriptions (2 hops)

```gql
// Traverses TWO relationships:
//   Provider -[:diagnoses]-> Diagnosis -[:treated_by]-> Prescription
// Power of graphs: shows how many prescriptions flowed from each provider's diagnoses
MATCH (prov:Provider)-[:diagnoses]->(dx:Diagnosis)-[:treated_by]->(rx:Prescription)
RETURN prov.name              AS provider_name,
       prov.specialty,
       count(DISTINCT dx)     AS diagnoses_made,
       count(rx)              AS prescriptions_from_diagnoses
GROUP BY prov.name, prov.specialty
ORDER BY prescriptions_from_diagnoses DESC
```

---

### Query 9 — Diagnoses made by providers who treated a patient (3 hops)

```gql
// Traverses THREE relationships across FOUR entities:
//   Patient -[:has_appointment]-> Appointment <-[:sees]- Provider -[:diagnoses]-> Diagnosis
// Power of graphs: finds every condition diagnosed by any provider a patient has visited
MATCH (pat:Patient)-[:has_appointment]->(appt:Appointment)<-[:sees]-(prov:Provider)
MATCH (prov)-[:diagnoses]->(dx:Diagnosis)
RETURN DISTINCT pat.name     AS patient_name,
                prov.name    AS provider_name,
                prov.specialty,
                dx.description AS diagnosis_made_by_this_provider
ORDER BY pat.name, prov.name
```

---

### Query 10 — High-severity conditions and their prescribed treatments (3 hops)

```gql
// Traverses THREE relationships:
//   Provider -[:diagnoses]-> Diagnosis (severity = high) -[:treated_by]-> Prescription
// Power of graphs: surfaces the most critical patients and exactly what was prescribed
MATCH (prov:Provider)-[:diagnoses]->(dx:Diagnosis)-[:treated_by]->(rx:Prescription)
FILTER dx.severity = 'high'
RETURN prov.name        AS diagnosing_provider,
       prov.specialty,
       dx.icdCode,
       dx.description   AS condition,
       dx.severity,
       rx.medication,
       rx.dosage
ORDER BY prov.name, dx.icdCode
```

---

### Query 11 — Prescription burden per patient across all diagnoses (2 hops, aggregation)

```gql
// Traverses TWO relationships:
//   Patient -[:diagnosed_with]-> Diagnosis -[:treated_by]-> Prescription
// Power of graphs: counts both active conditions and total medications for each patient
MATCH (pat:Patient)-[:diagnosed_with]->(dx:Diagnosis)-[:treated_by]->(rx:Prescription)
RETURN pat.name               AS patient_name,
       count(DISTINCT dx)     AS active_conditions,
       count(rx)              AS total_prescriptions
GROUP BY pat.name
ORDER BY total_prescriptions DESC
```

---

### Query 12 — Patients who share the same provider for both appointments AND diagnoses (3 hops, two paths)

```gql
// Traverses TWO paths that both resolve to the same provider:
//   Patient -[:has_appointment]-> Appointment <-[:sees]- Provider
//   Patient -[:diagnosed_with]-> Diagnosis   <-[:diagnoses]- Provider (same one!)
// Power of graphs: variable reuse of 'prov' enforces the "same provider" constraint automatically
MATCH (pat:Patient)-[:has_appointment]->(appt:Appointment)<-[:sees]-(prov:Provider)
MATCH (pat)-[:diagnosed_with]->(dx:Diagnosis)<-[:diagnoses]-(prov)
RETURN pat.name      AS patient_name,
       prov.name     AS provider_name,
       count(DISTINCT appt) AS shared_appointments,
       count(DISTINCT dx)   AS shared_diagnoses
GROUP BY pat.name, prov.name
ORDER BY shared_appointments DESC
```

---

### Query 13 — Which medications reach the most patients? (2 hops, ranked by patient reach)

```gql
// Traverses TWO relationships:
//   Patient -[:diagnosed_with]-> Diagnosis -[:treated_by]-> Prescription
// Power of graphs: surfaces the most widely used medications by counting unique patients
MATCH (pat:Patient)-[:diagnosed_with]->(dx:Diagnosis)-[:treated_by]->(rx:Prescription)
RETURN rx.medication,
       count(DISTINCT pat) AS patients_receiving,
       count(DISTINCT dx)  AS diagnoses_treated
GROUP BY rx.medication
ORDER BY patients_receiving DESC
```

---

### Query 14 — Full care chain: patient → appointment → provider → prescription (4 entities)

```gql
// Traverses FOUR relationships across the full ontology:
//   Patient -[:has_appointment]-> Appointment <-[:sees]- Provider -[:prescribes]-> Prescription
//   (then validates the prescription links to a diagnosis the patient actually has)
// Power of graphs: reveals the entire patient journey in a single query with no hand-coded joins
MATCH (pat:Patient)-[:has_appointment]->(appt:Appointment)<-[:sees]-(prov:Provider)
MATCH (prov)-[:prescribes]->(rx:Prescription)
MATCH (pat)-[:diagnosed_with]->(dx:Diagnosis)-[:treated_by]->(rx)
RETURN pat.name          AS patient_name,
       appt.scheduledTime AS appointment_date,
       prov.name          AS provider_name,
       dx.description     AS condition,
       rx.medication,
       rx.dosage
ORDER BY pat.name, appt.scheduledTime DESC
```

---

### Query 15 — Per-provider: appointment count vs. prescription volume per patient seen (4 entities, aggregation)

```gql
// Traverses FOUR relationships across all five entity types:
//   Patient -[:has_appointment]-> Appointment <-[:sees]- Provider -[:prescribes]-> Prescription
//   Patient -[:diagnosed_with]-> Diagnosis -[:treated_by]-> Prescription (same rx)
// Power of graphs: compares how active a provider's appointment schedule is vs 
// how many prescriptions they generate for those same patients —
// revealing providers who diagnose and treat vs those who primarily refer
MATCH (pat:Patient)-[:has_appointment]->(appt:Appointment)<-[:sees]-(prov:Provider)
MATCH (prov)-[:prescribes]->(rx:Prescription)
MATCH (pat)-[:diagnosed_with]->(dx:Diagnosis)-[:treated_by]->(rx)
RETURN prov.name                      AS provider_name,
       prov.specialty,
       count(DISTINCT pat)            AS unique_patients,
       count(DISTINCT appt)           AS total_appointments,
       count(DISTINCT rx)             AS prescriptions_for_own_patients
GROUP BY prov.name, prov.specialty
ORDER BY prescriptions_for_own_patients DESC
```

---

## Quick Reference

| Query | Hops | Entities Traversed |
|---|---|---|
| 1 — Patient appointments | 1 | Patient → Appointment |
| 2 — Provider diagnoses by specialty | 1 | Provider → Diagnosis |
| 3 — Prescriptions by ICD code | 1 | Diagnosis → Prescription |
| 4 — Appointments by department | 1 | Provider → Appointment |
| 5 — Prescription count per provider | 1 | Provider → Prescription |
| 6 — Providers seen by each patient | 2 | Patient → Appointment ← Provider |
| 7 — Patient medication trail | 2 | Patient → Diagnosis → Prescription |
| 8 — Provider prescribing output | 2 | Provider → Diagnosis → Prescription |
| 9 — Diagnoses by provider-patient chain | 3 | Patient → Appointment ← Provider → Diagnosis |
| 10 — High-severity treatments | 3 | Provider → Diagnosis → Prescription |
| 11 — Prescription burden per patient | 2 + agg | Patient → Diagnosis → Prescription |
| 12 — Shared provider (appts + diagnoses) | 3 dual-path | Patient ↔ Appointment/Diagnosis ↔ Provider |
| 13 — Most-used medications by reach | 2 + agg | Patient → Diagnosis → Prescription |
| 14 — Full care chain | 4 | Patient → Appointment ← Provider → Prescription + Diagnosis |
| 15 — Appointment vs prescription rate | 4 + agg | All 5 entities |
