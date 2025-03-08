🎯 Hands-on Workshop: Decoding Bacterial Communities Using Short-Read 16S rRNA Metagenomics

📅 Session 1: March 11th, 2025👨‍🏫 Instructor: XX👥 Participants: Staff of XXX

🌍 Welcome to the Journey of Microbial Exploration! 🌱🔬
Imagine stepping into an invisible world, where trillions of bacteria interact to shape ecosystems, from the human gut to soil, oceans, and beyond. This hands-on workshop will empower you to decode these unseen microbial communities using QIIME 2, a powerful bioinformatics tool for 16S rRNA metagenomic analysis.

Get ready to explore, analyze, and interpret microbial data like a pro! 🚀

🚀 QIIME 2 Installation and Analysis Workflow

Welcome to the QIIME 2 workflow guide! 🎉 Let's set up and analyze microbial communities step by step. You got this! 💪

📌 Outline

Install and Download QIIME 2

Set Up the QIIME 2 Environment

Prepare the Working Directory

Import and Process Data

Filter, Denoise, and Analyze Features

Taxonomic Classification and Diversity Analysis

Statistical Analysis and Visualization

7️⃣ Statistical Analysis and Visualization

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





