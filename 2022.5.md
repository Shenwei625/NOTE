# 笔记
## 5月9日
### sed -i 's/指定的字符/要插入的字符&/' 文件
```bash
sed -i 's/[A-Z]:[A-Z]/:&/' LOVD.tsv #在指定的字符前面插入字符
```

### R绘制分布图
```R
library(ggplot2)
HISTABLE <- "F.EXON.tsv"
data <- read.table(HISTABLE,header=TRUE,sep = "\t")
# 单独做图
p1 <- ggplot(data, aes(x=EXON,fill=group)) +
+ geom_histogram(binwidth = 1,colour="black", position="identity")
# 或(更改y后面的参数count或者density可以选择作图的形式)
ggplot() +
+ geom_histogram(aes(x=EXON,fill=group,y=..density..), data, binwidth = 1,colour="black", position="identity") +

# 合并作图
p <- ggplot() +
+ geom_histogram(aes(x=EXON,fill=group,y=..density..), data, binwidth = 1,colour="black", position="identity", fill="#00F5FF") + # 设置颜色
+ geom_histogram(aes(x=EXON,fill=group,y=-..density..), data2, binwidth = 1,colour="black", position="identity")

# 修饰
p + scale_x_continuous(breaks = seq(55)) + ylab("density") + theme(panel.grid = element_blank())# 坐标轴刻度+y标签+删去网格线

EXON <- data[,1]
ggplot() +
+ geom_histogram(aes(x=EXON,fill=group,y=..density..), data, binwidth = 1,colour="black", position="identity") +
+ geom_vline(aes(xintercept=median(EXON, na.rm=T)),color="black", linetype="dashed", size=.5) # 添加中位数线           
```





## 5月11日
### 绘制树形图
+ 可以利用paste()函数来构建一些变量
```R
install.packages("treemapify")
library(treemapify)
library(ggplot2)
library(patchwork) # 可以将多张图组合在一起(p1 + p2左右组合，或者p1 / p2上下组合)

COUNTF <- "F.Amino_acids.tsv" 
TREEDATA <- read.table(COUNTF,header=TRUE,sep = "\t")
ggplot(TREEDATA, aes(area = count, fill = Amino_acids, label = paste(Amino_acids, count,sep = "," ),subgroup = Amino_acids)) +  
# area后面添加计数列，fill后面添加被统计的列
    geom_treemap() +
    geom_treemap_text(colour = "white",place = "bottomleft",size = 10) +   
# 添加标签,place设置位置（top,bottom.center,left,rght）,前面的label选项可以设置标签内容
    geom_treemap_subgroup_border(colour = "white", size = 3) 
# 设置每个区块之间的间隔，对应于前面的subgroup

ggsave(filename = <variants>,width=20, height=10, dpi=300) # 保存图片，filename后面可以直接使用变量

labs(title = "content") # 设置图片标题，filename后面可以直接使用变量

theme(legend.position = "none",plot.title=element_text(hjust=0.5)) # 删除图例,标题居中
```

### 绘制饼状图
+ 通过ggplot2利用极坐标将柱状图折叠成饼状图
```R
library(ggplot2)
FILE1 <- "T.Consequence.tsv"
PIEDATA <- read.table(FILE1,header=TRUE,sep = "\t")
TITLE1 <- "T.Consequence"
J <-"Consequence"
PLABEL = as.vector(PIEDATA[,J])
myLabel = as.vector(PIEDATA[,J])
myLabel = paste(myLabel, "(", round(PIEDATA$count / sum(PIEDATA$count) * 100, 2), "%" , ")") # 重新定义标签

ggplot(PIEDATA,aes(x="", y=count, fill=get(J))) +  #只需要一个柱子所以x="",y为计数列，fill为分类
    geom_bar(stat = "identity", width = 1) +
    coord_polar(theta = "y") +   # 把柱状图折叠成饼状图
    labs(x = "", y = "", title = TITLE1) + # 删去饼图旁边的标签，原柱状图x,y轴坐标
    theme(axis.ticks = element_blank(), plot.title=element_text(hjust=0.5)) + # 删去坐标轴刻度,标题居中
    theme(axis.text.x = element_blank()) + # 删去x轴标签
    theme(panel.grid = element_blank()) + # 删除网格线
    theme(panel.background = element_rect(fill = "transparent",colour = NA)) + # 删除灰色的背景
    theme(legend.title = element_blank(), legend.position = "bottom") + # 删除图例标题
    scale_fill_discrete( breaks = PIEDATA[,J], labels = myLabel ) + # 重新更换图例中的标签
    theme(legend.text=element_text(size=10)) + # 图例中字体大小
    geom_text(aes(2, label = myLabel),position = position_stack(vjust = 0.5),size = 4) # 在饼图中添加标签
```





## 5月13日
+ 将文件的奇数行和偶数行分开
```bash
sed '2~2d' file # 奇数行
sed '1~2d' file # 偶数行
```


## 5月17日
+ R中获取某列名称的方式
```R
colnames(PIEDATA1[1])
# 方括号中为第几列
```
+ R中利用正则表达式捕获特定文件名称
```R
list.files(path = ".", pattern = "*.lst")
# pattern后面接表达式
```

### Logistic Regression ###

+ R Squared

概念：拟合优度，回归直线对观测值的拟合程度。

+ P Value

概念：当原假设为真时，比所得到的样本观察结果更极端的结果出现的概率。

p 值是用来评估自变量参数的统计学显著性的一个统计量，即可用来评估该自变量与因变量之间的统计关系是否显著。一般常用的p 值阈值是0.05，p 值越小，检验结果越显著。在Logistic 回归分析中，我们通常使用Wald test 计算得到各个自变量参数的p 值，从而依此判断该自变量与因变量之间的联系是否显著。



+ logistic

特征
>预测事物是对还是错（二元或多元变量），而不是预测连续的事物（线性回归范畴）

>maximum likeihood:最大似然，类似于线性回归中的R方（由于因变量和自变量之间不存在线性关系）

>Logistic回归模型的因变量为二分类变量

>Logistic回归没有关于自变量分布的假设条件，可以是连续变量、离散变量和类别变量

与结果有关的相关概念

>标准误差：均方根误差。√[n个测量值的误差（测量值与真实值的差）的平方和 / n]

>标准差:方差（每个样本值与全体样本值的平均数之差的平方值的平均数）的算术平方根。标准差能反映一个数据集的离散程度。平均数相同的两组数据，标准差未必相同

>标准误：样本平均数的标准差。可用于衡量抽样误差的大小

>标准偏差：度量数据分布的分散程度的标准。√[n个测量值的偏差（测量值与平均值的差）的平方和 / n-1]

>B值：回归系数，自变量对因变量影响大小的参数

>瓦尔德（Wald卡方值）：对回归系数b值的假设检验，z值（回归系数/标准误）的平方

>Exp（B）（OR值）：e^b值

+ 利用R进行logistic回归分析
```R
glm(Pathogenicity~cDNA_position,data = DATA1,family="binomial")
# 将自变量写在~之后，因变量在~之前，binomial表示二项分布

glm(Pathogenicity~.,data = DATA1,family="binomial")
# ~之后用.可以考虑所有的自变量

# 删除不需要的自变量,X表示不需要的自变量的列，相当于在一个新的数据框中考虑所有自变量
glm(Pathogenicity~.,data = DATA1[,-X],family="binomial")


# 逐步法筛选自变量
mod.none <- glm(Pathogenicity~1,data = DATA1,family="binomial") #建立一个空模型
mod.full <- glm(Pathogenicity~.,data = DATA1,family="binomial") #建立一个全模型
step(mod.none, scope=formula(mod.full),direction='both',trace=F)
# 或者
-step(object = mod.full,trace = 0)
```
+ 建立回归模型之后的参数计算
```R
  #取P值
  p<-summary(mod)$coefficients[,4]
  #wald值
  wald<-summary(mod)$coefficients[,3]^2
  #B值
  valueB<-coef(mod)
  #OR值
  valueOR<-exp(coef(mod))
  #OR值得95%CI
  confitOR<-exp(confint(mod))
  data.frame(
    B=round(valueB,3),
    Wald=round(wald,3),
    OR_with_CI=paste(round(valueOR,3),"(",
               round(confitOR[,1],3),"~",round(confitOR[,2],3),")",sep=""),
    P=format.pval(p,digits = 3,eps=0.001)
  )
```



## 5月18日
+ sed正则表达式中使用变量
```bash
sed 's/get(C)/'$C'/' 1.tsv # 将变量用但引号括上
```



## 5月23日
+ perl一行式参数
```bash
perl -n #将参数当作文件读取进来，并且每次读取一行遍历文件。
perl -p #将参数当作文件读取进来，并且每次读取一行遍历文件。参数不管你的代码中有什么判断语句来控制输出，每读取一行都会输出一下$_
perl -a #将读入的$_进行分割，保存到@F列表之中，类似于split /分隔符/ , $_; 而这个分隔符是由-F参数指定的
perl -F"" #在添加-a参数时候，指定分隔符（可以是正则表达式）
perl -Mmodulename #导入模块或者编译指示（Pragmas）到Perl单行程序中

BEGIN与END #BEGIN，END，它们可以分别包含一组脚本，用于程序体运行前或者运行后的执行。
```

+ perl模块PATH::Tiny的使用，将文件转化为字符串,perl一行式
```bash
cat template.md | 
  perl -n -MPath::Tiny -e '
  $data = path("max.tsv")->slurp; # 利用Path::Tiny进行转换
  s/maxtsv/$data/; # 修改
  print "$_";
  ' > tem&&
    mv tem total.md
```




## 5月25日
+ 利用sed显示指定的行
```bash
sed -n 'np' file # 单引号中的n为特定的行
```




## 5月26日
+ 利用RColorBrewer包进行颜色更换
```R
library(ggplot2)
library(RColorBrewer)
#创建数据集
x<-matrix(1:200,nrow=20,ncol=10)
x1<-x[,1:2]
colnames(x1)<-c("sample","num")
data<-data.frame(x1) 
data$sample<-as.factor(data$sample)
#将第一列定为样本类别，第二列定义为样本数量
#展示所有色板
display.brewer.all()
#展示"Set3"色板中的9个颜色
display.brewer.pal(9,'Set3')
brewer.pal(9,'Set3')
#统计sample的数量 20
colorcount = length(data$sample)
samplecolor=colorRampPalette(brewer.pal(9,'Set3'))
#使用palette控制scale_fill_brewer()中的颜色选择
ggplot(data,aes(x=sample,y=num,fill=sample))+geom_bar(stat = "identity")+scale_fill_manual(values=samplecolor(colorcount))
```

+ ggplot调整字体大小
```R
theme(text=element_text(size=15), axis.text=element_text(size=15, face = "bold")) 
# axis.text设置坐标轴字体大小，"bold"加粗字体
theme(legend.text=element_text(size=15)) # 设置图例字体大小
```



### 5月31日
+ 在bash中利用printf格式化输出（科学技术法）
```bash
printf "%*.*f\n" 16 15 $MEDIAN_RAW
```

