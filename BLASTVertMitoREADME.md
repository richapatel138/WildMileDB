# WildMileDB - Veterbrate Mitochondrial BLAST
This repository serves to explain how to build reference databases using CRABS for Illinois vertebrate and mussel species, with the generated databases attached along with other helpful files. This specific README files focuses on how to curate the vertebrate database.

This database is different than the others as we are not subsetting species, but rather using CRABS as a tool to download all vertebrate mitochondiral RefSeq sequences, which we can then use for the last step of broad BLAST assignment. This is an alternative to searching unassigned sequences against all of the nucleotide BLAST database as that can be computationally expensive to download the database locally. By just downloading the vertebrate mitochondiral RefSeq sequences, we still allow for a broader pool, but it doesn't take as much time and memory to create the database. 

## Creating Vertebrtae Mitochondrial RefSeq Database Using CRABS

To create the reference database, we will use [CRABS](https://github.com/gjeunen/reference_database_creator). The CRABS GitHub has great instructions for installation as well as different uses. Below I have outlined the commands I used. 

1. The first step is to download taxonomic information files -- these are needed to assign taxonomic lineage to each downloaded sequence in the reference database.
```
mkdir crabs_tax
crabs --download-taxonomy --output crabs_tax
```

2. Download all vertebrate mitochondrial refseq sequences from NCBI
```
crabs --download-ncbi --query 'mitochondrion[filter] AND refseq[filter] AND txid7742[Organism:exp]' --output ncbi_mito_refseq.fasta --email rpatel@luc.edu --database nucleotide
```

3. Import it into CRABS format
```
crabs --import --import-format NCBI --input ncbi_mito_refseq.fasta --names crabs_tax/names.dmp --nodes crabs_tax/nodes.dmp --acc2tax crabs_tax/nucl_gb.accession2taxid --output vert_mito.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
```

4. Since these sequences are used for general blast, we do not need to trim/filter them, we will just export them into QIIME format and use them like that. 
```
crabs --export --input vert_mito.txt --output mito_refseq_seq.fasta --export-format 'qiime-fasta'
crabs --export --input vert_mito.txt --output mito_refseq_tax.txt --export-format 'qiime-text'
```

5. Get into QIIME format. You will need to deactivate CRABS and activate the QIIME enviroment.
```
qiime tools import --type 'FeatureData[Sequence]' --input-path mito_refseq_seq.fasta --output-path mito_refseq_seq.qza
qiime tools import --type 'FeatureData[Taxonomy]' --input-format HeaderlessTSVTaxonomyFormat --input-path mito_refseq_tax.txt --output-path mito_refseq_tax.qza
```

## Useful Files

In the BLASTVertMito folder, I have included some files that might be helpful. 

* `vert_mito.txt`
  * Full downloaded NCBI sequences for vertebrate m,itochondrial RefSeq sequences in CRABS format. 
* `mito_refseq_tax.qza`
  * QIIME formated reference database taxonomy file.
* `mito_refseq_seq.qza`
  * QIIME formated reference database sequence file.
  


