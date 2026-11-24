preparing t2t data:

downloading
```
cd /media/ds1821p_III/aserikzhan/repeats
mkdir -p genomes/chm13 && cd genomes/chm13
BASE=https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/009/914/755/GCF_009914755.1_T2T-CHM13v2.0
wget $BASE/GCF_009914755.1_T2T-CHM13v2.0_genomic.fna.gz \
     $BASE/GCF_009914755.1_T2T-CHM13v2.0_assembly_report.txt \
     $BASE/GCF_009914755.1_T2T-CHM13v2.0_rm.run \
     $BASE/md5checksums.txt
grep genomic.fna.gz md5checksums.txt; md5sum GCF_009914755.1_T2T-CHM13v2.0_genomic.fna.gz   # the two hashes must match
```

Converting to lowercase (to avoid soft masked sequences messing it up for me):
```
zcat GCF_009914755.1_T2T-CHM13v2.0_genomic.fna.gz | awk '/^>/ {print; next} {print toupper($0)}' > chm13v2.0.upper.fa
grep -c "[acgt]" chm13v2.0.upper.fa    # should print 0
```

Didn't run yet:
Changing chromosome names to t2t truth set:
```
grep -v "^#" GCF_009914755.1_T2T-CHM13v2.0_assembly_report.txt | awk -F'\t' '{print $7"\t"$10}' > refseq_to_chr.tsv   # RefSeq name -> UCSC name
awk 'NR==FNR{m[$1]=$2; next} /^>/{split($1,a,">"); id=a[2]; print ">" (id in m ? m[id] : id); next} {print}' refseq_to_chr.tsv chm13v2.0.upper.fa > chm13v2.0.fa
grep ">" chm13v2.0.fa    # should list chr1 ... chr22, chrX, chrY, chrM
```

Indexing it:
```
samtools faidx chm13v2.0.fa
cut -f1,2 chm13v2.0.fa.fai     # chromosome lengths; total should be ~3.11 Gb
```
