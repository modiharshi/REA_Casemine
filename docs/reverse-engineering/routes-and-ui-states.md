# CaseMine: frontend routes, components and UI states

This is deliverable 3. Page routes are server-rendered; each route loads one page script on top of the shared `newuilogin.js` / `newuimain.js` bundle (jQuery 3 and Bootstrap). There are no source maps.

## Page routes

| Route | Page script | Main components | Evidence |
|---|---|---|---|
| `/` | `newuimain.js` | Search box with scope dropdown (`#sub_query`), autosuggest, marketing sections | F-15 |
| `/search/{country}/{query}` (form `GET /search/in`) | `search.js` v1.0.4, `d3/search.viz.js` | Result tabs, facets, infinite scroll, Visual graph, CaseIQ and AMICUS banners | F-15, F-16 |
| `/judgement/{country}/{id24}` | `judgement.js` v2.2.8, `judgment-pdf.js`, `d3/judgement.viz.js`, popline plugins | JUDGMENT / ORIGINAL PDF / Subsequent References / CaseIQ tabs; Smart Summary; citation graph; heat map; badges | F-04, F-05 |
| `/judgments/{country}/{court}/{yyyy}/{mm}/{dd}` | — | Daily listing (requires login) | Round 1 |
| `/citationsearch` | `citation-search.js` v1.0.1 | Journal, year, volume and page pickers | F-41 |
| `/chat`, `/chat/{chatId}` | `chat.js` v1.4.0, ProseMirror | Chat list, editor (`#chat-textarea`, `#chat-send`), streamed answer, export | F-20 |
| `/caseiq`, `/caseiq?id={run}` | `caseiq.js` v1.1.1, ProseMirror (ciq) | Upload / Add Text / Add URL tabs; result tabs | F-44 |
| `/acts/in` | — | Central Acts; State Acts by state | F-66 |
| `/commentary/{country}/{court}` | — | Commentary article list | F-66 |
| `/home/*` | — | Static pages (API marketing, takedown policy, FAQ, pricing) | F-25, F-26 |

## Component → handler map (static, REA)

REA mapped **265 listener registrations** to file and line (F-55). Selected controls:

| Control | Event | Handler location |
|---|---|---|
| `#show-pdf-tab` (ORIGINAL PDF) | click | `judgement.js:94` |
| `#generateSummary` | click | `judgement.js:160` |
| `#third-tab` (CaseIQ tab on a judgment) | click | `judgement.js:614` |
| `.jd-type.overruled` / `.jd-type.negative` | click (delegated) | `judgement.js` around L1177 (manual) |
| `#search-btn-ijs`, `#search-btn-inner` | click | `search.js:3342`, `search.js:3595` |
| `#query` | keydown | `search.js:3357` |
| `#getCaseIq`, `#getCaseIqUrl` | click | `caseiq.js:1350`, `caseiq.js:1378` |
| Journal, year, volume, page pickers | click | `citation-search.js:388–436` |
| `#chat-send` | click | Not resolved statically (REA found 0 listeners in `chat.js`). Live inspection shows only jQuery's dispatcher (F-61). Manual: `chatStart` at `chat.js:1399` |

The full list is in `evidence/raw/round4/rea-listeners-resolved.json` (gitignored).

## UI states observed

| State | Where | Trigger | Evidence |
|---|---|---|---|
| Login redirect | `/judgments/in` when anonymous | Not logged in | Round 1 |
| reCAPTCHA Enterprise interstitial ("Capcha - Judgment") | Judgment page | Rapid consecutive judgment loads | F-37 |
| Trial exhausted (`event: error`, subscribe link) | AMICUS | 5th question on the trial | F-47 |
| "Showing top 100 of N" | Subsequent References | Over 100 incoming citations | F-39 |
| "No more CiteTEXT excerpts to display." | Subsequent References | End of excerpt paging | code |
| "PDF unavailable – Please download text" | ORIGINAL PDF | `fileFormat == 'rtf'` | code (F-04) |
| "No citations automatically detected within uploaded document." | CaseIQ, Citations Found | Input text without case citations | F-64 |
| "Argument(s) not found." | CaseIQ, Suggested Arguments | `/caseiq/arguments` returned `arguments: null` | F-64 |
| Empty `[]` lookup (not found, or daily limit; the code can't tell them apart) | Citation Code Search | Missing citation | F-42 |
| HTTP 200 with a "Server Error" HTML page | `/commentary/search/…` direct request | Unknown | F-63 |
| JavaScript error "Cannot read properties of undefined (reading 'length')" | Search, Commentaries tab | Unknown | F-63 |
