---
title: 'Connecting FAIR data with TogoID'
title_short: 'BioHackEU22 #07: TogoID'
tags:
  - Bioinformatics
  - TogoID
  - API
authors:
  - name: Toshiaki Katayama
    orcid: 0000-0003-2391-0384
    affiliation: 1
  - name: Last Author
    orcid: 0000-0000-0000-0000
    affiliation: 2
affiliations:
  - name: Database Center for Life Science
    index: 1
  - name: Second Affiliation
    index: 2
date: 10 November 2022
cito-bibliography: paper.bib
event: BH22EU
biohackathon_name: "BioHackathon Europe 2022"
biohackathon_url:   "https://biohackathon-europe.org/"
biohackathon_location: "Paris, France, 2022"
group: Project 07
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/ktym/elixir-biohackathon-2022-togoid
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: First Author \emph{et al.}
---


# Introduction

TogoID[ref] is an open-source identifier conversion service. As part of the BioHackathon Europe 2022, we here report new enhancements and discussions raised.

# Results

## Rhea

We have incorporated identifier (ID) mappings from Rhea[ref] to UniProt[ref], Gene Ontology[ref], and PubMed[ref] databases. Note that mappings from Rhea to Enzyme code (EC), ChEBI[ref], and Reactome reaction[ref] were already supported in the TogoID. In the download section on the Rhea website, it provides tab-separated values (TSV) files in addition to RDF data.

As for the Rhea to UniProt mapping, it is alternatively included in the UniProt RDF but we decided to use TSV files (`rhea2uniprot_sprot.tsv` and `rhea2uniprot_trembl.tsv.gz`) for TogoID because the mapping data is curated by the Rhea project (it is strange that one file is gzip-compressed but the other is not, though). There are four categories in the Rhea reactions including `undefined`, `left-to-right`, `right-to-left`, and `bidirectional`. Most of the reactions are assigned to the `undefined` category, while there are some cases that curators confirmed the direction (`left-to-right` or `right-to-left`) of the enzymatic reaction of the given UniProt protein and there exists no UniProt protein that is annotated as `bidirectional` so far. For example, the reaction `RHEA:10101` is `left-to-right` that corresponds to four UniProt proteins (`P19835`, `P30122`, `P07882`, and `Q64285`) and the `right-to-left` reaction `RHEA:10102` and the `bidirectional` reaction `RHEA:10103` has not UniProt proteins assigned, while the `undefined` reaction `RHEA:10100` corresponds to 12 UniProt proteins that also includes four proteins assigned to `RHEA:10101` as these reactions are describing the same chemical equation. We decided to include all Rhea IDs regardless of their categories so that user can obtain all available ID mappings while it can also cause ID expansion as a downside, e.g., when a user converts one UniProt protein `P19835` and will obtain two reactions, `RHEA:10100` and `RHEA:10101`, in this case.

Because cross references from Rhea to PubMed are only available in the Rhea RDF dumps, we created a SPARQL query to extract the ID mappings. Rhea to GO mappings are available both in the TSV file and RDF dumps but we used the TSV file for the time being.

## BridgeDB


## EGA


## Bgee

Bgee[ref] is a gene expression database and we found that it uses Ensembl gene IDs which already suppored in the TogoID. Instead, we 

# Discussion

Convert natural language labels to IDs
* Multilingual labels
* Spelling variants
Map IDs and hierarchical ontologies
* retrieve all descendant IDs
* classification enrichment in each category



## Tables and figures

Tables can be added in the following way, though alternatives are possible:

| Header 1 | Header 2 |
| -------- | -------- |
| item 1 | item 2 |
| item 3 | item 4 |

Table: Note that table caption is automatically numbered.

A figure is added with:

![Caption for BioHackrXiv logo figure](./biohackrxiv.png)

# Other main section on your manuscript level 1

Lists can be added with:

1. Item 1
2. Item 2

# Citation Typing Ontology annotation

You can use CiTO annotations, as explained in [this BioHackathon Europe 2021 write up](https://raw.githubusercontent.com/biohackrxiv/bhxiv-metadata/main/doc/elixir_biohackathon2021/paper.md) and [this CiTO Pilot](https://www.biomedcentral.com/collections/cito).
Using this template, you can cite an article and indicate why you cite that article, for instance DisGeNET-RDF [@citesAsAuthority:Queralt2016].

Possible CiTO typing annotation include:

* citesAsDataSource: when you point the reader to a source of data which may explain a claim
* usesDataFrom: when you reuse somehow (and elaborate on) the data in the cited entity
* usesMethodIn
* citesAsAuthority
* discusses
* extends
* agreesWith
* disagreesWith

## Acknowledgements

Anne Morgat (SIB/Rhea)
Tarcisio Mendes (Bgee)
Egon Willighagen and ??? (BridgeDB)

## References
