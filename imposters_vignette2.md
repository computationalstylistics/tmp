---
title: "Authorship verification using the package stylo"
author: Maciej Eder
date: 26.05.2018
output: 
  html_document:
    highlight: pygments
---

## The General Imposters method 


those of you who follow non-standard additions to the package `stylo` might remember that some time ago, I (sort of) implemented the General Imposters method as introduced by Koppel and Winter (2014) and applied by Kestemont et al. (2016). The function, however, has remained somewhat underdeveloped – frankly, I never liked it.

Let me proudly introduce a brand new reimplementation, which is now available via the function `imposters()`. The new version of `stylo` is not available at CRAN yet, because I will have to introduce some other functionalities, too. If you’re interested in testing the new `imposters()` function, please install the package from GitHub:

``` r
library(devtools)
install_github("computationalstylistics/stylo")
```





## Quick start

To test at a glance what the new function can offer, type the following code:

``` r
# activating the package 'stylo':
library(stylo)

# activating one of the datasets provided by the package 'stylo';
# this is a table of frequences of a few novels, including "The Cuckoo's Calling"
# by Robert Galbraith, aka JK Rowling:
data(galbraith)

# to learn more about the dataset, type:
help(galbraith)

# to see the table itself, type:
galbraith

# now, time for the imposters method:
imposters(galbraith)
```





## Details

As you might have noticed, the dataset `galbraith` contains frequencies for different texts by a few authors, the class GALBRAITH being represented by a single text, though. Having no additional parameters, the function tries to identify such a single text and then assumes that this is the anonymous sample to be assessed. 

In a vast majority of cases, however, one would like to have some control on choosing the text to be contrasted against the corpus. There is a dedicated parameter `test` to do the trick. Note the following code:

``` r
# getting the 8th row from the dataset (it contains frequencies for Galbraith):
my_text_to_be_tested = galbraith[8,]

# building the reference set so that it does not contain the 8th row
my_frequency_table = galbraith[-c(8),]

# launching the imposters method:
imposters(reference.set = my_frequency_table, test = my_text_to_be_tested)
```

Consequently, if you want to test who wrote “The Lord of the Rings”, part 1, get the 24th row from the table: 

``` r
my_text_to_be_tested = galbraith[24,]
my_frequency_table = galbraith[-c(24),]
imposters(reference.set = my_frequency_table, test = my_text_to_be_tested)
```

So far, I’ve been neglecting one important feature of the imposters method. As Kestemont et al. (2016) show in their “Algorithm 1”, the method tries to compare an anonymous text against (1) a candidate set, containing the works of a probable candidate author, and (2) the imposters set, containing lots of text by people who could not have written the text in question. In the previous examples, where no canditate set was explicitly indicated, the method simply repeatedly tested all of possible authors as potential candidates. It is a time consuming task. If you want to do the _task_ properly, e.g. if you want to test if “The cuckoo’s Calling” was written by JK Rowling, you should define the parameters as follows:

``` r
# indicating the text to be tested (here, "The cuckoo's Calling"):
my_text_to_be_tested = galbraith[8,]

# defining the texts by the candidate author (here, the texts by JK Rowling):
my_candidate = galbraith[16:23,]

# building the reference set by excluding the already-selected rows
my_imposters = galbraith[-c(8, 16:23),]

# launching the imposters method:
imposters(reference.set = my_imposters, test = my_text_to_be_tested, candidate.set = my_candidate)
```

In practice, however, I’d rather test all the authors iteratively, even if this requires quite a lot of time to complete the task. In other words, I’d rather compare the behavior of JK Rowling in comparison to other candidate authors.






## Loading a corpus from files

I am fully aware that the function `imposters()` in its current form requires some advanced knowledge of R, since it does not provide any pre-processing. Specifically, one needs to know in advance how to produce a table of frequencies. This step has been already described elsewhere (Eder et al. 2016: 109–111), therefore I will not go into nuanced details here. A straightforward way to get from raw text files to the imposters results might look as follows:

``` r
# activating the package
library(stylo)

# setting a working directory that contains the corpus, e.g.
setwd("/Users/m/Desktop/A_Small_Collection_of_British_Fiction/corpus")

# loading the files from a specified directory:
tokenized.texts = load.corpus.and.parse(files = "all")

# computing a list of most frequent words (trimmed to top 2000 items):
features = make.frequency.list(tokenized.texts, head = 2000)

# producing a table of relative frequencies:
data = make.table.of.frequencies(tokenized.texts, features, relative = TRUE)

# who wrote "Pride and Prejudice"? (in my case, this is the 4th row in the table):
imposters(reference.set = data[-c(4),], test = data[4,])
```





## Parameters

In its current form, the function `imposters()` works with the Delta method only. Next versions will provide SVM, NSC, kNN and NaiveBayes. As most of you know very well, the general Delta framework can be combined with many different distance measures. E.g. in their paper introducing the imposters method (Kestemont et al. 2016), the authors argue that the Ruzicka metrics (aka Minmax) outperforms other measures. Similarly, the Wurzburg guys (Jannidis et al. 2015) show that Cosine Delta rocks when compared to other distances. My implementation of the `imposters()` applies Classic Delta by default, but other measures can be used as well. Try the following options:


``` r
# activating the package 'stylo':
library(stylo)

# activating one of the datasets provided by the package 'stylo':
data(galbraith)

# Classic Delta distance
imposters(galbraith, distance = "delta")

# Cosine Delta (aka Wurzburg Distance)
imposters(galbraith, distance = "wurzburg")

# Ruzicka Distance (aka Minmax Distance)
# (please keep in mind that it takes AGES to compute it!)
imposters(galbraith, distance = "minmax")
```

Not really impressed, right? This is because the signal of JK Rowling is really strong, and all the measures perform just fine. Let’s try something more difficult. Did you know that “In Cold Blood” by Truman Capote is stylometrically hard to associate with its actual author? Execute the following code:

``` r
# activating the package 'stylo':
library(stylo)

# activating another dataset, which contains Southern American novels:
data(lee)

# defining the test text, i.e. "In Cold Blood"
my_text_to_be_tested = lee[1,]

# defining the comparison corpus
my_reference_set = lee[-c(1),]

# Classic Delta distance
imposters(my_reference_set, my_text_to_be_tested, distance = "delta")

# Eder's Delta distance
imposters(my_reference_set, my_text_to_be_tested, distance = "eder")

# Cosine Delta (aka Wurzburg Distance)
imposters(my_reference_set, my_text_to_be_tested, distance = "wurzburg")

# Ruzicka Distance (aka Minmax Distance)
# (please keep in mind that it takes AGES to compute it!)
imposters(my_reference_set, my_text_to_be_tested, distance = "minmax")
```

Have you noticed the amazing improvement of Wurzburg Delta over the other measures? It’s really cool!






## References

Eder, M., Rybicki, J. and Kestemont, M. (2016). Stylometry with R: a package for computational text analysis. “R Journal”, 8(1): 107-121.

Jannidis, F., Pielstrom, S., Schoch, Ch. and Vitt, Th. (2015). Improving Burrows’ Delta: An empirical evaluation of text distance measures. In: “Digital Humanities 2015: Conference Abstracts” <URL: http://dh2015.org/abstracts>.

Kestemont, M., Stover, J., Koppel, M., Karsdorp, F. and Daelemans, W. (2016). Authenticating the writings of Julius Caesar. “Expert Systems With Applications”, 63: 86-96.

Koppel, M. , and Winter, Y. (2014). Determining if two documents are written by the same author. “Journal of the Association for Information Science and Technology”, 65(1): 178-187.


