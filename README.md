# July-2018-single-cell-workshop
This repository is a resource for PICI members performing end-to-end single-cell analysis.

# Installation
**The following R packages are required for this tutorial:**
1. [devtools](https://github.com/r-lib/devtools)
2. [flowCore](http://bioconductor.org/biocLite.R)
3. [premessa](https://github.com/ParkerICI/premessa)
4. [scgraphs](https://github.com/ParkerICI/scgraphs)
5. [scfeatures](https://github.com/ParkerICI/scfeatures)
6. [scaffold](https://github.com/nolanlab/scaffold)
7. [Rtsne](https://github.com/jkrijthe/Rtsne)

To install these packages, open an R session and enter the following command lines:
```
install.packages("devtools")

source("http://bioconductor.org/biocLite.R")
biocLite("flowCore")

install.packages("premessa")

install.packages("scgraphs")

install.packages("scfeatures")

install.packages("scaffold")

install.packages("Rtsne")
```

## Usage
In this tutorial, we will go step-by-step covering methods to optimally visualize single cell data. After downloading the [datasets](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) (from a 2015 [publication](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4537647/) in _Science_), we will go through this method in order of the following general steps:

* [Normalization](#normalization)
* [De-barcoding](#de-barcoding)
* [Gating](#gating)
* [Visualization](#visualization)
  * [Unsupervised visualization](#unsupervised-visualization)
  * [Supervised visualization](#supervised-visualization)
* [Associating cell populations with endpoints of interest](#associating-cell-populations-with-endpoints-of-interest)
  

## Panel editing
When you need to harmonize panels across a number of files, panel editing and renaming is useful so that they can be prepared for downstream analysis (most analysis tools expect files that are part of the same analysis to have identical panels). The [premessa](https://github.com/ParkerICI/premessa) package allows for such editing.

(create a case from existing files where we'd have to apply this step)

### Starting the GUI and selecting the working directory

You can start the normalizer GUI by typing the following commands in your R session:
```
library(premessa)
paneleditor_GUI()
```
This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory containing the data you want to analyze, and select any file in that directory. The directory itself will then become the working directory for the software.

To stop the software simply hit the "ESC" key in your R session.
_Note_: If the GUI does _not_ open a new web browser, hit the "ESC" key and re-enter the above command.



# Normalization
There are several sources of variability in flow-based data, and normalization is an essential method in controlling such variability. [Bead-based normalization](https://www.ncbi.nlm.nih.gov/pubmed/23512433) provides several advantages, as it controls variability over time and 


You can start the normalizer GUI by typing the following commands in your R session:


This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory containing the data you want to analyze, and select any file in that directory. The directory itself will then become the working directory for the software.

To stop the software simply hit the "ESC" key in your R session.

```
library(premessa)
normalizer_GUI()
```

This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory containing the data you want to analyze, and select any file in that directory. The directory itself will then become the working directory for the software.

To stop the software simply hit the "ESC" key in your R session. Check normalization worked by plotting bead intensitites before and after.


# De-barcoding
* De-barcoding steps also contained in [premessa](https://github.com/ParkerICI/premessa) package
* Use excel file titled "ScienceDataset_DebarcodedKey" in the [Science datasets](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) folder
* Based on the distance between positive and negative channels
* Check barcoding worked by viewing summary plots of barcode yields

```
library(premessa)
debarcoding_GUI()
```

# Gating
* Check data, clean to make sure doublets, dead cells and unwanted populations removed
* Manually gated data will establish landmark nodes, for this use desired software system (we use Cell Engine)

Cluster data 3 ways:
* Cluster each sample independently
* Pool all files then cluster--> extract features from each group of cells (like change in expression, fold change) and look at association with endpoint
* Pool all files from a given tissue type and cluster

# Visualization
## Unsupervised visualization 
The first task is to get and idea of the structure of the sample— what cells are in there? How many different populations can we identify as a first pass? What are the markers combinations that define such populations? Rather than using standard dimension-reduction methods PCA or tSNE, both of which have limitations, we will use **Force-directed graphs** to achieve this.

Like other graphical methods, **Force-directed graphs** (_cite_) offer flexibility of visulization, as each node represents one cell (or cluster of cells) and each edge represents similarity between cells which can correspond to a variety of distance metrics or functions, but for our purposes we will use cosine (_link scgraphs package_).  In this layout the edges act as springs, whose strength is proportional to the weight of the edge (i.e. the similarity between the nodes). The algorithm then proceeds as a physical simulation by pulling together nodes that are similar (i.e. are connected by strong springs), and repelling dissimilar nodes. The end result is a layout where groups of similar cells are located close on the page, exactly what we set out to do for our visualization. (_cite Zunder:2015gp, Levine:2015ew, Samusik:2016ev, Spitzer:2015jd_?}

* Number of neighbours=15

## Supervised visualization 
* [Scaffold](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4537647/) method using [scaffold](https://github.com/nolanlab/scaffold) package

# Associating cell populations with endpoint(s) of interest
* Finding  cell populations that are significantly associated with an outcome (which in our example is tissue type but could be predicting disease states, prognosis or survival times).
* Statistical analysis of microarrays with Citrus(?)
* Pool all files then cluster--> extract features from each group of cells (like change in expression, fold change) 
* Generates a matrix of cluster/feature combinations 
