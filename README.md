# Amblyopia_annotation
INCEpTION 38.6 User Manual
Amblyopia KG annotation (ClinicalConceptMention + ClinicalRelation)
What this project is doing (conceptually)

You are creating a document-level mini knowledge graph (KG) from clinical notes:

Nodes = ClinicalConceptMention spans
Each node is a mention in text mapped to a KB concept IRI (entity linking) and optionally has attributes:

assertion_status (present/absent/suspected/resolved)

temporality (current/history/planned)

laterality (left/right/bilateral)

Edges = ClinicalRelation arcs
Each edge connects two nodes and must have a relation_type (tagset) such as:

has_treatment, has_adherence, has_outcome, has_followup_interval, has_followup_plan, etc.

This structure is exactly what you need for the R21-style pipeline: consistent extraction of diagnosis/treatment/adherence/outcome/follow-up from messy notes, and later training NER+linking+relation extraction models.

1) Import the shared project (best way to share settings)
1.1 Where “Import Project” is in 38.6

Import Project does not appear inside an open project.
It appears only at the global level:

Top menu → Dashboard (or Projects) → Import Project

If a collaborator doesn’t see it, it’s almost always one of:

They are still inside an open project page

They don’t have permission (often needs Admin / Project Manager role)

1.2 What to share with collaborators

Share two things:

Project backup archive (.zip) from INCEpTION

amblyopia_kb.skos.ttl (KB content file)

Some installations include KB content in export inconsistently, so TTL is the safest “source of truth.”

2) Knowledge Base setup (Amblyopia_KG) — exact steps for 38.6
2.1 Create or verify KB exists

Project → Knowledge Bases → Create

Recommended values:

Name: Amblyopia_KG

Language: en

Type: Local

Read-only: ON (once finalized)

Base prefix: your chosen prefix (example: http://example.org/amblyopia-kb/)

2.2 Schema mapping (SKOS)

Set the KB to SKOS-style mapping so INCEpTION knows what label/definition/hierarchy mean:

Recommended mapping:

Class IRI: skos:Concept

Label IRI: skos:prefLabel

Hierarchy / broader: skos:broader

Description IRI:

Use skos:definition if your TTL stores definitions there

Or use rdfs:comment if your TTL uses that

Synonyms: skos:altLabel (see next section)

2.3 Import KB content (TTL)

Project → Knowledge Bases → Amblyopia_KG → Import content → Import knowledge base…
Select: amblyopia_kb.skos.ttl

2.4 Make synonyms searchable (critical)

If you have synonyms in skos:altLabel, add it to matching:

Amblyopia_KG settings → Additional matching properties → add:

http://www.w3.org/2004/02/skos/core#altLabel

Save → verify in annotation UI by typing a synonym and checking it returns the intended concept.

3) Tagsets — exact setup (and why they matter for KG)
Why Tagsets are “CRITICAL” in KG annotation

Tagsets are what prevent “free text chaos” in attributes/edges.

For nodes, tagsets make attributes analyzable:

assertion and temporality are essential for clinical truth conditions
(e.g., “no amblyopia” ≠ amblyopia; “plan to patch” ≠ patching now)

For edges, tagsets are literally the predicate vocabulary of the KG:

relation_type is your KG edge label set

Without it, relations are not machine-readable.

3.1 Create tagsets in 38.6 (UI reality)

Project → Tagsets → Create

In 38.6, after saving the tagset, the tag list is often below on the same page.

If you don’t “see Add tag”: scroll inside the right panel/page.

Sometimes the tags table appears only after Save.

Create these Tagsets (exact values)

Tagset: assertion_status

present

absent

suspected

resolved

Tagset: temporality

current

history

planned

Tagset: laterality

left

right

bilateral

Tagset: relation_type (CRITICAL)
Include all you plan to annotate (minimum set you’ve used):

has_severity

has_treatment

has_adherence

has_outcome

has_followup_interval

has_followup_plan

associated_with

Recommendation:

Keep “Annotators may add new tags” OFF (prevents drift)

4) Layers + Features — exact setup you used
4.1 Layer: ClinicalConceptMention (Span)

Project → Layers → Create

Name: ClinicalConceptMention

Type: Span

Enable it

Recommended layer behavior

Granularity: Token-level (best export stability)

Overlap: Any (safe; clinical text often needs nested mentions)

Optional: “Allow crossing sentence boundaries” ON (span layer only)

Add features to ClinicalConceptMention

Feature 1: concept

Type: KB: Concept/Instance/Property

KB: Amblyopia_KG

Required: ON

Curatable: ON

Visibility: Show in label ON (helpful)

Feature 2: assertion_status

Type: Primitive String

Tagset: assertion_status

Editor type: Radio group (small tagsets)

Required: usually OFF (unless you enforce always)

Feature 3: temporality

Type: Primitive String

Tagset: temporality

Editor type: Radio group

Feature 4: laterality

Type: Primitive String

Tagset: laterality

Editor type: Radio group

Options + Visibility (your “what should we choose?”)

Recommended defaults for your project:

Options

Enabled: ✅

Required:

✅ for concept

Optional for others (depends on guideline strictness)

Remember: optional ✅ (speeds annotation when many mentions share the same value)

Curatable: ✅ (allows manual correction when recommenders are used later)

Visibility

Show in label:

✅ for concept (always)

Optional for attributes (can clutter labels)

Show on hover:

✅ good compromise if labels get too busy

4.2 Layer: ClinicalRelation (Relation)

Project → Layers → Create

Name: ClinicalRelation

Type: Relation

Attach to layer: ClinicalConceptMention ✅ (must)

Critical relation layer behavior

Overlap: Any ✅
(prevents the “Cannot create another relation annotation… stacking not allowed” pain)

Allow crossing sentence boundaries: ON ✅
(your export failures + cross-line relations strongly suggest keeping this ON)

Add feature: relation_type (CRITICAL)

Name: relation_type

Type: Primitive String

Tagset: relation_type

Required: ON

Visibility: Show in label ON (so arcs display the relation type)

5) How to annotate (day-to-day workflow)
5.1 Import a sample note file

Project → Documents → Import

Format: Plain text

Upload .txt

Then:
Annotation → open document

5.2 Create nodes (ClinicalConceptMention)

Highlight token(s) (prefer token-aligned spans)

Ensure layer = ClinicalConceptMention

Set:

concept (KB search → pick correct IRI)

optionally laterality/assertion/temporality

Span hygiene rules

Don’t include trailing punctuation if possible

Prefer minimal mention span (“patching” not whole sentence)

If two concepts appear, create two mentions

5.3 Create edges (ClinicalRelation)

Make sure both nodes exist

Create a relation from source → target

Set relation_type

If you get:

“Cannot create another annotation … no overlap or stacking is allowed”

Fix:

Project → Layers → ClinicalRelation → Overlap = Any

6) Output / Export — what to use and why
6.1 Best option for full KG: XMI (what you’re already doing)

Use:

UIMA CAS XMI (XML 1.1)

Why:

preserves spans + attributes + relations

supports cross-sentence / cross-line relations robustly

6.2 Export “project settings” to another user

Use the project backup archive export:
Project → Export backup archive

Secondary format: UIMA CAS XMI (XML 1.1) recommended

This is the best method to share:

KB config (schema mapping etc.)

tagsets

layers and features

documents/annotations (depending on export options)

Also share the TTL KB file separately (amblyopia_kb.skos.ttl) for safety.

6.3 Why WebAnno TSV was failing for you

Your error pattern is consistent with TSV struggling when:

relations cross sentence boundaries / lines

the export can’t find an “ID-defining unit” in the expected tokenization constraints

So for KG (nodes+edges), prefer XMI.
