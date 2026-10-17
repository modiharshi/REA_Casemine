# ELI architecture decisions, informed by the CaseMine investigation

**Status:** proposed, 2026-10-17
**Sources:** [CaseMine architecture report](../reverse-engineering/casemine-architecture-report.md) (Rounds 1–3) and its [evidence ledger](../reverse-engineering/evidence-ledger.md) (F-01 to F-54).

## How to read this document

Each decision record follows the same layout: **context → CaseMine evidence → decision → alternatives considered → open questions**.

- Every statement about CaseMine carries the classification the report gives it: **OBSERVED** (seen in a response, the DOM or shipped JavaScript), **INFERRED** (supported but unproven) or **UNKNOWN**. Each one cites ledger IDs (`F-nn`).
- The **Decision** sections are ELI proposals, not findings. They borrow what CaseMine does well and avoid the failures the investigation measured.
- CaseMine's sample was small (one account, a handful of judgments, 4 AMICUS answers). Percentages quoted from it describe that sample only.

## Summary of decisions

| # | Decision | Key evidence |
|---|---|---|
| ADR-1 | One opaque judgment ID; one paragraph-ID scheme for every document, separate from printed paragraph numbers | F-01, F-02, F-05, F-32, F-36, F-49, F-52 |
| ADR-2 | Judgment record with explicit identity, metadata, text, quality and derived-counter groups | F-01, F-04, F-06, F-14, F-22, F-26 |
| ADR-3 | Citation edges as first-class records with separate treatment and status-mentioned fields, paragraph provenance, review fields, and counts derived from the edge store | F-07–F-14, F-38–F-40 |
| ADR-4 | Deterministic exact-citation resolver, separate from full-text search, with measured coverage and explicit failure states | F-18, F-41–F-43, F-37 |
| ADR-5 | Search with deterministic tie-breaking, explicit caps, and two indexes (judgments, citation excerpts) | F-15–F-19, F-13, F-39 |
| ADR-6 | AI answers whose anchors come from retrieval metadata, validated before rendering; a source panel; per-answer metering | F-20, F-27–F-31, F-35, F-46–F-51, F-30, F-47 |

---

## ADR-1. Canonical judgment IDs and a single paragraph-ID scheme

### Context
Every ELI feature (citation edges, search hits, AI anchors, bookmarks, deep links) addresses either a judgment or a paragraph inside one. If the paragraph address is ambiguous, every downstream link can point to the wrong place without anyone noticing.

### CaseMine evidence
- **OBSERVED:** one 24-hex judgment ID is used everywhere: URL `/judgement/in/{id}`, inline `judgementId`, edge `fjid`/`tjid`, search `id`, graph `id1`/`id2` (F-01).
- **INFERRED (med-high):** the ID is a MongoDB-style ObjectId minted at ingestion; its timestamp prefix trails the judgment date by a median of 6 days, never earlier, n=110 (F-02, F-03).
- **OBSERVED:** paragraphs are addressed by DOM-sequential `p_N` ids; `citedParas` is keyed `{p_n: count}` (F-05).
- **OBSERVED:** `p_N` is not the printed paragraph number: N.N. Global `p_523` is printed ¶212 and `p_797` is printed ¶323 (F-32); PSA Sical's printed ¶55 is id `106` (F-50).
- **OBSERVED:** CaseMine runs two paragraph-ID schemes: bare numbers (`0`–`78` on Mayavati; Saroj, Pratibha, PSA Sical) and `p_N` elsewhere (F-52). The citation-excerpt index mixes them too: `tpid` is a bare number in 8 of 30 results, `fpid` is sometimes a list string like `"[p_7]"`, and the same query returned `tpid` as `"39"` once and `"p_18"` another time (F-36, F-19).
- **OBSERVED:** AMICUS emitted the format matching each page in 8 of 8 links (F-49), so the mixed scheme was not itself the cause of its bad anchors. **UNKNOWN:** whether the mixed scheme contributes to misalignment elsewhere in their pipeline.

### Decision
1. **Judgment ID:** an opaque, immutable ID (UUIDv7 or ULID), minted once at ingestion and never reused or re-keyed. It does not encode court, year or date. Human-readable keys (neutral citation, reporter citation, case number) are lookup attributes resolved through ADR-4, never primary keys.
2. **Paragraph ID:** one scheme for every document: `{judgment_id}:{seq}` where `seq` is a zero-padded ordinal of the segmented paragraph (`p000001`…). Within a page it renders as `id="p000123"`. No bare numbers, no list strings, no per-source variants.
3. **Printed number is a separate field.** Each paragraph stores `printed_label` (e.g. `"212"`, `"12.3"`, `"(iv)"`, or null). The UI shows the printed label; links use the stable `seq` ID. Users can cite "¶212" while the system addresses `p000523`.
4. **Text versioning.** Re-segmenting a judgment (OCR fix, better parser) produces a new `text_version`. Paragraph IDs are stable within a version; a mapping table `(old_version, old_seq) → (new_version, new_seq)` is written at re-ingestion so that stored edges, bookmarks and AI anchors can be migrated or flagged.
5. **One canonical serializer.** Every API, index and AI pipeline emits paragraph IDs through one library function; a schema test rejects any other format.

### Alternatives considered
- **Use printed paragraph numbers as IDs.** Rejected: many judgments have none, restart numbering inside annexures, or use sub-numbering; CaseMine shows the two diverge widely (F-32, F-50).
- **Keep CaseMine-style DOM-sequential ids per source format.** Rejected: that is exactly the two-scheme situation in F-52/F-36.
- **Content-hash paragraph IDs.** Stable across re-segmentation of unchanged text, but break on any OCR correction and are not ordered. Could be kept as an extra `text_hash` field for migration matching.

### Open questions
- Segmentation rules for headnotes, quoted statutory text, tables and multi-opinion judgments (separate opinion IDs within one judgment?). CaseMine's segmentation method is **UNKNOWN** (F-05 does not prove it).
- Whether the public URL should carry the version (`/j/{id}@v3#p000123`) or always resolve to the latest version with a redirect map.

---

## ADR-2. Judgment data model

### Context
ELI needs one authoritative judgment record from which search projections, page rendering and AI retrieval are derived. CaseMine shows that projections drift from the page (sparser fields, stale counters).

### CaseMine evidence: observed fields
These are field **names** seen in CaseMine responses. They are listed to inform ELI's model, not copied as a schema.

| Group | CaseMine fields | Class | F-ID |
|---|---|---|---|
| Identity | `id`/`jid`, URL `/judgement/{country}/{id}` | OBSERVED | F-01 |
| Page metadata | `judgementTitle, judgementStitle, judgementDate, judgementCourt, judgementYear, citedByCount, paraIds, has_summary, is_order, fileFormat, free_access, sensitive` | OBSERVED (report Q1) | F-01, F-05 |
| Search projection | `title, shortTitle, courtName, courtBench, judgeName, date, caseNumber, citationCodes[], nCitation, acts, advocates, disposition, citesToCount, citedByCount, prio` and more | OBSERVED; several (`judgeName`, `caseNumber`, `citationCodes`, `nCitation`, `acts`, `advocates`) empty in all 110 records although the page shows them | report Q2 |
| Quality flags | `scanned, junk, rlang, table, ai, order, ur, noIndex, sensitive` | OBSERVED names; meanings INFERRED | F-06 |
| Treatment roll-up | `ovrRuled, sentimentCategory, sentimentIds[], sentimentTitle[]` | OBSERVED; denormalization INFERRED | F-14 |
| Original file | `/judgement/pdf/{c}/{id}`, RTF fallback | OBSERVED; storage UNKNOWN | F-04 |
| Generated summary | Smart Summary with fixed sections, generated on demand | OBSERVED; method UNKNOWN | F-22 |
| Policy | verbatim text, de-indexing on request, no redaction without court order | OBSERVED (policy text, not technical proof) | F-26 |
| `prio` | values {0, 50, 60, 80, 100}, not a court mapping | OBSERVED values; meaning UNKNOWN | report Q2 |

### Decision: ELI proposed model
Everything below is **proposed**. Fields are grouped so ownership is clear: *source* fields come from ingestion, *derived* fields are recomputed from other tables and never edited by hand.

```
judgment
  # identity (ADR-1)
  judgment_id            ULID, immutable
  text_version           int
  # source metadata
  court_id               FK court (hierarchy: SC / HC + bench / tribunal)
  bench_type             single | division | full | constitution
  judges[]               FK judge, with role (author, concurring, dissenting)
  decision_date          date
  case_numbers[]         {type, number, year} normalized, plus raw string
  parties                {petitioners[], respondents[]} plus raw title
  short_title, full_title
  neutral_citation       e.g. "2026 INSC 5" (nullable)
  reporter_citations[]   FK citation_key (ADR-4), many per judgment
  disposition            controlled vocabulary + raw string
  advocates[]            raw strings, normalized later
  statutes_referenced[]  {act_id, section, paragraph_ids[]}
  language               ISO code; translation_of (nullable FK)
  doc_type               judgment | order | interim order
  # text
  paragraphs[]           {seq, printed_label, opinion_id, text, text_hash}
  source_file            {storage_key, mime, sha256, page_count, fetched_from, fetched_at}
  # quality and policy
  ocr                    {is_scanned, engine, confidence}
  quality_flags          {junk, has_tables, incomplete, ...}
  visibility             public | no_index | sensitive | withdrawn, with reason + date
  # derived (never hand-edited; ADR-3)
  cites_count, cited_by_count, cited_by_retrievable_count
  negative_treatment     {flag, by_judgment_ids[], computed_at}
  # generated
  summaries[]            {kind, model, prompt_version, generated_at, sections}
```

Rules:
- **Search and API projections are generated from this record by one job**, so a field present on the page is present in search (avoids the sparse-projection gap in report Q2).
- **Quality and visibility are first-class.** CaseMine carries the same idea (F-06, F-26); ELI makes the reason and date explicit so a hidden judgment can be explained, and so counts can say how many citing judgments are hidden (see ADR-3).
- **Generated content is stored with its model and prompt version**, never mixed into source fields (CaseMine's summary method is UNKNOWN, F-22).
- **No unexplained ranking fields.** If ELI needs an authority weight like `prio`, it is documented and computed, not hand-set.

### Alternatives considered
- **Document store with a free-form record (CaseMine-like, INFERRED Mongo, F-02/F-03).** Fast to start, but encourages projection drift. Proposed instead: relational core (Postgres) for identity, metadata and edges, with paragraph text in the same database or object storage, and search indexes built from it.
- **Single wide table including derived counters.** Rejected: counters must be recomputable from edges.

### Open questions
- Court and judge master data: build or license? How to normalize High Court bench names.
- How to represent multiple opinions (majority, concurring, dissent) for paragraph IDs and treatment.
- Whether `statutes_referenced` is extracted at ingestion or treated as another edge type.

---

## ADR-3. Citation edges

### Context
Citation edges power "cited by", treatment flags, citation graphs and AI retrieval. A lawyer acting on "overruled" or "cited 1,288 times" needs these to be correct and explainable.

### CaseMine evidence
- **OBSERVED:** edges are their own records: `{id, fjid, tjid, sentiment, snippet[{pid[], opinion}], reason, snippet_span, eBy, eByName, eAt}` (F-07).
- **OBSERVED:** direction is consistent: `fjid` = citing, `tjid` = cited; outgoing via `linkDetails`, incoming via `citeText` (F-08), re-verified on 117 incoming records (F-40). **UNKNOWN:** whether each edge is reciprocally present in both lists (F-08).
- **OBSERVED:** treatment vocabulary `positive, referencing, distinguished, overruled, negative`; missing value shows as "uncategorized" (F-09). Labels are per edge, not per paragraph.
- **INFERRED:** `overruled` sometimes records the status *mentioned* in the citing text, not something the citing court did (Motilal edges to Himangni and SBP & Co.) (F-10).
- **OBSERVED:** editor fields `eBy`, `eByName`, `eAt` exist but were null on every edge; **INFERRED:** provision for manual overrides; labelling method **UNKNOWN** (F-07, report Q4).
- **INFERRED:** `snippet.pid` refers to the citing judgment's paragraphs (report Q3). **OBSERVED:** `citedParas` gives per-paragraph counts for the cited judgment (F-05).
- **OBSERVED:** treatment is rolled up onto the judgment (`ovrRuled`, `sentimentCategory`, `sentimentIds`) (F-14).
- **OBSERVED counter mismatch (Vidya Drolia, incoming):** headline `citedByListSize` 1,288; search `citedByCount` 1,173; excerpts actually retrievable by paging 1,171 (F-11, F-12, F-38). The ~117 gap's cause is **UNKNOWN** (candidates: edges without excerpts, hidden citing judgments, stale counter).
- **OBSERVED:** the citing-judgments list is hard-capped at the top 100; "Load More" pages excerpts only (F-39).

### Decision
1. **Edge record (proposed):**
   ```
   citation_edge
     edge_id              ULID
     citing_judgment_id   FK  (always the later judgment that contains the citation)
     cited_judgment_id    FK  (nullable until resolved)
     cited_raw            the citation string as written, plus resolver result (ADR-4)
     occurrences[]        {citing_para_id, char_start, char_end, excerpt}   -- one per mention
     cited_para_ids[]     paragraphs of the cited judgment relied on, if identifiable
     treatment            followed | applied | relied_on | referred | distinguished |
                          doubted | not_followed | overruled_by_citing | per_incuriam | unclear
     status_mentioned     none | overruled_elsewhere | reversed_on_appeal | stayed | referred_to_larger_bench | ...
     status_mentioned_ref FK judgment that took that action, if named
     label_source         rule | model | human
     classifier_version, model_confidence
     review_state         unreviewed | accepted | corrected | disputed
     reviewed_by, reviewed_at, review_note
     created_at, updated_at
   ```
2. **Treatment and status are separate columns.** `treatment` answers "what did the citing court do with this case?" `status_mentioned` answers "what does the citing text say happened to it?" Only `treatment = overruled_by_citing`, or a curated status with a named overruling judgment, can raise ELI's red flag. This avoids the F-10 ambiguity.
3. **Paragraph-level provenance is required.** An edge without at least one `occurrence` (citing paragraph + character span) is not shown as a treatment, only as "cited (location unknown)". Provenance uses ADR-1 paragraph IDs on both sides.
4. **Human review from day one.** Review fields are populated by an editorial queue that prioritizes negative treatments, Supreme Court and Constitution Bench edges, and low-confidence labels. CaseMine has the fields but they were never filled in the sample (F-07); ELI tracks review coverage as a published metric.
5. **One counting rule, three numbers.** All counters are computed from `citation_edge` by the same query that serves the list:
   - `cited_by_total`: distinct citing judgments with a resolved edge;
   - `cited_by_visible`: those whose citing judgment is visible to this user;
   - `hidden_count = total − visible`, displayed as "N not shown (restricted or withdrawn)".
   The list API pages through all of `cited_by_visible` with a stable sort and cursor; no silent top-100 cap (F-39). A nightly check compares stored counters with a fresh recount and alerts on any difference (prevents an F-12/F-38-style drift).
6. **Roll-ups are derived.** `negative_treatment` on the judgment is a materialized view over edges with review state, recomputed on edge change, and links back to the edges that caused it (F-14 shows the value of the roll-up; ELI keeps its derivation traceable).

### Alternatives considered
- **Single sentiment label (CaseMine, F-09).** Rejected for the reasons in F-10.
- **Graph database for edges.** Not needed for adjacency lists and induced subgraphs; CaseMine serves both without visible graph-DB use (F-13, storage UNKNOWN). Postgres with indexes on both endpoints, plus a precomputed induced subgraph per search, is enough to start.
- **Denormalized counters updated incrementally only.** Rejected without the recount check, given F-12/F-38.

### Open questions
- Final treatment vocabulary: needs review by practising lawyers (Indian usage of "per incuriam", "doubted", "explained").
- How to treat citations of a judgment that is itself under appeal or reversed in the same case (case-history edges vs citation edges).
- Whether to expose `model_confidence` to users or only `review_state`.

---

## ADR-4. Deterministic exact-citation resolver

### Context
Indian lawyers routinely paste reporter citations (`(2021) 2 SCC 1`, `AIR 1973 SC 1461`, `2026 INSC 5`). The expected result is one judgment, not a keyword search.

### CaseMine evidence
- **OBSERVED:** in the main search and CiteTEXT index, `(2021) 2 SCC 1` is matched token by token and returns 1,440,078 excerpt hits (F-18).
- **OBSERVED:** the `parallel:` scope is also token-matched (about 25 results); the target judgments were absent (F-43).
- **OBSERVED:** a separate Citation Code Search does exact lookup through journal → year → volume → page → judgment ID; `(2021) 2 SCC 163` resolved (F-41).
- **OBSERVED:** coverage gap: SCC 2021 volume 2 lists 37 starting pages and not page 1, so `(2021) 2 SCC 1` (Vidya Drolia) returns `[]` (F-42). Overall coverage rate **UNKNOWN**.
- **OBSERVED (code):** a commented-out "daily limit exhausted" message sits on the same empty-result path, so "not found" and "rate limited" can look identical (report R2-3).
- **OBSERVED:** CaseMine also rate-limits page loads with a CAPTCHA challenge (F-37, F-54).

### Decision
1. **Separate service, separate endpoint:** `GET /citations/resolve?q=` returns a typed result, never a search result list.
2. **Parser:** a grammar per reporter family (SCC, SCC OnLine, AIR, SCR, INSC neutral, HC neutral citations, major state reporters), normalizing to a `citation_key` = `{reporter, year, volume, series, page, court?}`. Unparseable input falls through to search with a visible "not a recognised citation" note.
3. **Citation table:** `citation_key → judgment_id` with `source` (publisher data, extracted from judgment text, editorial), `verified_by`, `verified_at`. Many keys per judgment (parallel citations); a key can map to more than one judgment only with an explicit `ambiguous` state.
4. **Explicit result states:**
   `resolved` (one judgment) · `ambiguous` (list of candidates) · `not_in_table` (parsed, no mapping) · `reporter_not_covered` (reporter/year outside measured coverage) · `parse_failed` · `rate_limited` (with retry-after) · `error`.
   The UI wording differs for each. `rate_limited` is never shown as "no results".
5. **Measured coverage:** for each reporter and year, store `expected_count` (from an index of the volume, where available) and `mapped_count`, and publish the ratio on a coverage page. CI fails if a known-good fixture set (e.g. 500 landmark citations, including `(2021) 2 SCC 1`) stops resolving.
6. **Search integration:** if the main search box receives a string that parses as a citation, ELI calls the resolver first and shows the resolved judgment as a pinned top card, then normal search results below.

### Alternatives considered
- **Rely on lexical search with phrase boosting.** Rejected: F-18/F-43 show this fails for the most common lawyer query.
- **Hierarchical picker only (CaseMine's journal→year→volume→page UI, F-41).** Kept as a secondary UI, but the free-text resolver is primary.

### Open questions
- Licensing of reporter citation data (SCC, AIR are commercial); what coverage is achievable from citations extracted from judgment text alone.
- How to handle pin cites (`(2021) 2 SCC 1, 45`) and map them to paragraphs.

---

## ADR-5. Search

### Context
Search must be reproducible: a lawyer who reruns a query, shares it, or cites "the first result" should see the same order. Counts must be honest about caps.

### CaseMine evidence
- **OBSERVED:** two hit spaces with different shapes and counts: judgments (SSR page + `/search/scroll`, `/search/graph`) and citation excerpts (`/search/opinion?sort=_score`) (F-15, F-16).
- **INFERRED (medium):** Elasticsearch/OpenSearch-class engine from `_score` and `<em>` highlighting (F-17). Semantic retrieval in search: **UNKNOWN**.
- **OBSERVED:** `/search/scroll` carries no query parameter, so query state is kept server-side (F-15; storage INFERRED).
- **OBSERVED:** the same query twice gave the same hit count (2,869) but a different top 10 (F-19). Cause **UNKNOWN**.
- **OBSERVED:** `/search/graph` returns up to 100 nodes and the induced edges among them (F-13); the citing list is capped at 100 (F-39).

### Decision
1. **Deterministic ordering.** Every sort ends with explicit tie-breakers: `score DESC, court_rank ASC, decision_date DESC, judgment_id ASC`. Score computation is pinned to a single index snapshot per query (no mixing replicas mid-pagination), and replica scoring differences are removed by using consistent shard routing or a preference key per session. A regression test runs each benchmark query twice and fails if the top 50 differ.
2. **Stateless, shareable queries.** The full query (text, filters, sort, index version) is encoded in the URL; pagination uses a cursor (`search_after` on the tie-broken sort key), not server-side session state.
3. **Explicit caps.** Any capped list or graph says so: "Showing 100 of 1,288 citing judgments — refine or export", with the cap value returned in the API (`{total, returned, cap}`). Hit counts state whether they are exact or estimated.
4. **Two indexes, justified:**
   - **Judgment index:** one document per judgment (metadata + full text + headnote fields). Answers "which judgments are about X".
   - **Citation-excerpt index:** one document per `citation_edge.occurrence` (citing paragraph text, cited judgment, treatment, both paragraph IDs). Answers "how have courts used case Y on point X", which a judgment-level index cannot rank. CaseMine's separate CiteTEXT index (F-16) suggests users value this view.
   Both are built from the canonical tables (ADR-2, ADR-3) and carry the same paragraph-ID format (ADR-1), so the format mixing seen in F-36 cannot occur. A paragraph-level passage index for AI retrieval (ADR-6) can share infrastructure with the excerpt index but is a separate index.
5. **Exact citations go to the resolver first** (ADR-4).
6. **Hybrid lexical + vector retrieval** is allowed only if it keeps rule 1: the fused score is computed deterministically and ties broken as above.

### Alternatives considered
- **One index with nested excerpts.** Simpler to operate, but ranking excerpts inside judgments gets awkward and counts become ambiguous. Revisit if the excerpt index is too costly.
- **Server-side scroll state (CaseMine, F-15).** Rejected: breaks shareable links and makes results depend on session.

### Open questions
- Ranking signals for Indian courts (court hierarchy, bench strength, recency, citation authority): which ones, and how to explain them to users.
- Engine choice (OpenSearch vs Postgres full-text + pgvector at launch scale).

---

## ADR-6. AI answers: anchors, validation, source panel, metering

### Context
ELI's AI answers will cite judgments and paragraphs. A wrong paragraph link is worse than no link: it looks authoritative and wastes the reader's time, or misleads a court filing.

### CaseMine evidence
- **OBSERVED:** AMICUS is `POST /chat/ask/stream` returning SSE: connected → 5 status events → answer deltas → `final` → `done` (F-20).
- **OBSERVED labels; implementation UNKNOWN:** stages understanding_question → searching_semantic → searching_authorities → collecting_citations → generating_answer (F-27).
- **OBSERVED:** answers cite `/judgement/in/{id}[#p_n]` inline plus a References block with "Proposition relied on" and "Applicability" (F-28).
- **OBSERVED:** paragraph links are produced server-side; the frontend only inserts them (F-35).
- **OBSERVED (n=8 distinct linked pairs over 4 answers, F-46):** 6 of 8 links support the claim, 1 is off by one (PSA Sical 55 → 56), 2 are far off (N.N. Global `p_523` vs `p_797`; BSNL `p_15` vs `p_122`/`p_123`); every miss points earlier (F-48, F-31). Judgment-level attribution was correct in every checked case (F-31, F-33).
- **OBSERVED:** ruled-out causes: link format (F-49), printed-number confusion (F-50), most-cited-paragraph data (F-51), borrowing from a co-cited judgment (F-34). Remaining cause (unvalidated ID selection, or chunk-start offsets) is **UNKNOWN**.
- **OBSERVED:** answers include many unlinked citations, 22–28 per answer (report Round 3).
- **OBSERVED:** credit metering: `chatLeft` / `limitMessage` in `final`, decremented per question (F-30); the trial ended with `event: error` at the 5th question (F-47).
- **OBSERVED label / INFERRED provider:** CaseIQ shows "AMICUS (Powered by GPT)" (F-45); uploads map to `openaiFileId` (F-21). The model for `/chat` answers is **UNKNOWN**.

### Decision
1. **Anchors come from retrieval, never from model text.** Retrieval returns passages `{passage_id, judgment_id, para_ids[], text_version, text}`. The model is shown passages with short opaque handles (`[S3]`) and may only cite handles. A post-processor maps handles to `judgment_id` + `para_ids` from retrieval metadata. Any judgment ID, URL or paragraph number the model writes itself is stripped. This removes the F-35 class of failure at the source.
2. **Pre-render validation for every anchored claim:**
   - Split the answer into claims, each with its cited handle(s).
   - Check entailment of the claim against the anchored paragraph(s): lexical overlap screen plus an NLI/LLM verifier, with a stored score.
   - If support is found only in an adjacent paragraph (±2), move the anchor to the best paragraph and log the correction (would fix the PSA Sical off-by-one, F-48).
   - If no paragraph in the anchored passage supports the claim, search the whole cited judgment; if found, re-anchor and log; if not, render the citation at judgment level with a "paragraph not verified" marker, or drop the claim.
   - Never render a deep link that failed validation.
   - Each claim stores `verification = {method, score, verified_para_id, corrected_from}` for audit.
3. **Quality gate:** a fixed evaluation set of questions with hand-labelled supporting paragraphs; release blocks if paragraph-anchor precision falls below a set target (e.g. 98% at ±0 after correction). CaseMine's sample (6/8 at first pass) is the floor to beat, not a target.
4. **Source panel:** a side panel listing every cited authority with: title, court, date, neutral and reporter citations; treatment flag from ADR-3 (e.g. "overruled by X" with link); the exact quoted paragraph text with its printed paragraph label; a "view in judgment" link that scrolls and highlights the paragraph; the verification badge. Unlinked mentions in the answer (CaseMine had 22–28 per answer) are also resolved through ADR-4 and listed, marked "mentioned, not relied on".
5. **Answer record:** store `question, retrieval_query, retrieved passages (ids + text_version), model + version, prompt_version, raw output, post-processed output, verification results`. Lets ELI rerun validation after re-segmentation (ADR-1 rule 4).
6. **Streaming protocol:** keep the useful CaseMine shape (status events, deltas, a final summary event, F-20) but stream only *validated* citations: deltas carry text with handles; anchors are resolved in the `final` event after validation.
7. **Metering:** charge per answer, with the cost model visible before submission (effort level → credits). The `final` event returns `credits_used` and `credits_remaining`; exhaustion returns a typed `quota_exhausted` error, not a generic error, and the UI shows it before the user types (F-30, F-47). Failed or refused answers are not charged. Validation and re-anchoring cost is ELI's, not the user's.

### Alternatives considered
- **Let the model emit paragraph IDs and validate afterwards.** Validation alone catches errors, but generating from retrieval metadata removes the most common error source; we do both.
- **Judgment-level links only.** Safe but much less useful for lawyers; acceptable only as the fallback in rule 2.
- **Hide unverified claims entirely.** May drop correct legal statements; the "not verified" marker is preferred, with an option to hide.

### Open questions
- Which verifier (NLI model vs LLM judge) gives acceptable latency for streaming answers.
- Model provider and data residency for Indian legal data (CaseMine's provider is UNKNOWN/INFERRED, F-21, F-45).
- Credit pricing per effort level, and whether institutional plans get pooled credits.
- Whether to show users the verification score or only a pass / corrected / unverified badge.

---

## Cross-cutting notes

- **Rate limiting:** CaseMine challenges rapid page loads with a CAPTCHA (F-37); spaced loads raised none (F-54). ELI should rate-limit with explicit `429` + `Retry-After` on APIs and keep that state distinct from "not found" everywhere (ADR-4 rule 4).
- **Single origin:** CaseMine serves all API calls from one origin, with only analytics and translation as third parties (F-24). ELI can follow the same pattern for simpler CSP and privacy review.
- **What was not learned:** CaseMine's storage, search engine, model provider and service split are UNKNOWN (report "Limitations"). None of the decisions above depends on knowing them.
