# Scaffolding Bluehead genome with SSPACE

## Set up 

This was run on local server boros

**for latest version of samtools (1.6):**

`module load SAMTOOLS`

SSPACE variable used (in my .bashrc, but just a preference):

`SSPACE="/usr/local/SSPACE-STANDARD-3.0_linux-x86_64/SSPACE_Standard_v3.0.pl"`


## Mapping and converting sam files to table format

a nice option in SSPACE is that allows us to map the sequences to the genome first, then convert the same files to simple tab-delimited files to input to SSPACE. This saves a lot  of time (otherwise SSPACE would map all reads first), and allows us to choose the mapper we want.

### Insert libraries

#### mapping first using bwa:

```
bwa mem -t 16 /data/bluehead_scaffold/data/bluehead_350bp_assembly_sept2016.fasta blue_350bp_pe_1.fq blue_350bp_pe_2.fq > blue_350bp_insert.bam
bwa mem -t 16 /data/bluehead_scaffold/data/bluehead_350bp_assembly_sept2016.fasta blue_550bp_pe_1.fq blue_550bp_pe_2.fq > blue_550bp_insert.bam
```

In order for SSPACE table conversion to work, bam files must be sorted by name, then output to sam:
```
samtools sort -@ 16 -n blue_350bp_insert.bam -O sam -T tmp > sortN_blue_350bp_insert.sam
samtools sort -@ 16 -n blue_550bp_insert.bam -O sam -T tmp > sortN_blue_550bp_insert.sam
```

#### use SSPACE perl script to convert to tables

**NOTE: the bwa mapping seems to remove the /1 and /2 from read names, which results in a warning from SSPACE table conversion**

I wrote a script to add back the /1 and /2. However, the conversion still worked, and resulting tab files were only slightly different. I used the converted sam files.

`convert_sam_readnames.py `

SSPACE perl script to convert sam files to tables: 

```
perl /usr/local/SSPACE-STANDARD-3.0_linux-x86_64/tools/sam_bam2tab.pl mod_sortN_blue_350bp_insert.sam /1 /2 blue_350bp_insert.tab
perl /usr/local/SSPACE-STANDARD-3.0_linux-x86_64/tools/sam_bam2tab.pl mod_sortN_blue_550bp_insert.sam /1 /2 blue_550bp_insert.tab
```

### Mate pair libraries

#### Merge bam files

mate pairs were merged (trimmed and mapped with Nextclip software)

original bam files are here, check Nextclip documentation for details:

**/scratch/bluehead_wrasse/nextclip/align_to_genome**

merge bam files, sorting by name:
```
samtools merge -n -@ 12 -b mates6k_list /data/bluehead_scaffold/trim_data/sortN_blue_6k_mate_pair.bam
samtools merge -n -@ 12 -b mates8k_list /data/bluehead_scaffold/trim_data/sortN_blue_8k_mate_pair.bam
```

convert to sam (should already be sorted by name):
```
samtools view -@ 8 -h sortN_blue_6k_mate_pair.bam > sortN_blue_6k_mate_pair.sam
samtools view -@ 8 -h sortN_blue_8k_mate_pair.bam > sortN_blue_8k_mate_pair.sam
```

convert to tab
```
perl /usr/local/SSPACE-STANDARD-3.0_linux-x86_64/tools/sam_bam2tab.pl mod_sortN_blue_6k_mate_pair.sam /1 /2 blue_6k_mate_pair.tab
perl /usr/local/SSPACE-STANDARD-3.0_linux-x86_64/tools/sam_bam2tab.pl mod_sortN_blue_8k_mate_pair.sam /1 /2 blue_8k_mate_pair.tab
```

## RUNNING SSPACE


### Library file

I have included several examples from trial runs, but here is the one I used

*blue_scaf1_lib*

(Also see SSPACE documentation )

### Running (finally!)

Another nice feature of SSPACE is that it allows you to rerun the same library file using different parameters, without having to remap (unpaired reads cannot be pre-mapped).

The first run used the following command:
```
$SSPACE -l blue_scaf1_lib -s \
  /data/bluehead_scaffold/data/bluehead_350bp_assembly_sept2016.fasta \
  -x 1 -T 12 -b blue_scaf1 -v 1 -p 1
```

Subsequent runs can be done on original data using -S 1 option

I reran this a few times with different parameters, Just moving the output files to a new folder so they were not overwritten. Here is the final command used for downstream gapfilling.
```
$SSPACE -l blue_scaf1_lib -s \
 /data/bluehead_scaffold/data/bluehead_350bp_assembly_sept2016.fasta \
 -x 1 -T 12 -b blue_scaf1 -v 1 -S 1 -n 15 -k 3 -o 10 -r 0.8 -a 0.7 -m 50
```

There are many output files, but here are log and summary files from first run:

* *first_run_blue_scaf1.logfile.txt*

* *first_run_blue_scaf1.summaryfile.txt*

And here from last run:

* *last_run_blue_scaf1.logfile.txt*

* *last_run_blue_scaf1.summaryfile.txt*


## Gapfilling

The program GapFiller (Boetzer & Pirovano, 2012) was used to close gaps in scaffolds produced by SSPACE. As with scaffolding, all libraries were used, including both insert sizes and mate pairs. The specific libraries used are found in the library file (*gapfill\_lib1*), and the parameters and results are detailed in the GapFiller summary output file (*blue_gapfill1.summaryfile.final.txt*)



