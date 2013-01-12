Wessim
=======

Wessim: A targeted resequencing (exome sequencing) simulator

For a manual for installation, preparation and usage, please go to http://sak042.github.com/Wessim/

### Introduction
**Wessim** is a simulator for a targeted resequencing as generally known as exome sequencing. Wessim basically generates a set of *artificial* DNA fragments for next generation sequencing (NGS) read simulation. In the targeted resequencing, we constraint the genomic regions that are used to generated DNA fragments to be only a part of the entire genome; they are usually exons and/or a few introns and untranslated regions (UTRs).

### Install Wessim
Download Wessim using the links in this page, or go to https://github.com/sak042/Wessim   
To run Wessim, Python 2.7 or later is required. To install Python, go to http://python.org/

### Requirements
The following programs are required to run Wessim or to prepare input files:
* **pysam** library: go to http://code.google.com/p/pysam/ to install pysam
* **numpy** library: go to http://numpy.scipy.org/ to install numpy
* **gfServer** and **gfClient**: In probe hybridization mode, Wessim runs more than 100,000 queries again the reference genome. This essentially requires a local blat server. gfServer and gfClient are pre-compiled programs for establishing private blat server on your computer. go to http://hgdownload.cse.ucsc.edu/admin/exe/ to download gfServer and gfClient (and set your local path to access the two programs anywhere). For more details about the tools, please refer to http://genome.ucsc.edu/FAQ/FAQblat.html#blat5
* **faToTwoBit**: go to http://hgdownload.cse.ucsc.edu/admin/exe/ and download faToTwoBit. This is required to convert your FASTA file to .2bit 
* **samtools**: samtools is needed to index your sample genome FASTA file (samtools faidx).
* **GemSim** error models: Wessim uses GemSim's empirical error models for NGS read generation. Go to GemSim's project page (http://sourceforge.net/projects/gemsim/) to download GemSim. You will find several model files (e.g. ill100v4_p.gzip) under 'models' directory. Save them and remember their location.
 
### Preparing Input Files 
Wessim requires two major inputs. One is the sample genome sequence, and the other is the target region information.
* **Sample genome sequence**: This is a FASTA file (e.g. ref.fa). You will need to index the file and generate .2bit
<pre><code>
>samtools faidx ref.fa
>faToTwoBit ref.fa ref.2bit
</code></pre>
* **Target region information**: Target regions can be specified by two different ways. 
    1. **Ideal targets**: In ideal target mode, you will provide a list of genomic coordinates in a BED  file (e.g. chr1   798833 799125). Ideal targets of major exome capture platforms are freely available from vendor's website. For Agilent's SureSelect platforms, go to https://earray.chem.agilent.com/suredesign/ . You must register at their site. After logging in, go to Find Designs and select Agilent Catalog at the menu tab. You will be able to download all information of currently available platforms including ideal target BED files and probe sequence text files.   For NimbleGen's SeqCap go to http://www.nimblegen.com/products/seqcap/index.html and find BED files under Design and Annotation Files. 
    2. **Probe sequences**: Probe sequences are available for SureSelect platforms in the SureDesign homepage (https://earray.chem.agilent.com/suredesign/) (see above). Usually those files are named "[platform]_probe.txt"

### Running Wessim
The basic synopsis of Wessim1 is like below:
<pre><code>
# Run Wessim1 in ideal target mode
>python Wessim1.py -R ref.fa -B target.bed -n 1000000 -l 100 -M model.gzip -z -o result -t 4
</code></pre> 
This will generate *result.fastq.gz* (single-end mode / gzip compressed) using 4 threads (CPU cores).

For Wessim2:
<pre><code>
# Generate a FASTA file of probe sequence
>python Prep_Probe2Fa.py probe.txt (this generates probe.txt.fa)
# Establish your local blat server
>gfServer start -canStop localhost 6666 ref.2bit
# Run blat search to generate the match list
>python Prep_BlatSearch.py ref.2bit probe.txt.fa probe.txt.fa.psl
# Run Wessim2 in probe hybridization mode.
>python Wessim2.py -R ref.fa -P probe.txt.fa -B probe_match.txt.fa.psl -n 1000000 -l 76 -M model.gzip -pz -o result
</code></pre>
This will generate *result_1.fastq.gz* and *result_2.fastq.gz* (paired-end mode / gzip compressed).

### Wessim Options
You can use '-h' for detailed help in command line.

```
Mandatory input files (for Wessim1 and Wessim2 in common):
  -R FILE     faidx-indexed (R)eference genome FASTA file
For Wessim1 only:
  -B FILE     Target region .(B)ED file
For Wessim2 only:
  -P FILE     (P)robe sequence FASTA file
  -B FILE     (B)lat matched probe regions .PSL file

Parameters for exome capture:
  -f INT      mean (f)ragment size. this corresponds to insert size when sequencing in paired-end mode. [200]
  -d INT      standard (d)eviation of fragment size [50]
  -m INT      (m)inimum fragment length [read_length + 20]
  -x INT      slack margin of the given boundaries [0] (only for Wessim1)

Parameters for sequencing:
  -p          generate paired-end reads [single]
  -n INT      total (n)umber of reads
  -l INT      read (l)ength (bp)
  -M FILE     GemSim (M)odel file (.gzip)
  -t INT      number of (t)hreaded subprocesses [1]

Output options:
  -o FILE     (o)utput file header. ".fastq.gz" or ".fastq" will be attached automatically. Output will be splitted into two files in paired-end mode
  -z          compress output with g(z)ip [false]
  -q INT      (q)uality score offset [33]
  -v          (v)erbose; print out intermediate messages.
```

### Support or Contact
For GitHub use, check out the documentation at http://help.github.com/pages or contact support@github.com and we’ll help you sort it out.
