# 建树

```

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

# 
