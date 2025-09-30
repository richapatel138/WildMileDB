# WildMileDB - Vertebrate
This repository serves to explain how to build refrence databases using CRABS for Illinois vertebrate and mussel species, with the generated databases attached along with other helpful files. This specific README files focuses on how to curate the vertebrate database.

## Making a List of Species of Interest

The first (and most tedious) step of creating the refrence database if creating a list of species of intrest. This is what we will use to subset the sequences we get from public repositories, and it allows for a more focused scope. There are many different ways that you can choose to go about this process. For my project, I wanted to focus on vertebrate species found in Illinois, encompasing fish, mammals, bird, amphibians, and reptiles.

The most comprehensive approach I found was utilizing the Illinois Natural History Survey (INHS), and going to the respective website to collect species found in Illinois. I used all the species found on each website to create the refrence subset list. 

* [Fish](https://fish.inhs.illinois.edu/illinois-species-list/)
* [Mammals](https://mammals.inhs.illinois.edu/mammals-of-illinois/)
* [Birds](https://bird.inhs.illinois.edu/birds-of-illinois-checklist/?_gl=1*1vx9wqg*_ga*MjU0MzE3MTI4LjE3NTc0Mzc4MTM.*_ga_8XRWZCXCM7*czE3NTc0Mzc4MTMkbzEkZzAkdDE3NTc0Mzc4MjEkajUyJGwwJGgw)
* [Amphibians & Reptiles](https://herpetology.inhs.illinois.edu/species-lists/ilspecies/)

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

Additionally, depending on what types of positive controls are used in your project design, you will also want to add those to the list as well. For us, they were:
* Negaprion (Lemon sharks)
* Squatina (Angel sharks)

The complete list of vertebrates for Illinois using the above info is included in this repository, [here](https://github.com/richapatel138/WildMileDB/blob/main/VertebrateDB/vertebrate_complete.txt). 

## Creating Refrence Database Using CRABS

To create the refrence database, we will use [CRABS](https://github.com/gjeunen/reference_database_creator). The CRABS GitHub has great instructions for installation as well as different uses. Below I have outlined the commands I used. 

1. The first step is to download taxonomic information files -- these are needed to assign taxonomic lineage to each downloaded sequence in the reference database.
```
mkdir crabs_tax
crabs --download-taxonomy --output crabs_tax
```

2. Download necessary sequences from databases.

This step will depend on what sequences you need. For the vertebrate database, we will be downloading the total srRNA database (this is 12S, what we PCR'd with) and the MitoFish database (for more fish species). This is suggested as downloading sequences straight from NCBI can be really time and memory expensive. Make sure to note the MIDORI release number, as new releases could potentially change things. 

```
crabs --download-midori --output midori_267.fasta --gb-number 267_2025-06-19 --gene srRNA --gb-type total
crabs --download-mitofish --output mitofish.fasta
```

3. Import both databases into CRABS format.

```
crabs --import --import-format MIDORI --input midori_267.fasta --names crabs_tax/names.dmp --nodes crabs_tax/nodes.dmp --acc2tax crabs_tax/nucl_gb.accession2taxid --output midori_267.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
crabs --import --import-format MITOFISH --input mitofish.fasta --names crabs_tax/names.dmp --nodes crabs_tax/nodes.dmp --acc2tax crabs_tax/nucl_gb.accession2taxid --output mitofish.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
```

4. Merge the two databases.

```
crabs --merge --input 'midori_267.txt;mitofish.txt' --uniq --output merged_vert.txt
```

5. Dereplicate & filter BEFORE in-silico PCR.

This step was suggested in the CRABS documnetation as to reduce the time it takes for the later steps of in-silico PCR and pairwise alignment, though I cannot comment on the differences in the generated databses as I have not done it without first dereplicating and filtering.

```
crabs --dereplicate --input merged_vert.txt --output merged_vert_derep.txt --dereplication-method 'unique_species'
crabs --filter --input merged_vert_derep.txt --output merged_vert_filtered.txt --environmental --no-species-id --rank-na 2
```

6. In-silico PCR with 12S-V5 primers (relaxed). 

```
crabs --in-silico-pcr --input merged_vert_filtered.txt --output merged_vert_PCR_relaxed.txt --forward ACTGGGATTAGATACCCC --reverse TAGAACAGGCTCCTCTAG --relaxed
```

7. Rescue missed sequences with pairwise alignment.

This step can be pretty time intensive. Depending on the number of sequences, it can take anywhere from an hour to days. I suggest using a `screen`.  

```
crabs --pairwise-global-alignment --input merged_vert_filtered.txt --amplicons merged_vert_PCR_relaxed.txt --output merged_vert_aligned.txt --forward ACTGGGATTAGATACCCC --reverse TAGAACAGGCTCCTCTAG --percent-identity 0.95 --coverage 95
```

8. Dereplicate & filter again.

```
crabs --dereplicate --input merged_vert_aligned.txt --output merged_vert_aligned_dereplicated.txt --dereplication-method 'unique_species'
crabs --filter --input merged_vert_aligned_dereplicated.txt --output merged_vert_aligned_filtered.txt --minimum-length 100 --maximum-length 300 --maximum-n 1 --environmental --no-species-id --rank-na 2
```

9. Subset using species list.
```
crabs --subset --input merged_vert_aligned_filtered.txt --output vertebrate_subset.txt --include VertebrateDB/vertebrate_complete.txt
```

10. Visualize the subsetted database.

Something to remeber is that not every sequence that you have in your subset list will end up being in the subsetted refrence database. This is due to the factt that only species that have refrence sequences can be subsetted for. A good way to see the composition of the refrence database is to use the `--diversity-figure` functionality. It will show you the number of species and number of sequences for the level you specified (4 = order level classification, though this can be changed if you want something more/less specific). 

```
crabs --diversity-figure --input vertebrate_subset.txt --output vertebrate_subset_diversity_figure.png --tax-level 4
```

Once you have the final `.txt` file, this can be exported depending on your needs. For us, we use QIIME, so these are the commands to export the subsetted refrence database into QIIME format.

11. Export for QIIME use.

```
crabs --export --input vertebrate_subset.txt --output vertebrate_seq.fasta --export-format 'qiime-fasta' #this is the sequence file
crabs --export --input vertebrate_subset.txt --output vertebrate_tax.txt --export-format 'qiime-text' #this is the taxonomy file
```

12. Get into QIIME format. You will need to deactivate CRABS and activate the QIIME enviroment.

```
qiime tools import --type 'FeatureData[Sequence]' --input-path vertebrate_seq.fasta --output-path vertebrate_seq.qza
qiime tools import --type 'FeatureData[Taxonomy]' --input-format HeaderlessTSVTaxonomyFormat --input-path vertebrate_tax.txt --output-path vertebrate_tax.qza
```

## Useful Files

In the VertebrateDB folder, I have included some files that might be helpful. 

* `vertebrate_complete.txt`
  * List of species I subsetted with. Includes Illinois fish, mammals, birds, amphibians, and reptiles, with common contaminants, and positive controls. 
* `merged_vert_aligned_filtered.txt`
  * Full MIDORI & MitoFish database that has been PCR'd, aligned, dereplicated, and filtered but NOT subseted. This can be used if you wish to subset with a different list of species.
* `vertebrate_subset.txt`
  * CRABS formated subsetted refrence database file.
* `vertebrate_tax.qza`
  * QIIME formated subsetted refrence database taxonomy file.
* `vertebrate_seq.qza`
  * QIIME formated subsetted refrence database sequence file.
  















