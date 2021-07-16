# Annotate genes 

> This tool annotates genes with alias, symbol, full name, function also related papers.
>
> 目的就是：与基因id相关的操作（如转换、可视化交集等）、分析（如富集分析），都可以加进来（后期考虑加入网页版）

## Table of Contents

-   [Installation](#installation)
-   [Features](#features)
-   [Plans](#plans)
-   [Let's mining data!](#lets-mining-data)
-   [Let's plot!](#lets-plot)
-   [Supplement](#supplement)

## Installation

You can also install devel version of **AnnoGenes** from github with:

``` r
# install.packages("remotes")
remotes::install_github("GangLiLab/AnnoGenes")
```

If you want to build vignette in local, please add two options:

``` r
remotes::install_github("GangLiLab/AnnoGenes", build_vignettes = TRUE, dependencies = TRUE)
```



## Features

#### 信息获取 (Search)

- genecards虽然全，但是搜索数量有限制，于是整合了Ensembl 数据库和 `orgdb`中的基因信息 => `genInfo`

  - 与Ensembl数据库保持同步，目前更新到v104 
  - 做了`biomart`的数据接口，可以扩展其中各种数据（序列数据由于太长，不支持该函数直接显示；会有相应的序列函数去获取）
- 整合了相关的文献信息，可以自定义搜索关键词 => `genPubmed` 

#### 数据整理与转换（Tidy & Trans）

- 基因ID转换 => `transId`  

#### 数据分析（Analyse）

- 有了基因的id和对应的logFC（需要排序好），就可以做GSEA => `genGSEA`
- 有了基因id，就能做GO分析 => `genGO ` 
- 有了基因id，就能做KEGG分析 => `genKEGG`
  - 自己拿基因去做富集分析结果为数据框，并且新增一列：`FoldEnrich`
  - 拿网页结果，依然可以调整为特定格式 => `as.enrichdat`

#### 可视化（Visualize）

- 气泡图 => `plotEnrichDot ` 
- 交集韦恩图 =>`plotVenn` 

#### 导出结果 (Export)

- 每个操作都能得到一个数据框，可以继续探索，也可以作为不同的sheets导出到同一个excel => `expo_sheet`



## Plans

##### 信息获取 (Search)

- [x] `genInfo` 的`orgdb`数据根据每个物种保存为rda，以便快速加载【总共支持12种bioconductor org】

- [ ] `genInfo` 输入name是gene alias：如果有对应的symbol，那么symbol列就写输入的name，其他列用标准symbol对应的列；如果没有对应的alias，那么其他列就是NA

- [x] `genInfo`增加基因位置 【之前通过下载分析GTF，但现在用`biomart`接口更快更方便】

- [ ] `genInfo`支持多个不同版本的基因组 => 可以参考`liftover`

- [x] `genInfo`与biomart的融合

  同时也发现**一个很有趣的事情**：标准命名人类的HGNC和小鼠的MGI都是以Ensembl数据库中的alias为准，而genecards用的是ncbi gene数据库的alias，为啥呢？其实看它们的创建国就知道了：GeneCards，是由以色列威兹曼研究院和美国Lifemap 生物科技有限公司；HGNC是EBI和剑桥大学联合；MGI是Jackson Lab，位于美国，但它比较倾向于Ensembl

  不过这两个我都加入了`genInfo`中，分别是`ensembl_alias` 和`ncbi_alias` 

- [ ] 可以增加基因以及对应蛋白的序列 => `genSeq` ?

- [ ] `auto_install`增加镜像选择

##### 数据整理与转换（Tidy & Trans）

- [x] ID转换`transId` 允许错误的id匹配，结果为NA，并且提交的顺序和结果的顺序一致

##### 数据分析（Analyse）

- [ ] 设置自己的示例数据，like：`data(geneList, package="AnnoGenes")`

##### 可视化（Visualize）

- [x] 增加genVenn，先做成数据框结果。然后如果多于五组比较，就做成usetplot图
- [x] 图片的y轴label折叠（比如dotplot的y轴有很多的term，且长度不一，如果出现太长的term，最好可以折叠一下）=> `strwrap()`
- [x] 设定特定的作图格式，比如dotplot可以支持任何网站的结果，只要满足我们的作图格式`as.enrichdat`

##### 导出结果 (Export)

- [ ] ~~图片也能导入excel（后期再看看这个有没有意义）~~
- [ ] 想到一个R包名称：`genepedia` （看看以后会不会使用它）



## DEBUG

- [x] `genGO`的use_symbol参数不管用 （原因：如果提供的已经是symbol，那么就忽略了这个参数）
- [ ] `genInfo` 如果有symbol对应不到ncbi的name，那么就找ncbi 和 ensembl alias的对应



## Let's mining data!

#### Example gene ID

```R
mm_id =c('Ticam2
Arhgap33os
Insl3
Myo15
Gal3st2b
Bloc1s1') 
mm_id=str_split(mm_id,"\n")[[1]]
```

#### Method 1: All things about gene ID

- **AUTO** detect orgnism name (e.g. `human/hs/hg`  is fine)
  - support 12 organisms (maybe more in the future...)
  - use `biocAnno(org)`  to get data
- **AUTO** detect duplicate ID, ID alias or wrong spelled ID
- Make sure the input order is identical with the output

```R
# in this example, BCC7 is the alias of TP53; SXHFJG is a fake name
id = c("MCM10",  "CDC20",  "S100A9", "FOXM1",  "KIF23",  "MMP1",   "CDC45",  "BCC7" ,  "SXHFJG", "TP53"  )
genInfo(id, org = 'human')
```

![](https://jieandze1314-1255603621.cos.ap-guangzhou.myqcloud.com/blog/2021-07-13-145933.png)



#### Method 2: Search pubmed 

```R
test2=genPubmed(mm_id, keywords = 'stem cell', field = 'tiab')
# Search example: Ticam2 [TIAB] AND stem cell [TIAB] 

# or use much specific keyword
genPubmed(mm_id, keywords = 'stem cell AND epithelial', field = 'tiab')
```

![](https://jieandze1314-1255603621.cos.ap-guangzhou.myqcloud.com/blog/2021-06-29-081925.png)

#### Method 3: GSEA

- ~~之前的操作~~

  ```R
  # 加载示例数据
  data(geneList, package="DOSE")
  # 获得msigdb的gene set
  msigdb <- getMsigdb(org='human', category='C3',subcategory = 'TFT:GTRD')
  # 直接进行gsea
  egmt <- genGSEA(genelist = geneList,geneset = msigdb)
  # 如果是extrez id，可以用下面的函数将id变成symbol
  egmt2 <- DOSE::setReadable(egmt, OrgDb = org.Hs.eg.db, keyType = 'ENTREZID')
  ```

- 目前已经将`getMsigdb` 整合进`genGSEA `， 和GO、KEGG一样，提供一个物种名称即可，比如人类可以是`human/hs/hsa/hg`
  并且和GO、KEGG一样，增加了`use_symbol`参数

```R
# 加载示例数据
data(geneList, package="DOSE")
# 直接进行gsea
genGSEA(genelist = geneList,org = 'human', category='C3',subcategory = 'TFT:GTRD',use_symbol = F)
```

![](https://jieandze1314-1255603621.cos.ap-guangzhou.myqcloud.com/blog/2021-07-06-073517.png)

#### Method 4: GO

- 函数需要用到物种的`org.db`，**如果没有相关物种注释包**，函数内部的`auto_install()` 会帮助下载👍

```R
data(geneList, package="DOSE")
id = names(geneList)[1:100]
ego = genGO(id, org = 'human',ont = 'mf',pvalueCutoff = 0.05,qvalueCutoff = 0.1 ,use_symbol = T)
head(ego)
tmp=as.data.frame(ego)
```

![](https://jieandze1314-1255603621.cos.ap-guangzhou.myqcloud.com/blog/2021-07-02-035433.png)

**不知道物种名称？别怕！**

```R
biocOrg_name()
# full_name short_name
# 1   anopheles         ag
# 2      bovine         bt
# 3        worm         ce
# 4      canine         cf
# 5         fly         dm
# 6   zebrafish         dr
# 7    ecolik12      eck12
# 8  ecolisakai    ecSakai
# 9     chicken         gg
# 10      human         hs
# 11      mouse         mm
# 12     rhesus        mmu
# 13      chipm         pt
# 14        rat         rn
# 15        pig         ss
# 16    xenopus         xl
```



#### Method 5: Transform gene id

- `org` support many species from `biocOrg_name()` 
  - support common name (e.g. `human/hs/hg`, `mouse/mm`, `dm/fly`) ...
- user can choose output dataframe or not => using `return_dat`
- **AUTO detect** input gene id type

```R
library(AnnoGenes)
data(geneList, package = 'DOSE')
id = names(geneList)[1:5]
id
# "4312"  "8318"  "10874" "55143" "55388"

## trans ID is very easy!
transId(id, trans_to = 'symbol',org='hs', return_dat = T)
# 100% genes are mapped from entrezid to symbol
# entrezid symbol
#  4312   MMP1
#  8318  CDC45
# 10874    NMU
# 55143  CDCA8
# 55388  MCM10
```

If there are some ID could not transform to another type (like "type error ID", "entrez ID has no symbol/ensembl"), the output will show as NA, while keep the same order with the input

```R
# the id "23215326", "344263475" and "45" are fake, while "1" and "2" are real
fake_id = c(id,'23215326','1','2','344263475','45')

res = transId(fake_id, trans_to = 'sym',org='human', return_dat = T)

## input and output order is identical, even there are many NAs!
identical(fake_id, res$entrezid)
```

![](man/figures/example5.png)



Also, transform from symbol to entrez or ensembl is very easy...

```R
transId(na.omit(res$symbol), trans_to = 'ens',org='hs', return_dat = T)
transId(na.omit(res$symbol), trans_to = 'entrez',org='hg', return_dat = T)
```

![](man/figures/example6.png)

However, if user provides wrong orgnism, the function will report error...

```R
## try to trans human id to symbol, but choose wrong org (mouse)
transId(id, trans_to = 'sym',org='mouse', return_dat = F)
# Error in .gentype(id, org) : Wrong organism! 
```

Compare `AnnoGenes::transId` and `clusterProfiler::bitr`

![](man/figures/example8.png)



#### Method 6: KEGG

```R
ids = names(geneList)[1:100]
gkeg <- genKEGG(ids, org = 'human') 
# org可以是common name（如human、mouse），也可以是hg、hs等常见的称呼

# 默认支持readable 参数，结果以symbol name展示
keg_raw <- genKEGG(test, org = 'hs', use_symbol = F)
keg_readable <- genKEGG(test, org = 'hs', use_symbol = T)
# 差别就是：
```

![](man/figures/example7.png)



换个物种试试~理论上，**拿任意物种的symbol、entrez、ensembl基因，给函数投食即可**。不需要再提前进行id转换了

```R
# 小鼠基因为例
head(id)
[1] "Adora1"    "Insl3"    "AF067061"      "Alpk1"         "Arhgap20"      "B020004J07Rik" "Bmp6"
keg <- genKEGG(mm_id, org = 'mouse', use_symbol = T, pvalueCutoff = 1, qvalueCutoff = 1, maxGSSize = 3000)
```



## Let's plot!

#### P1: Enrichment dotplot =>  `plotEnrichDot ` 

- 默认按照 `FoldEnrich + p.adjust`
- 目前可以将大多数富集分析结果转换为作图需要的数据框：`as.enrichdat` 
  - 支持R包：clusterP
  - 支持网页：[panther](http://geneontology.org/)、
- 支持定义主图和legend的字体及大小；是否去除网格线、文字、图例；自定义渐变色的顶部和底部颜色；设定x轴起点；折叠y轴title；边框和刻度线宽度

```R
# First, feed any dataframe result to enrichDat 
test = as.enrichDat(test)
ego = as.enrichDat(ego)

# Second, easy plot
p1 = plotEnrichDot(test,legend_by = 'qvalue'))
p2 = plotEnrichDot(ego)

# Third, if you want to change more on plot...
# test dataframe was from GeneOntology panther web result
p3 = plotEnrichDot(test, xlab_type =  'FoldEnrich', legend_by = 'qvalue',
              show_item = 15, main_text_size = 14,legend_text_size = 10,
              low_color = 'red', high_color = 'blue',
              xleft = 0, font_type = 'Arial', remove_grid = T,
              wrap_width = 30,border_thick = 3 )

# ego dataframe was from clusterP result
p4=plotEnrichDot(ego, xlab_type =  'GeneRatio', legend_by = 'p.adjust',
                 show_item = 10, main_text_size = 14,legend_text_size = 10,
                 low_color = 'orange', high_color = 'green',
                 xleft = 0, font_type = 'Times New Roman', remove_grid = F,
                 wrap_width = NULL ,border_thick = 1)

library(patchwork)
wrap_plots(list(p1,p2,p3,p4))+ plot_layout(ncol = 2) + 
  plot_annotation(tag_levels = 'a')
```

![](man/figures/example3.png)



#### P2: Venn plot =>  `plotVenn ` 

- 如果venn_list的长度大于4，那就默认使用`UpSet plot`；否则使用常规的venn plot
- venn plot可以调整：透明度、字体大小、边框粗细/有无、颜色
- upset plot可以调整：字体大小、边框粗细、内部网格线

```R
library(AnnoGenes)
library(dplyr)
library(patchwork)

set1 <- paste(rep("gene" , 100) , sample(c(1:1000) , 100 , replace=F) , sep="")
set2 <- paste(rep("gene" , 100) , sample(c(1:1000) , 100 , replace=F) , sep="")
set3 <- paste(rep("gene" , 100) , sample(c(1:1000) , 100 , replace=F) , sep="")
set4 <- paste(rep("gene" , 100) , sample(c(1:1000) , 100 , replace=F) , sep="")
set5 <- paste(rep("gene" , 100) , sample(c(1:1000) , 100 , replace=F) , sep="")

two_gene_list = list(gset1 = set1, gset2 = set2)
sm_gene_list = list(gset1 = set1, gset2 = set2, gset3 = set3)
la_gene_list = list(gset1 = set1, gset2 = set2, gset3 = set3, gset4 = set4, gset5 = set5 )

p1 = plotVenn(two_gene_list, alpha_degree = 1, border_thick = 0)

p2= plotVenn(sm_gene_list,alpha_degree = .3, border_thick = 1)
p3 = plotVenn(sm_gene_list,text_size = 2,alpha_degree = 1,
              remove_grid = T, color = ggsci::pal_lancet()(3))

p4 = plotVenn(la_gene_list,use_venn = F,
              text_size = 10, border_thick = 2,remove_grid = T)

(p1+p2+p3)/p4
```

![](man/figures/example4.png)





## Supplement

- support pipe ` %>% ` 

```R
library(openxlsx)
wb <- createWorkbook()
wb <- expo_sheet(wb, sheet_dat = test1, sheet_name = 'genInfo') %>% 
  expo_sheet(., sheet_dat = test2, sheet_name = 'genPub')
saveWorkbook(wb, "~/Downloads/test.xlsx", overwrite = T)
```

<img src='man/figures/example1.png' align="below" />







## References

### Things about GSEA

- https://www.biostars.org/p/132575/
- https://www.biostars.org/p/367191/
- https://www.gsea-msigdb.org/gsea/doc/GSEAUserGuideFrame.html

