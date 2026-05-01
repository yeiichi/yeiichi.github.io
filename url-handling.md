---
layout: default
title: "URL handling hints"
permalink: /url-handling-hints
---

# URL handling hints
_Last Updated: 2026-05-01_

## 1. Introduction
This memorandum outlines the technical strategy for a multi-layered website credibility and fact-checking tool. The approach combines structural validation, reputational analysis, and content verification.

---

## 2. Technical Implementation Roadmap

### A. URL Extraction & Filtering
To ensure high-integrity data collection, the extraction phase prioritizes secure protocols and specific string parsing.

* **Primary Method (Custom Regex):**  `HTTPS_URL_RE = re.compile(r"https://[^\s,\"'<>\)]+[^\s,\"'<>\)\.]")`
    * *Rationale:* Empirically identified as 90% effective for this use case; avoids common sentence-ending punctuation.
* **Secondary Method (External Package):** [urlextract](https://pypi.org/project/urlextract/)
    * *Use Case:* Deploy as a fallback for extracting URLs from dense or messy text where valid TLD matching is required.
* **Security Policy:** **TLS-Only.** The system must ignore `http` schemes to mitigate security risks and filter out low-trust legacy sources.

### B. Website Reputation Check (The "Hygiene" Layer)
Before evaluating content, the tool assesses the source's structural and historical trustworthiness.

* **Structural Parsing:** Use [tldextract](https://pypi.org/project/tldextract/) to accurately separate subdomains and domains based on the [Public Suffix List (PSL)](https://publicsuffix.org/).
* **Logic Criteria:**
    1.  **TLD Analysis:** Prioritize restricted suffixes (e.g., `.gov`, `.edu`) and flag high-risk or "disposable" TLDs.
    2.  **Imposter Detection:** Flag domains mimicking reputable brands (e.g., `reuters-news.co` vs `reuters.com`).
    3.  **Third-Party Reputation:** Integrate External APIs (e.g., **VirusTotal** or **Google Safe Browsing**) to check for malicious history.
    4.  **Domain Age:** Query WHOIS data; newly registered domains are marked as higher risk.

### C. Website Content Check (The "Fact" Layer)
The final layer evaluates the specific claims made within the article content.

* **Primary Engine:** [Google Fact Check Tools API](https://developers.google.com/fact-check/tools/api).
* **Workflow:**
    1.  **Claim Extraction:** Summarize the article under the URL into 1–3 core "Main Claims."
    2.  **API Query:** Send claims to the `claims:search` endpoint.
    3.  **Evaluation Criteria:** Obtain the `textualRating` (e.g., "False", "Mostly True") and the reviewing `publisher`.
* **Cost Management:** Utilization is subject to **Quotas** rather than per-request billing. Monitor usage in the Google Cloud Console.

---

## 3. Maintenance Notes
- **PSL Updates:** `tldextract` caches the Public Suffix List; refresh the cache monthly to stay current with new TLD releases.
- **API Quotas:** If daily search limits are reached, a quota increase can be requested through Google Cloud.