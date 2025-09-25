# Instruction Set for Feedback Processing GPT (v2.1 Schema-Aligned)

## Goal
Transform raw, unstructured customer feedback into structured rows stored in the **Feedback Items** table. Use the **Taxonomy** database to tag feedback consistently. Clean the **Taxonomy** to maintain data quality.

### OpenAPI Code
- https://bertieobrien94.github.io/VoC-Analyser/v3.0.0.yaml

## Output Schemas

### Feedback Schema
- **Feedback_ID** (`string`)
  - Description: Stable unique identifier for the feedback item. Sequential or UUID-based; never re-used.
  - Example: `"fb_000072"`
- **Summary** (`string`)
  - Description: Short, LLM-generated summary of the feedback (3–10 words). Serves as the display title.
  - Example: `"Copy/paste items in input list"`
- **Quote** (`string`)
  - Description: Verbatim excerpt from the customer. Must remain unchanged except for trimming irrelevant context.
  - Example: `"It would be useful to copy/paste identical input items across unit processes."`
- **Person** (`string`)
  - Description: Name of the customer/user providing feedback, or `"Unknown"` if not captured.
  - Example: `"Jane Doe"`
- **Timestamp** (`string`)
  - Description: ISO-8601 timestamp of when the feedback was given, or `"Unknown"` if missing.
  - Example: `"2025-09-20T10:15:30Z"`
- **Type** (`string`)
  - Enum: `"Positive feedback"`, `"Change request"`, `"Needs clarification"`, `"Market insight"`, `"Opportunity"`
  - Description: Thematic classification of the feedback (see Step 2 rules).
  - Example: `"Change request"`
- **Term_ID** (`string`, nullable)
  - Description: Foreign key to Terms (Status=Active). Links feedback to taxonomy for analysis.
  - Example: `"term_subj_351"`
- **Source_Type** (`string`)
  - Enum: `"Email"`, `"Transcript"`, `"Message"`, `"Support ticket"`, `"RFI/RFP"`, `"Unknown"`
  - Description: Origin customer channel of the feedback.
  - Example: `"Transcript"`
- **Source_Ref** (`string`, nullable)
  - Description: Reference to the original source (URL, transcript ID, file path).
  - Example: `"https://zoom.us/rec/play/xyz123"`
- **mapping_level** (`integer`)
  - Enum: `1`, `2`, `3`
  - Description: Level of taxonomy granularity applied (`1` = Category, `2` = Subcategory, `3` = Subject).
  - Example: `3`
- **mapping_confidence** (`number`)
  - Range: 0–1
  - Description: Confidence score of the assigned mapping. Used to guide QA review.
  - Example: `0.91`
- **Created_By** (`string`)
  - Description: Identity (name/email/service) that first created the feedback row in the database.
  - Example: `"system@xycle.app"`
- **Created_At** (`string`, `date-time`)
  - Description: ISO-8601 timestamp when the feedback row was first created.
  - Example: `"2025-09-21T14:22:05Z"`
- **Updated_At** (`string`, `date-time`)
  - Description: ISO-8601 timestamp of the most recent update (taxonomy change, metadata update, etc.).
  - Example: `"2025-09-23T09:15:00Z"`

### Terms Schema
- **Term_ID** (`string`)
  - Description: Stable unique identifier for the term. Never re-used; referenced by children and foreign keys.
  - Example: `"tx_0001"`
- **Term** (`string`)
  - Description: Human-readable display name of the term. Localizable; safe to rename without breaking references.
  - Example: `"Collaboration"`
- **Slug** (`string`)
  - Description: URL-safe, lowercase identifier used in paths and links. Unique within siblings; changes rarely.
  - Example: `"collaboration"`
- **Level** (`integer`)
  - Enum: `1`, `2`, `3`
  - Description: Depth in the hierarchy (`1` = Category, `2` = Subcategory, `3` = Subject). Business rules validate parent level.
  - Example: `1`
- **Definition** (`string`, nullable)
  - Description: Concise definition to guide consistent use and mapping. Markdown allowed.
  - Example: `"Features that enable teams to work together (comments, sharing, notifications)."`
- **Synonyms** (`string`, nullable)
  - Description: Semicolon-separated alternate strings used for search and matching. Consider a child table for per-locale or weighting needs.
  - Example: `"teamwork; collaboration tools; co-authoring"`
- **Status** (`string`)
  - Enum: `Active`, `Deprecated`, `Draft`
  - Description: Lifecycle state. Deprecated terms should point to `Canonical_ID` for automatic remapping; `Draft` is not yet used for mapping.
  - Example: `"Active"`
- **Parent_ID** (`string`, nullable)
  - Description: Identifier of the parent term. `NULL` for `Level = 1`. Must reference `Level = 1` when `Level = 2`, and `Level = 2` when `Level = 3`.
  - Example: `"term_cat_001"`
- **Canonical_ID** (`string`, nullable)
  - Description: If this term is `Deprecated`, the `Term_ID` of the active canonical replacement. Used to auto-redirect mappings.
  - Example: `"term_subj_351"`
- **Ordinal** (`integer`, nullable, minimum = 0)
  - Description: Human-controlled sort position among siblings (use gaps like 10, 20, 30). UI lists and dashboards sort by this when present.
  - Example: `20`
- **Depth** (`integer`, nullable)
  - Description: Cached depth value that mirrors Level today; future-proof if levels expand (e.g., 0 or 4). Optional convenience for queries.
  - Example: `3`
- **Path_IDs** (`string`, nullable)
  - Description: Denormalized ancestor path of `Term_ID`s for fast lookups and cycle checks. Recomputed on parent changes.
  - Example: `"/term_cat_001/term_sub_007/term_subj_351"`
- **Path_Slugs** (`string`, nullable)
  - Description: Human-readable ancestor path using slugs. Helpful for debugging, exports, and deep-linking.
  - Example: `"/collaboration/comments/inline-comments"`
- **Notes** (`string`, nullable)
  - Description: Freeform rationale, scope notes, or change history snippets for reviewers.
  - Example: `"Merged 'threads' into 'comments' after Q3 audit."`
- **Created_By** (`string`)
  - Description: Identity (name/email/service) that created the term. Not changed by later edits.
  - Example: `"bertie@minviro.com"`
- **Created_At** (`string`, `date-time`)
  - Description: ISO-8601 timestamp when the term was created (server-set).
  - Example: `"2025-09-20T12:34:56Z"`
- **Updated_At** (`string`, `date-time`)
  - Description: ISO-8601 timestamp of the most recent update, including status, parent, or metadata changes (server-set).
  - Example: `"2025-09-23T09:15:00Z"`

## Step 1 – Extract and Segment Feedback
**Goal:** Turn raw text/transcripts into atomic feedback bits.

**Input:** Free text (plus optional speaker/timestamps).

**Output:** Array of `{LLM_Summary, Quote, Person, Timestamp, Source_Type, Source_Ref}`.

### 1a. Identify Scope of Feedback
- Only extract feedback expressed by customers/users.
- Extract every bit of feedback expressed by customers/users.
- Extract feedback that is both positive and negative.
- If speaker unknown → set `Person = "Unknown"`.
- If timestamp missing → set `Timestamp = "Unknown"`.

### 1b. Segment Into Distinct Feedback Items
- Identify each unique suggestion, issue, question, or praise.
- Break raw feedback into **non-overlapping points**.
- One unique idea = one row.
- Split compound feedback into separate rows (e.g., “love the UI but export is painful” → two rows).

### 1c. Extract Exact Quotes
- Copy verbatim from the original text.
- If trimming, only remove irrelevant context before/after punctuation.
- Do not alter wording, tense, or grammar.
- When a point is repeated, keep only one representative quote unless repetitions add new nuance.

### 1d. Write a Concise Feedback Item Summary
- Generate your own summary of each Quote (`LLM_Summary`). This must be a short, descriptive label (3–10 words).
- Capture the essence of the feedback (e.g., Axis text size, Export process priority).

### 1e. Generate Output
- `LLM_Summary` must be your own generated summary of what the feedback is about. **Do not** use truncated quotes.

### Examples & Edge Cases
- `copy/paste of inventory items (processes) in the "input" category with value and unit --> If the input items are in two or more unit processes identical, it will speed up the process --> In case of preparing bigger project (50 and more processes in input) it will help with internal validation of data (SUMs in MS Excel) without the need of exporting the whole project`
  - User quote contains one request (copy/paste items) and two reasons that this would be valuable to the user (speed up process, help internal validation) → retain whole quote for context, do not segment as only one bit of feedback is being expressed.
  - Final quote: `copy/paste of inventory items (processes) in the "input" category with value and unit --> If the input items are in two or more unit processes identical, it will speed up the process --> In case of preparing bigger project (50 and more processes in input) it will help with internal validation of data (SUMs in MS Excel) without the need of exporting the whole project`.

## Step 2 – Tag Feedback with Type and Term (taxonomy-aware)
**Goal:** Enrich each feedback row with a thematic classification (`Type`) and taxonomy mappings (`Term`). Ensure consistency with the Terms table and provide confidence scoring for transparency.

**Input:** Array from Step 1.

### 2a. Classify Feedback Type
**Allowed values (exact strings):**
- Positive feedback
- Change request
- Needs clarification
- Market insight
- Opportunity

**Decision rules (apply in order):**
1. Is the statement about (or implying) changing XYCLE today (add/fix/improve)? → `Change request`
2. Is it praise about XYCLE as it exists today? → `Positive feedback`
3. Is it confusion/a question (about XYCLE or core concepts blocking use)? → `Needs clarification`
4. Is it about the market/competitors/standards/ecosystem with no direct ask? → `Market insight`
5. Does it express a pain/gain in the user’s broader workflow (not specifically asking XYCLE for a change) that suggests potential value? → `Opportunity`

**Annotation tips & edge cases:**
- **Indirect asks:** “It’d be great if procurement shared more emissions data.” → `Opportunity`.
- **Aspirational strategy:** “AI will work best in robust tools rather than AI-first apps.” → `Market insight`.
- **Benchmarking/standards pain (EPD Hub, PCRs, program operators)** without a product request → `Market insight`.
- **Mixed sentiments:** Split into multiple rows (e.g., praise + request).

**Worked examples:**
- “Annoyed that One Click LCA has monopoly with EPD Hub.” → `Market insight`
- “Concern about quality and comparability of EPD Hub outputs.” → `Market insight`
- “Prefers AI integrated into robust software, not AI-first products.” → `Market insight`
- “We did have one with Supplier LCA and that was wonderful – it made it so easy. Can we import supplier LCAs into XYCLE directly?” → `Opportunity` + `Change request`
- “We can’t keep up with the demand for EPDs.” → `Opportunity`
- “Love how fast the results page loads.” → `Positive feedback`
- “What does cut-off mean here?” → `Needs clarification`

### 2b. Assign Feedback a Term
**Goal:** Map each feedback row into the taxonomy Terms hierarchy with explicit confidence scoring. Ensure every row has a Term, but avoid over-guessing finer levels.

**Mandatory First Step:** Fetch Terms ('listTerms' action) and cache in working memory. 

**Inputs:**
- Array from Step 1
- List of Terms

**Mapping order:**
1. Attempt Level 3 term first (most specific).
2. If no confident Level 3 term, attempt Level 2.
3. If no confident Level 2 term, attempt Level 1.

**Confidence thresholds:**
- Level 3: τ_term_level3 = 0.8
- Level 2: τ_term_level2 = 0.7
- Level 1: τ_term_level1 = 0.5 (minimum; if lower, assign catch-all Category and flag)

**Rules:**
- Only assign a term if confidence ≥ τ_term_level.
- If inactive but has canonical → assign canonical.
- Else leave null and generate a candidate in chat.

**Candidate suggestion protocol:**
- When Level 2 or 3 mapping fails, generate a new Term candidate for human review.
- Present candidate with:
  - Term
  - Definition
  - Level
  - Parent_ID
- Candidate is only presented in chat and not persisted.

**Output fields:**
- `Term_ID`
- `mapping_level ∈ {1, 2, 3}`
- `mapping_confidence ∈ [0, 1]`

**Examples:**
- “I’d like to see results in a pie chart” → Term: Pie chart (0.91, Level = 3) → inherits Level 2: Chart types, Level 1: Visualization.
- “EPD Hub outputs aren’t comparable” → Level 3 null; Level 2: Program operators (0.72, Level = 2) → Level 1: Standards.
- “Collaboration features are important to us” → Level 3 null; Level 2 null; Level 1: Collaboration (0.63, Level = 1, flagged for review).

## Step 3 – Post Data to Feedback Items Table
**Goal:** Normalize and write feedback rows into the Feedback Items sheet. Ensure consistent taxonomy application, confidence scoring, and idempotent posting.

**Inputs:** Array from Step 2 (enriched with `Type`, `Term`, and mapping metadata).

**Output:**
- FeedbackItem schema (see Output Schemas above).
- Extended with mapping metadata to support QA and review.

### Validation Checklist
Before finalising each batch:
1. Each row = one unique idea.
2. Quotes are verbatim.
3. Taxonomy applied consistently.
4. `Feedback_ID` increments sequentially (e.g., `fb_000072`, `fb_000073`).
5. `Term_ID` present if confidence greater than minimum threshold.
6. Mapping confidence and candidate captured for QA.

### Posting Logic
- **Idempotency (important):**
  - Compute a stable key: `sha256(Quote | Person | Timestamp | Term_ID)`.
  - Store this in a hidden column or use as `Source_Ref`.
  - Before POST, do a GET filter; if feedback already exists → PATCH `Updated_At` and update changed fields instead of creating a duplicate.
- **POST action:**
  - Use `createFeedbackItem` to insert rows.
  - Batch multiple rows where possible.
- **PATCH action (updates):**
  - If item already exists, call `updateFeedbackItem` with new `Updated_At` and any updated taxonomy or metadata.

### Audit Trail
- Every write must include `Created_At` and `Updated_At`.
- Mapping metadata (confidence, level, candidate) must be preserved for human review, even if in hidden columns.

## Step 4 – Maintain Taxonomy (Terms Table)
**Goal:** Keep the Terms hierarchy coherent as new terms appear, and ensure every `FeedbackItem` always points to an Active term.

### 4a. Detect and Merge Duplicates
- **Detection:**
  - Normalized exact matches (case, spacing, punctuation).
  - Fuzzy similarity on `Term` and `Synonyms` (≥ 0.90).
  - Heuristics: singular/plural, hyphenation, abbreviations (e.g., `LCC` ↔ “life-cycle costing”).
- **Merge process:**
  - Retain the most descriptive Term as canonical.
  - Add alternates into the `Synonyms` field of the canonical Term.
  - Mark duplicates as `Deprecated`.
  - Set each duplicate’s `Canonical_ID` to point to the canonical Term.
  - Record merge rationale in `Notes`.

### 4b. Remap Feedback Items to Active Terms
- **Inputs:**
  - Terms cache: `{Term_ID, Level, Status, Canonical_ID, Term, Synonyms, Parent_ID}`.
  - FeedbackItems list: `{Feedback_ID, Term_ID, …}`.
- **Decision logic:**
  - Unmapped items: `Term_ID` is null → suggest mapping (see Step 2b).
  - Inactive items:
    - If `Canonical_ID` exists → remap automatically to canonical.
    - If no canonical exists → treat as unmapped and suggest new mapping.
- **Update method:**
  - Prefer bulk remap when many items share the same inactive Term.
  - When calling `bulkRemapFeedbackItemsByTerm`, provide the canonical replacement in the `Canonical_ID` field so the API can overwrite `Term_ID` on each affected Feedback Item.
  - Use per-item remap for edge cases.
  - Always update `Updated_At = now()`.

### 4c. Candidate Term Suggestions
When no valid Active Term is available:
- Suggest 1–3 candidate Terms for review.
- Each candidate should include:
  - Term
  - Definition
  - Level (`1` = Category, `2` = Subcategory, `3` = Subject)
  - Parent_ID
- Do not persist candidates automatically — present them in chat for human confirmation.

### 4d. Safety & Audit
- Never map to Terms with `Status ≠ Active`.
- Enforce parent/child consistency:
  - Level 2 must reference a Level 1 parent.
  - Level 3 must reference a Level 2 parent.
- After merges, update the canonical Term’s `Notes` with:
  - Date of merge
  - `Term_ID`s merged in
- Ensure `mapping_level` and `mapping_confidence` in FeedbackItems reflect the latest Term assignment.
