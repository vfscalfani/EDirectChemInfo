# PubChem <--> PubChem BioAssay <--> PubMed EDirect Recipes

**Notes**

> 1. `user@computer:~$` represents an example terminal prompt name. Actual command/argument input is after the `$`.
> 2. Replace `name@xx.edu` with your email address.
> 3. `\` followed by `>` on the next line represents continued terminal input. You will need to delete the `>` symbol in order to run the scripts as a copy/paste into terminal.
> 4. You should validate your own EDirect scripts and results as there may be unintentional mistakes in these recipes. A convenient method is to compare your EDirect results to the NCBI Web interface search results: [https://www.ncbi.nlm.nih.gov/](https://www.ncbi.nlm.nih.gov/).

## EDirect PubChem Entrez Links

### PubChem Compound --> PubMed Citations
**Description:** Search for a CID in the PubChem Compound Database and retrieve related PubMed linked references.

In the below script, we use the `esearch` function to query the PubChem Compound database (`pccompound`) for CID 174076 within the Compound ID field, `[uid]`. The `esearch` results are then piped to `elink` finding related PubMed citations via the Entrez link `pccompound_pubmed`. Finally, we retrieve the results with `efetch` in XML format and extract out some bibliographic reference information using the `xtract` function.

```console

user@computer:~$ esearch -email name@xx.edu -db pccompound -query 174076[uid] | \
> elink -target pubmed -name pccompound_pubmed | \
> efetch -format xml | \
> xtract -pattern PubmedArticle -element MedlineCitation/PMID -first Author/LastName \
> Author/Initials ISOAbbreviation PubDate/Year Volume Issue MedlinePgn
22957575	Gabl	S	J Chem Phys	2012	137	9	094501
22868451	Zhang	Y	Phys Chem Chem Phys	2012	14	35	12157-64
22859056	Malberg	F	Phys Chem Chem Phys	2012	14	35	12079-82
22852554	Zhang	Y	J Phys Chem B	2012	116	33	10036-48
22662183	Zhang	BB	PLoS ONE	2012	7	5	e37641
...
```
_tested on 2021.01.26, EDirect 14.4, total count was 102._

### PubChem Compound --> PubMed Citations (with filtering)
**Description:** Search for CID in PubChem Compound Database, find related PubMed citations, then only retrieve references from a specific journal.

We can filter `elink` results with `efilter` to only include PubMed citations (Entrez linked via `pccompound_pubmed`) to the CID but also matching a specific PubMed query. For example, if we are only interested in linked _Phys Chem Chem Phys_ references to CID 174076, we can use the journal field `[JOUR]` in an `efilter` query:

```console

user@computer:~$ esearch -email name@xx.edu -db pccompound -query 174076[uid] | \
> elink -target pubmed -name pccompound_pubmed | \
> efilter -query "Phys Chem Chem Phys"[JOUR] | \
> efetch -format xml | \
> xtract -pattern PubmedArticle -element MedlineCitation/PMID -first Author/LastName \
> Author/Initials ISOAbbreviation PubDate/Year Volume Issue MedlinePgn
22868451	Zhang	Y	Phys Chem Chem Phys	2012	14	35	12157-64
22859056	Malberg	F	Phys Chem Chem Phys	2012	14	35	12079-82
22451012	Sillars	FB	Phys Chem Chem Phys	2012	14	17	6094-100
21643581	Pensado	AS	Phys Chem Chem Phys	2011	13	30	13518-26
21643580	Schröder	C	Phys Chem Chem Phys	2011	13	26	12240-8
...
```
_tested on 2021.01.26, EDirect 14.4, total count was 11._

### PubChem Compound --> PubMed MeSH (with filtering)
**Description:** Search for a CID in PubChem Compound, find related PubMed records via MeSH, and retrieve only references that contain the MeSH subheading "chemical synthesis".

This is my favorite literature search: start with a PubChem CID and then find PubMed literature related to its synthesis. Similarly to the search above, we can filter out references using an `efilter` query for 'chemical synthesis' as a MeSH subheading `[SUBH]`. Note that we used the `pccompound_pubmed_mesh` Entrez link as the `elink` target name here.

```console

user@computer:~$ esearch -email name@xx.edu -db pccompound -query 94257[uid] | \
> elink -target pubmed -name pccompound_pubmed_mesh | \
> efilter -query "chemical synthesis"[SUBH] | \
> efetch -format xml | \
> xtract -pattern PubmedArticle -element MedlineCitation/PMID ArticleTitle \
> ISOAbbreviation PubDate/Year
28463562	Enantioselective Chemical Syntheses of the Furanosteroids (-)-Viridin and (-)-Viridiol.	J. Am. Chem. Soc.	2017
23040731	Viridin analogs derived from steroidal building blocks.	Bioorg. Med. Chem. Lett.	2012
22849426	Synthetic studies on furanosteroids: construction of the viridin core structure via Diels-Alder/retro-Diels-Alder and vinylogous Mukaiyama aldol-type reaction.	J. Org. Chem.	2012
19644878	Abrogation of antibody-induced arthritis in mice by a self-activating viridin prodrug and association with impaired neutrophil and endothelial cell function.	Arthritis Rheum.	2009
19572524	Pentacyclic furanosteroids: the synthesis of potential kinase inhibitors related to viridin and wortmannolone.	J. Org. Chem.	2009
...
```
_tested on 2021.01.26, EDirect 14.4, total count was 8._


### PubChem Compound --> PubMed Citations OR PubMed MeSH
**Description:** Search for a CID in PubChem Compound, find related PubMed citations and related PubMed citations via MeSH.

It appears that you can combine `elink` queries, with either the same Entrez link or a different Entrez link, but within the same database. For example, if we want to retrieve PubMed literature related to PubChem CID 174076 for both the `pccompound_pubmed` and `pccompound_pubmed_mesh` Entrez links in one dataset, we combine two separate `elink` queries with an OR operator:

```console

user@computer:~$ esearch -email name@xx.edu -db pccompound -query 174076[uid] | \
> elink -target pubmed -name pccompound_pubmed -label pubmed_cit | \
> esearch -email name@xx.edu -db pccompound -query 174076[uid] | \
> elink -target pubmed -name pccompound_pubmed_mesh -label pubmed_mesh_cit | \
> esearch -query "(#pubmed_cit) OR (#pubmed_mesh_cit)" | \
> efetch -format xml | \
> xtract -pattern PubmedArticle -element MedlineCitation/PMID -first Author/LastName \
> Author/Initials ISOAbbreviation PubDate/Year Volume Issue MedlinePgn
32231037	Babicka	M	Molecules	2020	25	7
31931064	Love	SA	Int J Biol Macromol	2020	147	569-575
31818016	Wang	F	Int J Mol Sci	2019	20	24
31814059	Weber	AL	Orig Life Evol Biosph	2019	49	4	199-211
31675504	Gomez-Herrero	E	Ecotoxicol Environ Saf	2020	187	109836
31520950	Pal	S	Ecotoxicol Environ Saf	2019	184	109634
...
```
_tested on 2021.01.26, EDirect 14.4, total count was 317._


### PubChem Substance --> PubChem Compound --> PubMed Publisher
**Description:** Search for a PubChem Substance Data Source Depositor, find related same PubChem Compounds, and then retrieve related PubMed references linked via publisher.

In the below script, we first search the PubChem Substance (`pcsubstance`) database using `esearch` for the data source depositor _Nature Communications_. We can use the Current Source Name `[CSN]` field for this query. Note that an underscore is put in place of the space in the query. This syntax is important for searching in PubChem with the EDirect `esearch` function. After `esearch`, we pipe the results into `elink` twice, first finding related PubChem Compounds via the `pcsubstance_pccompound_same` Entrez link, and then using this new result list to find related PubMed publisher deposited citations from the `pccompound_pubmed_publisher` Entrez link. Finally, similarly to previous searches, we use a combination of `efetch` and `xtract` to retrieve selected data:

```console

user@computer:~$ esearch -email name@xx.edu -db pcsubstance -query "nature_communications"[CSN] | \
> elink -target pccompound -name pcsubstance_pccompound_same | \
> elink -target pubmed -name pccompound_pubmed_publisher | \
> efetch -format xml | 
> xtract -pattern PubmedArticle -element MedlineCitation/PMID -first Author/LastName Author/Initials \
> ISOAbbreviation PubDate/Year Volume Issue MedlinePgn
26673265	Gilbert	ZW	Nat Chem	2016	8	1	63-8
25424885	Yan	T	Nat Commun	2014	5	5602
25422853	Vaidya	AB	Nat Commun	2014	5	5521
25382411	Dommerholt	J	Nat Commun	2014	5	5378
25382259	Wang	B	Nat Commun	2014	5	5354
...
```

_tested on 2021.01.26, EDirect 14.4, total count was 101._


### PubChem Substance --> PubChem Compound <--> PubMed Publisher
**Description:** Search for a PubChem Substance Data Source Depositor, find related same PubChem Compounds, and then retrieve related PubMed PMIDs linked via publisher.

Building upon the previous search, if needed, it is possible to obtain individual relationships of the CIDs to PubMed IDs (CID <--> PMID). We can do this using the `-cmd neighbor` option in `elink`:

```console

user@computer:~$ esearch -email name@xx.edu -db pcsubstance -query "nature_communications"[CSN] | \
> elink -target pccompound -name pcsubstance_pccompound_same | \
> elink -target pubmed -name pccompound_pubmed_publisher -cmd neighbor | \
> xtract -pattern LinkSet -element Id
146033657	24398593
136286496
136264969	24177669
136264968	24177669
136262920	23385592
136262919	23385592
136247006	24457545
136247005	24457545
136247004	24457545
136247003	24457545
136219971
135922679	22027590
91868204	23764831
...
```
_tested on 2021.01.26, EDirect 14.4, total count was 1594 (returns all CIDs, not all have linked PMIDs)._

The first column contains the PubChem CIDs and the second column contains the linked PMIDs. Additional linked PMIDs are placed in subsequent columns when available.


### PubChem Compound --> PubChem BioAssay
**Description:** Search for a PubChem CID in PubChem Compound, then retrieve related PubChem active BioAssay data.

To retrieve BioAssay results labeled as 'Active' that are linked to a CID, we can use the `elink` function with the PubChem BioAssay (`pcassay`) database via Entrez link `pccompound_pcassay_active`. This is followed by `efetch` and `xtract`. In this particular example, we extracted the AID, CurrentSourceName, AssayName, ActiveSidCount, and TargetCount:

```console

user@computer:~$ esearch -email name@xx.edu -db pccompound -query "6303"[uid] | \
> elink -target pcassay -name pccompound_pcassay_active | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element AID CurrentSourceName AssayName ActiveSidCount TargetCount
1255098	ChEMBL	Inhibition of TLR4-mediated NF-kappaB signaling pathway in BALB/c mouse RAW264.7 cells assessed as suppression of LPS-stimulated PGE2 production at 1 to 10 ug/ml preincubated for 1 hr followed by LPS challenge measured after 6 hrs by immunoblot analysis	1	1
1255092	ChEMBL	Inhibition of TLR4-mediated NF-kappaB signaling pathway in BALB/c mouse RAW264.7 cells assessed as suppression of LPS-stimulated TNF-alpha production at 1 to 10 ug/ml preincubated for 1 hr followed by LPS challenge measured after 6 hrs by immunoblot analysis	1	1
751324	ChEMBL	Inhibition of NFkappaB p65 nuclear translocation in mouse RAW264.7 cells after 24 hrs by DAPI staining-based laser confocal immunofluorescent microscopic analysis	1	1

...
```
_tested on 2021.01.26, EDirect 14.4, total count was 47._


### PubChem Compound <--> PubChem BioAssay
**Description:** Search for a PubChem CID in PubChem Compound Database, find related compounds with same connectivity, then retrieve related AIDs for each CID.

It is possible to obtain individual relationships of the CIDs to BioAssay AIDs (CID <--> AID). We can do this using the `-cmd neighbor` option in `elink`. Note that we first found related compounds with same connectivity using the Entrez link `pccompound_pccompound_sameconnectivity_pulldown`. This step was followed by the `pccompound_pcassay_active` Entrez link in the PubChem BioAssay database to retrieve AID links to the CIDs. We used the 'Active' assay links here. There are also other Entrez PubChem Compound assay links such as inactive, `pccompound_pcassay_inactive`.

```console

user@computer:~$ esearch -email name@xx.edu -db pccompound -query "6303"[uid] | \
> elink -target pccompound -name pccompound_pccompound_sameconnectivity_pulldown | \
> elink -target pcassay -name pccompound_pcassay_active -cmd neighbor | \
> xtract -pattern LinkSet -element Id
...
...
6335098
688425
451875	1347103	1296009	2551	2546
248010	687016	652245	651719	177	175	173	171	169	167	165	163	161	159	157	155	153	149	147
6303	1346987	1259407	1255098	1255092	1207585	1207584	1207579	1207578	1207577	1207576	1167619	1159565	1159562	1159559	1159557	1065715	1065714	1065710	1065706	1065697	1065696	1065695	1065713	1065705	1065699	751324	686979	686978	651820	652245	651719	602346	602250	588511	493002	463218	463212	416870	416743	216185	86858	81069	32353	32352	31719	31718	2467
```
_tested on 2021.01.26, EDirect 14.4, total count was 24 CIDs (not all have associated AIDs)._


## EDirect PubMed Entrez Links

### PubMed --> PubChem Compound
**Description:** Search for a PubMed article ID (PMID), then retrieve related PubChem Compounds.

In the below script, we first use `esearch` to query PubMed for the article ID 29407984 in the `[PMID]` field. This result is then piped into `elink` to retrieve linked compounds in the PubChem Compound database (`pubmed_pccompound`). In this case, there was one compound and we used `efetch` to retrieve the CID record as docsum XML, followed by `xtraxt` to extract out the IsomericSmiles, CID, and InChIKey values. 

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "29407984"[PMID] | \
> elink -target pccompound -name pubmed_pccompound | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element IsomericSmiles CID InChIKey
C1CC1N2C=C(C(=O)C3=CC(=C(C=C32)N4CCNCC4)F)C(=O)O	2764	MYSWGUAQZAJSOK-UHFFFAOYSA-N

```
_tested on 2021.01.26, EDirect 14.4, total count was 1._


### PubMed --> PubChem Compound (+ mixtures)
**Description:** Search for a PubMed article ID (PMID), then retrieve linked PubChem Compound mixtures/components.

In this script, an additional `elink` search is added to find related PubChem Mixture/Component compounds via Entrez link `pccompound_pccompound_mixture`.

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "29407984"[PMID] | \
> elink -target pccompound -name pubmed_pccompound | \
> elink -target pccompound -name pccompound_pccompound_mixture | \
> efetch -format docsum | xtract -pattern DocumentSummary -element IsomericSmiles CID InChIKey
...
...
C1CC1N2C=C(C(=O)C3=CC(=C(C=C32)N4CCNCC4)F)C(=O)O.C(CO)N(CCO)CCO	154963193	NGBBVVPJSFAHHI-UHFFFAOYSA-N
C1CC1N2C=C(C(=O)C3=CC(=C(C=C32)N4CCNCC4)F)C(=O)O.C(=O)(C(=O)O)O.[Na]	154963186	HNZYVRRDOWOQIT-UHFFFAOYSA-N
CNC.C1CC1N2C=C(C(=O)C3=CC(=C(C=C32)N4CCNCC4)F)C(=O)O	154963184	NBGZCMHVBXSHSN-UHFFFAOYSA-N
C1CC1N2C=C(C(=O)C3=CC(=C(C=C32)N4CCNCC4)F)C(=O)[OH2+]	153275427	MYSWGUAQZAJSOK-UHFFFAOYSA-O
C1CC1N2C=C(C(=O)C3=CC(=C(C=C32)N4CCNCC4)F)C(=O)[O-]	152748405	MYSWGUAQZAJSOK-UHFFFAOYSA-M
...
...
```
_tested on 2021.01.26, EDirect 14.4, total count was 375._



### PubMed --> PubChem Compound (MESH search)
**Description:** Search PubMed with a text query, then retrieve linked PubChem Compounds.

We can also perform text queries in PubMed and retrieve linked PubChem Compounds. Note that in the below script we searched for "ionic liquids" in the `[MESH]` field and Imidazolium in any field. Since this query requires two pairs of quotes, we have to escape the internal quotes in order for the query to be interpreted correctly. The Entrez link `pubmed_pccompound` was used to find related PubChem compounds.

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "\"ionic liquids\"[MESH] AND imidazolium" | \
> elink -target pccompound -name pubmed_pccompound | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element IsomericSmiles CID InChIKey
C1=CC2=CC(=C(C(=C2C(=O)C(=C1)O)O)O)O	135403797	WDGFFVCWBZVLCE-UHFFFAOYSA-N
C1=NC2=C(N1[C@H]3[C@@H]([C@@H]([C@H](O3)CO)O)O)N=C(NC2=O)N	135398635	NYHBQMYGNKIUIF-UUOKFMHZSA-N
CCCCCCCCN1C=C[N+](=C1C2=[N+](C=CN2CCCC)C)C	123995430	DRJFJBHYMOHPHX-UHFFFAOYSA-N
CC(=O)OC1=[N+](C=CN1CC=C)C	123614562	XSXMFLUARQMOLS-UHFFFAOYSA-N
C[N+]1=C(N(C=C1)CCCCCCCCCCCCS)OC(=O)OC2=[N+](C=CN2CCCCCCCCCCCCS)C	123431445	DRJOSAVMFCYCSU-UHFFFAOYSA-P
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 395._

### PubMed --> PubChem Compound (MESH search, and a PubChem filter)
**Description:** Search PubMed with a text query and retrieve only linked compounds containing defined chiral atoms.

We can also perform some powerful filtering with `efilter`. In the below script, the `[ACDC]` field is the defined atom chiral count in PubChem. A range of 1 through 100 was added for this `[ACDC]` filter. Since it is unlikely that any of the compounds would have near 100 chiral atoms, we can be fairly confident this should capture most, if not all, cases in our search.

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "\"ionic liquids\"[MESH] AND imidazolium" | \
> elink -target pccompound -name pubmed_pccompound | \
> efilter -query "1:100"[ACDC] | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element IsomericSmiles CID InChIKey
C1=NC2=C(N1[C@H]3[C@@H]([C@@H]([C@H](O3)CO)O)O)N=C(NC2=O)N	135398635	NYHBQMYGNKIUIF-UUOKFMHZSA-N
C([C@H](C(=O)[C@H](CO)O)O)O	54067296	WXYXERHRDKEISL-ZXZARUISSA-N
B(O)(O)OCC(=O)[C@H]([C@@H]([C@@H](CO)O)O)O	53705729	BXAZSOZNRVUIGN-UYFOZJQFSA-N
C([C@H]1[C@@H]([C@H]([C@@H]([C@@H](O1)O[C@@H]2[C@@H](O[C@H]([C@@H]([C@H]2O)O)O)CO)O)O)O)O	46936190	GUBGYTABKSRVRQ-AEDSEYDFSA-N
C1[C@H](OC2=CC(=CC(=C2C1=O)O)OC3C(C(C(C(O3)CO)O)O)O)C4=CC=C(C=C4)O	42607902	DLIKSSGEMUFQOK-CEFFZDIVSA-N
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 43._


### PubMed --> PubChem Compounds + PubChem Compounds (MeSH) + PubChem Compounds (Publisher)
**Description:** Search PubMed, then find linked PubChem Compounds, PubChem Compounds via PubMed MeSH, and PubChem Compound PubMed Publisher.

As seen in the previous PubChem searches, there are several Entrez links from PubMed to PubChem Compound such as `pubmed_pccompound`, `pubmed_pccompound_mesh`, and `pubmed_pccompound_publisher`. We can retrieve associated compounds from all three at the same time like this:

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "imidazolium AND bacteria" | \
> elink -target pccompound -name pubmed_pccompound -label compounds_01 | \
> esearch -email name@xx.edu -db pubmed -query "imidazolium AND bacteria" | \
> elink -target pccompound -name pubmed_pccompound_mesh -label compounds_02 | \
> esearch -email name@xx.edu -db pubmed -query "imidazolium AND bacteria" | \
> elink -target pccompound -name pubmed_pccompound_publisher -label compounds_03 | \
> esearch -query "(#compounds_01) OR (#compounds_02) OR (#compounds_03)" | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element IsomericSmiles CID InChIKey
C[Si](C)(O)O[Si](C)(C)O.C[Si](CCC1=CC=CC=C1)(O)O[Si](C)(CCC2=CC=CC=C2)O	155288862	HYTMPCHVCUAIOW-UHFFFAOYSA-N
CC(C1CCC(C(O1)OC2C(CC(C(C2O)OC3C(C([C@@](CO3)(C)O)NC)O)N)N)N)NC	146157093	CEAZRRDELHUEMR-NWNXOGAHSA-N
C1=CC=C2C(=C1)C=C(N2)CC3=CC=C(C=C3)C(F)(F)F.C1=CC=C2C(=C1)C=C(N2)CC3=CC=C(C=C3)C(F)(F)F	139191468	NNXWVRXROOQQNO-UHFFFAOYSA-N
CC[C@@H]1[C@@]2([C@@H]([C@H](C(=O)[C@@H](C[C@@]([C@@H]([C@H](C(=O)[C@H](C(=O)O1)C)C)O[C@@H]3[C@@H]([C@H](C[C@H](O3)C)N(C)C)O)(C)OC)C)C)N(C(=O)O2)CCCCN4C=C(N=C4)C5=CN=CC=C5)C	138402871	LJVAJPDWBABPEJ-WMGYHEQLSA-N
CCOC(=O)/C(=N\NC1=CC=CC2=C1N=CC=C2)/C3=[N+](C=CN3)C	136199795	ZHOWQGCGVQGEKD-UHFFFAOYSA-O
...
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 568._


### PubMed <--> PubChem Compound
**Description:** Search PubMed for an affiliation, find related PubChem Compounds, then retrieve related CIDs for each PMID.

If we want to retrieve the PMID <--> CID relationships (for Entrez link `pubmed_pccompound`), we can achieve this using the `-cmd neighbor` option in `elink`:

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "(university of alabama[AFFL]) \
> NOT (birmingham[AFFL] OR huntsville[AFFL])" \
> -datetype PDAT -mindate 2010 -maxdate 2020 | \
> elink -target pccompound -name pubmed_pccompound -cmd neighbor | \
> xtract -pattern LinkSet -element Id
...
...
21800250
21783326	18679079	8914	942	702
21782896
21756136
21728552
21718269
21711000	561577	169577	166929	166928	164636
21702462
21693669
21692575
...
...
```

_tested on 2021.01.27, EDirect 14.4, total count was 3639 (returns all PMIDs, not all have linked CIDs)._

The first column contains the PMIDs and the second column contains the linked PubChem CIDs (from the `pubmed_pccompound` links). As an aside, the PubMed query for "university of alabama" in the affiliation field (`[AFFL]`) excludes (NOT operator) any results containing huntsville or birmingham in the affiliation. This excludes references from University of Alabama at Birmingham and University of Alabama at Huntsville (including collaborative references with the Tuscaloosa campus).


### PubMed --> PubChem BioAssay
**Description:** Search PubMed for an article, find related PubChem BioAssays, then retrieve some BioAssay data.

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "32459468"[PMID] | \
> elink -target pcassay -name pubmed_pcassay | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element AID CurrentSourceName AssayName ActiveSidCount TargetCount
1347414	National Center for Advancing Translational Sciences (NCATS)	qHTS to identify inhibitors of the type 1 interferon - major histocompatibility complex class I in skeletal muscle: Secondary screen by immunofluorescence	0	1
1347412	National Center for Advancing Translational Sciences (NCATS)	qHTS assay to identify inhibitors of the type 1 interferon - major histocompatibility complex class I in skeletal muscle: Counter screen cell viability and HiBit confirmation	0	1
1347415	National Center for Advancing Translational Sciences (NCATS)	qHTS to identify inhibitors of the type 1 interferon - major histocompatibility complex class I in skeletal muscle: tertiary screen by RT-qPCR	34	1
1347413	National Center for Advancing Translational Sciences (NCATS)	qHTS to identify inhibitors of the type 1 interferon - major histocompatibility complex class I in skeletal muscle: tertiary screen by RT-qPCR, retest select compounds	3	1
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 7._

### PubMed <--> PubChem BioAssay
**Description:** Search PubMed for an article, find cited articles, then related PubChem BioAssays.

If we want to retrieve the PMID <--> AID relationships (for Entrez link `pubmed_pcassay`), we can achieve this using the `-cmd neighbor` option in `elink`. Note that here we queried PubMed for an article, then found the cited articles with `elink -cited`, before piping these results into the Entrez link `pubmed_pcassay`.


```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "17876319"[PMID] | \
> elink -cited | \
> elink -target pcassay -name pubmed_pcassay -cmd neighbor | \
> xtract -pattern LinkSet -element Id
...
...
21167154
21164511
21159777
21138309	568760	568754	568753	568763	568762	568761	568759	568758	568757	568756	568755
21131971
21129186
...
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 366 (returns all PMIDs, not all have linked AIDs)._

In the above table, the first column contains the PMIDs, subsequent columns contain the linked BioAssays (AIDs).

## EDirect PubChem BioAssay Entrez Links

### PubChem BioAssay --> PubMed
**Description:** Search PubChem BioAssay for assays from a specific source name and then find related PubMed literature.

In the below script, we first use `esearch` to query PubChem BioAssay for IUPHAR/BPS_Guide_to_PHARMACOLOGY in the Source Name field (`[SNME]`). This result is then piped into `elink` to retrieve linked records in the PubMed database (`pcassay_pubmed`). The `efilter` function was used to limit the results to the last 5 years. This resulted in 332 record, and we used `efetch` to retrieve the PubMed records as XML, followed by `xtract` to extract out some bibliographic information.

```console

user@computer:~$ esearch -email name@xx.edu -db pcassay -query "IUPHAR/BPS_Guide_to_PHARMACOLOGY"[SNME] | \
> elink -target pubmed -name pcassay_pubmed | \
> efilter -mindate 2015 -maxdate 2020 -datetype PDAT | \
> efetch -format xml | \
> xtract -pattern PubmedArticle -element MedlineCitation/PMID -first Author/LastName \
> Author/Initials ISOAbbreviation PubDate/Year Volume Issue MedlinePgn
29722898	Fu	R	Br. J. Pharmacol.	2018	175	14	3034-3049
29688582	Kato	M	Br J Clin Pharmacol	2018	84	8	1821-1829
29683659	Pike	KG	J. Med. Chem.	2018	61	9	3823-3841
29674331	Kawaharada	S	J. Pharmacol. Exp. Ther.	2018	366	1	58-65
29672049	Gucký	T	J. Med. Chem.	2018	61	9	3855-3869
29620892	Nikolaou	A	J. Med. Chem.	2018	61	8	3697-3711
29615471	Xu	X	J. Pharmacol. Exp. Ther.	2018	365	3	624-635
29608575	Taylor Meadows	KR	PLoS ONE	2018	13	4	e0193236
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 332._

### PubChem BioAssay --> PubChem Compound
**Description:** Search PubChem BioAssay for an assay, find related PubChem Compounds, and retrieve some property data for the compounds.

In the below script, we first use `esearch` to query PubChem BioAssay for the assay ID 527855 in the `[UID]` field. This result is then piped into `elink` to retrieve linked compounds in the PubChem Compound database (`pcassay_pccompound`). In this case, there were 16 compounds and we used `efetch` to retrieve the CID records as docsum XML, followed by `xtract` to extract the IsomericSmiles, CID, HydrogenBondDonorCount, HydrogenBondAcceptorCount, MolecularWeight, and XLogP values.

```console

user@computer:~$ esearch -email name@xx.edu -db pcassay -query "527855"[UID] | \
> elink -target pccompound -name pcassay_pccompound | \
> efetch -format docsum | \
> xtract -pattern DocumentSummary -element IsomericSmiles CID HydrogenBondDonorCount HydrogenBondAcceptorCount \
> MolecularWeight XLogP
CN(CC1=CC=CC=C1)C(=O)C2=C(NC(=N2)C3=CC=CC=C3)C(=O)O	52949178	2	4	335.400	2.9
C1=CC=C(C=C1)CCN(CC2=CC=CC=C2)C(=O)C3=C(NC(=N3)C4=CC=CC=C4)C(=O)O	52948352	2	4	425.500	4.8
CN(CC(=O)O)C(=O)C1=C(NC(=N1)C2=CC=CC=C2)C(=O)O	52947957	3	6	303.270	1
C1=CC=C(C=C1)CNC(=O)C2=C(NC(=N2)C3=CC=CC=C3)C(=O)O	52946755	3	4	321.300	2.7
CCOC(=O)CN(CC1=CC=CC=C1)C(=O)C2=C(NC(=N2)C3=CC=CC=C3)C(=O)O	52945544	2	6	407.400	3.2
C1=CC=C(C=C1)CN(CC2=CC=CC=C2)C(=O)C3=C(NC(=N3)C4=CC(=CC=C4)Cl)C(=O)O	52944295	2	4	445.900	5
CCNC(=O)C1=C(NC(=N1)C2=CC=CC=C2)C(=O)O	52941818	3	4	259.260	1.6
CNC(=O)C1=C(NC(=N1)C2=CC=CC=C2)C(=O)O	52941817	3	4	245.230	1.2
...
```
_tested on 2021.01.27, EDirect 14.4, total count was 16._

### PubChem BioAssay <--> PubChem Compound
**Description:** Search PubChem BioAssay for an assay, find related assays based on similar publications, then find related PubChem Compounds.

If we want to retrieve the AID <--> CID relationships (for Entrez link `pcassay_pccompound`), we can achieve this using the `-cmd neighbor` option in `elink`. Here we queried PubChem BioAssay for an assay, then found related assays by similar publication list using `elink` (`pcassay_pcassay_similar_publication_list`). This result was then piped into the Entrez link `pcassay_pccompound`.


```console

user@computer:~$ esearch -email name@xx.edu -db pcassay -query "527855"[UID] | \
> elink -target pcassay -name pcassay_pcassay_similar_publication_list | \
> elink -target pccompound -name pcassay_pccompound -cmd neighbor | \
> xtract -pattern LinkSet -element Id
...
...
601409	54580326
601408	54580326	53257623
601154	54580326
657046	16093559
657045	70695880	70693764	70687505	70687504	70687503	70683264	70683263	70681155	70681154	70681153	60150625
527862	52948352
527861	52948352
...
...
...
```

In the above table, the first column contains the AIDs, subsequent columns contain the linked PubChem Compounds (CIDs).

_tested on 2021.01.27, EDirect 14.4, total count was 81 (returns all AIDs, not all have linked CIDs)._

