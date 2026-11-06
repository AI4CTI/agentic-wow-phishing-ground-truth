# Agentic WOW – phishing / benign URL ground truth

A labelled set of **848 URLs — 418 phishing and 430 benign** — used to evaluate
**Agentic WOW**, the agentic phishing-site analyser developed by Ermes Cyber
Security within the AI4CTI project. Agentic WOW visits a URL in an isolated
browser, collects evidence (page content, screenshot, TLS and domain data) and
asks an LLM agent for a verdict: `phishing`, `suspicious`, `benign` or
`inconclusive`.

The analyser itself is proprietary and is not part of this release. The
dataset contains **URLs and a few probe attributes only**: no page source, no
screenshots, no WHOIS or DNS records.

> **Warning.** The phishing URLs point to live phishing infrastructure. Do not
> open them outside an isolated environment.

## Contents

| File | Description |
|---|---|
| `agentic_wow_ground_truth_v1.csv` | The dataset (UTF-8, comma-separated, header row) |
| `LICENSE` | GNU Affero General Public License v3.0 |
| `NOTICE` | Copyright and attributions |

## Fields

| Field | Description |
|---|---|
| `url` | The URL analysed |
| `gt_label` | Ground-truth label: `phishing` or `benign` |
| `gt_source` | How the row was obtained: `logs_triple_judge` or `popular_site` (see below) |
| `brand` | Brand impersonated by the phishing page; empty for benign rows |
| `agentic_verdict` | Agentic WOW's verdict when the URL was labelled |
| `agentic_reasons` | Reasons Agentic WOW gave for its verdict, separated by `;` |
| `final_url` | URL reached after redirects |
| `http_status` | HTTP status of the final URL |
| `tls_issuer` | Issuer of the TLS certificate, when available |
| `tls_age_days` | Age of the TLS certificate in days, when available |
| `whois_age_days` | Age of the domain registration in days, when available |
| `date` | Date (UTC) on which Agentic WOW produced the verdict |
| `url_trimmed` | `true` if `url` or `final_url` was shortened for privacy (see below) |

## How it was built

**Phishing (418 rows, `gt_source=logs_triple_judge`).**
- **Source:** the URLs analysed by the deployed Agentic WOW service between
  2026-10-07 and 2026-11-06.
- **Deduplication:** one URL per site, keyed on the registrable domain; on
  shared hosting platforms (vercel.app, pages.dev, github.io, netlify.app,
  web.app, …) the first label identifies the site.
- **Dead pages removed:** only pages that answered HTTP 200 and were not
  reported as not found, suspended, parked or blocked were kept.
- **Labelling:** three LLM judges (Anthropic Claude models) labelled each site
  independently from the URL, the final URL, the TLS issuer and age, and the
  domain age. They never saw Agentic WOW's verdict or reasons and did not visit
  the pages. A site was kept as phishing only if all three judges labelled it
  phishing with high confidence.

**Benign (430 rows, `gt_source=popular_site`).**
- **Which sites:** the official domains of the brands impersonated in the
  phishing class (some through their real login pages); Italian banks,
  telecom operators, utilities and public services; user-facing sites from the
  Tranco top 3,000.
- **Kept:** the sites that answered HTTP 200 when run through Agentic WOW on
  2026-11-06. They are benign by construction: Agentic WOW's verdict played no
  part in choosing them.

### Labelling rules

- Brand clones count as phishing, including "coding practice" clones of login
  pages published on GitHub Pages or Vercel (111 rows are on github.io).
- Security-awareness simulations were excluded.
- Gambling or casino spam that impersonates no brand was considered unsure and
  excluded.

## Privacy

Before release, every row was screened for data identifying the people or
organisations targeted by a campaign or the users of the service:
- query strings, fragments and `;` path parameters were removed from all URLs;
- path segments carrying recipient e-mail addresses (plain or base64-encoded),
  mailbox item identifiers, per-victim tokens or account and page identifiers
  were replaced by `[redacted]`.

Rows affected are marked `url_trimmed=true` (51 rows). Their URLs no longer
resolve to the original page; the host is unchanged.

## Baseline

Agentic WOW's verdicts at labelling time:

| Rows | phishing | suspicious | benign | inconclusive |
|---|---:|---:|---:|---:|
| phishing (418) | 260 | 63 | 95 | 0 |
| benign (430) | 0 | 0 | 430 | 0 |

- **Detection:** 62% of the phishing URLs were called phishing, 77% phishing or
  suspicious.
- **Misses:** most of the 95 phishing URLs called benign are brand clones on
  free hosting, which the analyser judges to be student projects.
- **Benign:** the popular sites are easy cases; this release does not measure
  false positives on less known benign sites.

## Caveats

- Phishing pages go offline within days; many URLs will no longer answer.
- The labels are the agreement of three LLM judges, not human annotations.

## Acknowledgement

This work was carried out by Ermes Cyber Security S.p.A. within **AI4CTI**
(FISA-2023-00168), a joint research project with Politecnico di Torino funded
by the Italian Ministry of University and Research (MUR) under the FISA 2023
programme.
