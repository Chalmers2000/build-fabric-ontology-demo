# Fabric Data Agent Prompts for Healthcare Ontology

Use these prompts in Fabric Data Agent to generate GQL aligned to the 15 sample queries.

## Single-Hop Prompts (Queries 1-5)

Single-hop means the query traverses exactly one relationship edge in the graph.

1. Show me all appointments for patients whose names begin with Liam.
2. Show me all diagnoses made by cardiology providers.
3. Show me all prescriptions used to treat diagnosis code I10.
4. Show me all appointments handled by providers in the Emergency department.
5. Rank providers by how many prescriptions they have written.

## Multi-Hop Prompts (Queries 6-15)

Multi-hop means the query traverses two or more relationship edges to connect entities through intermediate nodes.

6. For each patient, show which providers they have seen and each provider's specialty.
7. For each patient, show their diagnoses and the medications prescribed for those diagnoses.
8. For each provider, show how many diagnoses they made and how many prescriptions came from those diagnoses.
9. For each patient, show diagnoses made by providers that the patient has had appointments with.
10. Show high-severity diagnoses and the treatments prescribed for them, including the diagnosing provider.
11. For each patient, show the number of distinct diagnoses and total prescriptions.
12. Show patients who have the same provider for both appointments and diagnoses, and count those appointments and diagnoses.
13. Rank medications by how many unique patients receive them, and show how many diagnoses each medication treats.
14. Show the full care chain for each patient: appointment, provider, diagnosis, and prescription.
15. For each provider, compare appointment volume and prescription volume for the same patients they saw.
