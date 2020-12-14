# 16s扩增子分析流程-参考qzp

**扩增子测序数据拆分**

### 1.首先，激活运行环境

1）conda（安装samtools、bowtie2、fastqc、bedtools）

2）qiime2-2019.04

3）vsearch（qiime2内置）

4）biopython（qiime2内置）

1）conda（安装samtools、bowtie2、fastqc、bedtools）

conda是一个环境管理软件，可以让你不用管理员权限就能安装一些常用的分析软件

#### 1. 安装教程

miniconda

推荐个人环境安装, 占用空间小。[官方教程](https://docs.conda.io/projects/conda/en/latest/user-guide/index.html)

```shell
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh #下载`
bash Miniconda3-latest-Linux-x86_64.sh #安装`
```
anaconda [官方教程](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)

```shell 
#安装fastqc
conda install -c bioconda fastqc 

#安装samtools

conda install -c bioconda samtools #在base环境下安装

# 安装bowtie2

conda install -c bioconda bowtie2

# 安装bedtools

conda install -c bioconda bedtools  #在base环境下,安装bedtools
```

#### 2. qiime2-2019.04

公共账户LD lab上已经安装，登录后可以直接调用。#前提是你使用公用账户啊

`conda activate qiime2-2020.2 #直接激活最新的qiime2环境`

其他账号不用安装conda也可以激活并使用公共账户下的conda环境：

`source /home/LDlab/BioSoft/anaconda3/bin/activate /home/LDlab/BioSoft/anaconda3/envs/qiime2-2020.2 #需附上具体路径；这是用别人的conda激活别人的qiime2`

自己账户先安装conda然后去调用别人的qiime2
```shell
conda activate /home/LDlab/BioSoft/anaconda3/envs/qiime2-2019.4

conda activate /home/LDlab/BioSoft/anaconda3/envs/qiime2-2020.2 #20201024
```
自己账户先安装conda然后去调用自己的qiime2

下载和安装qiime2环境
```shell
wget https://data.qiime2.org/distro/core/qiime2-2020.2-py36-linux-conda.yml #下载

conda env create -n qiime2-2020.2 --file qiime2-2020.2-py36-linux-conda.yml #安装

conda activate qiime2-2020.2
```


### 2.vsearch双端合并要在qiime2环境运行

```shell
vsearch --fastq_mergepairs FILENAME-1 --reverse FILENAME-2 --fastqout FILENAME-out --fastqout_notmerged_fwd FILENAME-a --fastqout_notmerged_rev FILENAME-b 1>log—file 2>>log—file

fastqc FILENAME-out
```


输入文件：

**16s3-1_S1_L001_R1_001.fastq.gz** **测序正向结果**

**16s3-2_S1_L001_R2_001.fastq.gz** **测序反向结果**

输入脚本：

**无**

**实际操作例子：**

```shell
vsearch --fastq_mergepairs /home/quzp/seqdata-20200421/QZP1_S1_L001_R1_001.fastq.gz --reverse /home/quzp/seqdata-20200421/QZP1_S1_L001_R2_001.fastq.gz --fastqout qzp_merged.fq --fastqout_notmerged_fwd qzp_unmapped_1.fq --fastqout_notmerged_rev qzp_unmapped_2.fq 1>vserach_log_qzp.txt 2>>vserach_log_qzp.txt **#20201024**
```

hyc测序文件路径：

`/home/LDlab/huyucan/16s/16S3-2L_S9_L001_R1_001.fastq.gz`

`/home/LDlab/huyucan/16s/16S3-2L_S9_L001_R2_001.fastq.gz`



 质检：`fastqc hyc_merged.fq` **要在==base==环境运行**

###  3.拆分样品（根据barcode序列的完美匹配）==要在qiime2环境运行==

```shell
python /home/yxtan/qiime2_custom_scripts/parse_sample_perfectmatch.py -i 合并后的fq路径 -t 每个样品对应的barcode序列 -f 正向引物长度 -r 反向引物长度
```

**输入文件：**

从20200420样品信息.xlsx中提取B、D、E三列就是**sample_indexes.txt** 【为了后续画图等方便，其实B列的数字最好前面用0来补全N位数，这个可以在excel里面弄】

（但是一定要删干净，不然会说有不对应的column数；且最后一行不能有回车，也就是不能最后有一行空的行）

**输入脚本：**

**parse_sample_perfectmatch.py**

**实际操作例子：**

```shell
python /home/LDlab/BioSoft/Scripts/**parse_sample_perfectmatch.py -i qzp_merged.fq -t /home/quzp/20200510测序数据拆分/sample_indexes.txt -f 20 -r 18 #-f F引物长度 -r R引物长度 去除引物

python /home/huyucan/script/parse_sample_perfectmatch_update.py -i hyc_merged.fq -t /home/huyucan/20201030qzp16sp/testqzp2/0scp/sample_indexes.txt -f 9 -r 15 #没有barcode我截短了引物
```
 `/home/quzp/script/parse_sample_perfectmatch_update.py`

`/home/quzp/20201024sequencing_data/sample_indexes.txt`

### 4.拆分样品（根据参考序列的完美匹配）==要在base环境运行==

`bash /home/yxtan/qzp_16S_pools/data/Bowtie2align_and_feature_extract.sh input_folder input_ref.fa input_ref.bed`

 

**输入文件：**

**22株根际菌.fa**可以直接用1-22号菌V5~V7序列信息.txt的内容

**22株根际菌.bed**需要从1-22号菌V5~V7序列信息.txt里面，通过excel里面的len命令生成每个序列对应的长度，从而生成（qzp没专门写脚本）。第一列为菌种名，第二列为从第几个碱基开始，第三列为检索长度。其余附加列另有用处，这三列必须有。

**输入脚本：**

**Bowtie2align_and_feature_extract.sh**

 

**实际操作例子：**
```shell
bash /home/LDlab/BioSoft/Scripts/Bowtie2align_and_feature_extract.sh /home/quzp/20200510seqdata_result/parse /home/quzp/20200510测序数据拆分/22株根际菌.fa /home/quzp/20200510测序数据拆分/22株根际菌.bed
```

```shell
conda deactivate #关闭qiime2环境进入base
screen -S hyc #挂起后台会话
bash /home/huyucan/script/Bowtie2align_and_feature_extract_30.sh /home/huyucan/20201030qzp16sp/parse/ /home/huyucan/20201030qzp16sp/0scp/pre_syn5_v3_v4.fa /home/huyucan/20201030qzp16sp/0scp/pre_syn5bed.bed #20201030
```
```bash
#路径
"/home/quzp/20201024sequencing_data/20strains_v5v7.fa"

"/home/quzp/20201024sequencing_data/20strains_v5v7.bed"

"/home/quzp/20201024sequencing_data/parse/"

"/home/quzp/script/Bowtie2align_and_feature_extract_30.sh"

 

#input1: the folder of fastqs

#input2: The fasta file of the refernce region of each strain; the name of strains should be the same as the ones in the bed file

#input3: The bed file (must be customized into strains in specific format) and the fasta with bowtie2 index of the refernce strains

 

#output_path: current folder

#final output: count_matrix.txt （主要的结果文件）

#intermediate outputs: intermediate folder with sam\bam\bai of each sample.
```


### 5.后台运行，防止连接断开导致的运行终止

`screen -S quzp #新建一个叫quzp的会话session`

再运行命令
`screen –ls #查看所有会话`
**关闭后台**
`screen -x quzp #进入之前的会话`
`Ctrl+A`
`kill`

### 6.常用命令
```shell
pwd #查看当前路径`
cd 路径 命令 #命令在当期路径下运行
ssh node1 #更换节点，有5个节点，node1~4和一个master
htop  #当前节点使用情况
clear #清屏幕
ctrl+C结束程序
#复制粘贴
cp –rf /home/xxx/a/* /home/xxx/b #把a文件的内容复制到b文件夹中
cp –rf /home/quzp/CMX_plant_microbiome_seqdata/LY_seq/* /home/quzp/CMX_plant_microbiome_seqdata/LY_seq_53ref/
"/home/quzp/CMX_plant_microbiome_seqdata/LY_seq/"
"/home/quzp/CMX_plant_microbiome_seqdata/LY_seq_53ref/"
"/home/quzp/CMX_plant_microbiome_seqdata/LY_seq/parse/"
cp –rf /home/luwu/wulu_data/Nuohe_0608/qzp_merged.fq /home/quzp/wulu
less #路径/文件名_查看文件内容
head #路径/文件名_查看文件内容的前几条
more #路径/文件名_查看文件内容
cat #路径/文件名_正序查看文件内容
tac #路径/文件名_倒序查看文件内容
```
### 7.参考资料

1）Conda 环境的安装&使用（Chen jy）

https://github.com/Junyu25/LD-lab/blob/master/conda-env.ipynb

2）数据拆分(Tan yx)

