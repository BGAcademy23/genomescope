# Genome profiling using GenomeScope

This session is part of [**Biodiversity Genomics Academy 2023**](https://BGA23.org)

## Session Leader(s)

Lucía Campos-Domínguez  
Kamil Jaron

## Programme

- Introduction to k-mer spectra, their interpretation and running GenomeScope (lecture by Lucía, ~30')
- Ex. 1: Running and understanding genome models (~30')
- Looking under the hood of the GenomeScope model (lecture by Kamil, ~20')
- Ex. 2: Running GenomeScope-like model yourself in R; Using GenomeTelescope package (~20')
- Discussion/Interpretation of wild k-mer spectra (~20')
    - We will bring a bunch of examples; but feel free to bring your own wild k-mer spectrum to share with others!

## Description

By the end of this session you will be able to:

1. Appreciate the art of the k-mer spectra gazing
2. Understand the idea behind genomescope model and its assumptions
3. Generate k-mer spectrum from a read set
4. Run GenomeScope in command-line and web-server
5. (if everything goes well) Create your own GenomeScope-like extended versions;

## Prerequisites

1. Basics knowledge of genomics
2. Be ready for some non-linear regression
3. (optional) familiarity with k-mers (Some materials developed by ourselves: <https://github.com/KamilSJaron/oh-know/wiki>)
4. (optional) Read relevant sections of <https://www.nature.com/articles/s41467-020-14998-3>

!!! warning "Please make sure you MEET THE PREREQUISITES and READ THE DESCRIPTION above"

    You will get the most out of this session if you meet the prerequisites above.

    Please also read the description carefully to see if this session is relevant to you.
    
    If you don't meet the prerequisites or change your mind based on the description or are no longer available at the session time, please email tol-training at sanger.ac.uk to cancel your slot so that someone else on the waitlist might attend.

## Practical session 1
### Setting up your environment using GitPod

### (optional) Generating k-mer spectra using KMC and FastK

This section is just to show you how to get a k-mer spectrum out of reads. You don't need to do this for the practical, but just in case you'd like to learn how to run a k-mer counter, here's an example of a [KMC](https://github.com/refresh-bio/KMC) run (k=21) on yeast short-read datasets (SRR3265401). You can find these `.fastq.gz` files in your GitPod workspace (`workspace/data`).

Note, there are many more k-mer counters. KMC is great one, but if you are dealing with lots of genomes, or with massive amount of data, we would recommend to switch to [FastK](https://github.com/thegenemyers/FASTK).

```
SAMPLE=SRR3265401
mkdir -p tmp

ls data/"$SAMPLE"* > FILES
kmc -k21 -t4 -m96 -ci1 -cs100000 -fq @FILES $SAMPLE.21.kmc tmp/

kmc_tools transform $SAMPLE.21.kmc histogram $SAMPLE.21.kmc.hist -cx100000
```

Have a look at the histogram file that was generated with the KMC transform command:

![Screenshot 2023-09-14 at 23 20 26](https://github.com/BGAcademy23/genomescope/assets/28604909/744e0a4c-7aea-4528-a636-831ca411bb9d)

The first column is the coverage, that is how many times a kmer is seen in the set of reads. The second column is the number of kmers that occur that many times. In this case, there are 10693844 kmers that only occur once. As we'll get into later, this is probably because these kmers overlap an error. But for now, just know that this `*.hist` format should contain two columns, with the coverage in the first column and the frequency in the second. This is the file will be an input for genomescope!

### Fitting genome models to your kmer spectrum using GenomeScope
Now you have your own generated .hist (SRR3265401) and many others in the `workspace/histograms/` directory to play with.
Let's start by making sure GenomeScope works fine by typing `genomescope2.0/genomescope.R` from the workspace:

![Screenshot 2023-09-14 at 23 39 31](https://github.com/BGAcademy23/genomescope/assets/28604909/481a59bf-957b-45d4-a4d4-1ad8a956a173)

As you can see, the only necessary parameters are the input and `-k`. It recommends naming the output directory and specifying the ploidy level (`-p`), otherwise it is going to assume diploid. There's other parameters such as `-l` and '-m', which we will talk about later.

#### Example 1
Let's start with one of the histograms from `workspace/histograms/`: the stick insect *Timema monikensis*. Let's assume we know nothing about its genome, and just run GenomeScope with default parameters:

```
genomescope2.0/genomescope.R -i genomescope/hitograms/Timema_monikensis_k21.hist -o Tmonikensis_k21_GS_out -k 21

```
As you can see, after running you will get a printed message with the model fit data:
```
GenomeScope analyzing genomescope/hitograms/Timema_monikensis_k21.hist p=2 k=21 outdir=Tmonikensis_k21_GS_out
aa:99.8% ab:0.157%
Model converged het:0.00157 kcov:26.3 err:0.00401 model fit:0.866 len:1124307247
```
You can find the outputs in the left menu from your gitpod workspace.
Let's have a look at our histogram `(linear.plot.png)`:

![Screenshot 2023-09-15 at 00 25 57](https://github.com/BGAcademy23/genomescope/assets/28604909/d0e28749-944a-42c7-bff0-95f8c7bbcb65)

As you can see from our k-mer distribution (blue bars), we have here an extremelly haploid individual. Even though our heterozygosity levels are also very low in our model, we can see it doesn't adjust very well to a "diploid" model because of its extremelly high homozygosity. You can try changing your command to add `-p 1` to see if a haploid model works best. How else does the model change? does genome size estimate change much?

#### Example 2

Let's look at another example, a plant: *Begonia luxurians*. Again, let's assume we know nothing about its genome:

```
genomescope2.0/genomescope.R -i genomescope/hitograms/Begonia_luxurians_k21.hist -o Bluxurians_k21_GS_out -k 21
```


In this case we have a much more heterozygous genome, and it looks diploid. Looks like the model adjusts just fine. 


#### Try it on your own

There are a few more histogrmas here. Try to fit the models right. Play with `-p` if necessary 

<details>
<summary><b> Unfold here to see all the histograms</b></summary>

*Bombina sp.* (frog)
#### 
####
</details>

### Modifying GenomeScope parameters for a better model fit

Once we've made sure GenomeScope is installed and you can make it run, let's start modelling our with our own-generated `SRR3265401.21.kmc.hist`

### Fitting your own model



