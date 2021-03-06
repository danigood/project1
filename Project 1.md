---
title: "Goodman Project 1"
output: html_document
---

In my project, I work with mouse adenovirus type 1 (MAV-1). Specifically, I'm looking at a cellular kinase and its potential role in antiviral defense. Canonically, protein kinase R (PKR) responds to the presence of double-stranded RNA and it becomes activated to phosphorylate the transcription factor eIF2alpha. When phosphorylated, eIF2alpha halts protein synthesis. GCN2 (general control nonderepressible 2) is another kinase that phosphorylates eIF2alpha; it's activated by amino acid deprivation and UV irradiation. However, recent studies have shown that GCN2 is also activated by Sindbis virus (Berlanga *et al.*, 2006) and HIV-1 (Cosnefroy *et al.*, 2013). I hypothesize that GCN2 is also important during MAV-1 infection.

To test my hypothesis, I have been working with GCN2 mutant mice, termed *Atchoum* mice. These mice were created during a screen done by the Beutler group. They used ENU mutagenesis to create a library of C57BL/6-background mice, each with single point mutation. They then harvested peritoneal macrophages from each strain of mouse and screened them for susceptibility to human adenovirus and mouse cytomegalovirus. They found that the *Atchoum* strain of mice showed increased susceptibility to infection by both human adenovirus and mouse cytomegalovirus. When they sequenced the genome of the *Atchoum* mouse, they found that the single point mutation was in the GCN2 gene. Further investigation determined that the mutation caused a missense mutation and caused incorrect posttranslational splicing that resulted in an inactive kinase.

My working hypothesis is that the *Atchoum* mice and macrophages will be more susceptible to infection by MAV-1. To test my hypothesis, I started with attempting to recapitulate the macrophage experiments as done by the Beutler group. I harvested peritoneal macrophages from *Atchoum* and C57BL/6 mice and plated them in 6 well plates. I infected them the next day with MAV-1 at an MOI of 1. After 24, 48, and 72 hours, I harvested individual wells of cells (each is one replicate) and separated the cells from the supernatant. I purified DNA from both the cell and supernatant fraction and used qPCR to quantify the viral loads.

I get the output in an excel document, which I converted to text documents, one for the cell fraction data and one for the supernatant fraction data.

```{r echo=FALSE}
#load data into r studio
celldata <- read.table(file="qPCR_Raw_Data_Cells.txt", header=T)
superdata <- read.table(file="qPCR_Raw_Data_Supernatant.txt", header=T)
```

Let's start with the cell fraction data. I don't get quantifiable levels of DNA from the cell fraction so I have to use housekeeping genes to normalize the amount of DNA over all of the samples. I use GAPDH, so I get GAPDH output in my data. Since I use the qPCR machine program to normalize the data, I no longer need the GAPDH information, I only need the mE1A data (viral load data). So I want to delete all of the rows that contain GAPDH data.

```{r echo=FALSE}
#remove GAPDH data
celldata <- celldata[celldata$target_name == "mE1A",]
```

Now I can focus on the viral load data. I'm going to separate the *Atchoum* and C57BL/6 data into two different tables, using the same method I did to remove the GAPDH data.

```{r echo=FALSE}
atc_celldata <- celldata[celldata$strain == "Atc",]
b6_celldata <- celldata[celldata$strain == "B6",]
```

When I load my samples into the qPCR plate, I do three replicates for each sample. So in the data output, there are three rows of data for each sample. I want to average the RQ data because that is the relevant data (it is the fold change in viral load with the C57BL/6 data set as a basepoint).

```{r echo=FALSE}
#average RQ data
avg_atc_celldata <- aggregate(.~sample_name, FUN=mean, data=atc_celldata)
avg_b6_celldata <- aggregate(.~sample_name, FUN=mean, data=b6_celldata)
```

Now that I have the average of all my samples, I want to plot the data so I can see it.

```{r echo=FALSE}
#plot the data
atc24_celldata <- avg_atc_celldata[avg_atc_celldata$time_point == 24,]
atc48_celldata <- avg_atc_celldata[avg_atc_celldata$time_point == 48,]
atc72_celldata <- avg_atc_celldata[avg_atc_celldata$time_point == 72,]
b624_celldata <- avg_b6_celldata[avg_b6_celldata$time_point == 24,]
b648_celldata <- avg_b6_celldata[avg_b6_celldata$time_point == 48,]
b672_celldata <- avg_b6_celldata[avg_b6_celldata$time_point == 72,]
xbar <- c(mean(atc24_celldata$RQ), mean(b624_celldata$RQ), mean(atc48_celldata$RQ), mean(b648_celldata$RQ), mean(atc72_celldata$RQ),mean(b672_celldata$RQ))
sd <- c(sd(atc24_celldata$RQ), sd(b624_celldata$RQ), sd(atc48_celldata$RQ), sd(b648_celldata$RQ), sd(atc72_celldata$RQ),sd(b672_celldata$RQ))
compiled <- rbind(avg_atc_celldata,avg_b6_celldata)
data <- tapply(compiled$RQ, list(compiled$strain,compiled$time_point), mean)
mp <- barplot(data,beside=T,col=c("blue","orange"), ylim=c(0,4),main="Cell Fraction Data",xlab="Hours Post Infection",ylab="Fold Change Over C57BL/6")
segments(mp, xbar - sd, mp, xbar + sd, lwd=2)
segments(mp - 0.1, xbar - sd, mp + 0.1, xbar - sd, lwd=2)
segments(mp - 0.1, xbar + sd, mp + 0.1, xbar + sd, lwd=2)
legend(1,4, c("Atchoum","C57BL/6"), fill=c("blue","orange"))
```

Now that I've visualized the data, I have an idea of whether the difference between the viral loads in the *Atchoum* versus C57BL/6 macrophages is going to be significantly different. I need to do some t-tests to confirm my hypothesis. I will do two sample t-tests for each time point, comparing the *Atchoum* macrophage viral loads to the C57BL/6 macrophage viral loads. See t-test outputs below, in order from 24 hours post infection to 72 hours post infection.

```{r echo=FALSE}
#analyze data
t.test(avg_atc_celldata$RQ[avg_atc_celldata$time_point == 24], avg_b6_celldata$RQ[avg_b6_celldata$time_point == 24])
t.test(avg_atc_celldata$RQ[avg_atc_celldata$time_point == 48], avg_b6_celldata$RQ[avg_b6_celldata$time_point == 48])
t.test(avg_atc_celldata$RQ[avg_atc_celldata$time_point == 72], avg_b6_celldata$RQ[avg_b6_celldata$time_point == 72])
```

That completes my analysis of the cellular fraction. Now I want to look at the supernatant data. I'm going to start again by separating the *Atchoum* and C57BL/6 data.

```{r echo=FALSE}
atc_superdata <- superdata[superdata$strain == "Atc",]
b6_superdata <- superdata[superdata$strain == "B6",]
```

Again, like with the cellular data, I do three replicates of each sample in the qPCR plate so I need to average each of these replicates to get a single viral load quantity for each sample. Since there is no cellular DNA in this fraction, I did not have to worry about having housekeeping genes for normalization. I just purified DNA from the same amount of supernatant for each sample and I plated the same amount of purified DNA for each sample and just quantitated the viral load directly.

```{r echo=FALSE}
#average the data replicates quantity
avg_atc_superdata <- aggregate(.~sample_name, FUN=mean, data=atc_superdata)
avg_b6_superdata <- aggregate(.~sample_name, FUN=mean, data=b6_superdata)
```

Now that I have the average of all the data, I want to graph it.

```{r echo=FALSE}
atc24_superdata <- avg_atc_superdata[avg_atc_superdata$time_point == 24,]
atc48_superdata <- avg_atc_superdata[avg_atc_superdata$time_point == 48,]
atc72_superdata <- avg_atc_superdata[avg_atc_superdata$time_point == 72,]
b624_superdata <- avg_b6_superdata[avg_b6_superdata$time_point == 24,]
b648_superdata <- avg_b6_superdata[avg_b6_superdata$time_point == 48,]
b672_superdata <- avg_b6_superdata[avg_b6_superdata$time_point == 72,]
xbar <- c(mean(atc24_superdata$quantity), mean(b624_superdata$quantity), mean(atc48_superdata$quantity), mean(b648_superdata$quantity), mean(atc72_superdata$quantity),mean(b672_superdata$quantity))
sd <- c(sd(atc24_superdata$quantity), sd(b624_superdata$quantity), sd(atc48_superdata$quantity), sd(b648_superdata$quantity), sd(atc72_superdata$quantity),sd(b672_superdata$quantity))
compiled <- rbind(avg_atc_superdata,avg_b6_superdata)
data <- tapply(compiled$quantity, list(compiled$strain,compiled$time_point), mean)
mp <- barplot(data,beside=T,col=c("blue","orange"), ylim=c(0,2e7),main="Supernatant Fraction Data",xlab="Hours Post Infection",ylab="Genome Copies Per 4 µL Supernatant")
segments(mp, xbar - sd, mp, xbar + sd, lwd=2)
segments(mp - 0.1, xbar - sd, mp + 0.1, xbar - sd, lwd=2)
segments(mp - 0.1, xbar + sd, mp + 0.1, xbar + sd, lwd=2)
legend(7,2e7, c("Atchoum","C57BL/6"), fill=c("blue","orange"))
```

Now that I've visualized the data, I have an idea of whether the difference between the viral loads in the *Atchoum* versus C57BL/6 supernatants is going to be significantly different. I need to do some t-tests to confirm my hypothesis. I will do two sample t-tests for each time point, comparing the *Atchoum* supernatant viral loads to the C57BL/6 supernatant viral loads. See t-test outputs below, in order from 24 hours post infection to 72 hours post infection.

```{r echo=FALSE}
#analyze data
t.test(avg_atc_superdata$quantity[avg_atc_superdata$time_point == 24], avg_b6_superdata$quantity[avg_b6_superdata$time_point == 24])
t.test(avg_atc_superdata$quantity[avg_atc_superdata$time_point == 48], avg_b6_superdata$quantity[avg_b6_superdata$time_point == 48])
t.test(avg_atc_superdata$quantity[avg_atc_superdata$time_point == 72], avg_b6_superdata$quantity[avg_b6_superdata$time_point == 72])
```

####Data Summary:
In the cellular fraction, there was a significanct difference in the viral loads between the *Atchoum* and C57BL/6 macrophages (p<0.0001). In the supernatant fraction, there was no significant difference in the viral laods between the *Atchoum* and C57BL/6 supernatants. The cell fraction data does support my hypothesis that *Atchoum* macrophages will be more susceptible to infection by MAV-1. While the supernatant fraction did not support this result, it is possible that if I had assayed the supernatant at later time points (i.e. 96 hours), then I would begin to see an increased viral load in the *Atchoum* supernatant (since the viral load only became significantly higher within the cell at 72 hours post infection). Future directions for this project will include an in vivo appraoch of infecting the *Atchoum* and C57BL/6 mice themselves and harvesting the brains at 3, 5, and 7 days post infection to see if the viral loads differ between the *Atchoum* and C57BL/6 brains.