# Prioritized implementation roadmap for ELI

This is deliverable 7. It is a phased build order derived from the CaseMine evidence. The design detail for each item lives in the ELI architecture decisions (`docs/eli/architecture-decisions.md`, ADR-1 to ADR-6), committed on branch `eli-architecture-decisions`. Items cite the CaseMine finding that motivates them.

## Phase 0: Data foundations (prerequisite for everything)

1. **Canonical judgment ID and a single paragraph-ID scheme** (ADR-1)
   - CaseMine mixes bare-number and `p_N` paragraph IDs, and neither equals the printed paragraph number (F-32, F-36, F-52).
   - **Done when:** every ingested judgment has stable paragraph IDs, with the printed number stored separately.
2. **Judgment record and ingestion-quality flags** (ADR-2)
   - Flags: scanned/OCR, language, sensitive, no-index; plus provenance (F-06, F-26).
   - **Done when:** flags populated on ingestion and searchable.
3. **Exact-citation resolver** (ADR-4)
   - A reporter/year/volume/page table with measured coverage per reporter and explicit `not_found` and `rate_limited` states.
   - CaseMine's lookup misses `(2021) 2 SCC 1` and returns an ambiguous `[]` (F-41, F-42).
   - **Done when:** a coverage report exists per reporter, and the API returns distinct not-found and limit states.

## Phase 1: Citation graph (the main differentiator)

4. **First-class citation edges** (ADR-3)
   - Direction, separate `treatment` vs `status_mentioned` fields, paragraph provenance on both sides, and review fields.
   - CaseMine conflates these: "Overruled By" lists judgments that only *mention* overruling (F-10, F-60).
   - **Done when:** "overruled by" is computed only from `treatment = overruled`.
5. **Counts derived from the edge store**, with no hard cap without disclosure.
   - CaseMine shows 1,288 vs 1,171 retrievable vs 1,173, and caps lists at the top 100 (F-38, F-39).
   - **Done when:** the displayed count equals the number of retrievable edges.

## Phase 2: Search

6. **Two indexes: judgments and citation excerpts** (ADR-5), with deterministic tie-breaking (F-16, F-19, F-67).
   - Stateless pagination: pass the query or a cursor explicitly, unlike CaseMine's server-held query (F-15).
   - **Done when:** an identical query returns an identical order.
7. **Citation-aware query parsing** that routes recognised citations to the resolver instead of tokenizing them (F-18, F-43).

## Phase 3: AI research

8. **Grounded answers** (ADR-6)
   - Paragraph links generated from retrieval metadata and validated before rendering.
   - CaseMine: 6/8 links correct, 1 off by one, 2 far off (F-48).
   - **Done when:** 100% of rendered links pass validation, with failures shown as "unlinked" rather than wrong.
9. **Document-in research (CaseIQ-like)**
   - Extract citations and statutes, store results so they replay without regeneration (F-44, F-64), and keep source panels separate from generated prose.
10. **Metering and limits made explicit** in the UI, with no silent `[]` (F-30, F-47, F-42).

## Phase 4: Operational hygiene

11. **Abuse protection without breaking legitimate reading:** rate limits with clear UI states. CaseMine shows a reCAPTCHA interstitial (F-37).
12. **Correct error semantics:** no HTTP 200 for server errors (F-63); typed errors surfaced in the UI.

## Not recommended to copy

- **Hard-capped "top 100" lists** without pagination (F-39).
- **Mixed paragraph-ID schemes** (F-52).
- **Unlabelled AI provenance on `/chat`** (F-45).
- **A dead "similar judgments" endpoint** (F-56).
