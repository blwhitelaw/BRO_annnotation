# Genome and Transcriptome Assembly Data

The dovetail genome assembly can be obtained from Dropbox

```bash
wget "https://www.dropbox.com/s/gza7hr1kcolosu0/hmac_dovetail_assembly.fasta?dl=0" -O hmac_dovetail_assembly.fasta
```

As can the Trinity transcriptome assembly data used for gene modelling

```bash
wget "https://www.dropbox.com/s/nlcrj79bankkns3/hapalochlaena_maculosa_na.fasta?dl=0" -O hapalochlaena_maculosa_na.fasta
```

After downloading raw data generate the contig length file

```bash
cat hmac_dovetail_assembly.fasta | bioawk -c fastx '{print $name,length($seq)}' | sort -k 2 -n -r > hmac_dovetail_assembly.lengths.txt
```