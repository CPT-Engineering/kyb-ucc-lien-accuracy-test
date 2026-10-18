# US company, UCC lien and sanctions check: accuracy test (Oct 17, 2026)

A dated, reproducible-by-method accuracy test of three company-check tools, plus a vendor-neutral checklist of the mistakes to look for in **any** KYB, UCC lien or sanctions-screening tool.

**Disclosure:** CPT Engineering PLLC built and sells the three tools tested here, on Apify (pay per result). This page is ours, not an independent audit. Every number below comes from live cloud runs, checked against the official state and U.S. Treasury records listed under "How results were checked". No other vendor's tool is named or scored here.

Skip to the [buyer checklist](#what-to-check-in-any-kyb-or-lien-tool) if that's all you need.

Public summary of the test behind the "Tested accuracy" sections of three CPT Engineering Apify Actors:
[Company Due Diligence Check](https://apify.com/cpt_engineering/company-due-diligence-check),
[US Company Lookup & KYB](https://apify.com/cpt_engineering/us-company-lookup-kyb) and
[UCC Lien Filings](https://apify.com/cpt_engineering/ucc-lien-filings).
All inputs are businesses (or invented names); no personal data was used or is published here.

## Test set (20 inputs)
| Group | Inputs | What it tests |
|---|---|---|
| UCC lien debtors | 6 companies registered in Colorado (3, incl. New Belgium Brewing Company, Inc.) and Connecticut (3), and 1 Oregon nonprofit | Active lien counts against the official filing-office records |
| Large registered companies | Wegmans Food Markets, Inc. (NY), Sheetz, Inc. (PA), Whataburger Restaurants LLC (TX) and 1 other Texas company | Registry match, status, Texas right to transact |
| OFAC-listed entities | 3: one Texas-registered and one New York-registered company on the OFAC SDN list, plus Banco Nacional de Cuba (not U.S.-registered) | Sanctions screen |
| Name collision | 1 Colorado LLC formed in 2003 whose name equals an OFAC SDN entity established in 2019 | Date check on sanctions matches |
| Common names | ABC Construction LLC, First National Bank, Star Group LLC | Wrong-company matches |
| Invented names | 2 made-up company names | Explicit not-found rows |

Inputs were given as `Name | ST` when the state is known (14) and as a bare name otherwise (6). Short names were included on purpose (for example "M & C Concrete LLC", registered as "M & C Concrete LLC.").

## How results were checked
- **Liens:** Colorado Information Marketplace UCC datasets (filings, debtors; amendments, continuations, terminations and lapse dates followed back to the original filing) and the Connecticut UCC dataset (original financing statements, active, not lapsed). A lien counts as active only if it has not been terminated or released and has not lapsed.
- **Sanctions:** U.S. Treasury OFAC SDN and consolidated (non-SDN) lists as published on the test date, including the "Organization Established Date" remark.
- **Registry and Texas status:** the state open-data registries (CO, CT, NY, OR, PA, TX) and the Texas Comptroller franchise-tax `right_to_transact_business_code`.
- **Billing:** `chargedEventCounts` on the Apify run, compared with the rows returned, plus a run with a spend limit (`maxTotalChargeUsd`).

## Results on the live builds (Oct 17, 2026, about 11:05 AM ET)
| Check | Result |
|---|---|
| OFAC-listed entities flagged | **3 of 3** (POTENTIAL_MATCH), including the New York-registered SDN company |
| Name-collision sanctions match | Flagged and annotated: listed entity established 2019-10-23, after the company's 2003 formation (SANCTIONS_MATCH_ESTABLISHED_AFTER_FORMATION) |
| Active UCC lien counts vs official records | **6 of 6 lien debtors matched exactly** (CO and CT), e.g. 19 of 19 active filings for New Belgium Brewing |
| Expired or closed liens | **Not counted.** Colorado holds 58 financing statements for New Belgium Brewing since 2002; 39 have lapsed or been terminated, and only the 19 active ones were counted |
| Amendments | **Not double-counted.** Amendments are attached to their original filing (one Connecticut company has 30 rows in the state data, of which 9 are active original financing statements; we report 9) |
| Wrong-company matches | 0 counted. Common names returned similar-name filings separately and uncounted (Star Group LLC: 0 counted, 11 listed as possibly related; First National Bank: 1 counted, 15 listed) |
| Registry | 17 of 17 registered companies matched to the right entity (incl. the short name "M & C Concrete LLC."); 3 of 3 unregistered names returned an explicit NOT_FOUND row |
| Texas right to transact | 2 of 2 forfeited companies flagged FORFEITED |
| Rows | **One row per input**, 20 of 20, in every Actor; 0 failed runs |
| Speed (batch of 20, Apify cloud) | Due Diligence Check ~5.5 s per company (registry, UCC and government liens, OFAC, SAM.gov; 5.0 s measured alone, 5.5 s with the other two Actors running at the same time); Company Lookup ~3 s; UCC lien check ~4.8 s |
| Exact-match-only billing (UCC Lien Filings) | 51 charged filing events = the 51 exact-name filings returned; 21 similar-name filings returned free (`charged: false`). Spend-limit run: stopped cleanly at the limit, 7 charged events = 7 exact rows, 10 similar rows free |
| Free when a source fails | By design: a Due Diligence row with any failed source is PARTIAL and not charged; Company Lookup NOT_FOUND is free when a state portal failed; UCC NO_FILINGS_FOUND is free when a state failed |

The same 20 inputs were run a few hours earlier on the previous builds. That run found one wrong-company match (a short name matched a similar-named company and its liens) and counted other companies' liens for the three common names; both were fixed before this run.

## What to check in any KYB or lien tool
1. **Expired liens counted as active.** UCC filings lapse after 5 years unless continued, and can be terminated earlier. Ask whether "active" means not terminated *and* not lapsed.
2. **Amendments double-counted.** Amendments, continuations and assignments are separate filings in most state data. They should be attached to the original, not counted as new liens.
3. **Wrong-company matches.** Short and common names ("M & C Concrete", "Star Group") easily match a different company. Look for a match score, the matched legal name, and a separate list for similar names.
4. **Charged for failed or empty rows.** Check whether a row is billed when a state source fails or when the tool returns nothing for an input. Every input should get a row that says found, not found or failed.
5. **Sanctions name collisions.** A same-name OFAC entry is a lead, not a finding. Compare the listed entity's established date, country and programs with the company's record.

## Limits of this test
20 inputs; 6 states for the registry and 3 (CO, CT, OR) for liens; Oregon lien data covers only the previous month. Name matching normalizes punctuation, spacing and word order and treats equivalent legal forms as the same (Inc = Corp = Co; LLC = L.L.C.). Since the afternoon of Oct 17, a filing whose legal form conflicts with the company's (for example LP vs LLC) is treated as a different entity and not counted or charged; a filing with no legal form still matches on its words. Not a certified search.

## Re-test after the legal-form and speed updates (Oct 17, 2026, about 12:11 PM ET)
Same 20 inputs on the live builds after two changes: conflicting legal forms (LLC vs LP vs Inc/Corp vs LLP) are treated as a different entity, and the Due Diligence Check's UCC and government-lien searches share their requests to each state portal.
- All results above are unchanged (sanctions 3 of 3, lien counts 6 of 6, registry 17 of 17 plus 3 NOT_FOUND, 51 charged = 51 exact-name filings, 21 similar-name filings free), with one intended change: a municipal lien filed against "STAR GROUP LP" is no longer counted as a government lien for "Star Group LLC".
- Due Diligence Check: 99.8 s for 20 companies run alone (5.0 s per company) and 109.5 s alongside the other two Actors (5.5 s), down from 130 s before the shared requests.

## The tools
| Tool | What it returns | Price (Apify, pay per result) |
|---|---|---|
| [Company Due Diligence Check](https://apify.com/cpt_engineering/company-due-diligence-check) | One row per company: registry match and status (CO, CT, NY, OR, PA, TX), Texas right to transact, active UCC and government liens (CO, CT, OR), OFAC SDN and consolidated screen, SAM.gov exclusions | $0.10 per complete check; rows with a failed source are not charged |
| [US Company Lookup & KYB](https://apify.com/cpt_engineering/us-company-lookup-kyb) | Registry match, legal name, status, formation date, entity type | $3 per 1,000 found, $1 per 1,000 not found |
| [UCC Lien Filings](https://apify.com/cpt_engineering/ucc-lien-filings) | New UCC-1s, lapsing filings, or a lien check by company name (CO, CT, OR) | $4 per 1,000 exact-name filings; similar-name filings returned free and flagged |

Related free data samples: [us-new-business-filings-sample](https://github.com/CPT-Engineering/us-new-business-filings-sample), [form-d-weekly-sample](https://github.com/CPT-Engineering/form-d-weekly-sample).

None of these tools return officer, organizer, agent or other personal names. Not legal advice and not a certified search; confirm any finding against the official source before acting on it.

Questions or corrections: open an issue here or email cpt-datatools@mail.grokbot.com.

*This page was drafted with AI assistance from our own test records. License: CC BY 4.0.*
