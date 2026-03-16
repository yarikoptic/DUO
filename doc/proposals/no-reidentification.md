# Proposal: "No Re-identification" and "No Relinking" Data Use Modifiers for DUO

**Date:** 2026-03-16
**Author:** Yaroslav Halchenko
**GitHub issue:** https://github.com/EBISPOT/DUO/issues/131
**Related discussion:** https://github.com/EBISPOT/DUO/issues/129
**Target:** HL7 Security Working Group (via John Moehrke, per [comment](https://github.com/EBISPOT/DUO/issues/131#issuecomment-4069578751)) and DUO ontology

## Proposed Terms

### Term 1: No Re-identification (NRI)

**Code:**
:   `NRI`

**Label:**
:   no re-identification

**Definition:**
:   This data use modifier indicates that the user must not attempt to
    re-identify, establish, or retrieve the identity of data subjects
    from the shared data.

**Description:**
:   Prohibits any action aimed at determining the identity of
    individuals whose data are included.  This includes, but is not
    limited to: requesting or using pseudonymization keys, employing
    computational methods (e.g., facial reconstruction from
    neuroimaging, genetic re-identification, or inference from rare
    phenotypic combinations) to establish the identity of data
    subjects, or recognizing individuals through personal knowledge.
    This modifier is intended to be used in conjunction with a data
    use permission (e.g., GRU, HMB, DS) to constrain the scope of
    permitted use by excluding re-identification activities.

### Term 2: No Relinking (NRL)

**Code:**
:   `NRL`

**Label:**
:   no relinking

**Definition:**
:   This data use modifier indicates that the user must not link or
    associate the shared data with other databases or datasets in a
    manner that could result in disclosing information intended to be
    masked.

**Description:**
:   Prohibits associating de-identified, pseudonymized, or coded data
    with other information sources — whether or not such linking would
    result in identifying specific individuals.  This includes, but is
    not limited to: linking the data to external databases containing
    identifying information, performing record linkage attacks across
    datasets, or combining the data with other resources to enrich
    individual-level records beyond what was originally shared.  The
    prohibition applies even when the linkage target is itself
    de-identified (e.g., linking two anonymized datasets by similarity
    of brain scans to track individuals across studies).  This modifier
    is semantically aligned with HL7 `NORELINK`.

### Relationship between NRI and NRL

NRI and NRL are **complementary but distinct** modifiers addressing
different threat models:

- **NRI** targets the *outcome*: learning who a data subject is.
  A researcher who recognizes a participant from a distinctive brain
  lesion — without consulting any external database — violates NRI
  (if they act on that recognition) but not NRL.

- **NRL** targets the *act*: linking records across datasets.
  A researcher who links two anonymized datasets by scan similarity to
  track individuals across studies — without ever learning anyone's
  name — violates NRL but not necessarily NRI.

In practice, most human-subjects data sharing scenarios call for both:

- `GRU + NRI + NRL` — the typical combination for open neuroimaging data
- `NRI` alone — when cross-dataset linkage is acceptable (e.g., approved
  multi-site studies) but identity recovery is not
- `NRL` alone — when the concern is preventing unauthorized data
  enrichment rather than identity disclosure per se

## Justification

### The gap

The Data Use Ontology (DUO) provides machine-readable codes for tagging
datasets with permitted uses and restrictions.  It covers a broad range
of conditions: ethics approval requirements (IRB), commercial-use
restrictions (NCU, NPU), geographic limits (GS), population-research
prohibitions (NPOA), and more.  However, **DUO currently has no code that explicitly prohibits
re-identification of data subjects or relinking of de-identified
records**.

This is a critical gap because the prohibition on re-identification is a
**near-universal condition** for sharing human subjects data.  Every
major consent framework, data use agreement, and regulatory regime
includes such a restriction:

- **Open Brain Consent** (all three editions): The Data User Agreement
  states *"I will not attempt to establish or retrieve the identity of
  the study participants.* [→ NRI] *I will not link these data to any
  other database in a way that could provide identifying information."*
  [→ NRL]
  (see https://open-brain-consent.readthedocs.io/en/latest/gdpr/data_user_agreement.html)

- **GDPR** (Art. 89): Research exemptions to data subject rights are
  conditional on "appropriate safeguards" including technical measures
  that ensure re-identification is not possible.

- **U.S. Common Rule** (45 CFR 46): De-identified data are exempt from
  IRB review only if the investigator cannot readily identify subjects;
  re-identification would void this exemption.

- **HIPAA** (45 CFR 164.514): Safe Harbor de-identification requires
  that the covered entity has "no actual knowledge that the information
  could be used alone or in combination ... to identify an individual."

- **NIH Genomic Data Sharing Policy**: Requires that approved users
  "will not attempt to identify individual human research participants
  from whom the data were obtained."

### Why existing terms are insufficient

One might argue that re-identification is "obviously prohibited" and
therefore needs no explicit code.  This argument fails for several
reasons:

1. **Machine-actionability**: DUO exists precisely to make data use
   conditions machine-readable.  If an automated access-control system
   (e.g., GA4GH Passport / Data Use Registry) sees `GRU` (General
   Research Use), it permits *any* research purpose.  Without an
   explicit modifier, "any research purpose" technically includes
   re-identification research.  An NRI modifier makes the restriction
   computable.

2. **Explicit beats implicit**: Many DUO modifiers codify conditions
   that a reasonable person might consider "obvious" (e.g., publication
   required, ethics approval required).  Explicit coding removes
   ambiguity, enables auditing, and supports interoperability across
   jurisdictions with different baseline assumptions.

3. **Not all data are human-subjects data**: DUO is used for datasets
   ranging from human genomics to environmental samples.  A blanket
   assumption that "re-identification is always prohibited" does not
   hold across the full scope of DUO-tagged datasets.  An explicit
   modifier allows tagging specifically those datasets where the
   restriction applies.

4. **Consent form fidelity**: When a consent form explicitly promises
   participants that "no one will attempt to identify you," the
   corresponding DUO annotation should faithfully capture that promise.
   Omitting it creates a gap between what participants consented to and
   what the machine-readable metadata conveys.

### How it fits with existing DUO terms

The proposed `NRI` would be a **Data Use Modifier** (subclass of
DUO:0000017), following the same pattern as existing prohibitory
modifiers:

| Existing modifier                                              | Pattern                                      |
|-----------------------------------------------------------------|----------------------------------------------|
| NPOA — population origins or ancestry research **prohibited**   | Prohibits a specific research activity        |
| NMDS — **no** general methods research                          | Prohibits a specific research activity        |
| NCU — **non**-commercial use only                               | Restricts a category of use                  |
| **NRI — no re-identification**                                  | **Prohibits a specific activity on the data** |
| **NRL — no relinking**                                          | **Prohibits a specific activity on the data** |

Typical usage would combine NRI and/or NRL with a permission:

- `GRU + NRI + NRL` — General research use, no re-identification, no
  relinking (the common case for human subjects data)
- `HMB + NRI + NRL + IRB` — Health/medical research, no
  re-identification, no relinking, ethics approval required
- `DS + NRI + PUB` — Disease-specific research, no re-identification,
  publication required (relinking permitted for approved linkage studies)

### Cross-reference to HL7 terminology

HL7 Version 3 already defines a closely related concept in its
RefrainPolicy value set:

> **NORELINK** (*no relinking*): "Prohibition on associating
> de-identified or pseudonymized information with other information in a
> manner that could or does result in disclosing information intended to
> be masked."

The proposed DUO `NRL` modifier is a **direct mirror** of HL7
`NORELINK`.  Once adopted, a `skos:exactMatch` or `oboInOwl:hasDbXref`
cross-reference should be established between the two.

The proposed DUO `NRI` modifier has no direct HL7 counterpart — HL7's
`NORELINK` focuses on the linking act, not on identity recovery by other
means.  This is precisely why both terms are needed: NRL enables
interoperability with HL7-based clinical data systems, while NRI
captures the broader prohibition on re-identification that is standard
in research consent frameworks.

## Implementation Notes

### DUO ontology entries (OWL sketch)

```xml
<!-- NRI: no re-identification -->
<owl:Class rdf:about="http://purl.obolibrary.org/obo/DUO_00000XX">
    <rdfs:subClassOf rdf:resource="http://purl.obolibrary.org/obo/DUO_0000017"/>
    <obo:IAO_0000115 xml:lang="en">This data use modifier indicates that the
        user must not attempt to re-identify, establish, or retrieve the identity
        of data subjects from the shared data.</obo:IAO_0000115>
    <oboInOwl:id rdf:datatype="http://www.w3.org/2001/XMLSchema#string">DUO:00000XX</oboInOwl:id>
    <oboInOwl:shorthand rdf:datatype="http://www.w3.org/2001/XMLSchema#string">NRI</oboInOwl:shorthand>
    <rdfs:label xml:lang="en">no re-identification</rdfs:label>
</owl:Class>

<!-- NRL: no relinking -->
<owl:Class rdf:about="http://purl.obolibrary.org/obo/DUO_00000XY">
    <rdfs:subClassOf rdf:resource="http://purl.obolibrary.org/obo/DUO_0000017"/>
    <obo:IAO_0000115 xml:lang="en">This data use modifier indicates that the
        user must not link or associate the shared data with other databases
        or datasets in a manner that could result in disclosing information
        intended to be masked.</obo:IAO_0000115>
    <oboInOwl:id rdf:datatype="http://www.w3.org/2001/XMLSchema#string">DUO:00000XY</oboInOwl:id>
    <oboInOwl:shorthand rdf:datatype="http://www.w3.org/2001/XMLSchema#string">NRL</oboInOwl:shorthand>
    <oboInOwl:hasDbXref rdf:datatype="http://www.w3.org/2001/XMLSchema#string">HL7:NORELINK</oboInOwl:hasDbXref>
    <rdfs:label xml:lang="en">no relinking</rdfs:label>
</owl:Class>
```

### Impact on Open Brain Consent DUO annotations

With NRI available, the OBC DUO annotations (per
https://open-brain-consent.readthedocs.io/en/latest/duo.html) would
become:

| OBC version          | Current DUO codes | Proposed DUO codes                              |
|----------------------|-------------------|-------------------------------------------------|
| OBC-ULT (public)     | GRU               | GRU + **NRI** + **NRL**                         |
| OBC-ULT-2T (tiered)  | GRU; GRU + US     | GRU + **NRI** + **NRL**; GRU + US + **NRI** + **NRL** |
| OBC-GDPR-ULT         | HMB               | HMB + **NRI** + **NRL**                         |

### Governance criteria satisfied

Per [DUO Governance (2021)](https://github.com/EBISPOT/DUO/blob/master/Governance2021.md):

| Principle                                                       | Assessment                                                                            |
|-----------------------------------------------------------------|---------------------------------------------------------------------------------------|
| **Purpose** — promotes responsible and effective data sharing    | Directly encodes fundamental human subjects protections                                |
| **Simplicity** — understandable to non-experts                  | "No re-identification" and "no relinking" are intuitive and widely understood          |
| **Legitimate interests** — addresses common ethical concerns     | Near-universal requirement across consent forms and regulations                        |
| **Interoperability** — no breaking changes                      | Purely additive; NRL directly mirrors HL7 NORELINK                                    |
| **Common use cases** — affects 2+ implementations               | Open Brain Consent, NIH GDS Policy, GDPR-governed repositories, dbGaP, UK Biobank, etc. |
| **Machine-readability** — reduces free text                     | Replaces free-text DUA clauses with a computable code                                 |
| **Efficient GA4GH resources** — respects limited resources      | Minimal implementation effort; two simple new classes in OWL                           |

## Open Questions

1. **Scope of "re-identification"**: Should the NRI definition
   explicitly cover *attempted* re-identification (intent-based) or only
   *successful* re-identification (outcome-based)?  The current proposal
   uses "must not attempt," covering intent, which aligns with OBC DUA
   language and is more protective.

2. **One term or two?**: An alternative approach would be a single
   broader term (e.g., "no re-identification or relinking") rather than
   two separate modifiers.  The two-term approach is preferred because:
   (a) the concepts are genuinely distinct (see examples above),
   (b) some use cases need one without the other, and
   (c) NRL maps cleanly to the existing HL7 NORELINK code while NRI
   addresses a gap that HL7 itself does not cover.

3. **Relationship to "no redistribution"**: OBC's DUA also prohibits
   redistribution.  This could be a separate DUO modifier proposal
   (e.g., `NRD` — no redistribution).  Kept out of scope here to
   maintain focus.

4. **Code assignment**: The actual DUO IDs (e.g., DUO:0000047,
   DUO:0000048) would be assigned by DUO editors per their ID range
   allocation process.
