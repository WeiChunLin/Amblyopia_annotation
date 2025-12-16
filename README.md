# INCEpTION 38.6 User Manual  
## Amblyopia KG Annotation  
*(ClinicalConceptMention + ClinicalRelation)*

---
## Convert your tables into a KB format INCEpTION understands

INCEpTION supports:

SKOS / RDF (TTL) ← best



## What this project is doing (conceptually)

You are creating a **document-level mini knowledge graph (KG)** from clinical notes.

### Nodes = `ClinicalConceptMention` spans
Each node is a span in the text mapped to a **Knowledge Base (KB) concept IRI** (entity linking) and may include attributes:

- **assertion_status**: `present` / `absent` / `suspected` / `resolved`
- **temporality**: `current` / `history` / `planned`
- **laterality**: `left` / `right` / `bilateral`

### Edges = `ClinicalRelation` arcs
Each edge connects two nodes and must have a **relation_type** (from a tagset), such as:

- `has_treatment`
- `has_adherence`
- `has_outcome`
- `has_followup_interval`
- `has_followup_plan`
- `has_severity`
- `associated_with`

This structure supports the **R21-style pipeline**: consistent extraction of diagnosis, treatment, adherence, outcome, and follow-up from unstructured notes, and downstream training of **NER + entity linking + relation extraction models**.

---

## 1) Import the shared project (best way to share settings)

### 1.1 Where **Import Project** is in INCEpTION 38.6

**Import Project does NOT appear inside an open project.**  
It appears only at the global level:

If a collaborator doesn’t see it, common reasons are:
- They are still inside an open project
- They lack permissions (often requires Admin or Project Manager role)

### 1.2 What to share with collaborators

Share **two files**:

1. **Project backup archive (`.zip`)** exported from INCEpTION  
2. **`amblyopia_kb.skos.ttl`** (KB content file)

> Some installations include KB content inconsistently in project export,  
> so the TTL is the safest “source of truth.”

---

## 2) Knowledge Base setup (`Amblyopia_KG`) — exact steps for 38.6

### 2.1 Create or verify the KB exists


**Recommended values**
- **Name**: `Amblyopia_KG`
- **Language**: `en`
- **Type**: `Local`
- **Read-only**: ON (once finalized)
- **Base prefix**: your chosen prefix  
  (e.g., `http://example.org/amblyopia-kb/`)

---

### 2.2 Schema mapping (SKOS)

Configure SKOS so INCEpTION understands labels, hierarchy, and descriptions.

**Recommended mapping**
- **Class IRI**: `skos:Concept`
- **Label IRI**: `skos:prefLabel`
- **Hierarchy / broader**: `skos:broader`
- **Description IRI**:
  - `skos:definition` (if your TTL uses this)
  - OR `rdfs:comment` (if your TTL uses this)
- **Synonyms**: `skos:altLabel`

---

### 2.3 Import KB content (TTL)

amblyopia_kb.skos.ttl
---

### 2.4 Make synonyms searchable (CRITICAL)

If your TTL uses `skos:altLabel` for synonyms:


Save, then verify in the annotation UI by typing a synonym and confirming it returns the correct concept.

---

## 3) Tagsets — exact setup (and why they matter)

### Why Tagsets are **CRITICAL** in KG annotation

Tagsets prevent **free-text chaos**.

- **For nodes**: attributes become analyzable  
  - “no amblyopia” ≠ amblyopia  
  - “plan to patch” ≠ patching now
- **For edges**: `relation_type` is literally the **predicate vocabulary** of the KG  
  - Without it, relations are not machine-readable

---

### 3.1 Create tagsets in INCEpTION 38.6


> In 38.6, the tag list often appears **below on the same page** after saving.  
> If you don’t see “Add tag”, **scroll inside the panel**.

---

### Required Tagsets (exact values)

#### `assertion_status`
- `present`
- `absent`
- `suspected`
- `resolved`

#### `temporality`
- `current`
- `history`
- `planned`

#### `laterality`
- `left`
- `right`
- `bilateral`

#### `relation_type` (**CRITICAL**)
- `has_severity`
- `has_treatment`
- `has_adherence`
- `has_outcome`
- `has_followup_interval`
- `has_followup_plan`
- `associated_with`

**Recommendation**
- Keep **“Annotators may add new tags” OFF** to prevent drift

---

## 4) Layers + Features — exact setup

### 4.1 Layer: `ClinicalConceptMention` (Span)


- **Name**: `ClinicalConceptMention`
- **Type**: `Span`
- **Enabled**: ON

#### Recommended layer behavior
- **Granularity**: Token-level (best export stability)
- **Overlap**: Any
- **Allow crossing sentence boundaries**: ON (optional but helpful)

---

### Features for `ClinicalConceptMention`

#### Feature 1: `concept`
- **Type**: KB: Concept/Instance/Property
- **KB**: `Amblyopia_KG`
- **Required**: ON
- **Curatable**: ON
- **Visibility**: Show in label ON

#### Feature 2: `assertion_status`
- **Type**: Primitive String
- **Tagset**: `assertion_status`
- **Editor**: Radio group
- **Required**: usually OFF

#### Feature 3: `temporality`
- **Type**: Primitive String
- **Tagset**: `temporality`
- **Editor**: Radio group

#### Feature 4: `laterality`
- **Type**: Primitive String
- **Tagset**: `laterality`
- **Editor**: Radio group

---

### Options & Visibility (recommended defaults)

**Options**
- Enabled: ✅
- Required:
  - ✅ for `concept`
  - Optional for others
- Remember: ✅
- Curatable: ✅

**Visibility**
- Show in label:  
  - ✅ for `concept`  
  - Optional for attributes
- Show on hover: ✅ (good compromise)

---

### 4.2 Layer: `ClinicalRelation` (Relation)


- **Name**: `ClinicalRelation`
- **Type**: `Relation`
- **Attach to layer**: `ClinicalConceptMention` ✅

#### Critical relation behavior
- **Overlap**: Any ✅
- **Allow crossing sentence boundaries**: ON ✅

#### Feature: `relation_type`
- **Type**: Primitive String
- **Tagset**: `relation_type`
- **Required**: ON
- **Visibility**: Show in label ON

---

## 5) Annotation workflow

### 5.1 Import documents


- **Format**: Plain text
- Upload `.txt`

Then:
### 5.2 Create nodes (`ClinicalConceptMention`)

- Highlight token(s) (token-aligned spans preferred)
- Ensure layer = `ClinicalConceptMention`
- Set:
  - `concept` (KB search → select IRI)
  - optional `assertion_status`, `temporality`, `laterality`

**Span hygiene**
- Avoid trailing punctuation
- Prefer minimal spans (e.g., “patching”, not whole sentence)
- If two concepts appear, create two mentions

---

### 5.3 Create edges (`ClinicalRelation`)

- Ensure both nodes exist
- Create relation from **source → target**
- Set `relation_type`

If you see:
Cannot create another annotation … no overlap or stacking is allowed
set 
Project → Layers → ClinicalRelation → Overlap = Any

## 6) Output / Export — what to use and why

### 6.1 Best option for full KG export

**Use:**
UIMA CAS XMI (XML 1.1)
- **Secondary format**: `UIMA CAS XMI (XML 1.1)` (recommended)

This shares:
- KB configuration
- tagsets
- layers & features
- documents/annotations (depending on options)

**Also share separately**:
- `amblyopia_kb.skos.ttl`

---

### 6.3 Why WebAnno TSV may fail

TSV often fails when:
- relations cross sentence boundaries
- tokenization cannot define a stable ID unit

➡️ For KG (nodes + edges), **prefer XMI**.
