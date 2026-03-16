# Proposal: "No Re-identification" Data Use Modifier for DUO

**Date:** 2026-03-16
**Author:** Yaroslav Halchenko
**GitHub issue:** https://github.com/EBISPOT/DUO/issues/131
**Related discussion:** https://github.com/EBISPOT/DUO/issues/129
**Target:** HL7 Security Working Group (via John Moehrke, per [comment](https://github.com/EBISPOT/DUO/issues/131#issuecomment-4069578751)) and DUO ontology

## Proposed Term

**Code:**
:   `NRI`

**Label:**
:   no re-identification

**Definition:**
:   This data use modifier indicates that the user must not attempt to
    re-identify, establish, or retrieve the identity of data subjects
    from the shared data.

**Description:**
:   Prohibits any action aimed at associating de-identified,
    pseudonymized, or coded data with other information in a manner
    that could or does result in identifying the individuals whose data
    are included.  This includes, but is not limited to: attempting to
    link the data to external databases containing identifying
    information, requesting or using pseudonymization keys, or
    employing computational methods (e.g., facial reconstruction from
    neuroimaging, genetic re-identification, or record linkage attacks)
    to re-establish the identity of data subjects.  This modifier is
    intended to be used in conjunction with a data use permission
    (e.g., GRU, HMB, DS) to constrain the scope of permitted use by
    excluding re-identification activities.

## Justification

### The gap

The Data Use Ontology (DUO) provides machine-readable codes for tagging
datasets with permitted uses and restrictions.  It covers a broad range
of conditions: ethics approval requirements (IRB), commercial-use
restrictions (NCU, NPU), geographic limits (GS), population-research
prohibitions (NPOA), and more.  However, **DUO currently has no code
that explicitly prohibits re-identification of data subjects**.

This is a critical gap because the prohibition on re-identification is a
**near-universal condition** for sharing human subjects data.  Every
major consent framework, data use agreement, and regulatory regime
includes such a restriction:

- **Open Brain Consent** (all three editions): The Data User Agreement
  states *"I will not attempt to establish or retrieve the identity of
  the study participants. I will not link these data to any other
  database in a way that could provide identifying information."*
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

| Existing modifier                                              | Pattern                                    |
|-----------------------------------------------------------------|--------------------------------------------|
| NPOA — population origins or ancestry research **prohibited**   | Prohibits a specific research activity      |
| NMDS — **no** general methods research                          | Prohibits a specific research activity      |
| NCU — **non**-commercial use only                               | Restricts a category of use                |
| **NRI — no re-identification**                                  | **Prohibits a specific activity on the data** |

Typical usage would combine NRI with a permission:

- `GRU + NRI` — General research use, but no re-identification
- `HMB + NRI + IRB` — Health/medical research, no re-identification,
  ethics approval required
- `DS + NRI + PUB` — Disease-specific research, no re-identification,
  publication required

### Cross-reference to HL7 terminology

HL7 Version 3 already defines a closely related concept in its
RefrainPolicy value set:

> **NORELINK** (*no relinking*): "Prohibition on associating
> de-identified or pseudonymized information with other information in a
> manner that could or does result in disclosing information intended to
> be masked."

The proposed DUO `NRI` modifier is semantically aligned with HL7
`NORELINK`.  Once adopted, a `skos:exactMatch` or `oboInOwl:hasDbXref`
cross-reference should be established between the two, enabling
interoperability between DUO-based research data systems and HL7-based
clinical data systems.

## Implementation Notes

### DUO ontology entry (OWL sketch)

```xml
<owl:Class rdf:about="http://purl.obolibrary.org/obo/DUO_00000XX">
    <rdfs:subClassOf rdf:resource="http://purl.obolibrary.org/obo/DUO_0000017"/>
    <obo:IAO_0000115 xml:lang="en">This data use modifier indicates that the
        user must not attempt to re-identify, establish, or retrieve the identity
        of data subjects from the shared data.</obo:IAO_0000115>
    <oboInOwl:id rdf:datatype="http://www.w3.org/2001/XMLSchema#string">DUO:00000XX</oboInOwl:id>
    <oboInOwl:shorthand rdf:datatype="http://www.w3.org/2001/XMLSchema#string">NRI</oboInOwl:shorthand>
    <oboInOwl:hasDbXref rdf:datatype="http://www.w3.org/2001/XMLSchema#string">HL7:NORELINK</oboInOwl:hasDbXref>
    <rdfs:label xml:lang="en">no re-identification</rdfs:label>
</owl:Class>
```

### Impact on Open Brain Consent DUO annotations

With NRI available, the OBC DUO annotations (per
https://open-brain-consent.readthedocs.io/en/latest/duo.html) would
become:

| OBC version          | Current DUO codes | Proposed DUO codes                  |
|----------------------|-------------------|-------------------------------------|
| OBC-ULT (public)     | GRU               | GRU + **NRI**                       |
| OBC-ULT-2T (tiered)  | GRU; GRU + US     | GRU + **NRI**; GRU + US + **NRI**   |
| OBC-GDPR-ULT         | HMB               | HMB + **NRI**                       |

### Governance criteria satisfied

Per [DUO Governance (2021)](https://github.com/EBISPOT/DUO/blob/master/Governance2021.md):

| Principle                                                       | Assessment                                                                            |
|-----------------------------------------------------------------|---------------------------------------------------------------------------------------|
| **Purpose** — promotes responsible and effective data sharing    | Directly encodes the most fundamental human subjects protection                       |
| **Simplicity** — understandable to non-experts                  | "No re-identification" is intuitive and widely understood                             |
| **Legitimate interests** — addresses common ethical concerns     | Near-universal requirement across consent forms and regulations                        |
| **Interoperability** — no breaking changes                      | Purely additive; new modifier, no changes to existing terms                           |
| **Common use cases** — affects 2+ implementations               | Open Brain Consent, NIH GDS Policy, GDPR-governed repositories, dbGaP, UK Biobank, etc. |
| **Machine-readability** — reduces free text                     | Replaces free-text DUA clauses with a computable code                                 |
| **Efficient GA4GH resources** — respects limited resources      | Minimal implementation effort; single new class in OWL                                |

## Open Questions

1. **Scope of "re-identification"**: Should the definition explicitly
   cover *attempted* re-identification (intent-based) or only
   *successful* re-identification (outcome-based)?  The current proposal
   uses "must not attempt," covering intent, which aligns with OBC DUA
   language and is more protective.

2. **Relationship to "no redistribution"**: OBC's DUA also prohibits
   redistribution.  This could be a separate DUO modifier proposal
   (e.g., `NRD` — no redistribution).  Kept out of scope here to
   maintain focus.

3. **Code assignment**: The actual DUO ID (e.g., DUO:0000047) would be
   assigned by DUO editors per their ID range allocation process.
