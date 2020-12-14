# qiime2分类器训练调试-TJH

```shell
## qiime2-2019.7 版本
  ### Nr99_138
## wget 'https://www.arb-silva.de/fileadmin/silva_databases/release_138/Exports/SILVA_138_SSURef_NR99_tax_silva.fasta.gz'


python split_tax_fasta.py -i SILVA_138_SSURef_NR99_tax_silva.fasta

#####  美格 # V3-V4 338F 5′-ACTCCTACGGGAGGCAGCA-3′ and 806R 5′-GGACTACHVGGGTWTCTAAT-3′
 qiime tools import \
  --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \
  --input-path SILVA_138_SSURef_NR99_tax.txt \
  --output-path ref-taxonomy_Nr99.qza 
  
  qiime tools import \
  --input-path SILVA_138_SSURef_NR99_seq.fasta \
  --output-path SILVA_138_SSURef_Nr99_seq.qza \
  --type 'FeatureData[Sequence]'

nohup  qiime feature-classifier extract-reads \
  --i-sequences SILVA_138_SSURef_Nr99_seq.qza \
  --p-f-primer ACTCCTACGGGAGGCAGCA \
  --p-r-primer GGACTACHVGGGTWTCTAAT \
  --p-trunc-len 428 \
  --p-min-length 100 \
  --p-max-length 650 \
  --o-reads ref-seqs_Nr99_338-806.qza &

#V4区   华大  515_806 
 nohup qiime feature-classifier extract-reads \
  --i-sequences SILVA_138_SSURef_Nr99_seq.qza \
  --p-f-primer GTGCCAGCMGCCGCGGTAA \
  --p-r-primer GGACTACHVGGGTWTCTAAT \
  --p-trunc-len 253 \
  --p-min-length 100 \
  --p-max-length 400 \
  --o-reads ref-seqs_Nr99_515_806.qza &
#V3-V4雷朝碧 341F CCTACGGGNGGCWGCAG 805R GACTACHVGGGTATCTAATCC
 nohup  qiime feature-classifier extract-reads \
  --i-sequences SILVA_138_SSURef_Nr99_seq.qza \
  --p-f-primer CCTACGGGNGGCWGCAG \
  --p-r-primer GACTACHVGGGTATCTAATCC \
  --p-trunc-len 428 \
  --p-min-length 100 \
  --p-max-length 650 \
  --o-reads ref-seqs_Nr99_341-805.qza &



##train the classifier
nohup qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads ref-seqs_Nr99_341-805.qza \
  --i-reference-taxonomy ref-taxonomy_Nr99.qza \
  --o-classifier classifier_taxonomy_slv_138_Nr99_341_805.qza &

```

