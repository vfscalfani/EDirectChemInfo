# What is EDirect?

EDirect is a Unix command line tool from NCBI that allows programmatic retrieval of chemical/biological data and literature references from NCBI databases. EDirect reduces the barrier to accessing NCBI data programmatically; that is, with a basic knowledge of the Unix shell (e.g., bash), it is straightforward to obtain and format your own custom datasets, often with only a few lines of code. Moreover, you can input data retrieved from EDirect into other Unix tools for quick viewing and analysis ([Pipeline (Unix))](https://en.wikipedia.org/wiki/Pipeline_(Unix)).

## Installation Tips

Follow the installation instructions from NCBI: [Entrez Direct: E-utilities on the UNIX Command Line](https://www.ncbi.nlm.nih.gov/books/NBK179288/). There are several different methods to install EDirect. I used option 3 (EDirect v14.4) with `wget` in Gnome Terminal on a Linux Ubuntu 18.04 workstation. If you are using Windows, NCBI mentions that you can use the Cygwin Unix emulator. Another option for Windows users is to setup a Linux virtual machine. There are many tutorials for setting up virtual machines. For example, here is one for installing [Ubuntu on VirtualBox](https://askubuntu.com/questions/142549/how-to-install-ubuntu-on-virtualbox). When installing EDirect in a virtual machine, you may need to customize the VirtualBox network settings in order to use the `curl` or `wget` EDirect installation methods. In my testing on an Ubuntu 20.04 virtual machine, the fourth installation option for EDirect (using the longer perl script) worked fine with the standard VirtualBox network settings.

## Usage Tips

NCBI has specific data usage policies and disclaimers:

* [NCBI Website and Data Usage Policies and Disclaimers](https://www.ncbi.nlm.nih.gov/home/about/policies/)
* [Entrez Programming Utilities Help](https://www.ncbi.nlm.nih.gov/books/NBK25501/)

If you do not follow NCBI's usage policies (e.g., no more than 3 requests per second), NCBI may block your IP address. So be cautious and follow good programming practices of testing and adding sleep delays, particularly if executing multiple sequential calls in a loop. Moreover, it is always a good idea to include your email address in the requests so that NCBI can contact you if necessary. You can add your email address within each query like this:

```console

user@computer:~$ e-function -email name@xx.edu -arg input

```
Replace `name@xx.edu` with your email address. The `e-function` is a place holder for one of the actual EDirect functions like `einfo` or `esearch`, and `-arg input` is a placeholder for e-function argument(s) like `-db pccompound` or `-db pubmed -query "food allergies"`.

## EDirect Function Help

I generally refer to the official [Entrez Programming Utilities Help Document](https://www.ncbi.nlm.nih.gov/books/NBK25501/) or the [NIH NLM E-Utilities Documentation](https://dataguide.nlm.nih.gov/eutilities/utilities.html), however for a quick reference or reminder of the proper syntax, the `-help` option is useful. Here is an example with the `einfo` function:

```console

user@computer:~$ einfo -help
einfo 14.4

Database Selection

  -dbs       Print all database names
  -db        Database name (or "all")

Data Summaries

  -fields    Print field names
  -links     Print link names

Field Example

  <Field>
    <Name>ALL</Name>
    <FullName>All Fields</FullName>
    <Description>All terms from all searchable fields</Description>
    <TermCount>245340803</TermCount>
    <IsDate>N</IsDate>
    <IsNumerical>N</IsNumerical>
    <SingleToken>N</SingleToken>
    <Hierarchy>N</Hierarchy>
    <IsHidden>N</IsHidden>
    <IsTruncatable>Y</IsTruncatable>
    <IsRangable>N</IsRangable>
  </Field>

Link Example

  <Link>
    <Name>pubmed_protein</Name>
    <Menu>Protein Links</Menu>
    <Description>Published protein sequences</Description>
    <DbTo>protein</DbTo>
  </Link>
  <Link>
    <Name>pubmed_protein_refseq</Name>
    <Menu>Protein (RefSeq) Links</Menu>
    <Description>Link to Protein RefSeqs</Description>
    <DbTo>protein</DbTo>
  </Link>

```

## EDirect Query Translation via Debug Flag

When experimenting with searches in EDirect, it is often helpful to view the interpreted query. This can be accomplished using the `-debug` flag in EDirect 14.4 (thanks to NLM Support for the explanation and tip!):

```console

user@computer:~$ esearch -email name@xx.edu -db pubmed -query "hydrogel-based drug delivery" -debug
nquire -url https://eutils.ncbi.nlm.nih.gov/entrez/eutils/ esearch.fcgi -retmax 0 -usehistory y -db pubmed -term "hydrogel-based drug delivery" -tool edirect -edirect 14.4 -edirect_os Linux -email name@xx.edu
<ENTREZ_DIRECT>
  <Db>pubmed</Db>
  <WebEnv>MCID...</WebEnv>
  <QueryKey>1</QueryKey>
  <Count>436</Count>
  <Step>1</Step>
  <Email>name@xx.edu</Email>
  <Debug>Y</Debug>
</ENTREZ_DIRECT>

```
Next, copy and run the `nquire` command and pipe the results to `xtract`, extracting out the QueryTranslation element:

```console

user@computer:~$ nquire -url https://eutils.ncbi.nlm.nih.gov/entrez/eutils/ esearch.fcgi -retmax 0 -usehistory y -db pubmed -term "hydrogel-based drug delivery" | xtract -pattern eSearchResult -element QueryTranslation
hydrogel-based[All Fields] AND ("drug delivery systems"[MeSH Terms] OR ("drug"[All Fields] AND "delivery"[All Fields] AND "systems"[All Fields]) OR "drug delivery systems"[All Fields] OR ("drug"[All Fields] AND "delivery"[All Fields]) OR "drug delivery"[All Fields])

```

