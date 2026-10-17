# CaseMine: observed API contracts and data models

This is deliverable 4.
- **Status:** OBSERVED means a live request and response were captured; CODE means the endpoint appears only in shipped JavaScript, with its location given.
- **Shapes:** field names and types only; values are not reproduced.
- **Origin:** everything is same-origin on `www.casemine.com`. Responses carry `Server: CaseMineWeb` and `Cache-Control: no-store` (F-57).
- **CSRF:** `X-CSRF-TOKEN` is the page's `token` variable, sent where the table says.

## Judgments and citations

| Method / path | Params | Response (shape) | Status | Evidence |
|---|---|---|---|---|
| GET `/judgement/{c}/{id}` | — | HTML (server-rendered text, `p_N` or bare-number paragraph IDs, inline metadata variables) | OBSERVED | F-01, F-52 |
| GET `/judgement/{c}/{id}/linkDetails` | `sort=score`, `query` | `{citesToListSize:int, citesToGraphList:str(JSON), citesToList:[Edge]}` | OBSERVED | F-07 |
| GET `/judgement/{c}/{id}/citeText` | `sort=score\|date`, `query`, `from`, `courtType`, `paraId`, `citedByCount` | `{citedByListSize, opinionCount, packageLevel, opinionMap:str(JSON year→n), citesByList:[Edge ≤100], citesByGraphList:str, opinionList:[Edge ×10]}` | OBSERVED | F-11, F-38, F-39 |
| GET `/judgement/citedParas` | `judgementId`, `country`; CSRF | `{ "p_N" \| "N": count }` | OBSERVED | F-05, F-51 |
| GET `/judgement/overRuled/{c}/{id}` | — | `{sids:[id24], category:"overruled"\|…, sTitles:[str]}` | OBSERVED | F-60 |
| GET `/judgement/lawyers` | `judgementId` | `{lawyers:[], banner:str}` | OBSERVED | Round 1 |
| GET `/judgement/pdf/{c}/{id}` | — | `application/pdf` from the app server (no redirect) | OBSERVED (headers only) | F-59 |
| GET `/judgement/rtf/{c}/{id}`, `/judgement/{c}/{id}/content` | — | RTF; `{content}` (UK path) | CODE `judgement.js:98,220` | F-04 |
| GET `/judgement/{c}/{id}/summary/generate` | CSRF | HTML fragment | CODE `judgement.js:162` (may use credits; not called) | F-22 |
| GET `/judgement/similar/{c}/{id}` | — | — | CODE, dead (never used) | F-56 |

**Edge** = `{id, fjid:Stub, tjid:Stub, sentiment:"positive"|"referencing"|"distinguished"|"overruled"|"negative", snippet:[{pid:[str], opinion:html, groupId, length}], reason, snippet_span, eBy, eByName, eAt}`. `fjid` is the citing judgment and `tjid` the cited one.

**Stub** = `{jid, title, date:epoch-ms, courtName, year, courtFlag, judgmentDate, pid, sentiment?, score?}`.

## Search

| Method / path | Params | Response | Status | Evidence |
|---|---|---|---|---|
| GET `/search/autosuggest` (`/judgename`) | `searchfield` | `[{label, value, category}]` | OBSERVED | F-15 |
| GET `/search/{c}/{q}` | — | HTML, top 20 | OBSERVED | F-15 |
| GET `/search/scroll` | `start`, `from`, `isTribunal` (query is held server-side) | `{result:[SearchRecord], hits, allCourts[64], allCourtsMap, …buckets}` | OBSERVED | F-15 |
| GET `/search/opinion` | `start`, `sort=_score`, `query` | `{result:[{id, fjidId, fjidTitle, fjidCourtName, tjidId, tjidTitle, tjidCourtName, fpid, tpid, snippet:[html]}], hits}` | OBSERVED | F-16, F-36, F-67 |
| GET `/search/graph` | `query` | `{result:[SearchRecord ≤100], citedByLines:[{id1,id2,x1,x2,y1,y2,strokeColor,strokeWidth,sentimentPresent}], allYearsBucket, …}` | OBSERVED | F-13 |
| GET `/search/act` | `start`, `sort=_score`, `query`, `actType`, `state` | `{result:[{id, actName, fullName, shortTitle, sectionText, citations[], enacmentDate, date, jurisdiction, state, …}], hits, stateBucket}` | OBSERVED | F-62 |
| GET `/rca/search/{c}/{q}` | `start`, `sort`, `year`, `customYearFilter`, `state` | `{result:[{id, title, courtName, year, month, date, content, notificationNumber, gazetteRefNo, effectiveDate, …, noIndex, rlang, scanned}], hits}` | OBSERVED | F-62 |
| GET `/commentary/search/{c}/{q}` | same as RCA | UNKNOWN (body over budget; a direct request returned a 200 "Server Error" page) | OBSERVED (failure) | F-63 |
| GET `/citationsearch/codes` | `journal`, `year`, `volume`, `page`, `type=journal\|year\|volume\|page` | `[year]` / `[volume]` / `[page]` / `[jid]` | OBSERVED | F-41 |
| GET `/citationsearch/judgement` | `id`, `citationcode` | `{result:[SearchRecord]}` | OBSERVED | F-41 |

**SearchRecord** keys are listed in the report, section Q2.

## AI

| Method / path | Params | Response | Status | Evidence |
|---|---|---|---|---|
| POST `/chat/ask/stream` | form: `question, country, effort, verb, caseId[, chatId, trial, webSearch, mode]`; CSRF | SSE: `status{code,message}` ×5, data deltas (HTML with `cite<a href="/judgement/…#p">`), `final{id, chatId, chatLeft, limitMessage, mode, credits, walletUsed, walletBalance, fileSearch, webSearch, fileSearchSources, webSearchSources, fileStatusMap}`, `done`, or `error` | OBSERVED | F-20, F-27, F-47 |
| GET `/chat/meta/{chatId}` | — | `{id, title, productType, status, publicId, createdAt, country, skill, vectorStoreId, vectorStoreStatus, fileTokens, lastUsed, deletedAt, updatedAt}` | OBSERVED | F-29 |
| GET `/chat/list` | — | `[]` | OBSERVED | — |
| POST `/caseiq/text` | form: `docName, docText(html), caseIQId, isGraph, caseIQDetailsId`; CSRF | run ID (text) | OBSERVED | F-44 |
| GET `/caseiq/summary` | `caseIQId`, `caseIQDetailsId` | `{facts[], issues[], "petitioner arguments"[], "respondent arguments"[], holding, reasoning[]}` (stored; identical on replay) | OBSERVED | F-44, F-64 |
| GET `/caseiq/arguments` | `sort=_score`, `start`, `caseIQId` | `{html, arguments}` | OBSERVED | F-64 |
| `/caseiq/url`, `/docupload`, `/opinion`, `/filter` (fetch), `/citetext`, `/paraMatch`, `/askai/{id}`, `/uscitation`, `/brief` | — | — | CODE (`caseiq.js` fetch sites 291, 762, 1106, 1760, 2222 per REA) | F-55 |
| `/chat/upload/ask[/stream]`, `/ai/draft/*`, `/chat/promptEnhance`, `/chat/export/docx/` | — | — | CODE `chat.js` | F-21 |
