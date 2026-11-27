# CredAIble - Australian Personal Loan Reference Policy v0.1

> Status: DRAFT, inactive until a lender reviews and approves it (A-35). Sources checked 2026-11-22.
> Audience: lender Business Admins, credit and compliance staff, and the engineers who will encode these rules.
> Not legal advice. Every regulatory statement is to be confirmed with counsel (A-23, OP-01, OP-02).
> Scope: consumer personal loans from a non-bank lender to one PAYG applicant. Joint and self-employed cases go to manual assessment (A-25).

## 1. How to read this policy

| Label | Meaning | Can a lender change it? |
|---|---|---|
| **Statutory** | Required by legislation in the cited section | Not without counsel. It may be made stricter where lawful |
| **Guidance** | The regulator's published expectation | Yes, with a recorded reason |
| **Market** | Published by at least two of the three lenders sampled | Yes |
| **Illustrative** | Proposed by CredAIble as a starting point. No authority behind the value | Yes, and it should be calibrated |
| **Lender choice** | No default is proposed. The lender must choose | Yes, required |
| **Product control** | A CredAIble control from the project brief | No |

Outcomes:

| Outcome | Meaning |
|---|---|
| BLOCK | The process step cannot continue. This is never a decline |
| REFER | An assessor must look at the issue |
| REC-DECLINE | Recommend decline with a reason code. A human decides (D-06, A-26) |
| CONTROL | A required process or record |

`UNSET` means no defensible public value was found. The lender must set it before activation.

Source codes are defined in section 11.

---

## 2. Product boundary

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| PB-01 | Treat as Code-regulated where a natural person borrows wholly or predominantly for personal, domestic or household purposes, a charge may be made, and credit is provided in the course of a business | In scope | Statutory | CONTROL | NCC s 5 |
| PB-02 | A business-purpose declaration does not work if the lender knew or had reason to believe otherwise, or would have after reasonable inquiry | Refer any business-purpose claim | Statutory | REFER | NCC s 13 |
| PB-03 | Identify small amount contracts: non-ADI, not continuing or low cost credit, $2,000 or less, 16 days to 1 year, unsecured | BLOCK until a SACC rule set is configured (D-33) | Statutory | BLOCK | NCCP s 5 |
| PB-04 | Identify medium amount contracts: non-ADI, not continuing or low cost credit, $2,001 to $5,000, 16 days to 2 years | BLOCK until a MACC rule set is configured (D-33) | Statutory | BLOCK | NCC s 204 |
| PB-05 | Do not enter a prohibited short-term credit contract | BLOCK | Statutory | BLOCK | NCCP s 133CA |
| PB-06 | Annual cost rate must not exceed 48% (exceptions include ADIs and low cost, small amount and bridging contracts) | 48%, calculated by the statutory method | Statutory | BLOCK | NCC s 32A |
| PB-07 | Loan amount | $5,001 to $50,000 | Illustrative | REFER | L1; starts above the MACC band |
| PB-08 | Loan term | Up to 7 years | Market | REFER | L1, L2, L3 |
| PB-09 | Minimum loan term | 18 months | Illustrative | REFER | L3 |

A $5,000 loan over two years or less is a medium amount contract, so a common "$5,000 minimum" would fall into that regime. That is why PB-07 starts at $5,001.

---

## 3. Responsible lending

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| RL-01 | Make reasonable inquiries about requirements and objectives | Required | Statutory | BLOCK | NCCP s 130(1)(a) |
| RL-02 | Make reasonable inquiries about financial situation | Required | Statutory | BLOCK | NCCP s 130(1)(b) |
| RL-03 | Take reasonable steps to verify financial situation | Required | Statutory | BLOCK | NCCP s 130(1)(c) |
| RL-04 | Record an assessment that states the period it covers and whether the contract would be unsuitable | Required before a lender enters a contract | Statutory | BLOCK | NCCP s 129 |
| RL-05 | Assessment, inquiries and verification fall within 90 days before the credit day, unless the regulations prescribe another period (regulations not reviewed) | 90 days. Re-assess when expired | Statutory | BLOCK | NCCP s 128 |
| RL-06 | Make no unconditional representation that the consumer is eligible before that assessment | No "approved" or unconditional "pre-approved" wording in the portal, emails or agent output | Statutory | BLOCK | NCCP s 128(aa) |
| RL-07 | Assess as unsuitable if the consumer is likely to be unable to pay, or could pay only with substantial hardship | Required | Statutory | REC-DECLINE | NCCP s 131(2)(a) |
| RL-08 | Assess as unsuitable if the contract is unlikely to meet requirements or objectives | Required | Statutory | REC-DECLINE | NCCP s 131(2)(b) |
| RL-09 | Presume substantial hardship if payment is only possible by selling the principal home, unless the contrary is proved | Required | Statutory | REC-DECLINE | NCCP s 131(3) |
| RL-10 | Do not enter an unsuitable contract. An assessor override cannot waive this | Required | Statutory | BLOCK | NCCP s 133 |
| RL-11 | Give a written assessment on request before entry, or for 7 years after the credit day: within 7 business days in the first 2 years, otherwise 21. No charge | Lender process using export | Statutory | CONTROL | NCCP s 132 |
| RL-12 | Do not rely on information there is reason to doubt until further steps confirm it | Required | Guidance | REFER | RG 209.132-133 |
| RL-13 | Monitor applications from third parties for recurring reliability concerns | Report by broker | Guidance | CONTROL | RG 209.134 |

---

## 4. Eligibility

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| EL-01 | Minimum age | 18 | Market | REC-DECLINE | L1, L2, L3 |
| EL-02 | Maximum age | None. An age cut-off needs actuarial or statistical data that is reasonable to rely on | Statutory | CONTROL | ADA ss 28, 37(4)-(5) |
| EL-03 | Residency | Citizen or permanent resident | Market | REC-DECLINE | L1, L2, L3 |
| EL-04 | Minimum verifiable gross annual income | More than $25,000 | Market | REC-DECLINE | L1, L2 |
| EL-05 | Probation | Refer | Illustrative | REFER | L3 excludes probation |
| EL-06 | Casual tenure | 6 months | Illustrative | REFER | L3 publishes both 3 and 6 months |
| EL-07 | Self-employed | Manual route. The sampled practice is 24 months | Market | REFER | L2, L3; A-25 |
| EL-08 | Do not use sex, sexual orientation, gender identity, intersex status, marital or relationship status, pregnancy or potential pregnancy, or breastfeeding to refuse credit, set terms or vary service | Excluded from rules and prompts | Statutory | BLOCK at configuration review | SDA s 22 |
| EL-09 | Do not use disability to refuse credit, set terms or vary service, subject to the Act's exceptions | Excluded from rules and prompts, including inference from transactions | Statutory | BLOCK at configuration review | DDA s 24 |

State and territory discrimination law was not reviewed.

---

## 5. Income and verification

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| IN-01 | PAYG income evidence can include payslips, employer confirmation, tax returns and bank credits | Payslips plus bank credits | Guidance | REFER if missing | RG 209 Table 1 |
| IN-02 | Number and age of payslips | 2, the newest no more than 45 days old | Illustrative | REFER | None |
| IN-03 | Bank statement history | 90 days | Illustrative | REFER | NCCP s 130(1A) applies to SACCs only |
| IN-04 | Casual, seasonal or variable income needs evidence over a wider period | Period length is illustrative: 6 months | Guidance | REFER | RG 209 Table 1 |
| IN-05 | Overtime, bonus and commission counted | 80% of the verified average | Illustrative | REFER | None |
| IN-06 | Government benefit income needs evidence of amount and frequency | Statement plus bank credits | Guidance | REFER | RG 209 Table 1 |
| IN-07 | Payslip net pay differs from the bank credit | More than 2% | Illustrative | REFER | None |

---

## 6. Expenses, liabilities and affordability

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| AF-01 | A benchmark does not replace inquiries or verification | Required | Guidance | CONTROL | RG 209.136 |
| AF-02 | Expense method: declared, HEM, or the higher of the two (A-22) | UNSET | Lender choice | BLOCK activation | RG 209.141 does not require a benchmark floor |
| AF-03 | Use the current, income-adjusted benchmark and consider a buffer where it reflects conservative spending | Table UNSET. HEM is a subscription product | Guidance | BLOCK if HEM is selected without a table | RG 209.140, 209.142 |
| AF-04 | Compare like with like and collect items outside HEM separately | Required | Guidance | REFER | RG 209.140(d), 209.144-145 |
| AF-05 | Monitor the share of borrowers at or below the benchmark | Alert above 50% | Guidance | CONTROL | RG 209.140(e) |
| AF-06 | Monthly commitment for credit cards | 3.5% of the limit | Illustrative | Calculation | None |
| AF-07 | Income buffer | 5% of net monthly income | Illustrative | Calculation | None |
| AF-08 | Minimum monthly surplus after all items | $200 | Illustrative | Below zero: REC-DECLINE. Zero to $200: REFER | None |
| AF-09 | Maximum debt-to-income | UNSET | Lender choice | BLOCK activation unless explicitly disabled | None |

Calculation:

```text
surplus = accepted net monthly income
        - living expenses (AF-02) and items outside the benchmark
        - existing commitments (including AF-06)
        - proposed repayment, including fees
        - buffer (AF-07)
```

The repayment needs a rate and fees, so this calculation and PB-06 are blocked until DM-01 (pricing) is answered.

---

## 7. Credit history

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| CR-01 | Notify the individual of the credit reporting body and other required matters at or before collecting information likely to be disclosed to it | Required before enquiry | Statutory | BLOCK | PA s 21C |
| CR-02 | Make a written note of each use or disclosure of credit eligibility information | Ledger access event | Statutory | CONTROL | PA s 21G(6) |
| CR-03 | Disclose repayment history and hardship information only under the grounds in s 21G(5), which include a processor of the application | Deny unless a permitted ground is configured | Statutory | BLOCK | PA s 21G(3)-(5) |
| CR-04 | Before disclosure to an application processor without an Australian link, take reasonable steps to prevent breaches. The lender may be liable for them | No such disclosure by default (D-31) | Statutory | BLOCK | PA s 21NA |
| CR-05 | Refusal based wholly or partly on bureau-sourced information needs written notice within a reasonable period, including the bureau's details where the information is about that individual | Generate a task | Statutory | CONTROL | PA s 21P |
| CR-06 | Bankruptcy | Current or within 7 years | Illustrative | REC-DECLINE | L2; L3 excludes any |
| CR-07 | Unpaid defaults | Any listed (L2 limits this to 5 years) | Market | REC-DECLINE | L2, L3 |
| CR-08 | Current financial hardship | Refer, never an automatic knock-out | Illustrative | REFER | L2 excludes it |
| CR-09 | Score bands | UNSET for each bureau and score version | Lender choice | BLOCK activation | Not comparable across bureaus (D-28) |

The Credit Reporting Code was not reviewed.

---

## 8. Identity and financial crime

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| FC-01 | Establish identity, including anyone acting for the customer and their authority, before providing the service (s 29 allows limited exceptions) | Required before handoff for funding | Statutory | BLOCK | AML s 28 |
| FC-02 | Establish whether the customer is a politically exposed person or designated for targeted financial sanctions | Required. Not yet in product scope (PQ-01) | Statutory | BLOCK | AML s 28(2)(e) |
| FC-03 | Establish the nature and purpose of the relationship and the customer's ML/TF risk | Lender's AML program | Statutory | BLOCK | AML s 28(2)(f), 28(3) |
| FC-04 | Transitional use of the former identification procedures to 30 March 2029 | Off unless the lender confirms it qualifies | Statutory | CONTROL | AUSTRAC (search summary only, rules not read) |

---

## 9. Fraud screening and checker triggers

Every result is a signal, never a verdict (D-40).

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| FS-01 | Payslip created before its pay date | Allow up to 7 days. Beyond that, signal | Illustrative | REFER | D-40; payroll often runs early |
| FS-02 | Payslip arithmetic or year-to-date inconsistency | Above $1 rounding | Illustrative | REFER | D-40 |
| FS-03 | Font, layout or edit anomaly | Any medium or high signal | Illustrative | REFER | D-40 |
| FS-04 | Data mismatch across documents | Any material identity, employer or income mismatch | Illustrative | REFER | D-40 |
| FS-05 | Screening could not run | NOT_ASSESSABLE | Illustrative | REFER | D-40 |
| MC-01 | Checker for amount | Above $30,000 | Illustrative | Checker | D-42 |
| MC-02 | Checker for fraud signal | Any high signal | Illustrative | Checker | D-42 |
| MC-03 | Checker for deviation from the recommendation | Always | Illustrative | Checker | D-42 |
| MC-04 | Maker is never the checker | Cannot be disabled | Product control | BLOCK | D-42 |

---

## 10. Privacy and transparency

| ID | Rule | Default | Label | Outcome | Source |
|---|---|---|---|---|---|
| PT-01 | From 10 December 2026, the privacy policy lists the kinds of personal information and decisions where computer programs make, or substantially and directly support, significant decisions | Supply the lender with an inventory covering agents, tools and rules | Statutory | CONTROL | PLA Sch 1 Pt 15 |
| PT-02 | Assume human review does not remove PT-01 | In scope | Guidance | CONTROL | Allens commentary, guidance not read |
| PT-03 | A TFN recipient must not use, disclose or record an incidentally provided TFN inconsistently with the law. The individual may remove it | Do not extract. Mask before model calls | Statutory | BLOCK | TFN Rule s 9 |
| PT-04 | Have internal dispute resolution and AFCA membership | Lender process | Statutory | CONTROL | NCCP s 47(1)(h)-(i) |

The TFN Rule is due to sunset on 1 April 2027, and the OAIC is consulting on remaking it (search summary only).

---

## 11. Sources

### Primary text read (relevant sections only)

| Code | Source | Version |
|---|---|---|
| NCCP | [National Consumer Credit Protection Act 2009, volume 1](https://www.legislation.gov.au/C2009A00134/2026-07-01/2026-07-01/text/original/epub/OEBPS/document_1/document_1.html) | 1 July 2026 |
| NCC | [National Credit Code, volume 2 of that Act](https://www.legislation.gov.au/C2009A00134/2026-07-01/2026-07-01/text/original/epub/OEBPS/document_2/document_2.html) | 1 July 2026 |
| PA | [Privacy Act 1988](https://www.legislation.gov.au/C2004A03712/2026-06-04/2026-06-04/text/original/epub/OEBPS/document_1/document_1.html) | 4 June 2026 |
| PLA | [Privacy and Other Legislation Amendment Act 2024](https://www.legislation.gov.au/C2024A00128/asmade/2024-12-10/text/original/epub/OEBPS/document_1/document_1.html) | As made |
| AML | [AML/CTF Act 2006](https://www.legislation.gov.au/C2006A00169/2026-10-01/2026-10-01/text/original/epub/OEBPS/document_1/document_1.html) | 1 October 2026 |
| ADA | [Age Discrimination Act 2004](https://www.legislation.gov.au/C2004A01302/2026-07-01/2026-07-01/text/original/epub/OEBPS/document_1/document_1.html) | 1 July 2026 |
| SDA | [Sex Discrimination Act 1984](https://www.legislation.gov.au/C2004A02868/2025-11-01/2025-11-01/text/original/epub/OEBPS/document_1/document_1.html) | 1 November 2025 |
| DDA | [Disability Discrimination Act 1992](https://www.legislation.gov.au/C2004A04426/2026-10-08/2026-10-08/text/original/epub/OEBPS/document_1/document_1.html) | 8 October 2026 |
| TFN Rule | [Privacy (Tax File Number) Rule 2015](https://www.legislation.gov.au/F2015L00249/2015-03-04/2015-03-04/text/original/epub/OEBPS/document_1/document_1.html) | 4 March 2015 |
| RG 209 | [ASIC RG 209](https://download.asic.gov.au/media/hyeofbni/rg209-published-9-december-2019-20250306.pdf) | December 2019 |

### Market sample (public pages, not credit policies)

| Code | Source | Note |
|---|---|---|
| L1 | [Plenti](https://www.plenti.com.au/personal-loans) | $5,000 to $75,000, 1 to 7 years on this page |
| L2 | [Wisr](https://wisr.com.au/loans/personal-loans) | 3, 5 or 7 years |
| L3 | [NOW Finance](https://nowfinance.com.au/eligibility) | 18 months to 7 years |

Three lenders are not "the market". These are public minimums, and real credit policies are stricter and confidential.

### Secondary or summary only

| Code | Source | Limitation |
|---|---|---|
| Allens | [Commentary on the OAIC guidance](https://www.allens.com.au/insights-news/insights/2026/10/mandatory-automated-decision-making-disclosure-requirements-are-coming-key-takeaways-from-the-oaic-guidance/) | The OAIC PDF refused automated download (HTTP 403), so it was not read |
| AUSTRAC | [Transitional rules](https://www.austrac.gov.au/reforms/amlctf-transitional-rules-update) | Search summary only. An AUSTRAC page fetch timed out |
| OAIC | [TFN Rule consultation](https://www.oaic.gov.au/engage-with-us/consultations/consultation-on-remaking-the-privacy-tax-file-number-rule-2015) | Search summary only |

### Not reviewed

- National Consumer Credit Protection Regulations, including prescribed inquiries, SACC limits and the cost-rate inputs
- AML/CTF Rules
- Privacy (Credit Reporting) Code
- RG 271 (dispute resolution) and RG 274 (design and distribution obligations, including target market determinations)
- State and territory law
- Exceptions and case law, except as quoted in RG 209
- Amendments not yet commenced, except PLA Sch 1 Pt 15

---

## 12. Questions raised by this research

| ID | Question | Recommendation | Why it matters |
|---|---|---|---|
| PQ-01 | Who performs PEP and sanctions screening? | Product provides a vendor-neutral adapter and blocks handoff without a result or lender attestation | Required by AML s 28 and absent from the brief |
| PQ-02 | Who enforces the 90-day window, given that contracting is downstream? | Product sends `validUntil` in the handoff. The downstream system enforces it on the credit day | A late contract could breach s 128. Section 133 also tests suitability at the time of entry, so a change in circumstances matters within the window |
| PQ-03 | How does the lender meet the 7-year assessment-copy duty when retention is theirs (D-39)? | Keep D-39. Document the duty, make the assessment exportable and warn on purge | A purge could leave the lender unable to comply |
| PQ-04 | May bureau-derived data go to a model provider? | Minimise it, use AU processing only, and have counsel confirm the permitted ground for each provider | PA ss 21G, 21NA |
| PQ-05 | How is health or other protected information in bank statements kept from influencing recommendations? | Mask or generalise those categories before the summary agent, and test for it | Expense analysis can reveal disability or pregnancy |
