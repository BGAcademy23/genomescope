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

This section is just to show you how to get a k-mer spectrum out of reads. You don't need to do this for the practical, but just in case you'd like to learn how to run a k-mer counter, here's an example of a KMC run (k=21) on yeast short-read datasets (SRR3265401). You can find these .fastq files in your GitPod workspace (workspace/data).

```
SAMPLE=SRR3265401
mkdir -p tmp

ls data/"$SAMPLE"* > FILES
kmc -k21 -t2 -m96 -ci1 -cs1000000 -fq @FILES $SAMPLE.21.kmc tmp/

kmc_tools transform $SAMPLE.21.kmc histogram $SAMPLE.21.kmc.hist -cx1000000
```
Have a look at the histogram file that was generated with the KMC transform command:

![Screenshot 2023-09-14 at 23 20 26](https://github.com/BGAcademy23/genomescope/assets/28604909/744e0a4c-7aea-4528-a636-831ca411bb9d)

This will be our input for GenomeScope! 

### Fitting genome models to your kmer spectrum using GenomeScope
Now you have your own generated .hist (SRR3265401) and many others in the `workspace/histograms/` directory to play with.
Let's start by making sure GenomeScope works fine by typing `genomescope2.0/genomescope.R` from the workspace:
![Screenshot 2023-09-14 at 23 39 31](https://github.com/BGAcademy23/genomescope/assets/28604909/481a59bf-957b-45d4-a4d4-1ad8a956a173)

As you can see, the only necessary parameters are the input and `-k`. It recommends naming the output directory and specifying the ploidy level, otherwise it is going to assume diploid. There's other parameters such as `-l` and '-m', which we will talk about later.

#### example 1
Once we've made sure GenomeScope is installed and you can make it run, let's start modelling our with our own-generated `SRR3265401.21.kmc.hist` :

```

```

This is a simple case we show you

#### example 2

This is more complicated case we show you

#### Try it on your own

There are 5 more histogrmas here. Try to fit models right.

<details>
<summary><b> Unfold here to see all the histograms</b></summary>


</details>

### Modifying GenomeScope parameters for a better model fit

