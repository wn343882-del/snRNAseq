qsub -cwd -binding linear:2 -l vf=5g,num_proc=2 -P P21Z28400N0234 -l hostname=bjcas-compute-2-4 step2_optimze_para.sh
补充：seurat常用命令Reductions(obj)查看维度；head(obj@meta.data$Batch)

###################################################################################################################
数据背景介绍：
数据存储路径格式如下/jdfsbjcas1/ST_BJ/P21Z28400N0234/wangning12/1_project/4-multiOmics/RNA/rawdata/cot/cotC250416021/04.Matrix
Matrix数据里包含三个文件，分别是barcodes.tsv（一列，CELL1_N2，代表单个核数据），features.tsv（一列，AT1G01470的基因号），matrix.mtx(Matrix Market 标准格式下的coordinate = 稀疏矩阵，计数值为整数，三列列，第一列是行数（features数据里包含的基因数目，第二个barcode细胞核数，第三列非零元素个数表达过的基因×细胞，意义： 1 2 1 第一个基因在第二个细胞核里的count=1)
查看集群数据前几行数据zcat matrix.mtx.gz | head -n 30
#################################################################################################################
😉00-seurta对象的基础认知：slot都有什么，数据结构长什么样
😉step0-fq.gz文件借助云平台标准流程进行质量报告输出，输出文件为基因矩阵
😉step1-分时期整合文库，形成三个seurat的rds文件（tor bent cot)，此处可以看每个数据的基本情况（线粒体叶绿体比例，UMI，features-基因数量）
😉step2--整合所有时期的rds数据，形成mergerds文件mergeSilique，此处对rds进行质控（标准如下方法所示），snRNA数据分析基本流程（we followed the standard Seurat pipeline applying the NormalizeData, FindVariableFeatures, ScaleData and RunPCA with default parameters to create a joint principal component space.）以及去双胞处理，这里发现tor批次效应过于显著，需要对数据进行harmony整合；该rds没有去除双胞，但是在scRNA@meta$Doublet列有标注，/jdfsbjcas1/ST_BJ/P21Z28400N0234/wangning12/1_project/4-multiOmics/RNA/results/mergeSilique/silique_3stage_after_QC_doublet.rds.rds      
😉step2.0可以对双胞进行数据性检验（UMI和feature是否符合双胞特点，有双胞，可能导致UMAP图的两簇之间的粘连）---这个放在0认识seurat数据命令的后面了
😉step2.1 通过ElbowPlot / JackStraw / variance explained决定PCA的维度大小，拐点处明显平滑；
😉step3-对数据进行harmony处理去批次效应，注意harmony不是对counts矩阵做，而是对降维后的PCA数据作处理，counts  → PCA → Harmony → neighbors / cluster / UMAP
😉step4--确定聚类的最佳参数比，使用resolution树决定。(方法部分暂无）--做到这部分可以和snATAC数据进行整合分析了
😁steo5--根据质量控制确定的dims resolution重新进行聚类分析，展示分簇结果。
###########################################################################################################
😉step5--细胞类型注释，重点注意标记基因的选择（非常重要）以及Dotplot图的解读
方法描述
云平台流程描述（from chenchuan,2025,cell)略有修改
snRNA-seq data were processed to obtain the read count matrices for each gene and each cell via an open-source pipeline (https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software). First, raw sequencing reads were filtered (removing reads with an average base quality score lower than 4, more than 2 bases with a quality score lower than 10, or containing N bases or improper barcodes) and demultiplexed by barcode assignment. The obtained reads were subsequently aligned to the arabidopisis reference genome (TAIR10) using Spliced Transcripts Alignment to a Reference (STAR) (Dobin et al., 2013) and annotated to the gene set (Gmax_ZH13_V2.1) via a preprocessing and interative suite for single-cell data analysis (Shi et al., 2022). Valid cells were automatically identified based on the UMI number distribution of each cell via the “barcodeRanks” function of the DropletUtils tool (Lun et al., 2019) to remove background beads and beads with UMI counts less than 500. A preprocessing and interative suite for single-cell data analysis (Shi et al., 2022) was then applied to calculate the gene expression of cells and create a gene × cell matrix for each library. 
后续转到集群进行分析(结合陈钏老师，2025，cell以及xiaofeng gu et al; 2022水稻多组学methods）
Raw count matrix data were imported into R using the Seurat (v4.4.0) package for further data analysis. Different libraries were  merged via the “Merge” function of the R package Seurat(v4.4.0). Low-quality cells were removed by filtering out cells that had fewer than 500 genes or 1,000 UMIs.线粒体基因小于2%，叶绿体基因小于30%。 Doublets and multiplets were filtered out using DoubletFinder (v2.0.3)38. The parameters of DoubletFinder (nExp (pANN (proportion of artificial k nearest neighbors) threshold used to make final doublet and singlet predictions) and pK (PC neighborhood size used to compute pANN)) used to build artificial doublets for true doublet classification were determined automatically using recommended settings (parameters: PC (number of statistically significant principal components) = 1:30, pN (number of generated artificial doublets) = 0.25).
Finally, expression data from 8 libraries of the three stages(tor,bent and cot) were merged via the “Merge” function of the R package Seurat (v.4.1.0) , which was further normalized via the “NormalizeData” function with default parameters. After log normalization via the “ScaleData” function, 1500 highly variable genes were selected via the “FindVariableFeatures” function. For subsequent clustering and visualization, a dimensionality reduction algorithm based on principal component analysis was used to extract 30 principal components for Lovain clustering via the “FindNeighbors” function, with the resolution set to 0.5 (待定) . The clustering results were projected to two-dimensional space via the “RunUMAP” function.
细胞类型注释




