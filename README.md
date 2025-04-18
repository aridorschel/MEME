# Transcription factor motif enrichment analysis via MEME suite

Required data: e.g. ChIP-seq data, in bigwig and bed format
The bed file will need to be converted to fasta

Motif databases: e.g. JASPAR (different species), HOCOMOCO (human & mouse TFs), TRANSFAC

Paper describing MEME suite tools: [access here](https://pmc.ncbi.nlm.nih.gov/articles/PMC4489269/)

## 1. MEME installation on Mac (M1/M2, osx-arm64 architecture)

**1a. Install Homebrew (= package manager for macOS and Linux)**

In your terminal, copy this command:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```


**1b. Install MEME suite dependencies**

Copy the following commands in your terminal:

```bash
brew install gcc
brew install wget
brew install pkg-config
brew install cairo
brew install libxml2
brew install pcre

brew install imagemagick
brew install ghostscript

brew install coreutils
```

Check installations paths:

```bash
brew --prefix cairo
brew --prefix libxml2 
brew --prefix pcre
```

E.g. in my case, the installation paths were `/opt/homebrew/opt/cairo`, and same for libxml2

**1c. Install MEME suite from source**

```bash
wget http://meme-suite.org/meme-software/5.4.1/meme-5.4.1.tar.gz
tar -xvzf meme-5.4.1.tar.gz
cd meme-5.4.1
```

Configure the MEME suite build process, with the correct paths
`--prefix` specifies your installation directory, so this might have to be modified.
The $HOME environment variable (in Unix-based systems like macOS) refers to your home directory, e.g. in my case, /Users/dorschel
`--with-url` sets the URL for the MEME suite
`--with-libxml2` sets the path to the libxml2 library installed Homebrew, same is done for pcre and cairo.

```bash
./configure --prefix=$HOME/meme \
            --with-url=http://meme-suite.org/ \
            --with-libxml2=/opt/homebrew/opt/libxml2 \
            --with-pcre-dir=/opt/homebrew/opt/pcre \
            --with-cairo-dir=/opt/homebrew/opt/cairo
```

Now build and install the meme suite:
```bash
make
make test
sudo make install
```

Set environment variables (i.e. add the MEME suite bin directory to your path)
```bash
echo 'export PATH=/usr/local/meme/bin:$PATH' >> ~/.bash_profile
source ~/.bash_profile
```

Verify your installation:
```bash
meme -version
```

**1d. Install bedtools**

On Mac this is a bit tricky. Bedtools is not available for the ARM64 CPU architecture (`osx-arm64`) that new Apple Silicon (M1-M3) devices have. To work around this, we'll create a Conda environment that emulates the older Intel x86_64 architecture (`osx_64`), where Bedtools is available.

You can check your system architecture by using by running `uname -m` in your terminal. If the response is `arm64`, proceed to copy the following in your terminal:

```bash
CONDA_SUBDIR=osx-64 conda create -n bedtools_env bedtools
conda activate bedtools_env
export CONDA_SUBDIR=osx-64
bedtools --version
```

Alternatively (e.g. if the conda installation fails), try installing via Homebrew:
```bash
brew install bedtools
bedtools --version
```

**1e. Download JASPAR transcription factor database**

JASPAR is a database of known transcription factor motifs. There are different versions, the one we want is for - vertebrates - single batch file - non-redunant - MEME format.

Download the file as follows:
```bash
wget https://jaspar.genereg.net/download/data/2022/CORE/JASPAR2022_CORE_non-redundant_pfms_meme.txt
```

Note to self, my system locations (fetched via `find / -name “yourfile.extension” 2>/dev/null`):
```bash
/System/Volumes/Data/Users/dorschel/JASPAR2022_CORE_non-redundant_pfms_meme.txt
/System/Volumes/Data/Users/dorschel/meme-5.4.1/JASPAR2022_CORE_non-redundant_pfms_meme.txt
/Users/dorschel/JASPAR2022_CORE_non-redundant_pfms_meme.txt
/Users/dorschel/meme-5.4.1/JASPAR2022_CORE_non-redundant_pfms_meme.txt
```

**1f. Download hg19 reference genome**

We can download the hg19 reference genome from UCSC:
```bash
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/hg19.fa.gz
```

If this doesn't work, download from their FTP server:
```bash
curl -O ftp://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.fa.gz
```

Unzip the downloaded genome file, using gunzip:
```bash
gunzip hg19.fa.gz
```

Install samtools and index the genome:
```bash
brew install samtools
samtools faidx hg19.fa
```

Note to self, my hg19 genome file is located here:
```bash
/System/Volumes/Data/Users/dorschel/.local/share/genomes/hg19/hg19.fa
```

**1g. Prepare input files**

**Centering and adjusting regions to equal lengths**
MEME-ChIP expects all input sequences (i.e. the peak regions specified in your bed file) to be of consistent length. This prevents longer regions from skewing the analysis by providing more opportunities for random motif matches.

Consequently, we can adjust the input sequences to a uniform length. A common range seems to be around 150-200bp, but you can make this decision based on the summary statistics of your bed file. For example, in my ZFP57 ChIP-seq file (2372 peaks), the mean peak length is 272bp (SD+/- 181bp), and the median peak length is 223bp. 

The following command centers each peak of my ZFP57 bed file (add in your path here), keeps only the 200bp around the center of the peak, and also adjusts negative coordinates to zero, to handle boundary conditions.

```bash
awk 'BEGIN{OFS="\t"} {center=int(($2+$3)/2); start=center-100; end=center+100; if(start<0) start=0; print $1, start, end}' /Users/dorschel/Documents/ChIP_Resource/all_bed/ZFP57_n_vs_293T_TI_sampled_peaks_macs80.bed > ZFP57_n_vs_293T_TI_sampled_peaks_macs80_200bp.bed
```

**Convert bed file to FASTA**

We now need to convert our input BED file to FASTA files, using the genome we previously downloaded. This will allow the algorithm to actually scan the DNA sequence present in the genomic regions that our BED file specifies. Make sure to adjust the paths in this command.

```bash
bedtools getfasta -fi /path/to/hg19.fa -bed input_bed_file.bed -fo input_fasta_file.fa
```

`bedtools getfasta` is the command to extract FASTA sequences based on BED coordinates
`-fi hg19.fa` specifies the input reference genome file (hg19.fa)

Look at the converted fasta to ensure it is correctly generated:
```bash
head input_fasta_file.fa
```

**2. Motif discovery with MEME-ChIP**

There are several ways of doing motif enrichment analysis with MEME-Suite tools.
E.g.
- AME: To test whether known motifs are statistically enriched in your sequences compared to a background set
- FIMO: To find specific occurences of known motifs within your sequences.

MEME-ChIP runs the following workflow:
> Processing of your input sequence via getsize, fasta-most, fasta-center, fasta-get-markov
> MEME
> STREME (finds shorter motifs)
> CentriMo (finds centrally enriched motifs)
> Tomtom (compares discovered motifs to known motifs in a database)
> SpaMo (finds secondary motifs that are enriched in the vicinity of primary motifs)
> FIMO (finds occurrences of specific motifs)
> Conversion of TSV

Let's run MEME-ChIP. Here is an example command:

```bash
meme-chip -maxw 15 -meme-nmotifs 5 -db JASPAR2022_CORE_non-redundant_pfms_meme.txt ZFP57_n_peaks_sequences_200bp.fa -oc meme_chip_output
```



