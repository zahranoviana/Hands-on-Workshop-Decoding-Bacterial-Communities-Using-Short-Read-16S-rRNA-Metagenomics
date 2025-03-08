# 🎯 Hands-on Workshop: Decoding Bacterial Communities Using Short-Read 16S rRNA Metagenomics

## 📅 Date: March 11th, 2025
## 👨‍🏫 Instructor: Zahra Noviana
## 👥 Participants: Staff of RC Applied Microbiology - BRIN

🌍 Welcome to the Journey of Microbial Exploration! 🌱🔬
Imagine stepping into an invisible world, where trillions of bacteria interact to shape ecosystems, from the human gut to soil, oceans, and beyond. This hands-on workshop will empower you to decode these unseen microbial communities using QIIME2, a powerful bioinformatics tool for 16S rRNA metagenomic analysis.

Get ready to explore, analyze, and interpret microbial data like a pro! 🚀

References:
“Atacama soil microbiome” tutorial: https://docs.qiime2.org/2024.5/tutorials/atacama-soils/;
“Moving Pictures” tutorial: https://docs.qiime2.org/2024.5/tutorials/moving-pictures/

Welcome to the workshop! 

🎉 Let's set up and analyze microbial communities step by step. You got this! 💪

## 📌 Outline
1. Install QIIME2 and Download Database, Sequences, Metafile, and Barcode Information
2. Set Up the QIIME2 Environment and Prepare Working Directory
3. Import and Process Data
4. Filtering, Denoising, and Feature Analysis
5. Alpha and Beta Diversity Metrics and Visualization
6. Taxonomic Assignment and Visualization

## 📌 Let's get things started! 

### 1️⃣ Install QIIME2 and Download Database, Sequences, Metafile, and Barcode Information

#### 🔥 1.1 First, let's install QIIME2 and set up our environment!

This code is for the installation (Estimated time: 30s):
```
conda env create -n qiime2-amplicon-2024.5 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.5-py39-linux-conda.yml
```

#### 📥 1.2 Alternative --> Manual Installation

##### 🛠️ 1.2.a Download the Environment File

This is to download file for the installation of QIIME2:
```
# wget -O qiime2-amplicon-2024.5.yml https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.5-py39-linux-conda.yml
```

##### ⚡1.2.b Install via downloaded file

This is to install qiime2 directly from file:
```
# conda env create -n qiime2-amplicon-2024.5 --file qiime2-amplicon-2024.5.yml
```

#### 🧬 1.3 Download the Classifier

This is to download greengenes database (Estimated time: 30s):
```
wget -O "gg-13-8-99-515-806-nb-classifier.qza"  "https://data.qiime2.org/classifiers/sklearn-1.4.2/greengenes/gg-13-8-99-515-806-nb-classifier.qza"
```

#### 🗂️ 1.4 Download Metadata

This is to download metadata (Estimated time: 10s):
```
wget -O "sample-metadata.tsv" https://data.qiime2.org/2024.5/tutorials/atacama-soils/sample_metadata.tsv
```

#### 📥 1.5 Download Raw Sequence Data and Barcode

First create a folder to put the reads
```
mkdir raw/
```

This is to download forward reads (136.6 MB, Estimated time: 2 min):
```
wget -O "raw/forward.fastq.gz" https://data.qiime2.org/2024.5/tutorials/atacama-soils/10p/forward.fastq.gz
```

This is to download reverse reads (153.6 MB, Estimated time: 2 min):
```
wget -O "raw/reverse.fastq.gz" https://data.qiime2.org/2024.5/tutorials/atacama-soils/10p/reverse.fastq.gz
```

This is to download barcode information (Estimated time: 30s):
```
wget -O "raw/barcodes.fastq.gz" https://data.qiime2.org/2024.5/tutorials/atacama-soils/10p/barcodes.fastq.gz
```

### 2️⃣ Set Up the QIIME2 Environment and Prepare the Working Directory

#### Now, let's enter the interactive mode and activate our environment!

This is to enter interactive mode:
```
srun --partition=interactive --pty /bin/bash
```

This is to activate qiime2 environment:
```
conda activate qiime2-amplicon-2024.5
```

This is to check the installation:
```
qiime --help  
```

This is to check the active conda environment:
```
conda info  
```

#### 📂 Organize files before we begin!

First create our working folder:
```
mkdir qiime2_training/
```

Now, let's move to the directory :
```
cd qiime2_training/
```


### 3️⃣ Import and Process Data

Let's bring our sequences into QIIME2 and demultiplex them!

Import to QIIME2 (Estimated time: 2 min):
```
qiime tools import \
   --type EMPPairedEndSequences \
   --input-path raw \
   --output-path emp-paired-end-sequences.qza
```

Demultiplex (Estimated time: 3 min):
```
qiime demux emp-paired \
  --m-barcodes-file sample-metadata.tsv \
  --m-barcodes-column barcode-sequence \
  --p-rev-comp-mapping-barcodes \
  --i-seqs emp-paired-end-sequences.qza \
  --o-per-sample-sequences demux-full.qza \
  --o-error-correction-details demux-details.qza
```

🎯 Let's do subsampling and Summarizing Data

Subsample (Estimated time: 1 min):
```
qiime demux subsample-paired \
  --i-sequences demux-full.qza \
  --p-fraction 0.3 \
  --o-subsampled-sequences demux-subsample.qza
```

Summarize (Estimated time: 1 min):
```
qiime demux summarize \
  --i-data demux-subsample.qza \
  --o-visualization demux-subsample.qzv
```

You can open the qzv file in https://view.qiime2.org/. The result is something like this:

1. Count summary
![image](https://github.com/user-attachments/assets/26859cf8-24fc-4333-b7ba-8f3076bd645d)

2. Reads QC
![image](https://github.com/user-attachments/assets/996bcb6c-bc43-450c-a607-cbf4cc1ec4fe)


### 4️⃣ Filtering, Denoising, Feature Analysis, and Construction of Phylogenetic Tree

Now, let's clean and process the data for better accuracy!

Convert our file to appropriate format (Estimated time: 2 min):
```
qiime tools export \
  --input-path demux-subsample.qzv \
  --output-path ./demux-subsample/
```

Filter out low-reads samples (Estimated time: 1 min):
```
qiime demux filter-samples \
  --i-demux demux-subsample.qza \
  --m-metadata-file ./demux-subsample/per-sample-fastq-counts.tsv \
  --p-where 'CAST([forward sequence count] AS INT) > 100' \
  --o-filtered-demux demux.qza
```

Denoising with DADA2 (Estimated time: 4 min):
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left-f 13 \
  --p-trim-left-r 13 \
  --p-trunc-len-f 150 \
  --p-trunc-len-r 150 \
  --o-table table.qza \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza
```

Alignment and construction of phylogenetic tree (Estimated time: 1 min):
```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

### 5️⃣ Alpha and Beta Diversity Metrics and Visualization

#### 5.1 📊 Rarefaction curves

📉 Rarefaction Curve - Visualizes sequencing depth and richness 

This is to visualize rarefaction curve (Estimated time: 4-10 min):
```
qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 2000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv
```

Visualization of the result from https://view.qiime2.org/:
![image](https://github.com/user-attachments/assets/5b8fe36b-4c44-429c-ad11-1689d59e1536)


#### 5.2 📊 Test Alpha Diversity Metrics

First let's do alpha and beta diversity analysis (estimasi 1 menit):
```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 1000 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir core-metrics-results
```

🧬 Faith’s Phylogenetic Diversity (Faith PD) 

Evaluates phylogenetic diversity within samples.

Visualize Faith PD metric (Estimated time: 30s):
```
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/faith-pd-group-significance.qzv
```

Visualization of the result from https://view.qiime2.org/:
![image](https://github.com/user-attachments/assets/e093d9ab-0c6b-4594-b11b-b0803bf50d2f)


#### 5.3 🔬 Test Beta Diversity Metrics

🎭 Principal Coordinate Analysis (PCoA) Visualization

📌 Bray-Curtis PCoA Plot 

Visualize PCoA using Bray-Curtis distance (Estimated time: 1 min):
```
qiime emperor plot \
  --i-pcoa core-metrics-results/bray_curtis_pcoa_results.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/bray-curtis-emperor-days.qzv
```

Visualization of the result:
![image](https://github.com/user-attachments/assets/6979564b-c210-412b-986d-eb3e3b05b0f9)

### 6️⃣ Taxonomic Assignment and Visualization
🧬 Taxonomic Classification

🏷️ Run Classifier

Classifying reads (Estimated time: 30s):
```
qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza
```

📜 Taxonomy Table Visualization

Conversion of taxonomy file (Estimated time: 30s):
```
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```

📊 Taxa Bar Plot 

Visualize the data with barplot (Estimated time: 30s):
```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```

Visualization of the result:
![image](https://github.com/user-attachments/assets/9d708c11-f00c-4604-b64f-c05038715524)


🏆 Differential Abundance Testing with ANCOM-BC

🏅 Identify Differentially Abundant Features

Perform ancom on 'vegetation' (Estimated time: 30s):
```
qiime composition ancombc \
  --i-table table.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-formula 'vegetation' \
  --o-differentials ancombc-vegetation.qza
```

📊 ANCOM Visualization

Visualize the result with barplot (Estimated time: 30s):
```
qiime composition da-barplot \
  --i-data ancombc-vegetation.qza \
  --p-significance-threshold 0.001 \
  --o-visualization da-barplot-vegetation.qzv
```

Visualization of the result:
![image](https://github.com/user-attachments/assets/e14fda75-7442-4c7f-a882-ebfe1a63cac3)


🏗️ Collapse Taxonomic Levels

Agglomerate to genus level  (Estimated time: 30s):
```
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 6 \
  --o-collapsed-table table-l6.qza
```

📊 Test Statistics on Collapsed Data 

Perform statistic on genus level and across 'vegetation' (Estimated time: 1 min):
```
qiime composition ancombc \
  --i-table table-l6.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-formula 'vegetation' \
  --o-differentials l6-ancombc-vegetation.qza
```

📈 Barplot for Collapsed Data 

Visualize the data with barplot (Estimated time: 1 min):
```
qiime composition da-barplot \
  --i-data l6-ancombc-vegetation.qza \
  --p-significance-threshold 0.01 \
  --p-level-delimiter ';' \
  --o-visualization l6-da-barplot-vegetation.qzv
```


Visualization of the result:
![image](https://github.com/user-attachments/assets/73231fcb-b8f2-4029-98d0-7d2b0cf13646)


🎉 Awesome job! You've now completed the full QIIME 2 workflow, from installation to statistical analysis! 🚀 Keep exploring, and don't hesitate to dive deeper! 🔬💪

