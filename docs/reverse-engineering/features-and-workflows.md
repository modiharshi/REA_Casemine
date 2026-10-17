# CaseMine: feature inventory and user workflows

This is deliverable 1. Each feature is classed OBSERVED (used live or seen in captured UI), CODE (present in shipped JavaScript but not exercised), or UNKNOWN. Evidence IDs (F-xx) refer to [evidence-ledger.md](evidence-ledger.md).

**Scope:** India jurisdiction, with an operator-owned trial account. Private account areas (Research Centre, projects, bookmarks, alerts contents) were deliberately not inspected.

## Feature inventory

| Area | Feature | Status | Evidence |
|---|---|---|---|
| Search | Keyword search with autosuggest (≥3 chars, 500 ms debounce; categories such as "Judgements:") | OBSERVED | F-15 |
| Search | Scoped prefixes `parallel:`, `title:`, `appellant:`, `respondent:`, `lawyer:` | CODE + OBSERVED (`parallel:` only) | F-15, F-43 |
| Search | Result tabs: Judgments, CiteTEXT, Acts, Commentaries, Columns, Visual, RCA, Alerts | OBSERVED | F-16, F-62, F-63 |
| Search | Infinite scroll (`/search/scroll`, query held server-side) | OBSERVED | F-15 |
| Search | Visual citation graph of the result set | OBSERVED | F-13 |
| Search | Court, year and tribunal facets (`allCourtsMap`, `allYearsBucket`) | OBSERVED | Q5 |
| Citation lookup | Citation Code Search: journal → year → volume → page | OBSERVED | F-41, F-42 |
| Judgment viewer | Rendered text with paragraph anchors; copy and select blocked inside `#page-wrapper` | OBSERVED | F-05, F-52 |
| Judgment viewer | Original PDF tab (pdf.js; RTF fallback) | OBSERVED | F-04, F-59 |
| Judgment viewer | Metadata: neutral and equivalent citations, case number, disposition, judges, advocates, statutes | OBSERVED | Q2 |
| Judgment viewer | Smart Summary (fixed sections, "Table of Precedents Cited") and on-demand generation | OBSERVED (display) / CODE (generate) | F-22 |
| Judgment viewer | Cites-to list (`linkDetails`), Subsequent References (`citeText`), paragraph heat map (`citedParas`) | OBSERVED | F-07–F-12, F-38–F-40 |
| Judgment viewer | "Overruled By" and "Treated Negatively By" badges and modals | OBSERVED | F-60 |
| Judgment viewer | Bookmark, annotate, comment, share, print, lawyer claim | CODE (write actions, deliberately not used) | Q1 |
| Judgment viewer | "Similar judgments" URL | CODE (dead: defined, never used) | F-56 |
| AI | AMICUS chat (SSE stream, inline judgment and paragraph links, References block, credit metering) | OBSERVED | F-20, F-27–F-31, F-46–F-51 |
| AI | AMICUS upload Q&A, drafting (`/ai/draft/*`), prompt enhancement | CODE | F-21 |
| AI | CaseIQ: document, text or URL in; summary, arguments, citations found, suggested precedents, graph, AMICUS tab | OBSERVED | F-44, F-45, F-64, F-65 |
| Statutes | Acts browse (Central and per-State) and Acts search (section-level) | OBSERVED | F-62, F-66 |
| Commentaries | Per-court lists of analytical commentary articles | OBSERVED | F-66 |
| Notifications | RCA search (gazette and notification records) | OBSERVED | F-62 |
| Other | Briefs search, Columns, Alerts, Research Centre, Lawyers directory | CODE / not inspected | — |
| Platform | Login wall on listing pages; trial credit limits; reCAPTCHA on rapid judgment loads | OBSERVED | F-37, F-47, F-54 |

## Core user workflows (OBSERVED end to end)

1. **Keyword research**
   1. Type a query; `/search/autosuggest` offers suggestions.
   2. Submit; the browser navigates to `/search/{country}/{sanitized query}` (server-rendered top 20).
   3. Scrolling loads more via `/search/scroll`.
   4. Tabs fetch other indexes (`/search/opinion`, `/search/act`, `/rca/search/…`, `/search/graph`).
   5. Clicking a result opens `/judgement/in/{id}`.
2. **Reading a judgment**
   1. The page is rendered on the server.
   2. The browser then fetches `linkDetails` (outgoing citations), `citedParas` (heat map) and `lawyers`.
   3. The Subsequent References tab fetches `citeText` (top 100, with excerpts paged by `from`).
   4. The ORIGINAL PDF tab fetches `/judgement/pdf/…`.
   5. Badges open `overRuled`.
3. **Exact-citation lookup**
   1. In `/citationsearch`, pick journal → year → volume → page; each pick calls `/citationsearch/codes`.
   2. The final pick resolves to a judgment ID, then `/citationsearch/judgement` returns the result card.
4. **AMICUS question**
   1. On `/chat`, type at least 5 characters and press send; the browser calls `POST /chat/ask/stream`.
   2. The server streams 5 status stages, then answer deltas, then `final` (credits) and `done`.
   3. The URL changes to `/chat/{chatId}` and `/chat/meta/{id}` is fetched.
5. **CaseIQ**
   1. On `/caseiq`, use Add Text, Upload or URL, then Get CaseIQ; the browser calls `POST /caseiq/text|docupload|url`, which returns a run ID.
   2. The browser redirects to `/caseiq?id=…`, then fetches `/caseiq/summary` and `/caseiq/arguments`.
   3. The tabs (Citations Found, Suggested Precedents, Graphical View, AMICUS) render client-side.
   4. Reopening a run replays the stored results.
