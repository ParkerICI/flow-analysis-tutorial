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
When you need to harmonize panels across a number of files, panel editing and renaming is useful so that they can be prepared for downstream analysis (most analysis tools expect files that are part of the same analysis to have identical panels). The R package [premessa](https://github.com/ParkerICI/premessa) allows for such editing.

****(create a case from existing files where we'd have to apply this step)*****

### Starting the panel editing GUI and selecting the working directory and example data

You can start the normalizer GUI by typing the following commands in your R session:
```
library(premessa)
paneleditor_GUI()
```
This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory containing the data you want to analyze, and select any file in that directory. The directory itself will then become the working directory for the software.

![screen shot 2018-06-26 at 1 35 36 pm](https://user-images.githubusercontent.com/36977003/41937690-dac216a8-7945-11e8-95f2-ff9ad3a9dd9c.png)

To stop the software simply hit the "ESC" key in your R session.
_Note_: If the GUI does _not_ open a new web browser, hit the "ESC" key and re-enter the above command.


# Normalization
There are several sources of variability in flow-based data, and normalization is an essential method in controlling such variability. [Bead-based normalization](https://www.ncbi.nlm.nih.gov/pubmed/23512433) provides several advantages, as it controls variability over time uniquely for each event and accounts for instrument sensitivity changes in the short- and long- term.

The workflow for this normalization method invovles the following steps:
1. Bead identification through gating
2. Data normalization
3. Bead removal (optional)

Assuming you're using the [Bone marrow data](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) as your working directory and it contains two FCS files called BM_a_cells.FCS and BM_b_cells.FCS, at the end of the workflow the following directory structure and output files will be generated:

```
Bone_marrow
|--- BM_a_cells.FCS 
|--- BM_b_cells.FCS 
|--- normed
     |--- BM_a_cells_normalized.fcs
     |--- BM_b_cells_normalized.fcs
     |--- beads_before_and_after.pdf
     |--- beads_vs_time
          |--- BM_a_cells.pdf
          |--- BM_b_cells.pdf
     |--- beads_removed
          |--- BM_a_cells_normalized_beadsremoved.fcs
          |--- BM_b_cells_normalized_beadsremoved.fcs
          |--- removed_events
               |--- BM_a_cells_normalized_removedEvents.fcs
               |--- BM_b_cells_normalized_removedEvents.fcs
     |--- beads
          |--- BM_a_cells_beads.fcs
          |--- BM_b_cells_beads.fcs
```

- *BM_a_cells_normalized.fcs*: contains the normalized data, with an added parameter called *beadDist* representing the square root of the Mahalanobis distance of each event from the centroid of the beads population
- *beads_before_and_after.pdf*: a plot of the median intensities of the beads channels before and after normalization. This plot contains a single median value per sample. Therefore it will not be informative if you are normalizing a single sample
- *BM_a_cells.pdf*: a plot of the intensities of the beads channels along time, before and after normalization
- *BM_a_cells_normalized_beadsremoved.fcs*: the normalized data with the beads events removed
- *BM_a_cells_normalized_removedEevents.fcs*: the events that have been removed from the normalized data based on the Mahalanobis distance cutoff 
- *BM_a_cells_beads.fcs*: the beads events, as identified by gating

### Starting the normalization GUI and selecting the working directory and example data
You can start the normalizer GUI by typing the following commands in your R session:

```
library(premessa)
normalizer_GUI()
```

This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory (computer location) containing the data you want to analyze, and select any file in that directory. In this example, select a file from the [Bone marrow data](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets). The directory itself will then become the working directory for the software.

**The GUI contains two tabs**:

**1. Normalize data:** used for beads gating and normalization 

This panel contains the following controls:

- *Select beads type*: select the type of normalization beads that have been used for the experiment. Here we will use *Beta Beads (139, 141, 159, 175)*. The numbers indicate the beads channels used for normalization.
- *Select FCS file*: the FCS  that is currently being visualized for gating. This dropdown will contain all the FCS files located in the working directory. The gating plots will appear under the row of buttons.
- *Select baseline for normalization*: the baseline beads intensities to be used for normalization. You can either use the median beads intensities of the FCS files that you are currently using for normalization (*Current files* option), or the median intensities of an existing set of beads files (*Existing folder of beads files*). If you select the latter a file dialog window will pop-up when you select the option. Use the window to navigate to a directory containing FCS files containing beads events only (for instance the *BM_a_cells_beads.fcs* file in the above example) and select one of the files. The software will then load *all* the files contained in the same directory as the file you selected. The currently selected folder will be displayed in a text box on the right. 
![screen shot 2018-06-26 at 1 38 37 pm](https://user-images.githubusercontent.com/36977003/41937841-431a42ca-7946-11e8-9657-003f2cda5646.png)

- *Identify beads*: clicking this button will color in red the events that are recognized as beads events in the gating plots.
![screen shot 2018-06-26 at 1 39 19 pm](https://user-images.githubusercontent.com/36977003/41937870-5c8aadee-7946-11e8-8256-21085d60a2c3.png)

- *Apply current gates to all files*: applies the current gates to all the files.
![screen shot 2018-06-26 at 1 39 54 pm](https://user-images.githubusercontent.com/36977003/41937909-71794288-7946-11e8-9fea-dc391ac7652e.png)

- *Normalize*: starts the normalization routine. When the process is completed a confirmation dialog will appear.
The workflow involves cycling through all the files (there are five of them- a through e in our example) and adjusting the beads gates in the plot, in order to identify the beads. Only events that are included in *all* the beads gates are identified as beads. As detailed in the dialog box that is above the row of buttons, only files for which the gates have been defined will be used as input for normalization.

You can cycle back and forth between different files, as the GUI will remember the gates you have selected for each file.

**2. Remove beads**: used for beads removal 

This panel has the following controls:

- *Select beads type*: same as for the *Normalize data* panel: select the type of normalization beads that have been used for the experiment. In our example, we are using the Beta Beads option.
- *Select FCS file*: select the FCS file for plotting. The dropdown will contain all the FCS files located in the *normed* sub-folder of the working directory. The plots will appear below the row of buttons. See below for a description of what the plots represent
- *Cutoff for bead removal*: the Mahalanobis distance cutoff to be used for bead removal. This requires looking at the data and chosing an appropriate cut point (orange=beads, blue=no beads)



- *Remove beads (current file)*: use the current cutoff to remove beads from the currently selected file. When the process is completed a confirmation dialog will appear
![screen shot 2018-06-26 at 2 07 53 pm](https://user-images.githubusercontent.com/36977003/41939347-5a67c124-794a-11e8-8a3c-e1a15bf3aedb.png)

- *Remove beads (all files)*: use the current cutoff to remove beads from all the files in the folder (i.e. all the files that are listed in the *Select FCS file* dropdown). When the process is completed a confirmation dialog will appear
 ![screen shot 2018-06-26 at 2 11 28 pm](https://user-images.githubusercontent.com/36977003/41939495-daa079a8-794a-11e8-970d-94a1ae2a665f.png)
 
 
The bead removal procedure is based on the idea of looking at the distance between each event and the centroid of the beads population, and removing all the events that are closer than a given threshold to the beads population, and therefore are likely to represent beads as opposed to true cells.

To this end, during normalization, the software calculates the square root of the Mahalanobis distance of each event from the centroid of the beads population, and records this information in the *beadDist* parameter in the FCS file with the normalized data (i.e. the *_normalized.fcs* files in the *normed* sub-folder).

During the beads removal step, all the events whose *beadDist* is less or equal than the *Cutoff for bead removal* parameter are removed from the FCS. The removed events are saved in the *removed_events* sub-folder (see above).

The plots in the bottom half of the panel help you select an appropriate cutoff. They display all the pairs of beads channels. Beads should appear as a population in the upper right corner (as they will be double-positives for all the channel pairs). The color of the points represent the distance from the beads population. You should choose a cutoff so that most of the bead events are below the cutoff, and most of the non-beads events are above it. The legend for the color scale is located above the plots.

To stop the software simply hit the "ESC" key in your R session. _Note_: If the GUI does _not_ open a new web browser, hit the "ESC" key and re-enter the above command.


# De-barcoding
* De-barcoding steps also contained in [premessa](https://github.com/ParkerICI/premessa) package
* Use excel file titled "ScienceDataset_DebarcodedKey" in the [Science datasets](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) folder
* Based on the distance between positive and negative channels
* Check barcoding worked by viewing summary plots of barcode yields

### Starting the de-barcoding GUI and selecting the working directory and example data

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
