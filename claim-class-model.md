---
layout: default
title: "Claim Class Model"
permalink: /claim-class-model
---

# Claim Class Model
_Last Updated: 2026-06-18_

## 1. Introduction

This page summarizes a working data-class model for Cretan FactCheck. The purpose is to describe how a public statement can be represented before it is judged. Instead of forcing every article into a simple True/False verdict, the model separates the statement, the kind of fact involved, the evidence needed, and the possible status of the analysis.

The toy example “All Cretans are liars” is useful because it is simultaneously a historical quotation, a universal claim about a group, and a self-referential paradox when spoken by a Cretan. That makes it a good stress test for a fact-checking schema.

The model is intentionally internal-first. It should be useful for authors, editors, Django models, and public documentation. Where possible, however, the internal classes should also map cleanly to external vocabularies such as Schema.org `ClaimReview`.

---

## 2. Data Schema

### A. Column Definitions

* **Class**: The conceptual data class or category used to represent part of a fact-checking article.
* **Definition**: What the class means in editorial or analytical terms.
* **Example**: How the class applies to the example claim “All Cretans are liars.”

---

### B. Design Policy

* **Separate claim from verdict:** Store the claim as an object before deciding whether it is true, false, paradoxical, unsupported, or not fact-checkable.
* **Prefer dimensions over binaries:** A claim may have historical, logical, empirical, linguistic, and evidentiary dimensions rather than a single verdict.
* **Keep classes extensible:** The schema should allow new claim types, fact types, evidence types, and verdict dimensions without rewriting existing articles.
* **Align outward, model inward:** The internal model should be richer than public metadata, while still allowing export to external schemas such as Schema.org `ClaimReview`.

---

## 3. Proposed Classes for Fact-Check Articles

| Class | Definition | Example |
|---|---|---|
| `Claim` | A statement, quotation, allegation, or assertion selected for analysis. | “All Cretans are liars.” |
| `ClaimSource` | The person, document, platform, tradition, or record from which the claim is taken. | Traditionally associated with Epimenides of Crete. |
| `ClaimType` | The structural type of the claim. | Universal statement; group-level assertion; self-referential statement. |
| `FactType` | The kind of fact involved in the analysis. | Historical attribution, textual existence, speaker identity, logical consistency. |
| `EvidenceRecord` | A source item used to support, weaken, contextualize, or challenge the claim. | Ancient textual references, secondary scholarship, or logical analysis notes. |
| `AnalysisDimension` | One axis on which the claim can be evaluated. | Historical support, empirical testability, logical consistency, precision. |
| `VerdictDimension` | A structured assessment for one analysis dimension. | “Historical attribution: supported”; “Empirical claim: unsupported”; “Logic: paradoxical.” |
| `ArticleStatus` | The editorial status of the result, not necessarily a truth verdict. | Historical quotation; logical paradox; not fact-checkable as stated. |
{: .table-sticky-scope .table-scroll }

---

## 4. Related Standards and Vocabularies

The Cretan FactCheck data model is designed to be compatible with existing web, metadata, and knowledge-representation standards whenever practical.

| Standard                         | Purpose                                                 | Official Specification                                      |
| -------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------- |
| Schema.org ClaimReview           | Structured representation of fact-check reviews         | [https://schema.org/ClaimReview](https://schema.org/ClaimReview) |
| Schema.org CreativeWork          | General content model for articles and reports          | [https://schema.org/CreativeWork](https://schema.org/CreativeWork) |
| Schema.org Person                | Representation of people making or reviewing claims     | [https://schema.org/Person](https://schema.org/Person) |
| Schema.org Organization          | Representation of publishers, institutions, and sources | [https://schema.org/Organization](https://schema.org/Organization) |
| JSON-LD 1.1                      | Machine-readable linked data serialization              | [https://www.w3.org/TR/json-ld11/](https://www.w3.org/TR/json-ld11/) |
| RDF 1.1 Concepts                 | Foundation of linked-data graphs                        | [https://www.w3.org/TR/rdf11-concepts/](https://www.w3.org/TR/rdf11-concepts/) |
| Dublin Core Metadata Element Set | Common descriptive metadata vocabulary                  | [https://www.dublincore.org/specifications/dublin-core/dces/](https://www.dublincore.org/specifications/dublin-core/dces/) |
| PROV-O (W3C Provenance Ontology) | Provenance tracking for evidence and research workflows | [https://www.w3.org/TR/prov-o/](https://www.w3.org/TR/prov-o/) |
| SKOS                             | Controlled vocabularies and taxonomy management         | [https://www.w3.org/TR/skos-reference/](https://www.w3.org/TR/skos-reference/) |

### Mapping Notes

| Cretan FactCheck Concept | Candidate Standard Mapping |
| ------------------------ | -------------------------- |
| Claim                    | Schema.org `claimReviewed` |
| FactCheck Article        | Schema.org `ClaimReview`   |
| Source                   | Schema.org `CreativeWork`  |
| Evidence Record          | PROV-O Entity              |
| Verdict Type             | SKOS Concept               |
| Reviewer                 | Schema.org `Person`        |
| Publisher                | Schema.org `Organization`  |

### Design Principle

External standards are referenced for interoperability rather than strict compliance. The internal data model remains project-controlled and may extend beyond any individual standard when required for transparency, reproducibility, or editorial needs.

---

## 5. Internal Model to External Metadata Mapping

| Internal Class | Possible External Mapping | Notes |
|---|---|---|
| `Claim` | Schema.org `claimReviewed` | Use a concise, human-readable summary of the reviewed claim. |
| `ClaimSource` | Schema.org `itemReviewed`, `author`, `citation`, or `sameAs` | The best mapping depends on whether the source is a person, document, page, post, or tradition. |
| `ClaimType` | Internal taxonomy; optional keywords | Usually not directly represented in `ClaimReview`; may be exposed as tags or article metadata. |
| `FactType` | Internal taxonomy; optional keywords | Useful for internal classification and search filters. |
| `EvidenceRecord` | `citation`, `CreativeWork`, URL, archive URL, or bibliographic metadata | May later become a separate source model in Django. |
| `AnalysisDimension` | Internal section headings or structured article blocks | Usually richer than public search metadata. |
| `VerdictDimension` | Schema.org `reviewRating` plus internal rating details | Schema.org can express a public-facing rating, but multidimensional verdicts may need internal fields. |
| `ArticleStatus` | Page metadata, tags, or internal workflow/status field | May include statuses that are not simple truth ratings, such as “paradoxical” or “not fact-checkable as stated.” |
{: .table-sticky-scope .table-scroll }

---

## 6. Suggested JSON-LD Shape

The exact JSON-LD should be generated from the final Django models, but a future article could roughly export metadata like this:

```json
{
  "@context": "https://schema.org",
  "@type": "ClaimReview",
  "url": "https://www.cretan-factcheck.org/articles/example-all-cretans-are-liars/",
  "claimReviewed": "All Cretans are liars.",
  "itemReviewed": {
    "@type": "CreativeWork",
    "name": "Traditional Epimenides paradox attribution"
  },
  "author": {
    "@type": "Organization",
    "name": "Cretan FactCheck"
  },
  "reviewRating": {
    "@type": "Rating",
    "ratingValue": "Not fact-checkable as stated",
    "alternateName": "Logical paradox / unsupported universal claim"
  },
  "datePublished": "2026-06-18",
  "dateModified": "2026-06-18"
}
```

This example is intentionally illustrative. Before production use, validate generated JSON-LD against Schema.org and Google’s structured-data requirements.

---

## 7. Maintenance Notes

- **Extensibility:** Additional classes may be added later, such as `Actor`, `Document`, `Dataset`, `Method`, `Correction`, `ConfidenceLevel`, `ReviewStatus`, `EntityIdentifier`, `ArchiveRecord`, or `ProvenanceEvent`. The important rule is that each class should capture one stable editorial concept.
- **Generation:** This table can later be generated from Django model metadata, Python dataclasses, or a JSON/YAML schema used by the project documentation. Schema.org JSON-LD should eventually be generated from the same source of truth, not hand-written per article.
- **Interpretation:** Readers should not treat the table as a final ontology. It is a practical working model for transparent articles. In this model, a verdict is not mandatory; some articles may end with statuses such as “ill-defined,” “paradoxical,” “unsupported,” or “not empirically testable.”
- **Validation:** Public metadata should be tested with structured-data validation tools before deployment. Internal editorial classes may be more expressive than what public metadata validators understand.
