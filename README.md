# Kraken Tools
For news and updates, refer to the github page: https://github.com/jenniferlu717/KrakenTools/

KrakenTools is a suite of scripts to be used for post-analysis of 
Kraken/KrakenUniq/Kraken2/Bracken results. Please cite the relevant paper
if using KrakenTools with any of the listed programs. 

# Running Scripts:
No installation required. 
All scripts are run on the command line as described.

Users can make scripts executable by running

    chmod +x myscript.py
    ./myscript.py -h 

## extract\_kraken\_reads.py

This program extract reads classified at any user-specified taxonomy IDs. User
must specify the Kraken output file, the sequence file(s), and at least one
taxonomy ID. Additional options are specified below.

1. USAGE/OPTIONS
    
    `python extract_kraken_reads.py`
    *   `-k, --kraken MYFILE.KRAKEN.............`Kraken output file
    *   `-s, -s1, -1, -U SEQUENCE.FILE..........`FASTA/FASTQ sequence file (may be gzipped)
    *   `-s2, -2 SEQUENCE2.FILE.................`FASTA/FASTQ sequence file (for paired reads, may be gzipped)
    *   `-o, --output OUTPUT.FASTA..............`output FASTA file with extracted seqs
    *   `-t, --taxid TID TID2 etc...............`list of taxonomy IDs to extract (separated by spaces)        

    Optional:
    *   `-r, --report MYFILE.KREPORT.............`Kraken report file (required if specifying --include-children or --include-parents)
    *   `--include-children......................`include reads classified at more specific levels than specified taxonomy ID levels. 
    *   `--include-parents.......................`include reads classified at all taxonomy levels between root and the specified taxonomy ID levels.
    *   `--max #.................................`maximum number of reads to save.
    *   `--append................................`if output file exists, appends reads
    *   `--noappend..............................`[default] rewrites existing output file
    
2. INPUT FILES: SEQUENCE FILES

    Input sequence files must be either FASTQ or FASTA files. Input files
    can be gzipped or not. The program will automatically detect whether
    the file is gzipped and whether it is FASTQ or FASTA formatted based on
    the first character in the file (">" for FASTA, "@" for FASTQ)

3. PAIRED INPUT/OUTPUT
    
    Users that ran Kraken using paired reads should input both read files into
    extract_kraken_reads.py as follows:
        
    `extract_kraken_reads.py -k myfile.kraken -s1 read1.fq -s2 reads2.fq`
    
    By default, extracted paired reads will be output concatenated with 'N'.
    However, users have the option to specify a different delimiter using the
    `--delimiter` or `-d` option:
    
    `extract_kraken_reads.py -k myfile.kraken -s1 read1.fq -s2 reads2.fq -d X`
    
    Users may also get non-concatenated reads by specifying a second output
    file. If a 2nd output file is provided, any delimiter will be ignored and
    the individual pairs will be preserved in two separate files:

    `extract_kraken_reads.py -k myfile.kraken ... -o reads_S1.fa -o2 reads_s2.fa`

4. `--include-parents`/`--include-children` flags
    
    By default, only reads classified exactly at the specified taxonomy IDs
    will be extracted. Options --include-children and --include parents can be
    used to extract reads classified within the same lineage as a specified
    taxonomy ID. For example, given a Kraken report containing the following:

        [%]     [reads] [lreads][lvl]   [tid]       [name]
        100     1000    0       R       1           root
        100     1000    0       R1      131567        cellular organisms
        100     1000    50      D       2               Bacteria
        0.95    950     0       P       1224              Proteobacteria
        0.95    950     0       C       1236                Gammaproteobacteria
        0.95    950     0       O       91347                 Enterobacterales
        0.95    950     0       F       543                     Enterobacteriaceae
        0.95    950     0       G       561                       Escherichia
        0.95    950     850     S       562                         Escherichia coli
        0.05    50      50      S1      498388                        Escherichia coli C
        0.05    50      50      S1      316401                        Escherichia coli ETEC

    
    1.  `extract_kraken_reads.py  [options] -t 562` ==> 850 reads classified as _E. coli_ will be extracted
    2.  `extract_kraken_reads.py  [options] -t 562 --include-parents` ==> 900 reads classified as _E. coli_ or Bacteria will be extracted
    3.  `extract_kraken_reads.py  [options] -t 562 --include-children` ==> 950 reads classified as _E. coli_, _E. coli C_, or _E. coli ETEC_ will be extracted
    4.  `extract_kraken_reads.py  [options] -t 498388` ==> 50 reads classified as _E. coli C_ will be extracted
    5.  `extract_kraken_reads.py  [options] -t 498388 --include-parents` ==> 950 reads classified as _E. coli C_, _E. coli_, or Bacteria will be extracted
    6.  `extract_kraken_reads.py  [options] -t 1 --include-children` ==> All classified reads will be extracted 

## combine\_kreports.py 

This script combines multiple Kraken reports into a combined report file.

1. USAGE/OPTIONS
    
    `python complete_kreports.py`
    *    `-r 1.KREPORT 2.KREPORT........................`Kraken-style reports to combine 
    *    `-o COMBINED.KREPORT...........................`Output file 

    Optional:
    *   `--display-headers..............................`include headers describing the samples and columns [all headers start with #]
    *   `--no-headers...................................`do not include headers in output
    *   `--sample-names.................................`give abbreviated names for each sample [default: S1, S2, ... etc]
    *   `--only-combined................................`output uses exact same columns as a single Kraken-style report file. Only total numbers for read counts and percentages will be used. Reads from individual reports will not be included.

2. OUTPUT 
    Percentage is only reported for the summed read counts, not for each individual sample. 

    The output file therefore contains the following tab-delimited columns:
    *    `perc............`percentage of total reads rooted at this clade 
    *    `tot_all ........`total reads rooted at this clade (including reads at more specific clades) 
    *    `tot_lvl.........`total reads at this clade  (not including reads at more specific clades)
    *    `1_all...........`reads from Sample 1 rooted at this clade 
    *    `1_lvl...........`reads from Sample 1 at this clade 
    *    `2_all...........`""
    *    `2_lvl...........`""
    *    etc..
    *    `lvl_type........`Clade level type (R, D, P, C, O, F, G, S....) 
    *    `taxid...........`taxonomy ID of this clade
    *    `name............`name of this clade 

## kreport2krona.py 

This program takes a Kraken report file and prints out a krona-compatible TEXT file

1. USAGE/OPTIONS
    
    `python kreport2krona.py`
    *    `-r/--report MYFILE.KREPORT........`Kraken report file 
    *    `-o/--output MYFILE.KRONA..........`Output Krona text file
    
    Optional:
    *    `--no-intermediate-ranks...........`only output standard levels [D,P,C,O,F,G,S] 
    *    `--intermediate-ranks..............`[default] include non-standard levels

2. EXAMPLE USAGE 
    
        kraken2 --db KRAKEN2DB --threads THREADNUM --report MYSAMPLE.KREPORT \
            --paired SAMPLE_1.FASTA SAMPLE_2.FASTA > MYSAMPLE.KRAKEN2
        python kreport2krona.py -r MYSAMPLE.KREPORT -o MYSAMPLE.krona 
        ktImportText MYSAMPLE.krona -o MYSAMPLE.krona.html
    
    Krona information: see https://github.com/marbl/Krona. 

## filter\_bracken\_out.py

This program takes the output file of a Bracken report and filters the desired taxonomy IDs. 

1. USAGE/OPTIONS
    
    `python filter_bracken_out.py`
    *   `-i/--input MYFILE.BRACKEN..........`Bracken output file
    *   `-o/--output MYFILE.BRACKEN_NEW.....`Bracken-style output file with filtered taxids
    *   `--include TID TID2.................`taxonomy IDs to include in output file [space-delimited]
    *   `--exclude TID TID2.................`taxonomy IDs to exclude in output file [space-delimited]

    User should specify either taxonomy IDs with `--include` or `--exclude`. If
    both are specified, taxonomy IDs should not be in both lists and only
    taxonomies to include will be evaluated. 
    
    When specifying the --include flag, only lines for the included taxonomy
    IDs will be extracted to the filtered output file. The percentages in the
    filtered file will be re-calculated so the total percentage in the output 
    file will sum to 100%. 

    When specifying the --exclude flag alone, all lines in the Bracken file
    will be preserved EXCEPT for the lines matching taxonomy IDs provided. 
    
2. EXAMPLE USAGE

    This program can be useful for isolating a subset of species to 
    better understand the distribution of those particular species in the sample. 
    
    For example:

    * `python filter_bracken_out.py [options] --include 1764 1769 1773 1781
      39689` will allow users to get the relative percentages of _Mycobacterium
      avium, marinum, tuberculosis, leprae, and gallinarum_ in their samples.

    In other cases, users may want to focus on the distribution of all species
    that are NOT the host species in a given sample. This program can then
    recalculate percentage distributions for species when excluding reads for
    the host.
    
    For example, given this output:
        
        name                     tax_id      tax_lvl     kraken....  added...   new.... fraction...
        Homo sapiens             9606        S           ...         ....       999000  0.999000
        Streptococcus pyogenes   1314        S           ...         ....       10      0.000001
        Streptococcus agalactiae 1311        S           ...         ....       5       0.000000
        Streptococcus pneumoniae 1313        S           ...         ....       3       0.000000
        Bordetella pertussis     520         S           ...         ....       20      0.000002
        ...
    
    Users may not be interested in the 999,000 reads that are host DNA, but
    would rather know the percentage of non-host reads for each of the non-host
    species.  Using `python filter_bracken_out.py [options] --exclude 9606`
    allows better resolution of the non-host species, allowing each of the
    fraction of reads to be recalculated out of 1,000 instead of 1,000,000
    reads in the above example. The output would then be:

        name                     tax_id      tax_lvl     kraken....  added...   new.... fraction...
        Streptococcus pyogenes   1314        S           ...         ....       10      0.01000
        Streptococcus agalactiae 1311        S           ...         ....       5       0.05000
        Streptococcus pneumoniae 1313        S           ...         ....       3       0.03000
        Bordetella pertussis     520         S           ...         ....       200     0.20000
        ...
    
 
