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
Workdir: `GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/`

Run the code inside extract_20kwindow.ipynb to produce 20kwindows_trichomes.fasta with 20k windows for all interesting snps for trichomes and branching


## Annotations in MAKER
### Round 1
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_1`


#### Create config files

```bash
maker -CTL 
```




#### Change option for first round (maker_opts.ctl)
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_1`

Options changed (maker_opts.ctl): 
```text
genome=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/20kwindows_trichomes.fasta #genome sequence (fasta file or fasta embeded in GFF3 file)


est=/home/carlos/GDRIVE/viburnumThings/Viburnum-Transcriptome/Assembly/withTrinity/Trinity.fasta #set of ESTs or assembled mRNA-seq in fasta format

est_pass=1 #use ESTs in maker_gff: 1 = yes, 0 = no

rm_pass=1 #use repeats in maker_gff: 1 = yes, 0 = no

model_org= #select a model organism for RepBase masking in RepeatMasker
rmlib=/home/carlos/GDRIVE/viburnumThings/Viburnum-Transcriptome/RepeatMasking_gv3/RM_17994.MonMay181404322020/consensi.fa.classified #provide an organism specific repeat library in fasta format for RepeatMasker

est2genome=1 #infer gene predictions directly from ESTs, 1 = yes, 0 = no            set to 1 so that MAKER gene predictions are based on the aligned transcripts

alt_splice=1 #Take extra steps to try and find alternative splicing, 1 = yes, 0 = no

TMP=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/TMP/ #specify a directory other than the system default temporary directory for temporary files

```

#### Run Maker
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/selectedScaffolds/round_1`

```bash 
maker
```


Note: took 3 hours


#### Checking results
First convert the maker output (a bunch of folders with hexadecimal names) into an unique gff3 and an unique fasta

```bash
gff3_merge -d 20kwindows_trichomes.maker.output/20kwindows_trichomes_master_datastore_index.log
fasta_merge -d 20kwindows_trichomes.maker.output/20kwindows_trichomes_master_datastore_index.log
```

To maintain everything in order, I renamed the outputs to put the round in the last element (separated by periods), for example `20kwindows_trichomes.all.round1.gff` is the gff after round 1. Same for fastas

```bash
rename 's/all./all.round1./g' 20kwindows_trichomes.all.*
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
./20kwindows_trichomes.all.round1.gff
```

gFACs results
```text

```


Using busco to check results (internal note, do it in busco enviroment):
```bash
busco -f -i 20kwindows_trichomes.all.round1.maker.transcripts.fasta -o busco_round1 -m geno -l embryophyta_odb10 --augustus_species arabidopsis -c 20
```

Busco results
```text

```




### Round 2
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_2`

#### Download and preparation of additional material
```bash
mkdir proteins_independent
cd proteins_independent
wget https://www.arabidopsis.org/download_files/Proteins/Araport11_protein_lists/Araport11_pep_20220103.gz
wget ftp://download.big.ac.cn/gwh/Plants/Lonicera_japonica_jyh_GWHAAZE00000000/GWHAAZE00000000.Protein.faa.gz

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

maker_gff=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_1/20kwindows_trichomes.all.round1.gff   #previous maker round

est_pass=0
altest_pass=1
protein_pass=1
rm_pass=0
model_pass=1
pred_pass=1
other_pass=1

est= #remove


altest=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_2/GWHAAZE00000000.RNA.fasta #EST/cDNA sequence file in fasta format from an alternate organism

altest_gff=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_2/GWHAAZE00000000.gff #aligned ESTs from a closly relate species in GFF3 format

protein=/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_2/L_japonica_A_thaliana.protein.faa  #protein sequence file in fasta format (i.e. from mutiple organisms)


rmlib= #remove

est2genome=0        
protein2genome=1     #set to 1 so that MAKER gene predictions are based on the aligned proteins


```

#### Run Maker
Workdir: `/home/carlos/GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_2/`

```bash 
maker
```

Note: lasted 3 hours


#### Checking results
First convert the maker output (a bunch of folders with hexadecimal names) into an unique gff3 and an unique fasta (be sure Workdir is `GDRIVE/viburnumThings/Viburnum-Annotation/MAKER/20kwindows_trichomes/round_2`)

```bash
gff3_merge -d 20kwindows_trichomes.maker.output/20kwindows_trichomes_master_datastore_index.log
fasta_merge -d 20kwindows_trichomes.maker.output/20kwindows_trichomes_master_datastore_index.log
```

To maintain everything in order, I renamed the outputs to put the round in the last element (separated by periods), for example `20kwindows_trichomes.all.round2.gff` is the gff after round 2. Same for fastas

```bash
rename 's/all./all.round2./g' 20kwindows_trichomes.all.*
```

# Functional annotation:

## Using Blast2GO

Project in: `C:\CAML\Test_FunctionalAnnotation\four_selected_all_round1_maker_proteins.b2g`

We used the GUI free version of this tool. Here are the general steps:
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












