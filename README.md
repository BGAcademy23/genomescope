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

### (optional) Generating k-mer spectra using KMC and FastK

This section is just to show you how to get a k-mer spectrum out of reads. As an example, we can look at a yeast that we have already downloaded for you in `data/`. 

There are many k-mer counters. Here we are showing how to use [KMC](https://github.com/refresh-bio/KMC), still one of the fastest and very versatile k-mer counting tools. If you are dealing with lots of genomes, or with massive amount of data, we would recommend to switch to [FastK](https://github.com/thegenemyers/FASTK).

`KMC` needs a directory to save some temporary files, and also if more than one file is on the input, it likes them to be saved in a simple text file that lists all the readfiles that should be processed in a k-mer database. The `k` arugument just sepcify the k-mer size and `-ci` and `-cs` are the coverage limits the tool calculates.

```
mkdir -p tmp
ls data/SRR3265401_[12].fastq.gz > data/FILES

kmc -k21 -t4 -m96 -ci1 -cs100000 -fq @data/FILES data/SRR3265401_21.kmc tmp/
```

The output from the above command should look something like this (the screenshot is from drosophilla), giving you some numbers on the generated database: 

<img width="667" alt="Screen Shot 2023-01-27 at 10 57 22" src="https://user-images.githubusercontent.com/43171649/215130686-bfbfa553-89df-42c6-ba1d-e71c4bcc0bfe.png">

Now that the k-mer database exist, we can simply query it for the k-mer histogram. There is so much more you can do with tools like KMC, but for now we just want the histogram 

```
kmc_tools transform data/SRR3265401_21.kmc histogram data/SRR3265401_21.21.kmc.hist -cx100000
``` 

The generated histogram file looks will be a file like this: 

![Screen Shot 2023-02-20 at 17 30 07](https://user-images.githubusercontent.com/43171649/220207468-8b00028a-f7b0-4162-8487-43e40c1bd7d3.png)

The first column is the coverage, that is how many times a kmer is seen in the set of reads. The second column is the number of kmers that occur that many times. In this case, there are 10693844 kmers that only occur once. As we'll get into later, this is probably because these kmers overlap an error. But for now, just know that this *.hist format should contain two columns, with the coverage in the first column and the frequency in the second. 

This isn't terribly exciting output, however, when we run it through genomescope to build a genome model we can build this more informative plot.

### Simple diploid