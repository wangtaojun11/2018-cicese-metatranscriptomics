## Counting the number of transcriptomes in a metatranscriptome
In metagenomics, we use binning to approximate the number of genomes (or species) in a sample.
Although we can apply taxonomic classification methods like sourmash to metatranscriptomic 
samples, these methods only work reliably when sequences already exist for the organism of 
interest. This is still a relatively rare occurrence in environmental metatranscriptomics. 

We will make use of other data in our samples to approximate the number of species present.
Although our samples were poly-A selected, inevitably a some ribosomal RNA is always sequenced
given its abundance in samples. We can estimate the number of species in our metatranscriptome
by counting the unique number of ribosomal protein sequences we see. There are many ribosomal
proteins one could use for this analysis, and we provide a table of these at the bottom of 
this lesson. We will use one sequence here, however if we were to use more we would average 
the number of unique sequences we detect between the total sequences to estimate the number 
of transcriptomes in our sample.

We will first search for our protein in the amino acid sequences 
derived from our de novo assembly of our metatranscriptome. We will use
[Pfam](https://pfam.xfam.org/) domains and a tool called [HMMER](http://hmmer.org/) 
to help us locate all of the matching sequences. Then, we will extract our matches and 
filter those that are highly similar. 

Below is a picture representation of the HMM profile we will be working with. 

![rpsG seq logo](files/rpsg_logo_image.png)

Because we are only searching in our assembly, this method only captures the number of 
transcriptomes that assembled.

## 

First let's make a new directory and link in our annotation file. 
```
mkdir -p ${PROJECT}/count-transcriptomes
cd ${PROJECT}/count-transcriptomes
```

Then let's link in the peptide sequences (ORFs) predicted from our 
full transcriptome assembly.
```
ln -s /LUSTRE/bioinformatica_data/bioinformatica2018/assembly/tara_f135_megahit_full.transdecoder.pep .
```

Next let's link in an alignment of a Pfam domain for RpsG. 
In the future, you can download this alignment [here](http://pfam.xfam.org/family/PF00177/alignment/full) 
```
ln -s /LUSTRE/bioinformatica_data/bioinformatica2018/assembly/PF00177-full.sto .
```

Next we'll build a HMM profile of the Pfam domain. 

```
hmmbuild PF00177-full.hmm PF00177-full.sto
hmmpress PF00177-full.hmm
```

We then use the HMM profile to search the proteins from our assembly
```
hmmscan -T 100 --tblout PF00177-full-tbl.txt --domtblout PF00177-full-domtbl.txt PF00177-full.hmm tara_f135_megahit_full.transdecoder.pep 
```

Let's take a look at one of the files output by this search
```
less -S PF00177-full-tbl.txt
```

The third column contains the names of our protein sequences that matched
these domains. We can use those names to extract our matches from 
our assembly

```
# Grab the names
cat PF00177-full-tbl.txt | Rscript -e 'writeLines(noquote(read.table("stdin", stringsAsFactors = F)$V3))' > PF00177-names.txt

# extract the matches
ln -s /LUSTRE/bioinformatica_data/bioinformatica2018/extract-hmmscan-matches.py .
python extract-hmmscan-matches.py PF00177-names.txt tara_f135_megahit_full.transdecoder.pep > PF00177.faa
```

Let's count the number of sequences that matched 
```
grep ">" PF00177.faa | wc -l
```

Some matches are quite similar to each other. Let's cluster our sequences
at 97% similarity and see how this changes the number of unique proteins
we detect. 

```
cd-hit -i PF00177.faa -o PF00177-c97.faa -c .97
```

Let's see how much clustering at 97% reduced our estimate!
```
grep ">" PF00177-c97.faa | wc -l
```

From this, we estimate there are 82 transcriptomes assembled from our
metatranscriptome.

Given that this protein is highly conserved, we could BLAST these 
sequences against NCBI nr/nt database to see if there are any matches. 
We could then compare it to our sourmash results and see how much overlap 
vs. how much we are missing from each!

This technique can also be used to find any Pfam domain of interest. 
For instance, if you are interested in photosynthesis, you could
extract all photosynthetic proteins by searching with the Pfam domains.

## Other proteins to use for quantifcation

We used three proteins to quantify the number of transcriptomes in our
sample. There are more that can be used, and they are list in the table
below. They are originally derived from [Carradec et al. 2018](https://www.nature.com/articles/s41467-017-02342-1#Sec19).


| name | COG     | PFAM             |
|------|---------|------------------|
| RpsG | COG0049 | PF00177          |
| RpsB | COG0052 | PF00318          |
| RplK | COG0080 | PF03946          |
| RplA | COG0081 | PF00687          |
| RplC | COG0087 | PF00297          |
| RplD | COG0088 | PF00573          |
| RplV | COG0091 | PF00237          |
| RpsC | COG0092 | PF00189          |
| RplN | COG0093 | PF01929          |
| RplE | COG0094 | PF00281          |
| RpsH | COG0096 | PF00410          |
| RplF | COG0097 | NA               |
| RpsE | COG0098 | PF03719, PF00333 |
| RpsM | COG0099 | PF00416          |
| RpsK | COG0100 | PF00411          |
| RplM | COG0102 | PF00572          |
| RpsI | COG0103 | PF00380          |
| RpsO | COG0184 | NA               |
| RpsS | COG0185 | PF00203          |
| RpsQ | COG0186 | PF00366          |
| GyrB | COG0187 | PF00204          |
| GyrA | COG0188 | PF00521          |
| RimK | COG0189 | PF08443          |
| FolD | COG0190 | PF00763, PF02882 |
| Fba  | COG0191 | NA               |
| MetK | COG0192 | PF01941          |
| Pth  | COG0193 | PF01195          |
| Gmk  | COG0194 | PF00625          |
| RplO | COG0200 | PF00827          |
| RplR | COG0256 | NA               |
| RpsD | COG0522 | PF00163          |
