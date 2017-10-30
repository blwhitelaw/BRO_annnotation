## Training augustus 

```
Training set .gff3 produced from PASA is incompatible with the current version of 
augustus and cannot be used as a hints file. However, augustus can still be trained
by creating a custom "species" using the RNA based training file produced from the PASA pipeline

Perl scrips provided in the augusts package will be required
```

## Preperation

1. Convert gff3 to genbank format 
```
Each gene will be coverted into a LOCUS along with specified unstream and downstream flanking reions, in this case 1000bp

perl ~/bro_annotation/augustus/scripts/gff2gbSmallDNA.pl bro_pasa.assemblies.fasta.transdecoder.genome.gff3 hmac_dovetail_assembly.fasta 1000 hmac_dovetail_assembly.gb
```

2. Split genbank file into training and test files
```
perl ~/bro_annotation/augustus/scripts/randomSplit.pl hmac_dovetail_assembly.gb 200

outputs : hmac_dovetail_assembly.gb.test & hmac_dovetail_assembly.gb.train
```

3. Quality check for training file: provides number of genes
```
grep -c LOCUS hmac_dovetail_assembly.gb*
```

## Training

1. Create a species file
```
perl ~/bro_annotation/augustus/scripts/new_species.pl --species=h_mac
```

2. Run etraining
```
etraining --species=h_mac hmac_dovetail_assembly.gb.train
```

3. Test result
```
tee can be used to save standard output while it keeps printing on the screen.

augustus --species=h_mac hmac_dovetail_assembly.gb.test | tee test_h_mac.out

After first training, the sensitivity and specificity can be evaluated.

grep -A 22 Evaluation test_h_mac.out

Results can be compared agains gene predictuion using the species provided by augustus such as human and fly.
Using human/fly/other parameters:

augustus --species=human hmac_dovetail_assembly.gb.test | tee test_h_mac_human.out
grep -A 22 Evaluation test_h_mac_human.out

augustus --species=fly hmac_dovetail_assembly.gb.test | tee test_h_mac_fly.out
grep -A 22 Evaluation test_h_mac_fly.out

Quote from AUGUSTUS tutorial:
"If the gene level sensitivity is below 20% it is likely that the training set is not large enough, that it doesn't have a good quality or that the species is somehow 'special'."
```

4. Opitimization
```
For 12–24 rounds of training optimization, it takes about 7–14 days and at least 8 Gb of RAM.

nohup perl ~/optimize_augustus.pl \
--species=hmac hmac_dovetail_assembly.gb.train \
--rounds=12 \
--cpus=8 &

nohup perl ~/bro_annotation/augustus/scripts/optimize_augustus.pl \
--species=hmac hmac_dovetail_assembly.gb.train \
--rounds=12 \
--cpus=8 &
```

5. Evaluate after (run etraining again and compare with previous run)
```
etraining --species=h_mac hmac_dovetail_assembly.gb.train
augustus --species=h_mac hmac_dovetail_assembly.gb.test | tee test_h_mac_after.out
```
