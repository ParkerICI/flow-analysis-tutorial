# July-2018-single-cell-workshop
This repository is a resources for attendees of the workshop hosted at PICI in July 2018.

# Installation
## The following R packages are required for this tutorial:
1. [premessa](https://github.com/ParkerICI/premessa)
2. [scgraphs](https://github.com/ParkerICI/scgraphsScgraph)
3. [scfeatures](https://github.com/ParkerICI/scfeatures)
4. [scaffold](https://github.com/nolanlab/scaffold)
5. [Rtsne](https://github.com/jkrijthe/Rtsne)

## Usage
In this tutorial, we will go step-by-step covering methods to optimally visualize single cell data. After downloading the R packages and [datasets](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) (from a 2015 [publication](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4537647/) in _Science_), we will go through the following steps:

* [Normalization](#normalization)
* [De-barcoding](#de-barcoding)
* [Gating](#gating)
* [Visualization](#visualization)
  * [Unsupervised visualization](#unsupervised-visualization)
  * [Supervised visualization](#supervised-visualization)
* [Associating cell populations with endpoints of interest](#associating-cell-populations-with-endpoints-of-interest)
  

## Normalization
* Bead based normalization using premessa package
* Check normalization worked by plotting bead intensitites before and after


## De-barcoding
* Check barcoding worked by viewing summary plots of barcode yields

## Gating
* Check data, clean to make sure doublets, dead cells and unwanted populations removed
* Use desired software system (we use Cell Engine)

Cluster data 3 ways:
* Cluster each sample independently
* Pool all files then cluster--> extract features from each group of cells (like change in expression, fold change) and look at association with endpoint
* Pool all files from a given tissue type and cluster

## Visualization
### Unsupervised visualization 
The first task is to get and idea of the structure of the sample— what cells are in there? How many different populations can we identify as a first pass? What are the markers combinations that define such populations? Rather than using standard dimension-reduction methods PCA or tSNE, both of which have limitations, we will use **Force-directed graphs** to achieve this.

Like other graphical methods, **Force-directed graphs** (_cite_) offer flexibility of visulization, as each node represents one cell (or cluster of cells) and each edge represents similarity between cells which can correspond to a variety of distance metrics or functions, but for our purposes we will use cosine (_link scgraphs package_).  In this layout the edges act as springs, whose strength is proportional to the weight of the edge (i.e. the similarity between the nodes). The algorithm then proceeds as a physical simulation by pulling together nodes that are similar (i.e. are connected by strong springs), and repelling dissimilar nodes. The end result is a layout where groups of similar cells are located close on the page, exactly what we set out to do for our visualization. (_cite Zunder:2015gp, Levine:2015ew, Samusik:2016ev, Spitzer:2015jd_?}

* Number of neighbours=15

### Supervised visualization 
* Scaffold method

## Associating cell populations with endpoint(s) of interest
* Finding  cell populations that are significantly associated with an outcome (which in our example is tissue type but could be predicting disease states, prognosis or survival times).
* Statistical analysis of microarrays with Citrus(?)
* Pool all files then cluster--> extract features from each group of cells (like change in expression, fold change) 
* Generates a matrix of cluster/feature combinations 
