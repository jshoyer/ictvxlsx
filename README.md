
My goal here
is to make it a little bit easier
to use tables
from the ICTV
(the International Committee on the Taxonomy of Viruses)
in R,
either directly
or via dedicated R packages,
tracked on branches.

Depending on your goal,
you might not need to use R at all.
If you want to quickly browse the taxonomic hierarchy
from the top down,
the ICTV site
includes a nice [visual browser](https://ictv.global/taxonomy/visual-browser)
for current and past versions.
Another interactive view
that is cross-referenced by annotated host distribution
is at https://www.ncbi.nlm.nih.gov/labs/virus
(NCBI = US National Center for Biotechnology Information).
The NCBI Taxonomy database
provides a text-based hierarchical browser
that can be used to access many types of data
for each taxon.
A separate interface
at https://www.ncbi.nlm.nih.gov/datasets
allows one to browse the NCBI taxonomic hierarchy
while searching for multiple taxon names
at once.
As noted by Siddell et al. ([2023](https://doi.org/10.1099/jgv.0.001840))
in a paper on the role of the ICTV,
the NCBI Taxonomy team
uses ICTV releases
but there is a short lag each time,
so it can be worthwhile to cross-reference the ICTV tables.

The main product that the ICTV distributes
is an Excel file “species list” table
(the MSL).
Let's say that you have downloaded
the current Excel table
from ictv.global/msl
(download shortcut https://ictv.global/msl/current)
and put it in your current working directory.
```{r, warning=FALSE}
library(readxl)

msl2022 <-
  read_excel("ICTV_Master_Species_List_2022_MSL38.v3.xlsx",
             sheet = 2, .name_repair = "universal")
```

The ICTV distributes a second table,
the metadata resource
(ictv.global/vmr, 
 shortcut https://ictv.global/vmr/current).
This table includes
recommended common names
and abbreviations,
which are distinct from formal taxonomic names.
It also list an exemplar sequence
(GenBank accession numbers)
for each species
and in some cases one or more additional (A) sequence rows.
```{r, warning=FALSE}
library(dplyr)

vmr2022 <-
  read_excel("VMR_MSL38_v2.xlsx",
             sheet = 1, .name_repair = "universal")
```

Much higher-rank information
is duplicated
in the MSL and VMR,
so I like to combine the tables.
In effect
this adds the exemplar sequence (e) information to the MSL.
```{r}
library(dplyr)

vmr2022_e <- filter(vmr2022,
                    Exemplar.or.additional.isolate == "E") %>%
  select(-Exemplar.or.additional.isolate)

msl2022_e <- left_join(msl2022, vmr2022_e,
                       by = c("Realm", "Subrealm", "Kingdom", "Subkingdom",
                              "Phylum", "Subphylum", "Class", "Subclass",
                              "Order", "Suborder", "Family", "Subfamily",
                              "Genus", "Subgenus", "Species",
                              "Genome.Composition" = "Genome.composition"))
```

It is often convenient
to add NCBI taxon IDs to (subsets of) this data frame.
The best way I have found to do this so far
is to navigate the taxonomy database
and/or query it with a URL like so
https://www.ncbi.nlm.nih.gov/taxonomy/?term=txid10239[Subtree]
(possibly replacing 10239,
 the taxonomy ID covering all viruses,
 with a more specific ID).
By clicking on ‘Send to’
one can export both a Taxon name file
and a Taxid list file.
Reading these files into R,
cbinding them into a data frame,
and then joining that data frame
to the one above
by species names
(or genus names,
 or the names for any other rank of interest)
is straightforward.

## History of the taxonomy
Others have considered using one or more R packages
to access the full history of the taxonomy,
including taxon merging, splitting, reassignment-moves, and renaming
(taxa package [issue 136](https://github.com/ropensci/taxa/issues/136)
 and linked taxize issue 695).
 