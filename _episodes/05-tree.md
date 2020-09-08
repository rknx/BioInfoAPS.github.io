---
title: "Phylogenetic tree"
teaching: 0
exercises: 0
questions:
- "Key question (FIXME)"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## Sequence alignment

Sequence alignment is the basic and the most important step in phylogenetic analysis. 
A wrong sequence alignment will lead to wrong result. 
Various alignment tools are used to identify similarity regions that indicate fuctional, 
structural, and/or evolutionary relationships sequences. 
Multiple sequence alignment tools such as ClustalOmega, Muscle, MAFFT are commonly used. 
They vary in terms of algorithms used; ClustalOmega uses HMM profile-profile techniques, 
and MAFFT uses Fast Fourier Transforms and 
both are considered good for medium to large alignments. 
Muscle is regarded good with proteins for medium alignments.

We will use MAFFT sequence alignment tool.

```sh
$ module load mafft

$ mafft avrBs2_all_genomes.fas > avrBs2_all_genomes_aligned.fas

$ more avrBs2_all_genomes_aligned.fas
```
Warning: Make sure you are still in phylogeny folder. 
You can use `pwd` to check.

The two files avrBs2_all_genomes.fas and avrBs2_all_genomes.aligned.fas 
may not look very different at the moment considering the gene was present 
in all the sample genomes used here as seen from blast results. 
However, aligning the sequence prior to phylogenetic reconstruction is always valuable.
Now, we have an input file ready to use for a software that can run a 
maximum likelihood phylogentic analysis. 

## Phylogenetic tree

There are several phylogenetic analysis software and tools that can be used. 
We will run a Maximum likelhood phylogenetic analysis. 
There are other models/methods such as Neighbor-Joining, Maximum parsimony, and Bayesian. 
However, our focus is on application of command line, 
we will only focus on running a specific analysis here.
Maximum Likelihood uses probabilistic values to model the evolution and 
the tree with the highest probability is shared.

We will use RAxML - Randomized Axelerated Maximum Likelihood tool to 
construct the tree that we will submit through a batch file.

The SLURM submission script is present in `/blue/general_workshop/share/scripts/tree.sh

```sh
$ cp ../../scripts/tree.sh ./

$ tail -n8 tree.sh
# Load RAxML
module load raxml

# Mkae tree with RAxML
raxmlHPC -d -p 12345 -m GTRGAMMAI -s avrBs2_all_genomes_aligned.fas -n avrBs2_tree

# Success message
echo 'ML tree output in RAxML_bestTree.avrBs2_tree'
```
Tip: the argument `-n` allows us to name the output suffix. In our case, the output will
be name `RAxML_bestTree.avrBs2_tree`.

Tip: To understand what other arguments mean, you can check the help file `raxalHPC -h`.

Edit the email address in the SLURM submission script and submit the job.

Note: if you run bootstraps, there will be individual files for each bootstrap
in additing to `RAxML_bestTree.<suffix>` only.

We can download this tree only and visualize in our own computer using ‘Figtree’. 
Please use file transfer tool such as FileZilla or Cyberduck to download the file. 
The tree will open as follows:

![Phylogenentic tree](/fig/tree.png)

---

## Phylogenetic tree pipeline

We have now successfully extracted a gene of interest from our genomes and 
generated a phylogenetic tree. 
However, we completed the entire process using several distinct steps where
we evaluated output of one step before starting to run the next step.
In real world scenario, most of the intermediate results are not of 
interest to a bioinformatician, so those disjoint steps are often chained
into one or few longer workflow, referred to as **pipeline**. 

Question: Would it be possible to develop a script where we only need to 
provide external (generated outside the pipeline usch as gene and
genome sequences) inputs and get the final result (the phylogennetic tree) 
in return directly?

Answer: YES*

For this, we will merge all the steps together into one script.

Warning: *Chaining multiple steps is not always straightforward. 
We have to be vigilant for what might change when the steps are automated and 
intermediate  results are not evlauated.
For example, we have to make sure that the genes of interest are actually in the genome. 
Otherwise, the BLAST may output match to different closely related gene, 
gene sequence alignment may add gaps and give us a wrong output and so on. 
Understanding what might go wrong comes from experience in bioinformatics.

Let's put everything together into one script. The input files are
available in `/blue/general_workshop/share/all_in_one/` and the 
script is available as `/blue/general_workshop/share/scripts/all_in_one.sh`.

```sh
$ cd /blue/general_workshop/<username>

$ cp -r ../share/all_in_one ./

$ cd all_in_one

$ cp ../share/scripts/all_in_one.sh ./

$ cp ../share/scripts/blast2fasta.sh ./

$ ls
all_in_one.sh     avrBs2.fas     blast2fasta.sh     GEV1001.fasta     GEV1026.fasta
...
...

$ tail -n22 all_in_one.sh
```


```sh
# Load programs
module load ncbi_blast
module load mafft
module load raxml

# Loop over each genome
for genome in `ls *.fasta | sed 's/.fasta//g'`
do
  # Make database
   makeblastdb -in "$genome.fasta" --dbtype nucl -out "$genome"

  # Run blastn on that database
   blastn -query avrBs2.fas -db "$genome" -out $genome"_avrBs2.out" -outfmt 5 -evalue 0.001
done

# Combine all outputs and parse into multiFASTA file
cat *_avrBs2.out | ./blast2fasta.sh > avrBs2_all_genomes.fas

# Align sequences with MAAFT
mafft avrBs2_all_genomes.fas > avrBs2_all_genomes_aligned.fas

# Mkae tree with RAxML
raxmlHPC -d -p 12345 -m GTRGAMMAI -s avrBs2_all_genomes_aligned.fas -n avrBs2_tree
```

After editing the email address in `nano`, you are ready to submit the submission script.

```sh
$ sbatch all_in_one.sh
```

Tip: Once the job finishes, you can transfer all the outputs to your personal computer. 
Then, you can visualize the ‘RAxML_bestTree.avrBs2_tree’ using Figtree software.