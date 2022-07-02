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
```bash
mkdir blastp
cd blastp
mv human.fa blastp

# blastp
makeblastdb -in ./human.fa -dbtype prot -parse_seqids -out ./index # 建立索引
blastp -query ../sequence/Homo_sapiens.GRCh38.pep.all.fa -db ./index -evalue 1e-10 -outfmt 6 -num_threads 6 -out result.tsv

# 统计
cat result.tsv | cut -f 1 | sort | uniq | wc -l
# 626
cat result.tsv | cut -f 1 | sort | uniq > pretein_ID.lst
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f pretein_ID.lst | wc -l
# 626
cat ../sequence/Homo_sapiens.GRCh38.pep.all.fa | grep ">" | grep -f pretein_ID.lst | cut -d " " -f 4 | # 查询基因ID
  sort | uniq | wc -l
# 169
```

