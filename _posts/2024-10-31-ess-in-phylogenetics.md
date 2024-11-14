---
title: 'ESS in Phylogenetics'
date: 2024-10-31
permalink: /posts/2024/10/ess-in-phylogenetics/
tags:
  - ESS
  - Bayesian Phylogenetics
  - Sampling frequency
---

A brief discussion of what ESS is, why it matters, and how to boost it. A particular attention is payed to sampling frequency since this is oft overlooked in more general MCMC guides.

ESS in Phylogenetics
==========================

## ESS

What is ESS? ESS, or estimated sample size, is the estimated number of random draws from the posterior distribution which is equivalent to the Markov chain. While this sounds confusing, this basically means that ESS represents the number of independent samples with the same power as our Markov chain's autocorrelated samples. Autocorrelation is the correlation between two values separated by a time lag, i.e., the similarity between two samples in our Markov chain. The best explanation of autocorrelation I've come across is [this one from Alison Horst](https://allisonhorst.com/time-series-acf). ESS measures the amount of independent information in our Markov chain. 

Because ESS is a measure of the amount of independent information found in our Markov chains, having a high enough ESS score is necessary, but not sufficient, for a good Bayesian phylogenetic analysis. The most common rule of thumb is to have an ESS value of 200 or above, but [empirical analyses from Fabreti and Höhna have shown](https://doi.org/10.1111/2041-210X.13727) that 625 is a better cutoff for "good enough".

## Hold up, why do you say not sufficient?

If ESS is the amount of independent information in our Markov chain, why can't we just look at that for each parameter to determine if our MCMC output is good?

![image](https://github.com/user-attachments/assets/791d1707-392a-4aa5-8256-08315b9ea209)
Figure 1: A demonstration figure from [Vehtari, et al.](https://doi.org/10.48550/arXiv.1903.08008). On the left we have 2 chains which converge to different estimates (an error in reproducibility). On the right, we have two chains which are not yet stationary (an error in precision).

There are two properties that are common to chains which have converged: precision and reproducibility. Precision, meaning that even if we run the chain for much longer, we won't change our estimates. An example of changing estimates over time for two runs is above. This is what's measured by ESS. Reproducibility means that if we run another independent chain, we will get the same estimates. We can verify reproducibility by comparing the results from multiple independent MCMC runs and verifying that our results line up with one another. ESS values for a single run will help check for just precision, but not the reproducibility. We can see a case where this leads us astray above as well. 

## So high ESS is good… How do I increase it?

To paraphrase from the [Beast2 website](https://www.beast2.org/increasing-esss/):

1. *The easiest and best way is to increase your chain length!* However, you're probably here for alternatives because this is computationally impractical.
2. Optimize the operators which help you explore the probability space. BEAST has an auto-optimization system which is by default turned on, but you can manually adjust and tune this using ``` <operator optimize="false" ... >``` in your XML file. You can also use the report BEAST generates to help edit these.
3. Combine the results of multiple independent chains. To do this, you will need to ensure that chains have converged and are mixing adequately (compare the properties above). If so, then you can remove some burn-in and combine your runs. There are metrics to help ensure that these have sufficiently converged, but that's outside the scope of this article (this is a fun way of saying that I am too lazy to provide insight on this).
4. Increase your sampling frequency. If sampling frequency is very low, sampled states will be uncorrelated, and this will mean ESS approaches the number of states in the log file minus burn-in. Finally, we get into the meat and potatoes...

## Why does increasing sampling frequency increase ESS? And why does low sampling frequency lead to ESS=log file states?

Note that
$$ESS \approx \frac{\text{number of samples}}{1+2\cdot \sum(\text{autocorrelation of samples})}$$

As we reduce our sampling frequency, autocorrelation of samples will also go to 0 since the sample pairs have had plenty of time to diverge. When this happens, ESS will approach the number of samples since $ESS = \frac{\text{number of samples}}{1+2\cdot \sum(0)} = \text{number of samples}$.  As you increase sampling frequency, ESS will approach a maximum value for that chain length. However, increasing sampling frequency isn't free! This increases your computation time during the post-analysis and increases the demands on storage space. While in other fields, this is less of an issues, Bayesian phylogenetic analysis is somewhat unique. Each tree generated takes up a not-insignificant amount of storage space. For example, in a recent analysis, I used 24 MB of space to store just 1,000 trees containing 500 taxa. Considering chain lengths are usually at least 10 million or more, this would correspond to roughly 240 GB of storage space for just one run if you don't sample your chains. 

In other fields, many people have stopped thinning chains altogether. Thinning is known to be inefficient, and there are [ways to prove](https://hackmd.io/@khai/BkLMjmVVF) that it is nearly always the wrong choice. The exception, lucky us, is in special cases where storage space (or post-analysis) will be limited. 
