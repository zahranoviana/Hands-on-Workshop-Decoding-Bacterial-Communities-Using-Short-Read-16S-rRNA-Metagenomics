# 🎯 Hands-on Workshop: Decoding Bacterial Communities Using Short-Read 16S rRNA Metagenomics

📅 Session 1: March 11th, 2025👨‍🏫 Instructor: XX👥 Participants: Staff of XXX

🌍 Welcome to the Journey of Microbial Exploration! 🌱🔬
Imagine stepping into an invisible world, where trillions of bacteria interact to shape ecosystems, from the human gut to soil, oceans, and beyond. This hands-on workshop will empower you to decode these unseen microbial communities using QIIME 2, a powerful bioinformatics tool for 16S rRNA metagenomic analysis.

Get ready to explore, analyze, and interpret microbial data like a pro! 🚀

🚀 QIIME 2 Installation and Analysis Workflow

Welcome to the workshop! 🎉 Let's set up and analyze microbial communities step by step. You got this! 💪

## 📌 Outline
1. Install and Download QIIME 2
2. Set Up the QIIME 2 Environment
3. Prepare the Working Directory
4. Import and Process Data
5. Filter, Denoise, and Analyze Features
6. Taxonomic Classification and Diversity Analysis
7. Statistical Analysis and Visualization

## 📌 Let's get things started! 
### 1️⃣ Install and Download QIIME 2 (Estimated time: ~7 minutes)

🔥 First, let's install QIIME 2 and set up our environment!

time conda env create -n qiime2-amplicon-2024.5 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.5-py39-linux-conda.yml

📥 Manual Installation

🛠️ 1.1 Download the Environment File

wget -O qiime2-amplicon-2024.5.yml https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.5-py39-linux-conda.yml

🧬 1.2 Download the Classifier (Estimated time: 30s)

time wget \
  -O "gg-13-8-99-515-806-nb-classifier.qza" \
  "https://data.qiime2.org/classifiers/sklearn-1.4.2/greengenes/gg-13-8-99-515-806-nb-classifier.qza"

⚡ 1.3 Install QIIME 2

conda env create -n qiime2-amplicon-2024.5 --file qiime2-amplicon-2024.5.yml

### 2️⃣ Set Up the QIIME 2 Environment

Now, let's enter the QIIME 2 interactive mode and activate our environment!

srun --partition=interactive --pty /bin/bash
conda activate qiime2-amplicon-2024.5
qiime --help  # Test installation
conda info  # Check active conda environment

### 3️⃣ Prepare the Working Directory

📂 Organize files before we begin!

mkdir qiime2_training/
cd qiime2_training/

🗂️ 3.1 Download Metadata (Estimated time: 10s)

wget -O "sample-metadata.tsv" https://data.qiime2.org/2024.5/tutorials/atacama-soils/sample_metadata.tsv

📥 3.2 Download Raw Sequence Data

mkdir raw/
# Forward Reads (136.6 MB, Estimated time: 2 min)
wget -O "raw/forward.fastq.gz" https://data.qiime2.org/2024.5/tutorials/atacama-soils/10p/forward.fastq.gz
# Reverse Reads (153.6 MB, Estimated time: 2 min)
wget -O "raw/reverse.fastq.gz" https://data.qiime2.org/2024.5/tutorials/atacama-soils/10p/reverse.fastq.gz
# Barcode (Estimated time: 30s)
wget -O "raw/barcodes.fastq.gz" https://data.qiime2.org/2024.5/tutorials/atacama-soils/10p/barcodes.fastq.gz

### 4️⃣ Import and Process Data

Let's bring our sequences into QIIME 2 and demultiplex them!

# Import to QIIME (Estimated time: 2 min)
qiime tools import \
   --type EMPPairedEndSequences \
   --input-path raw \
   --output-path emp-paired-end-sequences.qza

# Demultiplex (Estimated time: 2 min)
qiime demux emp-paired \
  --m-barcodes-file sample-metadata.tsv \
  --m-barcodes-column barcode-sequence \
  --p-rev-comp-mapping-barcodes \
  --i-seqs emp-paired-end-sequences.qza \
  --o-per-sample-sequences demux-full.qza \
  --o-error-correction-details demux-details.qza

🎯 4.1 Subsampling and Summarizing Data

# Subsample (Estimated time: 1 min)
qiime demux subsample-paired \
  --i-sequences demux-full.qza \
  --p-fraction 0.3 \
  --o-subsampled-sequences demux-subsample.qza

# Summarize (Estimated time: 1 min)
qiime demux summarize \
  --i-data demux-subsample.qza \
  --o-visualization demux-subsample.qzv

### 5️⃣ Filtering, Denoising, and Feature Analysis

Now, let's clean and process the data for better accuracy!

# Filter out low-quality reads (Estimated time: 30s)
qiime tools export \
  --input-path demux-subsample.qzv \
  --output-path ./demux-subsample/

qiime demux filter-samples \
  --i-demux demux-subsample.qza \
  --m-metadata-file ./demux-subsample/per-sample-fastq-counts.tsv \
  --p-where 'CAST([forward sequence count] AS INT) > 100' \
  --o-filtered-demux demux.qza

# Denoising with DADA2 (Estimated time: 4 min)
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left-f 13 \
  --p-trim-left-r 13 \
  --p-trunc-len-f 150 \
  --p-trunc-len-r 150 \
  --o-table table.qza \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza

### 6️⃣ Taxonomic Classification and Diversity Analysis

Let's classify species and visualize the community composition!

# Run classifier (Estimated time: 30s)
qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

# Taxa Barplot Visualization
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv

### 7️⃣ Statistical Analysis and Visualization

📊 Test Alpha Diversity Metrics

🧬 Faith’s Phylogenetic Diversity (Faith PD) (Estimated time: 30s)

Evaluates phylogenetic diversity within samples.

time qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/faith-pd-group-significance.qzv

🌍 Evenness Test (Estimated time: 30s)

Measures how evenly species are distributed across samples.

time qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/evenness_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/evenness-group-significance.qzv

🔬 Test Beta Diversity Metrics

🔗 Unweighted UniFrac Distance Test (Estimated time: 1 min)

Compares community composition differences between groups.

time qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column transect-name \
  --o-visualization core-metrics-results/unweighted-unifrac-transect-name-significance.qzv \
  --p-pairwise

🎭 Principal Coordinate Analysis (PCoA) Visualization

🗺️ UniFrac PCoA Plot (Estimated time: 1 min)

time qiime emperor plot \
  --i-pcoa core-metrics-results/unweighted_unifrac_pcoa_results.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-custom-axes elevation \
  --o-visualization core-metrics-results/unweighted-unifrac-emperor-elevation.qzv

📌 Bray-Curtis PCoA Plot (Estimated time: 1 min)

time qiime emperor plot \
  --i-pcoa core-metrics-results/bray_curtis_pcoa_results.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-custom-axes elevation \
  --o-visualization core-metrics-results/bray-curtis-emperor-days-elevation.qzv

📉 Rarefaction Curve (Estimated time: 3-4 min)

Visualizes sequencing depth and richness.

time qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 2000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv

🧬 Taxonomic Classification

🏷️ Run Classifier (Estimated time: 30s)

time qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

📜 Taxonomy Table Visualization (Estimated time: 30s)

time qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv

📊 Taxa Bar Plot (Estimated time: 30s)

time qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv

🏆 Differential Abundance Testing with ANCOM-BC

🏅 Identify Differentially Abundant Features (Estimated time: 30s)

time qiime composition ancombc \
  --i-table table.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-formula 'vegetation' \
  --o-differentials ancombc-vegetation.qza

📊 ANCOM Visualization (Estimated time: 30s)

time qiime composition da-barplot \
  --i-data ancombc-vegetation.qza \
  --p-significance-threshold 0.001 \
  --o-visualization da-barplot-vegetation.qzv

🏗️ Collapse Taxonomic Levels (Estimated time: 30s)

time qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 6 \
  --o-collapsed-table table-l6.qza

📊 Test Statistics on Collapsed Data (Estimated time: 1 min)

time qiime composition ancombc \
  --i-table table-l6.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-formula 'vegetation' \
  --o-differentials l6-ancombc-vegetation.qza

📈 Barplot for Collapsed Data (Estimated time: 1 min)

time qiime composition da-barplot \
  --i-data l6-ancombc-vegetation.qza \
  --p-significance-threshold 0.01 \
  --p-level-delimiter ';' \
  --o-visualization l6-da-barplot-vegetation.qzv

🎉 Awesome job! You've now completed the full QIIME 2 workflow, from installation to statistical analysis! 🚀 Keep exploring, and don't hesitate to dive deeper! 🔬💪





