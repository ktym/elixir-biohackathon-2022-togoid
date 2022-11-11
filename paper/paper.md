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

TogoID[ref] is an open-source identifier (ID) conversion service. As part of the BioHackathon Europe 2022, we here report new enhancements and discussions raised.

# Results

During the hackathon, we have contacted several participants with use cases. Each dataset stores cross references in various format and we examined feasibility of inclusion to our TogoID one by one. Minimum requirements for inclusion are that (1) the dataset is officially provided by or synchronized to the original data source, (2) entire identifires used in the resource are covered. To include new ID mapping pairs, it is required to update the [TogoID ontology](https://togoid.dbcls.jp/ontology) that defines (1) a category and a class to which the source and target datasets belong if they are not yet registered, and (2) a semantic relationship of the new pair of datasets in both directions if there are no conformable predicate in the existing properties. Finally and most importantly, develop a method to extract ID mappings from the source data and include it in the [TogoID-config](https://github.com/dbcls/togoid-config) repository. When the source or target dataset is newly integrated, the `dataset.yaml` file in the TogoID-config also needs to be updated. To keep ID mappings in TogoID up-to-date, we currently update data every 10 days.

## Rhea

We have incorporated ID mappings from Rhea[ref] to UniProt[ref], Gene Ontology[ref], and PubMed[ref] databases. Note that mappings from Rhea to Enzyme code (EC), ChEBI[ref], and Reactome reaction[ref] were already supported in the TogoID. In the download section on the Rhea website, it provides tab-separated values (TSV) files in addition to RDF data. Because cross references from Rhea to PubMed are only available in the Rhea RDF dump, we developed a SPARQL query to extract ID mappings from the endpoint hosted at the RDF Portal[ref]. Rhea to GO mappings are available both in the TSV file and RDF but we used the TSV file for the time being. For example, we created `config/rhea-go/config.yaml` as follows. This configuration indicates that the `Rhea` to `GO` relation is `TIO_000004` (has GO annotation) and the `GO` to `Rhea` relation is is `TIO_000007` (GO annotation of), original data is updated bimonthly (we uses Dublin Core's Frequency Vocabulary [DCFreq](https://www.dublincore.org/specifications/dublin-core/collection-description/frequency/) terms to specify the update frequency), and the method to extract and format the ID mappings from the source `rhea2go.tsv` file. In this case, we also added the file download method in the `Rakefile` which checks size and timestamp and run the update procedure only when the remote file is renewed.

```
link:
  forward: TIO_000004
  reverse: TIO_000007
  file: sample.tsv
update:
  frequency: Bimonthly
  method: awk -F '\t' 'FNR!=1 && !a[$2 $3]++{print $1 "\t" gensub("GO:","","g",$4)}' $TOGOID_ROOT/input/rhea/rhea2go.tsv
```

As for the Rhea to UniProt mapping, it is alternatively included in the UniProt RDF but we decided to use TSV files (`rhea2uniprot_sprot.tsv` and `rhea2uniprot_trembl.tsv.gz`) for TogoID because the mapping data is curated by the Rhea project (it is strange that one file is gzip-compressed but the other is not, though). There are four categories in the Rhea reactions including `undefined`, `left-to-right`, `right-to-left`, and `bidirectional`. Most of the reactions are assigned to the `undefined` category, while there are some cases that curators confirmed the direction (`left-to-right` or `right-to-left`) of the enzymatic reaction of the given UniProt protein and there exists no UniProt protein that is annotated as `bidirectional` so far. For example, the reaction `RHEA:10101` is `left-to-right` that corresponds to four UniProt proteins (`P19835`, `P30122`, `P07882`, and `Q64285`) and the `right-to-left` reaction `RHEA:10102` and the `bidirectional` reaction `RHEA:10103` has not UniProt proteins assigned, while the `undefined` reaction `RHEA:10100` corresponds to 12 UniProt proteins that also includes four proteins assigned to `RHEA:10101` as these reactions are describing the same chemical equation. We decided to include all Rhea IDs regardless of their categories so that user can obtain all available ID mappings while it can also cause ID expansion as a downside, e.g., when a user converts one UniProt protein `P19835` and will obtain two reactions, `RHEA:10100` and `RHEA:10101`, in this case.

## SwissLipids

New ID mappings from SwissLipids [ref] to ChEBI and HMDB [ref] have also been incorporated. SwissLipids is the first lipid database to be introduced into TogoID, and we added new database category Lipids in the database configuration 'database.yaml'. We plan further inclusion of other lipid-related DBs, which will enrich the ID relation graph in TogoID. As in the case of Rhea, the ID mapping of SwissLipids is also derived from TSV files.

```
swisslipids:
  label: SwissLipids
  catalog: nbdc02026
  category: Lipids
  regex: '^SLM:(?<id>\d+)$'
  prefix: 'http://identifiers.org/slm/SLM:'
  examples:
    - ["SLM:000000002","SLM:000000003","SLM:000000006","SLM:000000007","SLM:000000035", "SLM:000000042","SLM:000000043","SLM:000000044", "SLM:000000047","SLM:000000048"]
```

## BridgeDB

[BridgeDB](https://bridgedb.github.io/) is another ID mapping database among genes, proteins, metabolites, metabolic reactions, diseases, complexes, and publications. It provides binary database which can be explored with Java APIs, REST APIs, and the PathVisio pathway editor. The gene and protein mapping data of BridgeDB is splet into species (currently 35 organisms are supported) and other ID mappings for complexes, interactions, metabolites, and publications are provided in separate files. It is offered that TSV dumps of their data to be included in the TogoID so that users can benefit from the graphical user interface to explore. We will continue collaboration in the future.

## EGA

The European Genome-phenome Archive (EGA)[ref] is a large archive of personal genomic and phenotypic data. It is suggested to host ID mappings among studies, datasets, samples etc. in TogoID. It would be beneficial for biomedical researchers if the EGA data can be reached from external IDs such as PubMed.

- bulk download of ID mappings?

## Bgee

Bgee[ref] is a gene expression database to retrive and compare gene expression patterns across multiple animal species. We found that it uses Ensembl gene IDs which already suppored in the TogoID. Instead, we had a discussion to utilize the Bgee data in [TogoDX](https://togodx.dbcls.jp/human/), our integrated database explorer, to search related human genes in each organ. As Bgee uses UBERON[ref] for the anatomical classification and cell lines, a list of genes can be classified hierarchically. We implemented Bgee in the development version of TogoDX by obtaining data from the Bgee SPARQL endpoint. As the expression data in Bgee can be filtered by the developmental stage, age, and sex, it brought us new idea to extend the TogoDX for embedding a complex query interface as a plugin.

# Discussion

We also investigated drug and side effect relations in the [T-ardis](http://www.bioinsilico.org/T-ARDIS/) database. 

and ATC classification
ATC - PubChem
Some DBs uses names (labels) of compounds as ID...


Convert natural language labels to IDs
* Multilingual labels
* Spelling variants
Map IDs and hierarchical ontologies
* retrieve all descendant IDs
* classification enrichment in each category

SSSOM (Mapping Commons) https://github.com/mapping-commons/sssom
Bioregistry (https://www.biorxiv.org/content/10.1101/2022.07.08.499378v3.full)

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

We appreciate greatful to Dr. Anne Morgat on Rhea, Dr. Tarcisio Mendes on Bgee, Egon Willighagen and ??? on BridgeDB, Marcos Casado Barbero on EGA for their valuable inputs.


## References
