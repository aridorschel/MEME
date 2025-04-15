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

*Copy the following commands in your terminal:*

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

**1c. Install MEME suite from source**

wget http://meme-suite.org/meme-software/5.4.1/meme-5.4.1.tar.gz
tar -xvzf meme-5.4.1.tar.gz
cd meme-5.4.1

