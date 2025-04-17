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

