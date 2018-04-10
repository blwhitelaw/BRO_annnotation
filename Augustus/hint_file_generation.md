#this tutorial borrows heavily from 

https://sites.google.com/site/yijyunluo/Bioinformatics/Gene-prediction

http://bioinf.uni-greifswald.de/bioinf/wiki/pmwiki.php?n=Augustus.ParallelPred

http://bioinf.uni-greifswald.de/bioinf/wiki/pmwiki.php?n=IncorporatingRNAseq.Tophat

http://bioinf.uni-greifswald.de/bioinf/wiki/pmwiki.php?n=Augustus.IncorporateRepeats


##Repeat masker 

1. Use RepeatMasker to generate repeat .gff file to create hints files

2. Generate tab sep hints file
```
cat hmac_dovetail_assembly.fasta.out | tail -n +3 | perl -ne 'chomp; s/^\s+//; @t = split(/\s+/);
print $t[4]."\t"."repmask\tnonexonpart\t".$t[5]."\t".$t[6]."\t0\t.\t.\tsrc=RM\n";' | sort -n -k 1,1 > repeats_hints.gff
```

##Exon hints

1. Find
```
nohup blat -noHead -minIdentity=92 hmac_dovetail_assembly.fa hapalochlaena_maculosa_na.fasta bro-rnaseq.psl &
```

2. Filter
```
nohup /kent/pslCDnaFilter -minId=0.9 -localNearBest=0.005 -ignoreNs -bestOverlap bro-rnaseq.psl bro-rnaseq.f.psl &
```

3. Generate tab sep hints file
```
perl /usr/local/sci-irc/sw/augustus-3.3/scripts/blat2hints.pl --in=bro-rnaseq.f.psl --out=exon_hints.gff --minintronlen=35 --trunkSS
```

##Intron hints (iterative method)

#Requires accepted_hits.bam from Trinity output

1. Sort
```
nohup samtools sort -n accepted_hits.bam accepted_hits.sorted &
```

2. Filter
```
nohup filterBam --uniq  --in accepted_hits.bam --out accepted_hits.nfnp.bam &
```

3. Extract header for later
```
samtools view -H accepted_hits.nfnp.bam > header.txt
```

4. Create preliminary intron hints
```
nohup samtools sort accepted_hits.nfnp.bam both.ssf &
nohup bam2hints --intronsonly --in=both.ssf.bam --out=intron_hints_1.gff &
```

5. Copy and modify the  the file extrinsic.M.RM.E.W.cfg file to match the paremeters below. 
```
[SOURCES]
M RM E W

exonpart    1   .992    M 1 1e+100 RM 1 1    E 1 1   W 1 1.005
intron      1   .34     M 1 1e+100 RM 1 1    E 1 1e5 W 1 1
CDSpart     1   1 0.985 M 1 1e+100 RM 1 1    E 1 1   W 1 1
UTRpart     1   1 0.985 M 1 1e+100 RM 1 1    E 1 1   W 1 1
nonexonpart 1   1       M 1 1e+100 RM 1 1.01 E 1 1   W 1 1
```

6. Running augustus with intron hints
```
Augustus can be run as a default (slow) 

augustus --species=h_mac --extrinsicCfgFile=extrinsic.M.RM.E.W.cfg --alternatives-from-evidence=true --hintsfile=intron_hints.gff --allow_hinted_splicesites=atac --introns=on --genemodel=complete h_mac_genome.fasta --outfile=all_aug_out.gff --errfile=all_aug_out.err
```

or in parallel to speed up the process

To run in parallel


7. Make required directories
```
mkdir intron_hints_aug

cd intron_hints_aug
```

8. Split genome into single seq
```
nohup perl /usr/local/sci-irc/sw/augustus-3.3/scripts/splitMfasta.pl ../hmac_dovetail_assembly.fa --outputpath=/fast/jc451635/BRO_Genome/augustus/split_genome &
```

9. Rename files with fasta header
```
 for f in hmac_dovetail_assembly.split.*; do NAME=`grep ">" $f`; mv $f ${NAME#>}.fa; done
```

10. Create summmary file
```
perl /usr/local/sci-irc/sw/augustus-3.3/scripts/summarizeACGTcontent.pl ../hmac_dovetail_assembly.fa > summary.out
```

11. Make chr.lst
```
grep 'bases' summary.out | awk '{printf("split_genome/%s.fa\tbro_all_hints.gff\t1\t%s\n",$3,$1)}' >chr.lst
```
12. Create jobs/commands
```
mkdir com_scripts
mkdir intron_output

/usr/local/sci-irc/sw/augustus-3.3/scripts/createAugustusJoblist.pl --sequences=/fast/jc451635/BRO_Genome/augustus/chr.lst --wrap="#" --overlap=5000 --chunksize=1252500 \
    --outputdir=../aug_intron_output/ --joblist=jobs.lst --jobprefix=aug_intron_hints_ --command "augustus --species=h_mac --extrinsicCfgFile=extrinsic.M.RM.E.W.cfg --alternatives-from-evidence=true --hintsfile=intron_hints.gff --allow_hinted_splicesites=atac --introns=on --genemodel=complete"

mv aug_intron_hints* com_scripts/
mv split_genome com_scripts/
```

13. Run jobs
```
module load parallel
run_aug_intron_hints(){
script_name=${1}
 bash ./${script_name}
}

export -f run_aug_intron_hints

parallel -j 48 run_aug_intron_hints ::: $(ls aug_intron_hints_* | tr "\n" " ")
```

14. Merge outputs
```
cat *.gff | /usr/local/sci-irc/sw/augustus-3.3/scripts/join_aug_pred.pl > aug_intron_all.gff

or if too many arguments/files

printf '%s\0' *gff | xargs -0 cat | /usr/local/sci-irc/sw/augustus-3.3/scripts/join_aug_pred.pl > aug_intron_all.gff
```

15. Create exon-exon junction database
```
grep -P "\tintron\t" aug_intron_all.gff > aug1.introns.gff
```

16. Filter for intron honts only (precaution)
```
cat /fast/jc451635/BRO_Genome/augustus/intron_hints.gff aug1.introns.gff | perl -ne '@array = split(/\t/, $_);print "$array[0]:$array[3]-$array[4]\n";'| sort -u > introns.lst
```

17. Extraxt junctions and convert to fasta format
```
perl /usr/local/sci-irc/sw/augustus-3.3/scripts/intron2exex.pl --introns=introns.lst --seq= ../hmac_dovetail_assembly.fa --exex=exex.fa --map=map.psl
```

18. Build a bowtie database
```
nohup bowtie2-build exex.fa bro_exex1 &
```

19. Align
```
nohup bowtie2  -x bro_exex1 -U EM8_C5CC3ACXX_ACTGAT_L004_R1.fastq,EM8_C5CC3ACXX_ACTGAT_L004_R2.fastq -S bowtie.sam &
```

20. Discard failed
```
nohup samtools view -S -F 4 bowtie.sam > bowtie.F.sam &
```

21. Map the local exex-alignments to global genome level
```
perl /usr/local/sci-irc/sw/augustus-3.3/scripts/samMap.pl bowtie.F.sam map.psl > bowtie.global.sam
```

22. Join preliminary intron hints with new hints 

##Each read has been aligned twice now, to the genome and to the exon-exon sequences. From the read-to-genome alignments we only take the unspliced ones (CIGAR does not contain the letter "N"), as the spliced ones are represented in bowtie.global.sam.

23. Remove all alignments containing introns from the original bam file 
```
bamtools filter -in ../accepted_hits.bam -out accepted_hits.noN.bam -script /usr/local/sci-irc/sw/augustus-3.3/auxprogs/auxBamFilters/operation_N_filter.txt
```

24. Create a bam file with header from the bowtie.global.sam file
```
cat header.txt bowtie.global.sam > bowtie.global.h.sam
nohup samtools view -bS -o bowtie.global.h.bam bowtie.global.h.sam &
```

25. Join bam files
```
samtools merge both.bam bowtie.global.h.bam accepted_hits.noN.bam
```
26. Sort
```
samtools sort -n both.bam both.s
```

27. Filter raw alignments
```
nohup filterBam --uniq  --in both.s.bam --out both.sf.bam &
```

28. Create intron hints
```
nohup samtools sort both.sf.bam both.ssf &
nohup bam2hints --intronsonly --in=both.ssf.bam --out=intron_hints_2.gff &
```

## Merge hints files

cat intron_hints_2.gff repeats_hints.gff exon_hints.gff > all_hints.gff

##Run augustus as described in steps 7-14 (remember to update all directory names, pathes and files as appropriate)
