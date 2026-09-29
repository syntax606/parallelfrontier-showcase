# pf-authors: who wrote it, who funded it

A pipeline that reads every new AI paper on arXiv and answers three questions with evidence:

1. Where do the authors work?
2. Who paid for the work?
3. Which author records belong to the same person?

## Stages

| Stage | What happens | How |
|---|---|---|
| List | Every new paper in cs.AI, cs.CL, cs.LG, cs.CV, cs.CR, cs.MA, cs.RO, cs.HC, cs.CY, stat.ML | arXiv bulk metadata (OAI-PMH) |
| Fetch | Full text of each paper | arXiv HTML, with the PDF's first page when the HTML drops affiliations; rate-limited; nothing kept on disk |
| Parse | Author block, affiliation lines, emails, names in characters | deterministic parsing |
| Affiliations | Each line → institution and the place it states | bilingual institution index |
| Funding | Every funding or in-kind acknowledgement, wherever it appears | sentence classifier, funder extraction, grant-number parser |
| Scope | In / out / unknown under each saved definition | a filter over the stored evidence, so changing a definition needs no re-run |
| Identity | Author records → people | log-likelihood matcher using pf-names |
| Review | Checks, funder labels, overrides | local web app; decisions survive re-runs |

## Hard problems and how they were solved

**"Hong Kong SAR, China" is not mainland China.** Scope is decided from the place a line actually names, matched on
whole words. That rule also fixed places turning up inside personal names ("Chuanhui" is not Anhui; "Jingxian" is not
Xi'an).

**Institution names are ambiguous.**
- The index combines a curated seed with about 8,800 organisations from the Research Organization Registry.
- An acronym counts only when it is unambiguous. HUST (Wuhan or Hanoi) and NTNU (Taipei or Trondheim) are excluded.
- Ordinary-word names count only when they make up a whole comma segment.
- A company match is dropped when the line states another country, so Huawei's Zurich lab is not China.

**A funder's country depends on the sentence.** "Ministry of Education" may be China's, India's or Korea's. Generic
names are resolved sentence by sentence from the place named, national markers (grant prefixes, programme names), or
the one country the sentence mentions.

**Identity without overclaiming.**
- Each link is graded:
  - **Confirmed** needs a hard identifier (email, ORCID, grant PI) or identical characters plus corroboration.
  - **Probable** is the ceiling for soft signals alone: name, co-authors, institutions and topic.
  - **Unresolved** means the evidence isn't enough.
- Weights are calibrated on pairs of records that share an email address, against random pairs.
- In a human review of 30 unresolved pairs, 20 were the same person and none were different people. The matcher errs
  on the side of caution by design.

**Scope definitions are separate from the evidence.** Mainland only, mainland plus Hong Kong and Macau, and with
Taiwan are all recorded side by side. An email-only signal is flagged for review but never enough on its own.
Chinese companies count as funders, and in-kind support (compute, data) counts as funding.

## Human in the loop

A local review app shows each machine decision with its evidence: the affiliation line, the funding sentence, and the
name comparison with its reason. A reviewer can confirm, reject or relabel. Decisions are stored separately from
machine output, override it, and survive every re-run. The app binds to localhost, and researcher data never leaves
the analyst's machine.
