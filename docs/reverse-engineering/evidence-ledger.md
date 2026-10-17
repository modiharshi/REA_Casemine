# Evidence ledger: CaseMine investigation (2026-10-17)

Raw evidence is in `evidence/raw/` (gitignored, about 430 MB). It is kept unmodified.

Files whose names end in `-AUTHENTICATED` were captured in the operator's own logged-in session and contain account identifiers. Never share them as-is.

## A. Captures

| File | REA operation | Evidence ID | SHA-256 (file) |
|---|---|---|---|
| 01-inspect_web_page-filtered.json | inspect_web_page (homepage, origin-filtered) | ev_674a7cf5…0065 | 2be282e4…4a56 |
| 01-capture_web_screenshot.json (+ 01-casemine-home.png) | capture_web_screenshot | ev_25a15bd7…74fb | e9ea3c6c…a2f |
| 02-inspect_web_page-with-sources.json | inspect_web_page + script sources (homepage) | ev_850359c1…0a9 | ae276f88…dd67 |
| scripts-homepage/ | export_web_scripts | ev_cb645fd3…bd43 | manifest 624ff8b0… |
| (retained in bundle) | analyze_web_bundle (homepage) | ev_71d1fa33…de36de | — |
| (retained in bundle) | analyze_javascript_application, summary (homepage scripts) | parent ev_4da3b0d3…21b4 | — |
| 03-trace_javascript_semantics-autosuggest.json | trace_javascript_semantics (0 seed matches; node budget) | ev_a0fd5397…1e0c | ec038296…87d3 |
| 04-inspect_web_page-judgment-AUTHENTICATED.json | inspect_web_page + sources + accessibility text (judgment 695cba20…) | ev_e1a34da2…4a0b | e673f3e1…19a4 |
| scripts-judgment/ | export_web_scripts | ev_8572c297…720d | manifest d88878b9… |
| 05-inspect_web_page-citation-click-AUTHENTICATED.json | 60 s passive window; **0 requests** (clicks opened new tabs) | ev_0b7f3f36…3c03 | 5da2ee7d…b49f |
| 06-scenario-judgment-tabs-AUTHENTICATED.json | capture_browser_scenario: load + Subsequent References (PDF step timed out) | ev_2413a9cf…819c | d5d469e4…4c98 |
| 07-scenario-caseiq-pdf-AUTHENTICATED.json | scenario: CaseIQ tab (no XHR); PDF role locator failed | ev_cf028590…b9af | ff6b42a2…ad42 |
| 08-scenario-pdf-and-incoming-citations-AUTHENTICATED.json | scenario: PDF tab (`#show-pdf-tab`) + Vidya Drolia page + Subsequent References | ev_4ceb449b…d6ca | deb3f462…378e |
| 09-scenario-keyword-search-AUTHENTICATED.json | scenario: autosuggest + keyword search | ev_160fa092…d10b | 628ee76d…06d0 |
| 10-inspect_web_page-search-results-AUTHENTICATED.json | inspect_web_page + sources + accessibility (results page) | ev_99a051bb…a2c | 18190362…150b |
| scripts-search/ | export_web_scripts | ev_8842efa4…0764 | manifest d9694695… |
| 11-scenario-search-repeat-scroll-citation-visual-AUTHENTICATED.json | scenario: repeat query, scroll, citation query, Visual tab | ev_fc0f1617…59a1 | 88a8fe60…636c |
| 12-scenario-amicus-chat-load-AUTHENTICATED.json | scenario: `/chat` load only (no question) | ev_46e469bd…613d | 2863c08a…127a |
| 13-inspect_web_page-amicus-chat-AUTHENTICATED.json | inspect_web_page + sources (`/chat`) | ev_871a01a7…4271 | c37a8852…a4cd |
| scripts-amicus/ | export_web_scripts | ev_02e88163…456f | manifest b1da8483… |
| 14-scenario-api-doc-and-takedown-policy.json | scenario: `/home/casemine-api`, takedown policy | ev_392f39a3…b782 | eabf7311…1d70 |
| session-004-evidence-bundle.json | export_evidence_bundle (27 records) | — | e71f4f1c…3adb |
| 15-scenario-amicus-one-question-AUTHENTICATED.json | scenario: suggested-prompt click + Enter. **No request sent** (Enter doesn't submit) | ev_a1d4d9c2…1dc4 | 514c90df…3465 |
| 16-scenario-amicus-question-sent-AUTHENTICATED.json | scenario: `#send-icon-bt` click timed out; **no request sent** | ev_adcc27fa…51ce | f541da46…1ac5 |
| 17-scenario-amicus-question-answered-AUTHENTICATED.json | scenario: fill `#chat-textarea` + click `#chat-send`, the **one question** sent; full SSE body (6,814 B) captured | ev_aec7411b…5ada | 7990f87f…5bfa |
| session-005-evidence-bundle.json | export_evidence_bundle (30 records) | — | see `shasum` |
| (navigation only) | scenario: open N.N. Global `/judgement/in/64482b18…` | ev_49fc9c71…282b | — |
| (navigation only) | scenario: open BSNL v. Nortel `/judgement/in/604a637d…` | ev_3a5c8fff…e4e7 | — |
| citation-check/nn-global-64482b18.json | CDP `Runtime.evaluate` read of p_522–p_524 (no network) | — | 0178e9c0…ae66 |
| citation-check/nn-global-search-srikrishna.json | CDP paragraph text search (no network) | — | 0b1b0ab4…3d3b |
| citation-check/bsnl-nortel-604a637d.json | CDP read of p_14–p_17 (no network) | — | 5d83b433…6040 |
| citation-check/bsnl-nortel-search.json | CDP paragraph text search (no network) | — | 9e2b7d05…18d9 |

**Supplementary (non-REA) bodies.** These were re-requested once through the debug page over CDP (`Runtime.evaluate` same-origin `fetch`), because REA's 64 KiB body budget was exceeded. Each has a `.meta.json` file with request path, status and hash.

| File | Bytes | SHA-256 |
|---|---|---|
| cdp-bodies/judgement_in_5fe97792…_linkDetails… | 248,763 | 837fb851e8154c05b590178c816617e1c313cedf5439a278dc36bfb706b56f41 |
| cdp-bodies/judgement_in_5fe97792…_citeText… | 446,097 | e1a6e242487a191194dfff21b254d2a82d4578e46ee82d14028cfb7a6eda0379 |
| cdp-bodies/search_scroll_start_20… | 25,544 | c78ec7ad7feda1f32ff96c95b4ac6d6703a8c4f42ed31f275e1683a5fa36d9e0 |
| cdp-bodies/search_graph_query_(2021) 2 SCC 1 | 288,365 | 59a34e531bbf82e3091f4abb8301525f69d32288e395ed3559b9151bf1096962 |

## B. Findings

Each row gives the finding, its class, evidence (source and location), what it does **not** prove, confidence, and the next step.

| ID | Layer | Finding | Class | Evidence | Does not prove | Conf. | Next step |
|---|---|---|---|---|---|---|---|
| F-01 | 1/2 | Judgment URL `/judgement/{country}/{id24}`; the same id appears in all APIs | OBSERVED | EV-04 inline `judgementId`; EV-06 linkDetails `fjid.jid`; search `id` | That the id is a primary key | High | — |
| F-02 | 1/2 | The id timestamp prefix trails the judgment date (median 6 days, min 0, n=110) | INFERRED (ObjectId-style ids at ingestion) | cdp-bodies search_graph + search_scroll; local decode | MongoDB is the system of record | Med-High | Test ids of very old judgments vs their ingestion era |
| F-03 | 2 | Field named `recordInMongo` in the recent-searches response | OBSERVED (name) | EV-09 `/research/recentSearches` shape (values withheld: personal data) | Which collections live in MongoDB | Med | — |
| F-04 | 1 | Original document at `/judgement/pdf/{c}/{id}`, pdf.js render; RTF fallback | OBSERVED | `judgement.js` L17, L94–118, L211; EV-08 step 3 fetch | File storage location | High | Capture headers (Content-Type/Length) with header capture on |
| F-05 | 1 | Paragraph anchors `p_{n}`; `citedParas` returns `{p_n: count}` | OBSERVED | EV-08 `/judgement/citedParas`; `judgement.js` L331–370 | How paragraphs are segmented at ingestion | High | — |
| F-06 | 1 | Quality flags `scanned, junk, rlang, table, ai, noIndex, sensitive` in search records | OBSERVED names | cdp-bodies search_* | Their exact semantics | Med | Find a scanned or regional-language judgment |
| F-07 | 3 | Edge `{id, fjid, tjid, sentiment, snippet[{pid[], opinion}], reason, snippet_span, eBy, eByName, eAt}` | OBSERVED | EV-06 linkDetails (31 KB); cdp-bodies linkDetails/citeText | Completeness of the private edge schema | High | — |
| F-08 | 3 | `fjid` is citing, `tjid` is cited; outgoing via `linkDetails`, incoming via `citeText` | OBSERVED | 10/10, 48/48 (fjid==page); 100/100 (tjid==page) | Reciprocity of each edge | High | Page `/citeText/scroll` to find the Motilal→Vidya Drolia edge |
| F-09 | 3 | Sentiment values: positive, referencing, distinguished, overruled, negative | OBSERVED | linkDetails (Motilal, Vidya Drolia), citeText (Vidya Drolia) | Full vocabulary | High | More judgments |
| F-10 | 3 | `overruled` can mark status *mentioned* by the citing case | INFERRED | Motilal edges to Himangni and SBP & Co. with snippets | Labelling method | Med | — |
| F-11 | 3 | Incoming list truncated: 100 of 1,288; year histogram `opinionMap` | OBSERVED | cdp-bodies citeText | Page size of `/scroll` | High | One `/citeText/scroll` call |
| F-12 | 3 | `citedByCount` 1,173 (search) vs `citedByListSize` 1,288 (citeText) | OBSERVED contradiction | cdp-bodies search_graph vs citeText | Which is current | High | — |
| F-13 | 3 | `/search/graph` returns an induced subgraph (643 edges, all within 100 nodes) | OBSERVED | cdp-bodies search_graph `citedByLines` | Graph database use | High | — |
| F-14 | 3/2 | Judgment-level roll-up: `ovrRuled`, `sentimentCategory`, `sentimentIds[]` | OBSERVED | cdp-bodies search_scroll (11 negative, 9 overruled of 110) | How the roll-up is computed | High | — |
| F-15 | 4 | Search flow: autosuggest → `/search/{c}/{sanitized q}` → SSR → `/search/scroll` (no query parameter) | OBSERVED | `newuimain.js` L72–81; EV-09; cdp-bodies search_scroll | Where session state is kept | High | — |
| F-16 | 4 | Two indexes: judgments and CiteTEXT (`/search/opinion?sort=_score`) | OBSERVED (distinct shapes and hits) | EV-09, EV-11 | Engine identity | High | — |
| F-17 | 4 | `_score` and `<em>` highlighting suggest an Elasticsearch/OpenSearch-class engine | INFERRED | EV-11 `/search/opinion` | Engine identity | Med | Error responses or header `Server` values |
| F-18 | 4 | Citation string searched lexically (1,440,078 hits) | OBSERVED | EV-11 | Behaviour of the `parallel:` scope | High | Try `parallel:` scope once |
| F-19 | 4 | Same query, same hits, different top 10 | OBSERVED | EV-09 vs EV-11 `/search/opinion` (sha 3ed4368d vs 64db9ab5) | Cause | High | A third repeat |
| F-20 | 5 | AMICUS: live `POST /chat/ask/stream` (question, country, effort=3, verb=2, caseId); SSE: connected → 5 status → data deltas → final → done | OBSERVED | EV-17 SSE body sha 3be4696e…; `chat.js` L1444–1467, L1982 | Behaviour in other modes | High | — |
| F-27 | 5 | Declared stages: understanding_question (term expansion), searching_semantic, searching_authorities, collecting_citations, generating_answer | OBSERVED (server labels) | EV-17 `event: status` lines | Actual retrieval or embedding implementation | High (labels) / Low (implementation) | — |
| F-28 | 5 | Answer cites `/judgement/in/{id24}[#p_n]` inline (5 citations, 4 authorities) plus a References block with "Proposition relied on" and "Applicability" | OBSERVED | EV-17 SSE data lines | That cited paragraphs support the propositions | High | Open the 2 anchored paragraphs |
| F-31 | 5 | 3/3 AMICUS paragraph anchors point to the wrong paragraph; the supporting text is elsewhere in the same judgment (N.N. Global p_797 not p_523; BSNL p_122/p_123 not p_15) | OBSERVED | citation-check/*.json (sha 0178e9c0…, 0b1b0ab4…, 5d83b433…, 9e2b7d05…); scenarios ev_49fc9c71…282b, ev_3a5c8fff…e4e7 | Error rate beyond this one answer; the cause | High (for this sample) | More answers; compare anchors with printed paragraph numbers |
| F-32 | 1 | `p_N` ids are DOM-sequential, not printed paragraph numbers (p_523 is printed ¶212; p_797 is printed ¶323; 1,057 p_ elements) | OBSERVED | citation-check/nn-global-*.json | How ids are assigned at ingestion | High | — |
| F-29 | 5 | Chat record `GET /chat/meta/{id}` has `productType:"AMICUS"`, `skill:"chat"`, `vectorStoreId`, `vectorStoreStatus`, `fileTokens` | OBSERVED | EV-17 meta (352 B) | Vector-store provider | High | — |
| F-30 | 5 | Trial-package credit counter (`chatLeft`, `limitMessage`) decremented by one question | OBSERVED (values withheld) | EV-17 `event: final` | Pricing | High | — |
| F-21 | 5 | Uploaded-file Q&A maps to `openaiFileId`/`file_id` | OBSERVED (code); OpenAI provider INFERRED for this feature | `chat.js` L2158–2197 | Model for normal answers | Med | — |
| F-22 | 5 | Smart Summary fixed sections; on-demand `GET …/summary/generate` | OBSERVED | EV-04 accessibility tree; `judgement.js` L160–200 | Generation method | High | — |
| F-23 | 5 | Judgment-page CaseIQ tab is client-only (image swap) | OBSERVED | `judgement.js` L615; EV-07 (no XHR) | CaseIQ tool behaviour | High | Load `/caseiq` |
| F-24 | 6 | Single application origin; third parties: Google Analytics/GTM, Google Translate | OBSERVED | All scenario `origins contacted` | Internal service split | High | — |
| F-25 | 6 | Public API marketed for **US** legal data | OBSERVED (marketing text) | EV-14 | API schema | High | Request docs from vendor |
| F-26 | 1 | Policy: judgments verbatim, de-indexing on request, no redaction without court order | OBSERVED (policy) | EV-14 | Technical enforcement | High | — |

## C. Failed or empty observations (kept on purpose)

- **Homepage bundle analysis:** found no routes or endpoints. REA's detector covers `fetch`/WebSocket, and CaseMine uses jQuery `$.ajax`.
- **Semantic trace from `/search/autosuggest`:** 0 seed matches. The per-file node budget (20,000) was exhausted inside inlined jQuery.
- **Two `observe_web_session` windows:** no navigation, because the operator was acting in a different window. `inspect_web_page` 60 s window: 0 requests, because clicks opened new tabs.
- **ORIGINAL PDF tab:** the role locator failed twice. It worked with the CSS selector `#show-pdf-tab`. The PDF fetch was aborted by pdf.js streaming (`ERR_ABORTED`), so no bytes were retained.
- **`/judgments/in` while anonymous:** ended at `/login`.
- **AMICUS submit, attempts 1–2:** sent nothing. Clicking a suggested prompt fills the editor without firing `input`, so `#chat-send` stays disabled; Enter doesn't submit, and `#send-icon-bt` isn't actionable. Exactly one question was sent (attempt 3, EV-17).
