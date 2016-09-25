#TKrelated
##**Estimating relatedness from low-coverage genomes using "forced homozygous" SNP data**

Copyright (c) Daniel Fernandes - dani.mag.fernandes@gmail.com

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as
    published by the Free Software Foundation, either version 3 of the
    License, or (at your option) any later version.
    
    This software is supplied without any warranty or guaranteed support.
    
    For more details on this license, see http://www.gnu.org/licenses/
***

###Citation
Fernandes DM, Sirak K, Novak M, Finarelli J, Byrne J, Connolly E, Carlsson J, Ferretti E, Pinhasi R, Carlsson J. 2016.
The Identification of a 1916 Irish Rebel: New Approach for Estimating Relatedness From Low Coverage Homozygous Genomes.
bioRxiv doi:10.1101/076992
***

###Description
This R script is composed by 9 different functions that can be used to:

1. estimate relatedness from *in silico* generated, and plot histograms for,
  * a) unrelated individuals (Queller and Goodnight's Rxy coefficient of relationship = 0%)
  * b) full siblings (Queller and Goodnight's Rxy coefficient of relationship = 50%)
  * c) half siblings (Queller and Goodnight's Rxy coefficient of relationship = 25%)
2. manipulate SNP data in non-binary PLINK format to be used for relatedness estimations

The main advantage of this script is the requirement of low amounts of genetic data for accurate estimations. This is possible due to the application of a "forced homozygous" approach, and has been shown to work with as few as 1400 common SNPs. It can be used for both forensic or archaeological applications, where reduced DNA amounts are frequently an issue. More information on the paper where this method was first applied: [doi:10.1101/076992 ](http://dx.doi.org/10.1101/076992)
***

###Third-Party Software Requirements
[**SPAGeDI**](http://ebe.ulb.ac.be/ebe/SPAGeDi.html "SPAGeDI")

    "SPAGeDi (Spatial Pattern Analysis of Genetic Diversity) is a computer package primarily 
    designed to characterise the spatial genetic structure of mapped individuals and/or
    mapped populations using genotype data of any ploidy level."
    http://ebe.ulb.ac.be/ebe/SPAGeDi.html

[**PLINK**](http://pngu.mgh.harvard.edu/~purcell/plink/ "PLINK")

    PLINK is a free, open-source whole genome association analysis toolset, designed to perform
    a range of basic, large-scale analyses in a computationally efficient manner.
    http://pngu.mgh.harvard.edu/~purcell/plink/
***

###Usage and Data Requirements
TKrelated requires individual data and allele frequencies both in non-binary PLINK format (*.ped/*.map and *.frq). Each PED file can only contain one individual.
More detailed input requirements and formats are described within the script.

####Example:
####Estimating relatedness on two given samples
For comparing two individuals one should start by having their PLINK files and an appropriate file with the allele frequencies. For this example, we will say that those frequencies were extracted from a PLINK file from the 1000 Genomes Project.

    ind1.ped
    ind1.map
    
    ind2.ped
    ind2.map
    
    1000genomes_European.bed
    1000genomes_European.bim
    1000genomes_European.fam
    1000genomes_allele_frequencies_European.frq

Use **relatedHomozSNP** function (line 605) to convert the PLINK data into SPAGeDI format.

    relatedHomozSNP(sample1="ind1",sample2="ind2",freqs="1000genomes_allele_frequencies_European.frq",run.spagedi=TRUE)

This should produce two files to be used with SPAGeDI.

    ind1_ind2_spagedi_input.txt
    ind1_ind2_spagedi_frequencies.txt

If argument *run.spagedi=TRUE*, it will also create an extra file in order to run the program automatically.

    SP_CMDS_ind1_ind2_spagedi_in.txt

The relatedness estimation generated by SPAGeDI will be found on the following section of *ind1_ind2_spagedi_out.txt*:
    
    Pairwise RELATIONSHIP coefficients (Queller & Goodnight, 1989)
    
    ALL LOCI	ind1	ind2
    ind1		0.1233
    ind2	0.1233

Due to the "forced homozygous" approach, the coefficients are expected to be the half-values of the originals, ie: a Queller and Goodnight coefficient of 50% under our approach will be presented as 25%, or 0.25. In the above example, 0.1233 would correspond to a Queller and Goodnight coefficient of 25%.

####Estimating relatedness on virtual individuals
At this point, one can use the frequencies of the common SNPs between the two samples to generate virtual individuals and simulate different degrees of relatedness. This way one can confirm the ranges for which the found coefficient of relatedness can vary, and/or assess if it lands on an interval where different relatedness coefficients overlap.

To do this, it is possible to use the generated SPAGeDI frequencies file to extract the frequencies of those specific SNPs from two different sources:

Any dataset, in this case, the original dataset: *1000genomes_European*:

    freqCommonSNPsAnyDataset(spagedi.freq.file="ind1_ind2_spagedi_frequencies.txt",dataset="1000genomes_European")

The same PLINK frequencies file used for the relatedness estimation (suggested use):

    freqCommonSNPsSameFreqFile(spagedi.freq.file="ind1_ind2_spagedi_frequencies.txt",plink.freq.file="1000genomes_allele_frequencies_European.frq")

The output of these functions is a new PLINK frequencies file:

    common_freqs_ind1_ind2.frq

This file can now be used to generate the virtual relatedness tests. For each of the three coefficients, 0, 25 and 50%, there are two functions. The first one generates the individuals and estimates their relatedness, and the second plots the corresponding histograms.

    unrelatedfunc = function(file,samples.tag,numUnrelatedPairs,identifier,run.spagedi=TRUE,reduce.SNPs=FALSE)
    unrelatedplot = function(samples.tag)
    
    fullsibsfunc = function(file,samples.tag,numFullSiblingsPairs,identifier,run.spagedi=TRUE,reduce.SNPs=FALSE)
    fullsibsplot = function(samples.tag) 
    
    halfsibsfunc = function(file,samples.tag,numHalfSiblingsPairs,identifier,run.spagedi=TRUE,reduce.SNPs=FALSE)
    halfsibsplot = function(samples.tag)

A detailed description of each argument exists within the function.

Below is a suggested set of commands to run the simulations, taking into consideration that SPAGeDI can become unstable due to memory issues.

    unrelatedfunc("common_freqs_ind1_ind2.frq","ind1ind2",250,1,run.spagedi = TRUE,reduce.SNPs = FALSE)
    unrelatedfunc("common_freqs_ind1_ind2.frq","ind1ind2",250,2,run.spagedi = TRUE,reduce.SNPs = FALSE)
    unrelatedfunc("common_freqs_ind1_ind2.frq","ind1ind2",250,3,run.spagedi = TRUE,reduce.SNPs = FALSE)
    unrelatedfunc("common_freqs_ind1_ind2.frq","ind1ind2",250,4,run.spagedi = TRUE,reduce.SNPs = FALSE)
    unrelatedplot("ind1ind2")

This will generate estimations for 1000 unrelated virtual individuals, in batches of 250. This way the script and SPAGeDI run faster and should not crash. It, however, is dependant on the amount of RAM available. The plotting function will recognize and load all four files with the specified *samples.tag*.

***

