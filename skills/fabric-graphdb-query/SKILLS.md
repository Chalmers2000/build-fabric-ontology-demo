---
name: fabric-graph-queries
description: Generates a Query.md file with 15 sample GQL queries (5 single-hop, 10 multi-hop) for a Microsoft Fabric Graph database built from a Fabric Ontology. Trigger when the user asks to create graph queries, a query file, or sample GQL for a Fabric ontology.
allowed-tools: Read Write WebFetch
argument-hint: [domain or ontology name]
---

# Skill: Generate Graph Queries for Microsoft Fabric

## Overview

Use this skill whenever you need to create a `Query.md` file containing sample GQL (Graph Query Language) queries for a Microsoft Fabric Graph database built from a Fabric Ontology.

---

## When to Apply This Skill

- A Fabric Ontology has been defined and a Graph database has been created from it.
- The user needs sample queries to demonstrate the ontology's graph capabilities in the Fabric graph query editor.
- The user requests a query file for a domain (healthcare, finance, retail, etc.).

---

## Output Requirements

Generate a `Query.md` markdown file with **15 sample GQL queries** structured as follows:

| Category | Count | Description |
|---|---|---|
| Single-hop queries | 5 | Traverse exactly one relationship |
| Multi-hop queries | 10 | Chain two or more relationships |

### Query File Structure

```
# <Domain> Ontology — Sample Graph Queries (GQL)

<Brief intro paragraph>

**Ontology entities:** <list all node types>

**Relationship types:**
| Relationship | Direction |
|---|---|
| `<rel>` | NodeA → NodeB |
...

---

## Single-Hop Queries
...

## Multi-Hop Queries
...

## Quick Reference
<summary table of all 15 queries>
```

---

## Rules for Writing Each Query

1. **Use GQL syntax** — not Cypher, not SQL. Fabric Graph uses ISO GQL.
2. **Open each query with a `//` comment block** that explains:
   - Which relationship(s) are traversed
   - What the query is useful for (one line)
3. **No `LIMIT` clause** — omit entirely when working with a local/demo dataset.
4. **Use `FILTER`** (not `WHERE`) for predicate conditions.
5. **Use `GROUP BY`** when aggregating with `count()`.
6. **Use `RETURN DISTINCT`** when duplicate rows are possible across multi-path traversals.
7. **Label return columns** with `AS <readable_name>` for clarity.
8. **Add an `ORDER BY`** clause wherever a meaningful sort exists.

---

## Single-Hop Query Patterns

Each single-hop query traverses **one relationship** between two node types.

```gql
// Traverses ONE relationship: NodeA -[:RELATIONSHIP]-> NodeB
// Useful for: <purpose>
MATCH (a:NodeA)-[:RELATIONSHIP]->(b:NodeB)
FILTER a.<property> = '<value>'
RETURN a.<prop> AS label, b.<prop> AS label
ORDER BY b.<prop> ASC
```

**Typical single-hop patterns to cover:**
- A node filtered by a property → its directly connected node
- A node → its connected nodes, aggregated and ranked (e.g., count per provider)
- Both directions of the most important relationships in the ontology

---

## Multi-Hop Query Patterns

Multi-hop queries chain two or more relationships. Each should highlight **why graphs are superior to SQL joins** for this traversal.

### 2-Hop Pattern

```gql
// Traverses TWO relationships: A -[:R1]-> B -[:R2]-> C
// Power of graphs: <explain the graph advantage>
MATCH (a:A)-[:R1]->(b:B)-[:R2]->(c:C)
RETURN a.<prop> AS label, b.<prop> AS label, c.<prop> AS label
ORDER BY a.<prop>
```

### 3-Hop Pattern (two MATCH clauses)

```gql
// Traverses THREE relationships across FOUR entities:
//   A -[:R1]-> B <-[:R2]- C -[:R3]-> D
MATCH (a:A)-[:R1]->(b:B)<-[:R2]-(c:C)
MATCH (c)-[:R3]->(d:D)
RETURN DISTINCT a.<prop>, c.<prop>, d.<prop>
ORDER BY a.<prop>, c.<prop>
```

### Dual-Path (same variable reuse)

```gql
// Two paths that converge on the same node variable:
//   A -[:R1]-> B <-[:R2]- C
//   A -[:R3]-> D <-[:R4]- C  (same C!)
MATCH (a:A)-[:R1]->(b:B)<-[:R2]-(c:C)
MATCH (a)-[:R3]->(d:D)<-[:R4]-(c)
RETURN a.<prop>, c.<prop>, count(DISTINCT b), count(DISTINCT d)
GROUP BY a.<prop>, c.<prop>
```

---

## Aggregation Patterns

Use these when a query needs counts or rankings:

```gql
RETURN <node>.<prop>,
       count(<other_node>)      AS total_<something>,
       count(DISTINCT <node2>)  AS unique_<something>
GROUP BY <node>.<prop>
ORDER BY total_<something> DESC
```

---

## Quick Reference Table (include at end of Query.md)

```markdown
| Query | Hops | Entities Traversed |
|---|---|---|
| 1 — <title> | 1 | NodeA → NodeB |
| 2 — <title> | 1 | NodeA → NodeB |
| 3 — <title> | 1 | NodeA → NodeB |
| 4 — <title> | 1 | NodeA → NodeB |
| 5 — <title> | 1 | NodeA → NodeB |
| 6 — <title> | 2 | NodeA → NodeB → NodeC |
...
| 15 — <title> | 4 + agg | All N entities |
```

---

## Example: Healthcare Ontology

See [query.md](../query.md) for a complete worked example using the following ontology:

- **Entities:** `Patient` · `Provider` · `Appointment` · `Diagnosis` · `Prescription`
- **Relationships:** `has_appointment` · `sees` · `diagnosed_with` · `diagnoses` · `treated_by` · `prescribes`

The healthcare example includes:
- 5 single-hop queries (Queries 1–5)
- 10 multi-hop queries (Queries 6–15), including 2-hop, 3-hop, 4-hop, aggregation, and dual-path patterns

---

## Resources

- [Tutorial: Query Fabric Graph database](https://learn.microsoft.com/en-us/fabric/graph/tutorial-query-code-editor)
- [GQL Quick Reference Guide](https://learn.microsoft.com/en-us/fabric/graph/gql-reference-abridged)
- [GQL Language Guide for Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/graph/gql-language-guide)
