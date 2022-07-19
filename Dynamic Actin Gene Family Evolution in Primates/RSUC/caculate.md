# Relative Synonymous Codon Usage(RSCU)
## 定义
+ 编码同一氨基酸的密码子使用的实际频率与理论频率的比值，即为RSCU。理论频率就是编码该氨基酸的密码子种类的倒数（每个密码子出现的频率相同）

## 分类
+ 因为要计算7个物种之间与不同组（group）之间的相对同义密码子实用度，要先根据group进行分类
```bash
cd RSUC

faops size total.fa | wc -l
# 210
head group.tsv
Group_name      Gene_name
orthologous1    ACTL6A
orthologous1    ACTL6A
orthologous1    ACTL6A
orthologous1    ACTL6A
orthologous1    ACTL6A
orthologous1    ACTL6A
orthologous1    ACTL6A
orthologous2    ACTR6
orthologous2    ACTR6
```
+ 直系同源组
```bash
cat group.tsv | grep "orthologous" | cut -f 2 | uniq | wc -l
# 12
cat group.tsv | grep "orthologous" | cut -f 2 | uniq > orthologous.lst

mkdir orthologous
JOB=$(cat orthologous.lst)
for J in $JOB;do
  cat total.fa | grep ">" | cut -d ">" -f 2 | grep "$J" > ./orthologous/$J.genename.lst
done
# 检查是否都为7
wc -l orthologous/*

# 提取序列
JOB=$(ls orthologous)
for J in $JOB;do
  NAME=$(echo $J | cut -d "." -f 1)
  faops some total.fa ./orthologous/$J ./orthologous/$NAME.fa
done  
```
+ complex组
```bash
mkdir complex
JOB=$(cat group.tsv | grep "complex" | cut -f 1 | uniq)
for J in $JOB;do
  cat group.tsv | grep -w "$J" | cut -f 2 > ./complex/$J.lst
done
# 检查
wc -l complex/*

# 提取序列
JOB=$(ls complex)
for J in $JOB;do
  NAME=$(echo $J | cut -d "." -f 1)
  faops some total.fa ./complex/$J ./complex/$NAME.fa
done  
```

## 计算









