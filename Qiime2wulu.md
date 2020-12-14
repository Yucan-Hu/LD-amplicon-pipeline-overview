# 吴璐流程

## 双端合并 ==要在qiime2环境运行==

```shell
vsearch --fastq_mergepairs ../Wulu-200608_FDDP202420784-1a_1.clean.fq --reverse ../Wulu-200608_FDDP202420784-1a_2.clean.fq --fastqout 0608_merged.fq --fastqout_notmerged_fwd 0608_unmapped_1.fq --fastqout_notmerged_rev 0608_unmapped_2.fq 1>vserach_log_0608.txt 2>>vserach_log_0608.txt
```
## 拆分reads：
整理好meta.txt : 没有抬头，三列，分别是id barcode F 和barcode R

 

#要在base环境运行,biopython 按照在base下的
`python /home/luwu/codes/nova_seq_parse_sample_perfectmatch.py -i 0608_merged.fq -t /home/luwu/wulu_data/Nuohe_0608/200608_meta.txt -f 17 -r 21`

所有文件在parese文件夹下，改名成：

`rename 's/\.fastq/\_0811_L001_R1_001.fastq/'`

对所有文件进行压缩，成gz 文件
`pigz .fastq`

## 1.	导入数据：

`qiime tools import --type 'SampleData[SequencesWithQuality]' --input-path parse --output-path single-end-demux.qza --input-format CasavaOneEightSingleLanePerSampleDirFmt`

## 1.	查看各样品的reads数目分布： 
`qiime demux summarize --i-data single-end-demux.qza --o-visualization 0612_trim_primer-summary-1.qzv`

## 1.	利用data2对数据进行denoise：（费时步骤）
`qiime dada2 denoise-single --i-demultiplexed-seqs single-end-demux.qza --p-trim-left 0 --p-trunc-len 0 --p-max-ee 4.2 --o-representative-sequences rep-seqs-dada2.qza --o-table table.qza --o-denoising-stats stats-dada2.qza`

==一般需要对table进行过滤：==
1,针对低丰度reads进行过滤：This filter can be applied to the feature axis to remove low abundance features from a table. For example, you can remove all features with a total abundance (summed across all samples) of less than 10 as follows.
`qiime feature-table filter-features --i-table table.qza  --p-min-frequency 10 --o-filtered-table feature-frequency-filtered-table.qza`

>来自 <https://docs.qiime2.org/2020.2/tutorials/filtering/> 

## 2.针对出现样品数少的reads进行过滤：Contingency-based filtering
Contingency-based filtering is used to filter samples from a table contingent on the number of features they contain, or to filter features from a table contingent on the number of samples they’re observed in.
This filtering is commonly used for filtering features that show up in only one or a few samples, based on the suspicion that these may not represent real biological diversity but rather PCR or sequencing errors (such as PCR chimeras). Features that are present in only a single sample could be filtered from a feature table as follows.
`qiime feature-table filter-features   --i-table table.qza   --p-min-samples 2  --o-filtered-table sample-contingency-filtered-table.qza`

回过来看看剩下些啥：`qiime feature-table summarize --i-table feature-frequency-filtered-table_filter10.qza --o-visualization table_contigingency_2_filter10.qzv --m-sample-metadata-file metadata.txt`


## 1.	taxonomy归类（费时步骤）
`nohup qiime feature-classifier classify-sklearn --i-classifier /home/luwu/database/Nr99/lab_source/classifier_taxonomy_slv_132_Nr99_341_805_wulu_modify.qza --o-classification taxonomy.qza --i-reads rep-seqs_0911.qza &`
## 7. 查看个样品的feature 数目，确定下游rarefraction 
```shell
qiime feature-table summarize --i-table table.qza --o-visualization table.qzv --m-sample-metadata-file metadata.txt
#这里的metadata.txt 是专门的可以用脚本生成：
```
对于
/home/luwu/wulu_data/0612_data//200429A-Stool-L_S3_L001_R1_001.fastq.gz
-s 
运行：`python /home/luwu/codes/manifest_paired.py -s _R -m _S -d /home/luwu/wulu_data/prcessed_seqs`
 生成索引文件metatable.txt得到 sample200429A-Stool-L

对于：200807D_0608_L001_R1_001.fastq.gz
运行程序：`python /home/luwu/codes/manifest_paired.py -s _R -m _L -d /home/luww/wulu_data/prcessed_seqs`
得到：
#SampleID        FileName
sample200511B_0608        200511B_0608

```shell
1.	进化树
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences rep-seqs.qza --o-alignment aligned-rep-seqs.qza --o-masked-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza --o-rooted-tree rooted-tree.qza

1.	core—diversity
qiime diversity core-metrics-phylogenetic   --i-phylogeny rooted-tree.qza   --i-table table.qza   --p-sampling-depth 10000   --m-metadata-file metadata.txt --output-dir core-metrics-results-10000

1.	taxonomy柱状图：
qiime taxa barplot   --i-table table.qza   --i-taxonomy taxonomy.qza   --m-metadata-file metadata.txt   --o-visualization taxa-bar-plots.qzv

1.	组间β多样性比较：

计算Aitchison距离：
pip install deicode
qiime dev refresh-cache
qiime deicode rpca --i-table table.qza --p-min-feature-count 10 --p-min-sample-count 500  --o-biplot ordination.qza --o-distance-matrix distance.qza

qiime emperor biplot --i-biplot ordination.qza --m-sample-metadata-file metadata.txt --m-feature-metadata-file taxonomy.qza --o-visualization biplot.qzv --p-number-of-features 10

qiime tools export --input-path distance.qza --output-path exported/Atichison_dis

qiime diversity beta-group-significance --i-distance-matrix distance.qza  --m-metadata-file sample-metadata.tsv --m-metadata-column BodySite  --p-method permanova --o-visualization BodySite_significance.qzv

来自 <https://forum.qiime2.org/t/robust-aitchison-pca-beta-diversity-with-deicode/8333> 


qiime diversity adonis --i-distance-matrix    --m-metadata-file   --p-formula TEXT   --p-permutations 999
--o-visualization 

组间距离计算和boxplot展示：

qiime diversity beta-group-significance --i-distance-matrix weighted_unifrac_distance_matrix.qza   --m-metadata-file ../metadata.txt  --m-metadata-column Cluster --o-visualization exported/weight_cluster_group --p-method permdisp

来自 <https://docs.qiime2.org/2020.8/tutorials/pd-mice/?highlight=beta> 

 

11.各种数据导出：（a-diverisity指数 各种distance）
qiime tools export --input-path  core-metrics-results-5000/shannon_vector.qza --output-path output/shannon

qiime tools export --input-path rooted-tree.qza --output-path exported

qiime tools export --input-path taxonomy_99.qza --output-path exported

qiime tools export --input-path table.qza  --output-path exported

qiime tools export --input-path rep-seqs.qza --output-path exported

qiime tools export  --input-path stats.qza --output-path exported
biom convert -i exported/feature-table.biom -o exported/feature-table.txt --to-tsv
```