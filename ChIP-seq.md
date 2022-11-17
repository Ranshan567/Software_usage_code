1、准备文件
原始测序数据
参考基因组
参考基因组index（可以通过samtools、bwa 、bowtie2 构建 index）
基因注释文件
2、#通过bwa比对到参考基因组(未能比对上，任务一直在跑，但是没有结果。20220719)
通过Bowtie2比对到参考基因组
3、peak-calling可以鉴定基因组中显著富集位点（peaks）。peak-calling结果通常以BED格式呈现。
MACS2是最常用的peak-calling工具![image](https://user-images.githubusercontent.com/93338266/202445479-81349c75-8c43-47b9-b73e-2a6d2cf4c0dd.png)
