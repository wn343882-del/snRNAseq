数据背景介绍：
数据存储路径格式如下/jdfsbjcas1/ST_BJ/P21Z28400N0234/wangning12/1_project/4-multiOmics/RNA/rawdata/cot/cotC250416021/04.Matrix
Matrix数据里包含三个文件，分别是barcodes.tsv（一列，CELL1_N2，代表单个核数据），features.tsv（一列，AT1G01470的基因号），matrix.mtx(Matrix Market 标准格式下的coordinate = 稀疏矩阵，计数值为整数，三列列，第一列是行数（features数据里包含的基因数目，第二个barcode细胞核数，第三列非零元素个数表达过的基因×细胞，意义： 1 2 1 第一个基因在第二个细胞核里的count=1)
查看集群数据前几行数据zcat matrix.mtx.gz | head -n 30
此步骤目的：
1，写入数据
2，根据数据质量报告提前排除异常文库

step1
