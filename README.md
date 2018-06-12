# July-2018-single-cell-workshop
This repository is a resources for attendees of the workshop hosted at PICI in July 2018.


## Repository content
1.	Outline of instructions (below)
2.	Datasets from Science (2015) paper         **Blood and 1 of the bone marrow files need upload via LFS**
3.	R codes

## Prior to beginning make sure download required R packages:
1. [premessa](https://github.com/ParkerICI/premessa)
2. [scgraphs](https://github.com/ParkerICI/scgraphsScgraph)
3. [scfeatures](https://github.com/ParkerICI/scfeatures)
4. [scaffold](https://github.com/nolanlab/scaffold)
5. [Rtsne](https://github.com/jkrijthe/Rtsne)

## Introduction
In this tutorial, we will go step-by-step covering methods to optimally visualize single cell data. The dataset we'll use as an example to go through these methods is from the a 2015 [publication in Science](http://science.sciencemag.org/content/349/6244/1259425). After downloading the R packages listed above, the next step is to download the datasets from this repository.

## Normalization
* Bead based normalization using premessa package
* Check normalization worked by plotting bead intensitites before and after


## Debarcoding
* Check barcoding worked by viewing summary plots of barcode yields

## Gating
* Check data, clean to make sure doublets, dead cells and unwanted populations removed
* Use desired software system (we use Cell Engine)

Cluster data 3 ways:
* Cluster each sample independently
* Pool all files then cluster--> extract features from each group of cells (like change in expression, fold change) and look at association with endpoint
* Pool all files from a given tissue type and cluster

## Unsupervised visualization 
The first task is to get and idea of the structure of the sample— what cells are in there? How many different populations can we identify as a first pass? What are the markers combinations that define such populations? Rather than using standard dimension-reduction methods PCA or tSNE, both of which have limitations, we will use **Force-directed graphs** to achieve this.

Like other graphical methods, **Force-directed graphs** (_cite_) offer flexibility of visulization, as each node represents one cell (or cluster of cells) and each edge represents similarity between cells which can correspond to a variety of distance metrics or functions, but for our purposes we will use cosine (_link scgraphs package_).  In this layout the edges act as springs, whose strength is proportional to the weight of the edge (i.e. the similarity between the nodes). The algorithm then proceeds as a physical simulation by pulling together nodes that are similar (i.e. are connected by strong springs), and repelling dissimilar nodes. The end result is a layout where groups of similar cells are located close on the page, exactly what we set out to do for our visualization. (_cite Zunder:2015gp, Levine:2015ew, Samusik:2016ev, Spitzer:2015jd_?}

* Number of neighbours=15

## Supervised visualization 
* Scaffold method

## Associating cell populations with endpoint(s) of interest
* Finding  cell populations that are significantly associated with an outcome (which in our example is tissue type but could be predicting disease states, prognosis or survival times).
* Statistical analysis of microarrays with Citrus(?)
* Pool all files then cluster--> extract features from each group of cells (like change in expression, fold change) 
* Generates a matrix of cluster/feature combinations 
