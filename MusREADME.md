# WildMileDB - Mussels 
This repository serves to explain how to build reference databases using CRABS for Illinois vertebrate and mussel species, with the generated databases attached along with other helpful files. This specific README files focuses on how to curate the mussel database.

## Making a List of Species of Interest

The first (and most tedious) step of creating the reference database if creating a list of species of intrest. This is what we will use to subset the sequences we get from public repositories, and it allows for a more focused scope. There are many different ways that you can choose to go about this process. For my project, I wanted to focus on mussel species found in Illinois.

The most comprehensive approach I found was utilizing the Illinois Natural History Survey (INHS). I used all the species found on the following website to create the reference subset list. 

* [Mussels](https://mollusk.inhs.illinois.edu/species/freshwater-bivalves-of-illinois/)

CRABS requires this list of species to be in a line seperated `.txt` file, with NO header.

After you have a list of species of interest, you will also want to add common contaminants to the subset list, these include:
* Homo sapiens (Human)
* Gallus gallus (Chicken)
* Bos taurus (Cow)
* Canis lupus familiaris (Dog)
* Equus caballus (Horse)
* Thunnini (Tuna)
* Felis catus (Cat)
* Capra hircus (Goat)

For our study, we did not have positive controls for the mussel runs, so those were not added, but in case you did make sure to add those species to the subset list as well. 

The complete list of vertebrates for Illinois using the above info is included in this repository, [here](https://github.com/richapatel138/WildMileDB/blob/main/MusselDB/mussels_complete.txt).

## Creating Reference Database Using CRABS

To create the reference database, we will use [CRABS](https://github.com/gjeunen/reference_database_creator). The CRABS GitHub has great instructions for installation as well as different uses. Below I have outlined the commands I used. 

1. The first step is to download taxonomic information files -- these are needed to assign taxonomic lineage to each downloaded sequence in the reference database.
```
mkdir crabs_tax
crabs --download-taxonomy --output crabs_tax
```

2. Download necessary sequences from databases.

This step will depend on what sequences you need. For the mussel database, we will be downloading the total lrRNA database (this is 16S, what we PCR'd with). This is suggested as downloading sequences straight from NCBI can be really time and memory expensive. Make sure to note the MIDORI release number, as new releases could potentially change things. 

**NOTE:** When advised on databases to use, I was told to use EMBL to download sequences as well, however, I found that using only MIDORI led to the same subsetted database as using MIDORI and EMBL. Additionaly, since downloading EMBL sequences creates a bigger pool of data to work with, the downstream steps take A LOT longer (for example, the pairwise alignment step took 3 days to run), and since I saw no difference in the subsetted database with EMBL versus MIDORI alone, I have chosen to provide documentation for the MIDORI only database. If you wanted to download the EMBL database, you would have to follow similar steps but have to add the merge step (see CRABS documentation).  

```
crabs --download-midori --output midori_267.fasta --gb-number 267_2025-06-19 --gene lrRNA --gb-type total
```

3. Import database into CRABS format.

```
crabs --import --import-format MIDORI --input midori_267.fasta --names crabs_tax/names.dmp --nodes crabs_tax/nodes.dmp --acc2tax crabs_tax/nucl_gb.accession2taxid --output midori_267.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
```

4. Dereplicate & filter BEFORE in-silico PCR.

This step was suggested in the CRABS documnetation as to reduce the time it takes for the later steps of in-silico PCR and pairwise alignment, though I cannot comment on the differences in the generated databses as I have not done it without first dereplicating and filtering.

```
crabs --dereplicate --input midori_267.txt --output midori_invert_derep.txt --dereplication-method 'unique_species'
crabs --filter --input midori_invert_derep.txt --output midori_invert_filtered.txt --environmental --no-species-id --rank-na 2
```

5. In-silico PCR with 12S-V5 primers (relaxed). 
```
crabs --in-silico-pcr --input midori_invert_filtered.txt --output midori_invert_PCR_relaxed.txt --forward AWGGWAGACGAAAAGACCCCGC --reverse AAGCCAACATCGAGGTCGCAAAC --relaxed
```

6. Rescue missed sequences with pairwise alignment.
```
crabs --pairwise-global-alignment --input midori_invert_filtered.txt --amplicons midori_invert_PCR_relaxed.txt --output midori_invert_aligned.txt --forward AWGGWAGACGAAAAGACCCCGC --reverse AAGCCAACATCGAGGTCGCAAAC --percent-identity 0.95 --coverage 95
```

7. Dereplicate & filter again.
```
crabs --dereplicate --input midori_invert_aligned.txt --output midori_invert_aligned_dereplicated.txt --dereplication-method 'unique_species'
crabs --filter --input midori_invert_aligned_dereplicated.txt --output midori_invert_aligned_filtered.txt --minimum-length 100 --maximum-length 300 --maximum-n 1 --environmental --no-species-id --rank-na 2
```

8. Subset using species list.
```
crabs --subset --input midori_invert_aligned_filtered.txt --output midori_mussels_subset.txt --include /home/rpatel/THESIS/REFDB/species_list/mussels_complete.txt```
```

9. Visualize the subsetted database.

Something to remeber is that not every sequence that you have in your subset list will end up being in the subsetted reference database. This is due to the factt that only species that have reference sequences can be subsetted for. A good way to see the composition of the reference database is to use the `--diversity-figure` functionality. It will show you the number of species and number of sequences for the level you specified (4 = order level classification, though this can be changed if you want something more/less specific). 

```
crabs --diversity-figure --input midori_mussels_subset.txt --output midori_mussels_subset_diversity_figure.png --tax-level 4
```

Once you have the final `.txt` file, this can be exported depending on your needs. For us, we use QIIME, so these are the commands to export the subsetted reference database into QIIME format.

10. Export for QIIME use.

```
crabs --export --input midori_mussels_subset.txt --output midori_mussels_seq.fasta --export-format 'qiime-fasta' #this is the sequence file
crabs --export --input midori_mussels_subset.txt --output midori_mussels_tax.txt --export-format 'qiime-text' #this is the taxonomy file
```

11. Get into QIIME format. You will need to deactivate CRABS and activate the QIIME enviroment.

```
qiime tools import --type 'FeatureData[Sequence]' --input-path midori_mussels_seq.fasta --output-path midori_mussels_seq.qza
qiime tools import --type 'FeatureData[Taxonomy]' --input-format HeaderlessTSVTaxonomyFormat --input-path midori_mussels_tax.txt --output-path midori_mussels_tax.qza
```

## Useful Files

In the MusselDB folder, I have included some files that might be helpful. 

* `mussel_complete.txt`
  * List of species I subsetted with. Includes Illinois mussels with common contaminants. 
* `midori_invert_aligned_filtered.txt`
  * Full MIDORI database that has been PCR'd, aligned, dereplicated, and filtered but NOT subseted. This can be used if you wish to subset with a different list of species.
* `midori_mussels_subset.txt`
  * CRABS formated subsetted reference database file.
* `midori_mussels_tax.qza`
  * QIIME formated subsetted refrence database taxonomy file.
* `midori_mussels_seq.qza`
  * QIIME formated subsetted refrence database sequence file.
