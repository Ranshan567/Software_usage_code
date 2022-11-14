# 建树

```

# 检查，修复，填充缺失的信息到排序和标准化的gff3中
agat_convert_sp_gxf2gxf.pl -g 2.gff3 -o 3.gff3

awk '{if ($3 =="CDS") print $0}' 3.gff3 > 4.gff3

agat_convert_sp_gff2bed.pl --gff 4.gff3 -o 4.bed

hal4dExtract --conserved species.hal C57B6J 4.bed 4D.bed

bed12ToBed6 -i 4D.bed > 5.bed

hal2maf --onlyOrthologs --noAncestors --noDupes --unique --refGenome fasta --refTargets 6.bed species.hal mm10_4D.maf

phyloFit --tree \"二叉树的拓扑结构;\" --msa-format MAF --out-root mm10_4D.mod mm10_4D.maf

tail -n 1 mm10_4D.mod | sed 's/TREE: //g' > tree.txt

```

# 运行GERP

```

hal2maf --refGenome species --refTargets species.bed --noDupes --noAncestors --global --onlyOrthologs --unique species.hal species.maf

sed -i 2d species.maf

egrep -nr "^$" species.maf | sed 's/://g' | awk '{ print prev"\t"$0} { prev = $0 }'| sed 1d | awk '{print $1+2","$2-1}' > list.txt
for x in $(seq 1  1 $(wc -l list.txt | cut -f1 -d" ")) ; do TMP=$(sed -n ${x}p list.txt) ; sed -n ${TMP}p ${JOB}.maf | sed 's/\./\t/g' | awk '{ print $1"\t"$2"\t"$3"\t"$4"\t"$5"\t"$6"\t"$7"\t"$8}' > maf_part${x}.txt ; done

python maf2fa.py 6

sed -e 's/^/>/g' prep.txt | sed -e 's/\t/\n/g' > out.fa

gerpcol -a -f out.fa -t tree.txt -e species -j

while read line ; do bits=($line) ; for x in $(seq ${bits[1]} 1 ${bits[2]}) ; do echo $x |  awk '{print ENVIRON["Y"]"\t"$1"\t"$1+1}' >> tmp.bed ; done ; sed '$d' tmp.bed | paste - out.fa.rates >> ind.bed ; done < species.bed

cd ..

```

```
# maf2fa.py

import pandas as pd
import sys
NUM = int(sys.argv[1])
df = pd.DataFrame()
for x in range(0，NUM):
  if df.empty:
  name = "maf_part"+str(x+1)+".txt"
  Name=“maf_Part”+str(x+1)+“.txt”
  a = pd.read_csv(name, sep="\t",header=None, index_col=1)
  df = a[7].loc[~a.index.duplicated(keep='first')]
  df = df[pd.to_numeric(df, errors='coerce').isnull()]
  df = pd.concat([df], axis=1, sort=False)
elif not df.empty:
  name = "maf_part"+str(x+1)+".txt"
  a = pd.read_csv(name ,sep="\t", header=None ,index_col=1)
  b = a[7].loc[~a.index.duplicated(keep='first')]
  b = b[pd.to_numeric(b, errors='coerce').isnull()]
  df = pd.concat([df,b], axis=1, sort=False)
  
  
namer = []
for x in range(0，NUM):
  inner = "a"+str(x+1)
  namer.append(inner)

df.columns = namer


for x in namer:
  filler = "_"*int( df[x].str.len().mean())
  df[x].fillna(filler, inplace=True)

df = df.apply(lambda x: ''.join(x ), axis = 1)
df.to_csv("prep.txt", sep="\t", header=False, index=True)

```
