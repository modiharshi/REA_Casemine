# CaseMine: evidence gaps, open hypotheses and failed experiments

This is deliverable 6, consolidated from rounds 1–4. F-IDs refer to [evidence-ledger.md](evidence-ledger.md).

## Open hypotheses (INFERRED, not proven)

| # | Hypothesis | For | Against / unknown | What would settle it |
|---|---|---|---|---|
| H1 | Judgment and edge IDs are MongoDB ObjectIds minted at ingestion | Timestamp prefix ≥ judgment date in 110/110 records (median 6 days); `recordInMongo` field name (F-02, F-03) | No server-side confirmation | Not settleable from outside |
| H2 | Search runs on an Elasticsearch/OpenSearch-class engine | `sort=_score`, `<em>` highlighting, token-by-token citation matching (F-17, F-18) | Engine never identified | An error page or header naming the engine (none seen; `Server: CaseMineWeb`, F-57) |
| H3 | The web tier is a Java servlet container (Tomcat-style) | Weak static ETag format `W/"<len>-<mtime ms>"` (F-58) | The custom `Server` header hides it | Not settleable without error pages |
| H4 | AMICUS (`/chat`) answers come from OpenAI GPT | "AMICUS (Powered by GPT)" and "Smart Summary (Powered by GPT)" labels in CaseIQ; `openaiFileId` / `vectorStoreId` fields (F-21, F-29, F-45, F-65) | `/chat` shows no label | Not settleable from outside |
| H5 | Wrong AMICUS paragraph links come from chunk-start labels or unvalidated model output | All misses point earlier; 1 off-by-one (F-48) | 2 far misses (−107, −274) don't fit a chunk-start explanation | Larger sample (needs a paid plan) |
| H6 | The ~117 incoming citations missing from `citeText` paging are hidden, excerpt-less, or a stale counter | 1,288 vs 1,171 retrievable vs 1,173 in search (F-38) | — | Not settleable without enumerating all edges |
| H7 | The `overruled` label marks status *mentioned*, so "Overruled By" overstates | SBP "Overruled By" includes Motilal Oswal, which only mentions SBP as overruled (F-10, F-60) | Labelling method unknown | Manual review of more `overRuled` lists |
| H8 | `prio` (0/50/60/80/100) is a ranking or authority weight | Present on search records; not court-only | Meaning unknown | — |
| H9 | Commentary articles are machine-generated | Uniform headline-style titles (F-66) | No label seen | An authorship label on an article page |

## Failed or empty experiments (kept on purpose)

| Experiment | Result | Why | Evidence |
|---|---|---|---|
| REA bundle analysis on the homepage | 0 endpoints | REA models `fetch`/WebSocket; CaseMine uses `$.ajax` | Ledger §C |
| REA semantic trace with jQuery bundled | 0 seed matches | Node budget used up inside jQuery | Ledger §C |
| REA literal-seed trace on CaseMine-only scripts (`/caseiq/text`) | 0 matches | Literal not retained as a seedable node; request nodes extracted from the record instead | F-55 |
| `observe_web_session` ×2 | No navigation recorded | Operator acted in a different window | Ledger §C |
| Live listener inspection (`#chat-send`) | Only jQuery's dispatcher | jQuery wraps every handler | F-61 |
| ORIGINAL PDF click ×3 | Timed out, twice by role locator and once after a modal | Actionability; `#show-pdf-tab` CSS click worked once | Ledger §C, R4-01 |
| AMICUS submit attempts 1–2 | Nothing sent | ProseMirror editor and send-button enablement | Ledger §C |
| AMICUS question 5 | Refused: trial credits exhausted | Account limit | F-47 |
| Mayavati judgment load | reCAPTCHA interstitial; operator solved it | Rapid loads | F-37 |
| `(2021) 2 SCC 1` via Citation Code Search | `[]` | Not in the code index (or the daily limit; indistinguishable) | F-42 |
| `/commentary/search` direct request | 200 "Server Error" HTML | Unknown | F-63 |
| `/research` inventory | Skipped on purpose | Private account area | F-68 |

## Gaps that remain (deliberately or by limits)

- **Not observable from outside:** storage engines, search engine, model, prompts, ranking and service boundaries.
- **Needs credits or a paid plan:** a larger paragraph-link sample, AMICUS upload Q&A and drafting, the CaseIQ `askai` tab, Smart Summary generation.
- **Not exercised:** `/search/briefs`, Columns, Alerts, the Lawyers directory, and the `title:`/`appellant:`/`respondent:`/`lawyer:` scopes.
- **Unresolved contract:** `/commentary/search`.
