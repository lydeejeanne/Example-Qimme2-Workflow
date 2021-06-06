# Qimme2 Workflow Scripts
This was used to analyze anglerfish symbionts present in both the animal and in the water sample

## Preparation Scripts
### Pairing the singletons
Run Trimmomatic to get rid of all the paired end reads that don’t match
`./pair.sh`

Made new files where paired and singletons were put into different files, the original stayed as is

`mv *.paired.fastq paired`

`mv paired/ ..`


## Environment Scripts
Getting qiime2 up and running
`export LC_ALL=en_US.utf-8`

`export LANG=en_US.utf-8`

`export PATH=/programs/Anaconda2/bin:$PATH`

`source activate qiime2-2018.11`

Put it in a format that works
`qiime tools import  --type 'SampleData[PairedEndSequencesWithQuality]'  --input-path manifest_paired.csv   --output-path paired-end-demux.qza  --input-format PairedEndFastqManifestPhred33`

Imported sequence.csv as PairedEndFastqManifestPhred33 to paired-end-demux.qza

To view the summary just made:
`qiime demux summarize --i-data paired-end-demux.qza --o-visualization  paired_end-demux.qzv`

Saved Visualization to: paired_end-demux.qzv

`nohup qiime dada2 denoise-paired --i-demultiplexed-seqs paired-end-demux.qza --p-trim-left-f 18 --p-trim-left-r 20 --p-trunc-len-f 220 --p-trunc-len-r 150 --o-table table.gza --o-representative-sequences rep-seqs.gza --o-denoising-stats denoising-stats.gza --verbose --p-n-threads 36 >& mel_run.log &`

## Summaries
For water use _with.missing.txt
`qiime feature-table summarize --i-table table.gza.qza --o-visualization table.gzv --m-sample-metadata-file AllAnglersandWater01_04_metadata_new.txt `

`qiime feature-table tabulate-seqs --i-data rep-seqs.gza.qza --o-visualization rep-seqs.qzv`

Qiime tools view table.qzv (on Google Chrome = view.qiime2.org)

`qiime metadata tabulate --m-input-file denoising-stats.gza.qza --o-visualization denoising-stats.gzv`


## Qimme Scripts
Alignment
`nohup qiime phylogeny align-to-tree-mafft-fasttree   --i-sequences rep-seqs.gza.qza --o-alignment aligned-rep-seqs.qza   --o-masked-alignment masked-aligned-rep-seqs.qza   --o-tree unrooted-tree.qza   --o-rooted-tree rooted-tree.qza  --p-n-threads 36 >& log2 &`

Classify
`qiime feature-classifier classify-sklearn --i-classifier silva-132-99-515-806-nb-classifier.qza --i-reads rep-seqs.gza.qza --o-classification taxonomy.qza --p-n-threads 36 >& classifier.log &`

`qiime metadata tabulate --m-input-file taxonomy.qza --o-visualization visualization/taxonomy.qzv`

Filtering 
`qiime taxa filter-table --i-table table.gza.qza --i-taxonomy taxonomy.qza --p-include Bacteria --o-filtered-table table-Bact.qza`

`qiime feature-table filter-features --i-table table-Bact.qza --p-min-frequency 2 --o-filtered-table table-Bact-no-single.gza`

Make visualizations
`qiime feature-table summarize --i-table table-Bact-no-single.gza.qza --o-visualization table-Bact-no-single.gzv --m-sample-metadata-file AllAnglersandWater01_04_metadata_new.txt `

`qiime taxa barplot --i-table table-Bact-no-single.gza.qza --i-taxonomy taxonomy.qza --m-metadata-file AllAnglersandWater01_04_metadata_new.txt --o-visualization taxa-bar-plots-Bact-no-single.gzv`

Exporting
`qiime tools export --input-path ./table-Bact-no-single.gza.qza --output-path ./for_export`

`qiime tools export --input-path ./rooted-tree.qza  --output-path ./for_export`

`qiime tools export  --input-path ./taxonomy.qza --output-path ./for_export`

Move everything into the qiime2_paired file for input into R

`biom convert -i feature-table.biom -o out_table.txt --to-tsv`

Concatonate all the singles left from fastq_pair
`./cat.sh`

`mv reads_single`

Prep qimme2 for run
`export LC_ALL=en_US.utf-8`

`export LANG=en_US.utf-8`

`export PATH=/programs/Anaconda2/bin:$PATH`

`source activate qiime2-2018.11`

Put it in a format that works
`qiime tools import  --type 'SampleData[SequencesWithQuality]'  --input-path manifest_single.csv   --output-path single-demux.qza  --input-format SingleEndFastqManifestPhred33`



Imported manifest_single.csv as SingleEndFastqManifestPhred33 to single-demux.qza

To view the summary just made:
`qiime demux summarize --i-data single-demux.qza --o-visualization  single-demux.qzv`


Saved Visualization to: single-demux.qzv

`nohup qiime dada2 denoise-single --i-demultiplexed-seqs single-demux.qza --p-trim-left 20 --p-trunc-len 190 --o-table table.gza --o-representative-sequences rep-seqs.gza --o-denoising-stats denoising-stats.gza --verbose --p-n-threads 16 >& dada2_run.log &`


## Summaries
`qiime feature-table summarize --i-table table.gza.qza --o-visualization table.gzv --m-sample-metadata-file AllAnglersandWater01_04_metadata_new.txt `

`qiime feature-table tabulate-seqs --i-data rep-seqs.gza.qza --o-visualization rep-seqs.qzv`

`qiime tools view table.qzv (on Google Chrome = view.qiime2.org)`

`qiime metadata tabulate --m-input-file denoising-stats.gza.qza --o-visualization denoising-stats.gzv`


Alignment
`nohup qiime phylogeny align-to-tree-mafft-fasttree   --i-sequences rep-seqs.gza.qza --o-alignment aligned-rep-seqs.qza   --o-masked-alignment masked-aligned-rep-seqs.qza   --o-tree unrooted-tree.qza   --o-rooted-tree rooted-tree.qza   >& log2 &`

Classify
`qiime feature-classifier classify-sklearn --i-classifier silva-132-99-515-806-nb-classifier.qza --i-reads rep-seqs.gza.qza --o-classification taxonomy.qza >& classifier.log &`


`qiime metadata tabulate --m-input-file taxonomy.qza --o-visualization taxonomy.qzv`


Filtering 
`qiime taxa filter-table --i-table table.gza.qza --i-taxonomy taxonomy.qza --p-include Bacteria --o-filtered-table table-Bact.gza`


`qiime feature-table filter-features --i-table table-Bact.qza --p-min-frequency 2 --o-filtered-table table-Bact-no-single.gza`

Make visualizations
`qiime feature-table summarize --i-table table-Bact-no-single.gza.qza --o-visualization table-Bact-no-single.gzv --m-sample-metadata-file AllAnglersandWater01_04_metadata_new.txt `

`qiime taxa barplot --i-table table-Bact-no-single.gza.qza --i-taxonomy taxonomy.qza --m-metadata-file AllAnglersandWater01_04_metadata_new.txt --o-visualization taxa-bar-plots-Bact-no-single.gzv`



Exporting
`qiime tools export --input-path ./table-Bact-no-single.gza.qza --output-path ./for_export`

`qiime tools export   --input-path ./rooted-tree.qza   --output-path ./for_export`


`qiime tools export   --input-path ./taxonomy.qza   --output-path ./for_export`


Move everything into the qiime2_single file.

With the for_export, change the taxonomy.tsv header to `#OTUID	taxonomy	confidence` then run biom 

`biom add-metadata -i feature-table.biom -o table-with-taxonomy.biom --observation-metadata-fp taxonomy.tsv --sc-separated taxonomy`
