# 1 Identification of Actin Genes

## 1.1 Download genome and protein sequence 
```bash
mkdir sequence
cd sequence

wget http://ftp.ensembl.org/pub/release-106/fasta/homo_sapiens/pep/Homo_sapiens.GRCh38.pep.all.fa.gz
gzip -d Homo_sapiens.GRCh38.pep.all.fa.gz
cat Homo_sapiens.GRCh38.pep.all.fa | grep ">" | wc -l
# 119068

head -n 2 Homo_sapiens.GRCh38.pep.all.fa
#ENSP00000488240.1 pep chromosome:GRCh38:CHR_HSCHR7_2_CTG6:142847306:142847317:1 gene:ENSG00000282253.1 transcript:ENST00000631435.1 gene_biotype:TR_D_gene transcript_biotype:TR_D_gene gene_symbol:TRBD1 description:T cell receptor beta diversity 1 [Source:HGNC Symbol;Acc:HGNC:12158]
#GTGG
```

## 1.2 Download protein sequence that limited to genes with actin domain(Pfam:PF00022)
![](./Fig/PF00022.png)

下载序列并重命名为human.fa
## 1.3 Identify actin genes
> biomart根据PFAM结构域初步筛选了一部分序列，这里利用blastp查找潜在的同源物（potential homologs）?
```bash
mkdir blastp
cd blastp
mv human.fa blastp

# blastp
makeblastdb -in ../sequence/Homo_sapiens.GRCh38.pep.all.fa -dbtype prot -parse_seqids -out ./index # 建立索引
blastp -query ./human.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result.tsv

# 统计
cat result.tsv | cut -f 2 | sort | uniq | wc -l
# 108
cat result.tsv | cut -f 2 | sort | uniq > protein_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein_ID.lst | wc -l
# 108
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 32
```
+ 第二轮blastp
```bash
cd repeat
mkdir ../blastp_two
cd ../blastp_two

faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa ../blastp/protein_ID.lst seed.fa

makeblastdb -in ../sequence/Homo_sapiens.GRCh38.pep.all.fa -dbtype prot -parse_seqids -out ./index
blastp -query ./seed.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result.tsv

# 统计
cat result.tsv | cut -f 2 | sort | uniq | wc -l
# 110
cat result.tsv | cut -f 2 | sort | uniq > protein_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 33 
```
+ 第三轮blastp
```bash
cd repeat
mkdir ../blastp_three
cd ../blastp_three

faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa ../blastp_two/protein_ID.lst seed.fa

makeblastdb -in ../sequence/Homo_sapiens.GRCh38.pep.all.fa -dbtype prot -parse_seqids -out ./index
blastp -query ./seed.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result.tsv

# 统计
cat result.tsv | cut -f 2 | sort | uniq | wc -l 
# 118
cat result.tsv | cut -f 2 | sort | uniq > protein_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 38
```
+ 重复blastp一直到没有新的结果
```bash
# 最终统计（6轮）
cat result.tsv | cut -f 2 | sort | uniq | wc -l
# 128
cat result.tsv | cut -f 2 | sort | uniq > protein_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 41
```
> 为了确保筛选出来的都是肌动蛋白基因，这里利用CDD对这些序列的Domain进行鉴定
+ [CDD](https://www.ncbi.nlm.nih.gov/Structure/bwrpsb/bwrpsb.cgi)
```bash
cd blastp_six

faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa protein_ID.lst raw.fa
```
![](./Fig/CDD.png)
```bash
mkdir CDD
cd CDD

mv hitdata.txt ./

cat hitdata.txt | perl -ne'
  if (/^Q/) {
    print "$_";
  }else {
    next
  }
' > hitdata.tsv

cat hitdata.tsv | grep -i "actin" | wc -l
# 66

cat hitdata.tsv | grep -i "actin" | cut -f 1 | cut -d ">" -f 2 > CDD_filter.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f CDD_filter.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 26
```

+ 根据PFAM提供的种子序列进行blastp检索

![](./Fig/PFAM_seed.png)

下载种子序列文件PF00022_seed.txt

```bash
mkdir new_blastp
cd new_blastp

mv PF00022_seed.txt ./
mv PF00022_seed.txt PF00022_seed.fa

makeblastdb -in ../sequence/Homo_sapiens.GRCh38.pep.all.fa -dbtype prot -parse_seqids -out ./index
blastp -query ./PF00022_seed.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result.tsv

# 统计
cat result.tsv | cut -f 2 | sort | uniq | wc -l
# 69蛋白
cat result.tsv | cut -f 2 | sort | uniq > protein_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 28基因
```
第二轮
```bash
faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa protein_ID.lst seed2.fa

blastp -query ./seed2.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result2.tsv

cat result2.tsv | cut -f 2 | sort | uniq | wc -l
# 77
cat result2.tsv | cut -f 2 | sort | uniq > protein2_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein2_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 32
```
第三轮
```bash
faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa protein2_ID.lst seed3.fa

blastp -query ./seed3.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result3.tsv

# 统计
cat result3.tsv | cut -f 2 | sort | uniq | wc -l
# 86
cat result3.tsv | cut -f 2 | sort | uniq > protein3_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein3_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 33
```
第四轮
```bash
faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa protein3_ID.lst seed4.fa

blastp -query ./seed4.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result4.tsv

# 统计
cat result4.tsv | cut -f 2 | sort | uniq | wc -l
# 106
cat result4.tsv | cut -f 2 | sort | uniq > protein4_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein4_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 38
```
第五轮
```bash
faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa protein4_ID.lst seed5.fa

blastp -query ./seed5.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result5.tsv

# 统计
cat result5.tsv | cut -f 2 | sort | uniq | wc -l
# 114
cat result5.tsv | cut -f 2 | sort | uniq > protein5_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein5_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 40
```
第六轮
```bash
faops some ../sequence/Homo_sapiens.GRCh38.pep.all.fa protein5_ID.lst seed6.fa

blastp -query ./seed6.fa -db ./index -evalue 1e-10 -qcov_hsp_perc 60 -outfmt 6 -num_threads 6 -out result6.tsv

# 统计
cat result6.tsv | cut -f 2 | sort | uniq | wc -l
# 117
cat result6.tsv | cut -f 2 | sort | uniq > protein6_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f protein6_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 41
```

## Sequence Alignment and Phylogenetic Analysis
+ 以biomart下载的结果尝试
```bash
mkdir Biomart
cd Biomart

JOB=$(ls *.fa)
for i in $JOB;do
  cat $i >> raw.fa
done

# 筛选出大于160aa的片段
faops size raw.fa | wc -l 
# 431
faops size raw.fa | tsv-filter --ge 2:160 | wc -l
# 382
faops size raw.fa | tsv-filter --ge 2:160 | cut -f 1 > 160.lst
faops some raw.fa 160.lst 160.fa
```











