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

## Setting up the GitPod environment

Open the [GitPod webpage](https://gitpod.io/). Click on "Try for Free", then click on "New Workspace", type to the url `github.com/BGAcademy23/genomescope` and finally hit `Continue` button. The worksspace will be created with the software installed and data avaialbe.

If you would like to install the tools at your own computer or cluster, you can take a look at [the configuration .yml file](https://github.com/BGAcademy23/genomescope/blob/main/.gitpod.yml) we used to set up the gitpod. These commands are setting up a conda environment with kmc, R and a bunch of r-packages, then fetching GenomeScope from GitHub and installing it. You should be able to follow similar steps to get a similar evironment anywhere you like.

## Practical session 1

This session is mostly to get you familiar with running GenomeScope in terminal. For completeness sake, we also show how k-mer spectra can be generated using KMC, but there are many tutorial on k-mer spetra generation out there. You can skip that part and start directly with modeling. 

### (optional) Generating k-mer spectra using KMC

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
genomescope2.0/genomescope.R -i genomescope/histograms/Timema_monikensis_k21.hist -o Tmonikensis_k21_GS_out -k 21

```
As you can see, after running you will get a printed message with the model fit data:
```
GenomeScope analyzing genomescope/histograms/Timema_monikensis_k21.hist p=2 k=21 outdir=Tmonikensis_k21_GS_out
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
genomescope2.0/genomescope.R -i genomescope/histograms/Begonia_luxurians_k21.hist -o Bluxurians_k21_GS_out -k 21
```


In this case we have a much more heterozygous genome, and it looks diploid. Looks like the model adjusts just fine. 


#### Try it on your own

There are a few more histogrmas here. Try to fit the models right.

<details>
<summary><b> Unfold here to see all the histograms</b></summary>

*Bombina sp.* (frog)

```
genomescope2.0/genomescope.R -i genomescope/hitograms/bombina_sp_k21.hist -o Bombinasp_k21_GS_out -k 21
```

![Screenshot 2023-09-15 at 00 59 19](https://github.com/BGAcademy23/genomescope/assets/28604909/9c0817c8-0902-4c20-b87d-e83ada2e3ebb)

Another well-behaved diploid! What differences can you see between *B. luxurians* and this one?


*Letharia vulpina* (lichen)
We will definitely be expecting something odd in the lichen. Why?

```
genomescope2.0/genomescope.R -i genomescope/hitograms/Letharia_vulpina_k21.hist -o Lvulpina_k21_GS_out -k 21
```

![Screenshot 2023-09-15 at 01 05 19](https://github.com/BGAcademy23/genomescope/assets/28604909/1c14094f-38fb-471a-817e-2f66e56a01b4)

That's right! 3 peaks. We've got more than one genome here! What can we do?
It is obvious there are no genome models that count on more than one genome in one sample... So to properly do a model fit we would have to separate the reads/kmers first with something like blobtoolkit.


*Fragaria iinumae* (strawberry)
```
genomescope2.0/genomescope.R -i genomescope/hitograms/Fragaria_iinumae_k21.hist -o Fiinumae_k21_GS_out -k 21
```

![Screenshot 2023-09-15 at 01 51 43](https://github.com/BGAcademy23/genomescope/assets/28604909/4ed2c0a4-152f-44ce-9f50-261b16fd3ec0)

Right! here we also see something odd... It looks like part of our true kmers are in the "errors", which is definitely affecting our model...

</details>


### Modifying GenomeScope parameters for a better model fit

Once we are familiar with different genome models, let's start playing with some parameters to improve our model fits. Let's go back to our strawberry plot. It looks like the small peak we see covered by the errors is right at half coverage (~150x) of the main peak (~300x). This is likely a monoploid (1n) peak that needs to be considered by our model. We can tell GenomeScope the **estimated coverage of our 1n peak*** by using the `-l` parameter:

```
genomescope2.0/genomescope.R -i genomescope/hitograms/Fragaria_iinumae_k21.hist -o Fiinumae_k21_GS_out -k 21 -l 150
```
Now it looks better!

![Screenshot 2023-09-15 at 01 59 48](https://github.com/BGAcademy23/genomescope/assets/28604909/559cd904-2b98-4207-be3b-c37f0b7bcc26)

Getting the 1n peak right is essential to get a good model fit. Another factor to take into account is **ploidy** (`-p`). Let's now try modelling our own-generated `SRR3265401.21.kmc.hist` dataset (it is also in the `workspace/histograms/` directory if you haven't run KMC):

```
genomescope2.0/genomescope.R -i SRR3265401.21.kmc.hist -o SRR3265401_k21_GS_out -k 21
```

![Screenshot 2023-09-15 at 02 08 41](https://github.com/BGAcademy23/genomescope/assets/28604909/c1373cb7-3ef1-4aeb-ad19-7f5a174c8495)

There's again something odd here. As in the strawberry plot we can see an extra, lower coverage peak not considered by the model. However in this case it is not half the coverage of our main peak but ~1/4. This is when we start considering we may not be dealing with a diploid genome! Let's play with `-p`:

```
genomescope2.0/genomescope.R -i SRR3265401.21.kmc.hist -o SRR3265401_k21_GS_out -k 21 -p 4
```

![Screenshot 2023-09-15 at 02 15 27](https://github.com/BGAcademy23/genomescope/assets/28604909/ff1279cf-b62a-45e5-bd62-65b3118d785c)

This does look better, looks like our yeast is tetraploid. Testing different ploidy models is always helpful when our k-mer distribution shows peaks that are not being included in the model. 

Another parameter that does impact the modeling is the **maximal counter value** (`-cx` with KMC, `-m` with GenomeScope). This is roughly equivalent to truncating the histogram at that value. Smaller -cs values can have a pretty dramatic impact on genome modeling. To see this, lets try re-running KMC with a much smaller value (-cs100). 

This is how a histogram on default max counter value looks like (`-cs10000`, `-m 10000`)

<img width="868" alt="Screenshot 2023-09-15 at 01 20 33" src="https://github.com/BGAcademy23/genomescope/assets/28604909/23ff63c3-7cff-45b4-b16a-30a9bb157d96">

This is how it would look like if we changed it into `-cs100`

<img width="861" alt="Screenshot 2023-09-15 at 01 20 52" src="https://github.com/BGAcademy23/genomescope/assets/28604909/a291d1dc-4a9c-49a7-a955-f6553f4d5b42">

The differences aren't quite too dramatic, but you can still see that the genome size estimate decreases by over 16mbp. This would be even more evident with a more repetitive genome. 

Try changing the `-m` values in any of our previous GenomeScope runs (default `-m 10000`) to a much lower one. How does this affect the model?

Why would you ever want to truncate the .hist file?? Because it makes the file smaller and speeds up the modeling. All valid reasons, but I think it's worth a few extra seconds and a little bit more space for more accurate genome models. 






### Fitting your own model

This exercise has one sole purpose - understanding the logic behind fitting genome models to k-mer spectra, in particular using GenomeScope genome model. 

K-mer spectra can have many forms and shapes dependent on both biology and the sequencing technique, but let's use an idealised case - a k-mer spectra, that does not have any sequencing errors, or repetitive DNA. It has been manually constructed from a k-mer spectra of _Timema cristinae_ (all the sequencing reads associated with [this biosample](https://www.ncbi.nlm.nih.gov/biosample/6311760)). We calculaded k-mer spectra (as in the last tutorial) and then manually trimmed both sides so it is close to the "ideal case" (`/workspace/histograms/advanced/Timema/Timema_cristinae_kmer_k21_simplified.hist`). This is how the idealised k-mer spectra looks like:

![simplified_linear_plot](https://user-images.githubusercontent.com/8181573/217258375-13116277-dcab-4487-be74-bdde167b03c1.png)

See? All the k-mers are unique, the error rate is 0. It can't get much better, so, let's fit an analogous model ouserlves. So, open R, load the k-mer spectra and plot it. From now on, all the code is going to be in `R`.

```R
ideal_kmer_spec <- read.table('/workspace/genomescope/histograms/advanced/Timema/Timema_cristinae_kmer_k21_simplified.hist', col.names = c('cov', 'freq'))
plot(ideal_kmer_spec$cov, ideal_kmer_spec$freq, type = 'l', xlab = 'Coverage', ylab = 'Frequency', main = 'ideal')
dev.off()
```

Of course, this looks the same as... the plot above (it's mostly that you see that the data is really the same).

There are just two peaks, so the model we will fit will be something like

```
frequency ~ 1n peak +
            2n peak
```

We could chose from various distributions to model the two peaks, but it just happens, that negative binomial, fits sequencing coverage very well. Negative bionomial has usually two parameters stopping parameter and the success probability, however, that is not a convinient notion for us, we will therefore use a reparametrised version of negative binomial with _mean_ 'mu', and 'size', the _dispersion parameter_. So in our model, we will fit mean, and the _dispersion parameter_ and the model can look like this

```
Probability ~ α NB(  kcov, dispersion) +
              β NB(2*kcov, dispersion2)
```

Now, the α and β will be the contributions of the two peaks. Notice that we fit the mean of the first peak with the same parameter as the mean of the second peak s, just multiplied by a factor of 2. That is because the heterozygous loci have 1/2 of the coverage of the homozygous loci and therefore, we actually desire to co-estimate the haploid coverage jointly for both of the at the same time.

There are still a few adjustments we need to do to the model to finally make a reasonable fit. In the last model we parametrise the dispersion parameters of the two models independently, that is however unnecessary - with increased coverage, the variance in sequencing depth increases too, but the "overdispersal" scales the same for the whole sequencing run. We therefore use the same trick we used for the coverage and fit the overdispersion as a single parameter scaled by the mean

```
Probability ~ α NB(  kcov,   kcov / overdispersion) +
              β NB(2*kcov, 2*kcov / overdispersion)
```

There are a few more things to add, but this is a minimal viable product. So, we can attempt to fit this model using non-linear least squared method (`nls`). This is a process that is very similar to classical least square method, but instead of calculating the exact best fit, it iterates through a range of possible solutions and finds the best one among them. These techniques are powerful, for they can fit parameters to any kind of model really, but they are also very sensitive to the initial parameters and the strategy of the parameter search.

**Fit and plot a basic model**

```R
y <- ideal_kmer_spec$freq
x <- ideal_kmer_spec$cov
gm1 <- nls(y ~ a * dnbinom(x, size = kcov   / overdispersion, mu = kcov    ) +
               b * dnbinom(x, size = kcov*2 / overdispersion, mu = kcov * 2),
           start = list(a = 300e6, b = 500e6, kcov = 25, overdispersion = 0.5),
           control = list(minFactor=1e-12, maxiter=40))

summary(gm1)
```

<details> 
  <summary>Model interpretation </summary>

Alright, we see that `a` (my `α`) is about 1/2 of `b` (`β`). That means there is about twice as much homozygous to heterozygous k-mers (the estimates are the estimates of how many exactly). We also see that the k-mer coverage is 27.6x. That is already quite some information, but nothing really 
</details>

To plot the model, we need to get the values predicted within the range of our data. Let let's define the model as a function

```R
predict_basic_model <- function(x, a, b, kcov, overdispersion){
    a * dnbinom(x, size = kcov   / overdispersion, mu = kcov    ) +
    b * dnbinom(x, size = kcov*2 / overdispersion, mu = kcov * 2)
}

gm1_coef <- coef(gm1)
gm1_y <- predict_basic_model(x, gm1_coef['a'], gm1_coef['b'], gm1_coef['kcov'], gm1_coef['overdispersion'])
```

and now we can finally plot it.

```R
plot(ideal_kmer_spec$cov, ideal_kmer_spec$freq, type = 'l', xlab = 'Coverage', ylab = 'Frequency', main = 'ideal')
lines(ideal_kmer_spec$cov, gm1_y, lty = 2, col = 'red')
dev.off()
```

Does the model fit well the data? Ya, but there are so many annoying things about it... for instance, what is the genome size? Or what is the heterozygosity? Let's make a few last modification so the model will give us these estimates too.

**Fit a model that estimates heterozygosity**

We can express α and β using probability of a heterozygous nucleotide (r) and the k-mer length (k). Look at the cascade of following expressions

![Screenshot 2023-02-07 at 13 17 51](https://user-images.githubusercontent.com/8181573/217258225-a3927db7-e8e4-484a-9a03-b78f7be0110c.png)

With this we could estimate the heterozygosity using `k` as a constant, while assuming that the heteorzygosity (probability of a nucleotidy being heterozygous) is the same for each nucleotide across the genome. Note there is a one less parameter too, instead of α and β (or `a` and `b`), there is just `r`, but there is one more consideration - while our previous model features large α and β coeficients, this model expects them to be relative proportions of the two peaks (the probability of heterozygous vs homozygous k-mers), which means we need to multiple the joint distribution by the genome length - this will be a one more added parameter to the equation, leading to the same total number as it was in the last case.

```R
k <- 21
gm2 <- nls(y ~ ((2*(1-(1-r)^k)) * dnbinom(x, size = kcov   / overdispersion, mu = kcov    ) +
                ((1-r)^k)       * dnbinom(x, size = kcov*2 / overdispersion, mu = kcov * 2)) * length,
           start = list(r = 0, kcov = 25, overdispersion = 0.5, length = 1000e6),
           control = list(minFactor=1e-12, maxiter=40))
summary(gm2)
```

So, what do you think? The heterozygosity seems to be in the right ballpark (0.99% by GenomeScope vs 0.98% by our model), K-mer coverage is 27.6x in both models, and the genome size is also remarkably similar ~760Mbp.


**Fit real data**

Till now we had an idealised case - trimmed of all repetitions and errors. In truth, the unidealised version of the k-mer spectra still looks pretty good: 

![real_linear_plot](https://user-images.githubusercontent.com/8181573/217610706-8a9c1cc1-3b93-408c-8bb7-11efa70123af.png)

Can you spot what's the effect of the "unidelised version"? Is the fit still good? Which estimated values get affected the most? Look at the genome size! What happens if you fit your own model to the full k-mer spectra? Load that k-mer spectra and fit the model and compare it to the estimates by genomescope.

<details> 
  <summary>Solution </summary>

```R
real_kmer_spec <- read.table('/workspace/genomescope/histograms/advanced/Timema/Timema_cristinae_kmer_k21.hist', col.names = c('cov', 'freq'))
x <- real_kmer_spec$cov
y <- real_kmer_spec$freq
k <- 21 

gm3 <- nls(y ~ ((2*(1-(1-r)^k)) * dnbinom(x, size = kcov   / overdispersion, mu = kcov    ) +
                ((1-r)^k)       * dnbinom(x, size = kcov*2 / overdispersion, mu = kcov * 2)) * length,
           start = list(r = 0, kcov = 25, overdispersion = 0.5, length = 1000e6),
           control = list(minFactor=1e-12, maxiter=40))
summary(gm3)
```

</details>

So, we did not model the error peak, but the coverage and heterozygosity is moreless right. This is in part because there are not many duplications in the genome (no bumps around 3n and 4n coverages), but also the majority of this estimate does come from the unique portion of the genome. However, our model, unlike GenomeScope still underestimates the genome size!

**Estimate genome size**

The basic GenomeScope estimates genome size as follow.

1. Fit the model to things that occur in one to four genome copies; (you have done that for one to two genomic copies).
2. Removing the residual errors from the k-mer spectra

```R
residuals <- y[1:40] - predict(gm3, newdata = list(x = 1:40))
errors <- residuals[1:(which.min(residuals > 0) - 1)]
y[1:length(errors)] <- y[1:length(errors)] - errors
```

3. The genome size is `sum(coverages * frequencies) / (1n_coverage * ploidy)` from the cleaned spectra

```R
sum(x * y) / (2 * coef(gm3)['kcov'])
```

You see that counting repetitive k-mers indeed changes a lot the genome size estimate.
