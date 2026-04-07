# Step-by-Step Guide: Build the Healthcare Ontology in Microsoft Fabric

**Who this guide is for:** Anyone who can open a web browser and log into Microsoft Fabric. No technical experience required.

**What you will build:** A Healthcare ontology that connects five types of data — Patients, Providers, Appointments, Diagnoses, and Prescriptions — so Fabric understands how they relate to each other.

**Before you start:** Make sure you have already run the five data-loading notebooks (01 through 05) so your Lakehouse tables exist. You will need:
- Access to a Microsoft Fabric workspace
- The Lakehouse that contains these five tables: `patient`, `provider`, `appointment`, `diagnosis`, `prescription`

---

## Part 1: Open Your Workspace

1. Open a web browser and go to **https://app.fabric.microsoft.com**
2. Sign in with your Microsoft account.
3. In the left-hand navigation panel, click **Workspaces**.
4. Click the name of the workspace where your Lakehouse lives. The workspace opens and you will see a list of items.

---

## Part 2: Create the Ontology Item

Think of an Ontology item as a blank map — you are about to draw all the connections between your data tables onto it.

1. In your workspace, click the **+ New item** button (usually near the top of the page).
2. In the search box that appears, type **Ontology** and press Enter.
3. Click the item called **Ontology (preview)**.
4. A dialog box appears asking for a name. Type exactly:
   ```
   HealthcareSystemOntology
   ```
   > **Tip:** Use no spaces or dashes in the name — only letters, numbers, and underscores are allowed.
5. Click **Create**.
6. Wait a few seconds. The new ontology opens automatically. It will look like a blank canvas.

---

## Part 3: Add the Five Entity Types

An **Entity Type** is just a category of thing — like "Patient" or "Provider." You will create one Entity Type for each of your five tables. The steps are the same for each one.

### 3.1 — Add the Patient Entity Type

1. On the blank canvas (or from the top ribbon), click **Add entity type**.
2. In the dialog that appears, type:
   ```
   Patient
   ```
3. Click **Add Entity Type**. A box labeled "Patient" appears on the canvas, and a configuration panel opens on the right side of the screen.
4. In the right panel, click the **Bindings** tab.
5. Click **Add data to entity type**.
6. A data source picker appears. Find and click your **Lakehouse** name, then click **Connect**.
7. A list of tables appears. Click **patient**, then click **Next**.
8. You will see a list of columns. Leave all settings at their defaults — the columns should already be mapped correctly:

   | Source Column | Property Name |
   |---|---|
   | name | name |
   | patientId | patientId |
   | mrn | mrn |
   | dateOfBirth | dateOfBirth |
   | bloodType | bloodType |
   | allergies | allergies |

9. Click **Save**.
10. Back in the right panel, click **Add entity type key**.
11. From the dropdown, select **patientId** and click **Save**.

    > **Why set a key?** The key is the unique ID that tells Fabric which row is which patient. Think of it like a student ID number at school — no two students share the same one.

---

### 3.2 — Add the Provider Entity Type

Repeat the same steps as above, with these values:

- **Entity type name:** `Provider`
- **Source table:** `provider`
- **Columns to map (leave defaults):** name, providerId, specialty, licenseNumber, department
- **Key property:** `providerId`

---

### 3.3 — Add the Appointment Entity Type

- **Entity type name:** `Appointment`
- **Source table:** `appointment`
- **Columns to map (leave defaults):** appointmentId, patientId, providerId, scheduledTime, duration, type, status
- **Key property:** `appointmentId`

---

### 3.4 — Add the Diagnosis Entity Type

- **Entity type name:** `Diagnosis`
- **Source table:** `diagnosis`
- **Columns to map (leave defaults):** diagnosisId, patientId, providerId, icdCode, description, severity, diagnosedDate
- **Key property:** `diagnosisId`

---

### 3.5 — Add the Prescription Entity Type

- **Entity type name:** `Prescription`
- **Source table:** `prescription`
- **Columns to map (leave defaults):** rxNumber, diagnosisId, providerId, medication, dosage, frequency, refillsRemaining
- **Key property:** `rxNumber`

---

After completing all five, you should see five boxes on the canvas: Patient, Provider, Appointment, Diagnosis, and Prescription.

---

## Part 4: Create the Six Relationship Types

A **Relationship Type** is a line you draw between two entity types that says "these two things are connected." For example, a Patient *has* Appointments.

You will create six relationships. The steps are the same each time.

### How to add a relationship (general steps):

1. From the top ribbon, click **Add relationship**.
2. Fill in the three fields:
   - **Relationship type name** — what the connection is called
   - **Source entity type** — the "starting" box
   - **Target entity type** — the "ending" box
3. Click **Add relationship type**.
4. A configuration panel opens. Fill in these additional fields:
   - **Source data** — select your workspace → your Lakehouse → the table listed below
   - **Source entity type > Source column** — the column that matches the Source entity's key
   - **Target entity type > Source column** — the column that matches the Target entity's key
5. Click **Create**.

---

### Relationship 1: Patient has_appointment → Appointment

| Field | Value |
|---|---|
| Relationship type name | `has_appointment` |
| Source entity type | Patient |
| Target entity type | Appointment |
| Source data table | `appointment` |
| Source entity > Source column | `patientId` |
| Target entity > Source column | `appointmentId` |

> **In plain English:** This line says "a Patient has Appointments." The `appointment` table connects them because each row in that table contains a `patientId` that tells us which patient the appointment belongs to.

---

### Relationship 2: Provider sees → Appointment

| Field | Value |
|---|---|
| Relationship type name | `sees` |
| Source entity type | Provider |
| Target entity type | Appointment |
| Source data table | `appointment` |
| Source entity > Source column | `providerId` |
| Target entity > Source column | `appointmentId` |

---

### Relationship 3: Patient diagnosed_with → Diagnosis

| Field | Value |
|---|---|
| Relationship type name | `diagnosed_with` |
| Source entity type | Patient |
| Target entity type | Diagnosis |
| Source data table | `diagnosis` |
| Source entity > Source column | `patientId` |
| Target entity > Source column | `diagnosisId` |

---

### Relationship 4: Provider diagnoses → Diagnosis

| Field | Value |
|---|---|
| Relationship type name | `diagnoses` |
| Source entity type | Provider |
| Target entity type | Diagnosis |
| Source data table | `diagnosis` |
| Source entity > Source column | `providerId` |
| Target entity > Source column | `diagnosisId` |

---

### Relationship 5: Diagnosis treated_by → Prescription

| Field | Value |
|---|---|
| Relationship type name | `treated_by` |
| Source entity type | Diagnosis |
| Target entity type | Prescription |
| Source data table | `prescription` |
| Source entity > Source column | `diagnosisId` |
| Target entity > Source column | `rxNumber` |

> **In plain English:** This says "a Diagnosis is treated by a Prescription." Every diagnosis in our data has at least one prescription linked to it.

---

### Relationship 6: Provider prescribes → Prescription

| Field | Value |
|---|---|
| Relationship type name | `prescribes` |
| Source entity type | Provider |
| Target entity type | Prescription |
| Source data table | `prescription` |
| Source entity > Source column | `providerId` |
| Target entity > Source column | `rxNumber` |

---

## Part 5: Check Your Work

When you are done, the canvas should show all five entity type boxes connected by six labeled relationship lines. It should look something like this:

```
Patient ──has_appointment──> Appointment <──sees────── Provider
   │                                                       │
   │diagnosed_with                               diagnoses │
   ▼                                                       ▼
Diagnosis ─────────treated_by──────────────────> Prescription
                                                       ▲
                                           prescribes  │
                                                   Provider
```

To confirm everything is connected:
1. Click on any entity type box (e.g., **Patient**).
2. Look at the right panel — you should see its **Bindings** (data columns) and any **Relationships** listed.
3. Click on the **Diagnosis** box to confirm it shows both `diagnosed_with` (from Patient) and `diagnoses` (from Provider) relationships.

---

## Part 6: Save Your Work

Microsoft Fabric saves automatically as you go, but to make sure:

1. Look for a **Save** button in the top ribbon and click it if available.
2. You can also just navigate away — Fabric will confirm the save.

---

## You're Done! 🎉

You have built a Healthcare Ontology in Microsoft Fabric. Here's a summary of what you created:

| What | How Many |
|---|---|
| Entity Types | 5 (Patient, Provider, Appointment, Diagnosis, Prescription) |
| Data Bindings | 5 (one per table in your Lakehouse) |
| Entity Keys | 5 (patientId, providerId, appointmentId, diagnosisId, rxNumber) |
| Relationship Types | 6 |

### What can you do next?

- Use **Fabric IQ** to ask natural language questions about your healthcare data — Fabric will use this ontology to understand what you mean.
- Explore the ontology canvas to see how data flows from patients all the way to their prescriptions.
- Continue to **Tutorial Part 2** on Microsoft Learn to enrich your ontology further:
  https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-2-enrich-ontology
