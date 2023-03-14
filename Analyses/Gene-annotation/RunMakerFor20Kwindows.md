# Maker annotation 2021 - with L50 reduced genome 

For reference and guidance:
https://bioinformaticsworkbook.org/dataAnalysis/GenomeAnnotation/Intro_To_Maker.html#gsc.tab=0
https://github.com/Joseph7e/MAKER-genome-annotations-tutorial

Note: some big files are excluded from the repository due to size limitation in Github

## Operational notes (for Sacra):
Two different enviornments were deployed for this task: maker and busco

For new maker environments, should be added the following paths to the PATH, and one variable to avoid mpi error:
```bash
conda activate maker
export PATH=/home/carlos/tools/EVidenceModeler-1.1.1:$PATH
export PATH=/home/carlos/gmes_linux_64/:$PATH
export PATH=/home/carlos/maker/bin/:$PATH
export LD_PRELOAD=/usr/lib/openmpi/lib/libmpi.so
OMPI_MCA_mpi_warn_on_fork=0
```

All buscos should be performed in busco enviorment.


## Reduce genome
Workdir: `GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/`

Run the code inside extract_20kwindow.ipynb to produce 20kwindows.fasta with 20k windows for all 36 interesting snps


## Annotations in MAKER
### Round 1
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_1`


#### Create config files

```bash
maker -CTL 
```




#### Change option for first round (maker_opts.ctl)
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_1`

Options changed (maker_opts.ctl): 
```text
genome=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/20kwindows.fasta #genome sequence (fasta file or fasta embeded in GFF3 file)


est=/home/carlos/GDRIVE/viburnumThings/Viburnum-Transcriptome/Assembly/withTrinity/Trinity.fasta #set of ESTs or assembled mRNA-seq in fasta format

est_pass=1 #use ESTs in maker_gff: 1 = yes, 0 = no

rm_pass=1 #use repeats in maker_gff: 1 = yes, 0 = no

model_org= #select a model organism for RepBase masking in RepeatMasker
rmlib=/home/carlos/GDRIVE/viburnumThings/Viburnum-Transcriptome/RepeatMasking_gv3/RM_17994.MonMay181404322020/consensi.fa.classified #provide an organism specific repeat library in fasta format for RepeatMasker

est2genome=1 #infer gene predictions directly from ESTs, 1 = yes, 0 = no            set to 1 so that MAKER gene predictions are based on the aligned transcripts

alt_splice=1 #Take extra steps to try and find alternative splicing, 1 = yes, 0 = no

TMP=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/TMP/ #specify a directory other than the system default temporary directory for temporary files

```

#### Run Maker
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/selectedScaffolds/round_1`

```bash 
maker
```


#### Checking results
First convert the maker output (a bunch of folders with hexadecimal names) into an unique gff3 and an unique fasta

```bash
gff3_merge -d 20kwindows.maker.output/20kwindows_master_datastore_index.log
fasta_merge -d 20kwindows.maker.output/20kwindows_master_datastore_index.log
```

To maintain everything in order, I renamed the outputs to put the round in the last element (separated by periods), for example `20kwindows.all.round1.gff` is the gff after round 1. Same for fastas

```bash
rename 's/all./all.round1./g' 20kwindows.all.*
```

Using gFACs I get the stats (internal note: not in Maker enviornment to avoid perl corruption with different requirements of gFACs,just use base)
```bash
mkdir gFACs

~/gFACs-master/gFACs.pl \
-f maker_2.31.9_gff \
-p round_1 \
--statistics \
--statistics-at-every-step \
--rem-5prime-3prime-incompletes \
--rem-all-incompletes \
--unique-genes-only \
-O ./gFACs/ \
./20kwindows.all.round1.gff
```

gFACs results
```text
STATS:
Number of genes:        25
Number of monoexonic genes:
Number of multiexonic genes:    25
```


Using busco to check results (internal note, do it in busco enviroment):
```bash
busco -f -i 20kwindows.all.round1.maker.transcripts.fasta -o busco_round1 -m geno -l embryophyta_odb10 --augustus_species arabidopsis -c 20
```

Busco results
```text
        --------------------------------------------------
        |Results from dataset embryophyta_odb10           |
        --------------------------------------------------
        |C:0.1%[S:0.1%,D:0.0%],F:0.1%,M:99.8%,n:1614      |
        |1      Complete BUSCOs (C)                       |
        |1      Complete and single-copy BUSCOs (S)       |
        |0      Complete and duplicated BUSCOs (D)        |
        |1      Fragmented BUSCOs (F)                     |
        |1612   Missing BUSCOs (M)                        |
        |1614   Total BUSCO groups searched               |
        --------------------------------------------------
```




### Round 2
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_2`

#### Download and preparation of additional material
```bash
mkdir proteins_independent
cd proteins_independent
wget https://www.arabidopsis.org/download_files/Proteins/Araport11_protein_lists/Araport11_pep_20220103.gz
wget ftp://download.big.ac.cn/gwh/Plants/Lonicera_japonica_jyh_GWHAAZE00000000/we use.Protein.faa.gz

gunzip *

#concatenate proteins from L japonica and A thaliana in a single fasta.
#cat Araport11_genes.201606.pep.fasta GWHAAZE00000000.Protein.faa > ../L_japonica_A_thaliana.protein.faa
cat * > ../L_japonica_A_thaliana.protein.faa


cd .. 
wget ftp://download.big.ac.cn/gwh/Plants/Lonicera_japonica_jyh_GWHAAZE00000000/GWHAAZE00000000.gff.gz

wget ftp://download.big.ac.cn/gwh/Plants/Lonicera_japonica_jyh_GWHAAZE00000000/GWHAAZE00000000.RNA.fasta.gz

gunzip *.gz


# copy ctl files from previous round
cp ../round_1/*.ctl ./

```


#### Options changed (maker_opts.ctl): 
```text

maker_gff=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_1/20kwindows.all.round1.gff   #previous maker round

est_pass=0
altest_pass=1
protein_pass=1
rm_pass=0
model_pass=1
pred_pass=1
other_pass=1

est= #remove


altest=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_2/GWHAAZE00000000.RNA.fasta #EST/cDNA sequence file in fasta format from an alternate organism

altest_gff=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_2/GWHAAZE00000000.gff #aligned ESTs from a closly relate species in GFF3 format

protein=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_2/L_japonica_A_thaliana.protein.faa  #protein sequence file in fasta format (i.e. from mutiple organisms)


rmlib= #remove

est2genome=0        
protein2genome=1     #set to 1 so that MAKER gene predictions are based on the aligned proteins


```

#### Run Maker
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_2/`

```bash 
maker
```

Note: lasted 3 hours


#### Checking results
First convert the maker output (a bunch of folders with hexadecimal names) into an unique gff3 and an unique fasta (be sure Workdir is `GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows/round_2`)

```bash
gff3_merge -d 20kwindows.maker.output/20kwindows_master_datastore_index.log
fasta_merge -d 20kwindows.maker.output/20kwindows_master_datastore_index.log
```

To maintain everything in order, I renamed the outputs to put the round in the last element (separated by periods), for example `20kwindows.all.round2.gff` is the gff after round 2. Same for fastas

```bash
rename 's/all./all.round2./g' 20kwindows.all.*
```

Using gFACs I get the stats (internal note: not in Maker enviornment to avoid perl corruption with different requirements of gFACs)
```bash
mkdir gFACs

~/gFACs-master/gFACs.pl \
-f maker_2.31.9_gff \
-p round_2 \
--statistics \
--statistics-at-every-step \
--rem-5prime-3prime-incompletes \
--rem-all-incompletes \
--unique-genes-only \
-O ./gFACs/ \
./20kwindows.all.round2.gff
```


gFACs results
```text
STATS:
Number of genes:        43
Number of monoexonic genes:     12
Number of multiexonic genes:    31
```


Checking results using BUSCO
```bash
busco -f -i 20kwindows.all.round2.maker.transcripts.fasta -o busco_round2 -m geno -l embryophyta_odb10 --augustus_species arabidopsis -c 20
```

Busco results
```text
        --------------------------------------------------
        |Results from dataset embryophyta_odb10           |
        --------------------------------------------------
        |C:0.1%[S:0.1%,D:0.0%],F:0.1%,M:99.8%,n:1614      |
        |2      Complete BUSCOs (C)                       |
        |2      Complete and single-copy BUSCOs (S)       |
        |0      Complete and duplicated BUSCOs (D)        |
        |2      Fragmented BUSCOs (F)                     |
        |1610   Missing BUSCOs (M)                        |
        |1614   Total BUSCO groups searched               |
        --------------------------------------------------
```




# Functional annotation:


>>>>> IMPORTANT: THIS PART WAS OVERWRITTEN BY BLAST2GO RUN

We can use  InterProScan (a domain finder) ran on the MAKER proteins and a BLAST report of homology of the MAKER proteins to UniProt/Swiss-Pro to get standarized names of some genes. (Important for assign GO terms)

Some people reported using Blast2GO v5.5.1 (Conesa et al. 2005; Götz et al. 2008) against the NCBI non-redundant [organism] protein database to get a functional annotation, however it needes a licence (now named OmicsBox). I will try the free version, unfortunately it is not the CLI version. And also I will check others like: DAVID GO and  UniProt-GOA DAS.

This can be done with any round, in theory the last round is the best but as I noticed in second round, this cannot be the case sometime.


## Installation
Blast is already installed by Maker so next is interproscan

First try to install InterProScan in my tools folder
```bash
#create dirs
mkdir my_interproscan
cd my_interproscan

#download software
wget https://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/5.54-87.0/interproscan-5.54-87.0-64-bit.tar.gz
wget https://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/5.54-87.0/interproscan-5.54-87.0-64-bit.tar.gz.md5

# verify md5
md5sum -c interproscan-5.54-87.0-64-bit.tar.gz.md5

# extract
tar -pxvzf interproscan-5.54-87.0-*-bit.tar.gz

# "install it"
cd interproscan-5.54-87.0/
./initial_setup.py

```

## BLASTP
Let's try first Blastp with round 1


Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/selectedScaffolds/round_1`

```bash
mkdir blastp
cd blastp
wget https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz
gunzip *.gz

#create index for blast search
makeblastdb -in uniprot_sprot.fasta -dbtype prot

#do the blastp
blastp -query ../four_selected.all.round1.maker.proteins.fasta -db uniprot_sprot.fasta -evalue 1e-6 -max_hsps 1 -max_target_seqs 1 -outfmt 6 -out output.blastp
```

## InterProScan
Now do InterProScan with round 1
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/selectedScaffolds/round_1`

```bash
mkdir interproscan
cd interproscan

interproscan.sh -appl pfam -dp -f TSV -goterms -iprlookup -pa -t p -i ../four_selected.all.round1.maker.proteins.fasta -o output.iprscan
``` 

>>>>>>>>> END OF IMPORTANT NOTE

## Using Blast2GO

Project in: C:\CAML\Test_FunctionalAnnotation\four_selected_all_round1_maker_proteins.b2g

I used the GUI free version of this tool. I am describing here the steps:
1. Download from https://ftp.ncbi.nlm.nih.gov/blast/db/ the `swissprot.tar.gz` database
2. Use `four_selected.all.round1.maker.proteins.fasta`
3. in Blast2GO:
- blast: local blastp using swissprot as db, 1e5 the expected e-value.
- mapping: all by default
- annotation: all by default
- interpro with all by default
- merge goslim 
- 

### Explainations following Blast2GO docs
blast: Use blast to identify similar, potentially homolog sequences from a large number of public protein databases. Choose one of the available subsets (plants, mammals, etc.) of the giant NR protein database to speed up your search. NCBI Blast can be performed in our cloud, via the NCBI web-services, locally or the Amazon web-services (AWS).
mapping: The Gene Ontology mapping is the step which matches all IDs of the similar sequences obtained in the previous step against the Gene Ontology Annotation database. This step has no parameters and your sequences will turn green if functional information could be retrieved.
interpro: This step can be executed in parallel to the blast step. It allows to identify protein domains and families which form part of the InterPro collection of databases. The InterPro classification is linked to the Gene Ontology which allows to directly identify biological functions based on domains and families. These functions (GO terms) can now be merged to the ones assigned previously.
annotation: This step evaluates the information obtained during the blast and the mapping step and assigns the most reliable gene ontology (GO) terms to the input sequences taking into account the GO hierarchy, sequences similarities as well as the abundance and quality of the source annotation. Finally the most specific GO terms above the user defined annotation cutoff will be assigned in a non-redundant way.
goslim: The goslim reduction is a step which allows to functionally summarise a sequence dataset in a uniform and species-specific way. The Gene Ontology is a structured vocabulary of around 40.000 terms with different levels of specificity. GOSlim abstracts specific GO terms to a more general level and allows in this way to summarise a dataset to a fixed subset of relevant functions. Comparisons of different datasets can also be done more easily in this way.











