# 锦辉16s 扩增子分析流程（双端数据）

```shell
/home/jhtang/mytest/zhangzhenkun/2020-05-08

###激活环境
source activate /home/jhtang/miniconda3/envs/qiime2-2019.7

#### produce manifest.txt file , file names like S2_S198_L001_R1_001.fastq.gz
python /home/jhtang/mytest/code/process_16S/manifest_paired.py -s _R -m _L001 -d /home/jhtang/mytest/zhangzhenkun/data 


qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality] ' \
  --input-path manifest.txt \
  --output-path paired-end-demux.qza \
  --input-format PairedEndFastqManifestPhred33V2 

qiime cutadapt trim-paired \
  --i-demultiplexed-sequences paired-end-demux.qza \
  --p-cores 8 \
  --p-front-f CCTACGGGNGGCWGCAG \
  --p-front-r GACTACHVGGGTATCTAATCC \
  --p-overlap 8 \
  --o-trimmed-sequences trim_primer.qza 


qiime dada2 denoise-paired \
  --i-demultiplexed-seqs trim_primer.qza \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 0 \
  --p-trunc-len-r 0 \
  --p-max-ee-f 2.5 \
  --p-max-ee-r 2.5 \
  --o-table table.qza \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats stats.qza \
  --p-n-threads 10 

# 标注
qiime feature-classifier classify-sklearn \
  --i-classifier /home/jhtang/mytest/silva/Nr99/classifier_taxonomy_slv_132_Nr99_341_805.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy_99.qza 

#进化树
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza 


#数据导出，导出进化树成nwk格式
qiime tools export \
  --input-path rooted-tree.qza \
  --output-path exported

qiime tools export \
  --input-path taxonomy_99.qza \
  --output-path exported

qiime tools export \
  --input-path table.qza \
  --output-path exported

qiime tools export \
  --input-path rep-seqs.qza \
  --output-path exported

qiime tools export \
 --input-path stats.qza \
 --output-path exported

#将table生成的biom 文件转化为txt文件, qza可以通过unzip解压  qiime2 里面有这个biom的软件
biom convert -i exported/feature-table.biom -o exported/feature-table.txt --to-tsv
```

