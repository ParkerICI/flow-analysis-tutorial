# July-2018-single-cell-workshop
This repository is a resource for PICI members performing end-to-end single-cell analysis.

# Installation
**The following R packages are required for this tutorial:**
1. [devtools](https://github.com/r-lib/devtools)
2. [flowCore](http://bioconductor.org/biocLite.R)
3. [premessa](https://github.com/ParkerICI/premessa)
4. [scgraphs](https://github.com/ParkerICI/scgraphs)
5. [scfeatures](https://github.com/ParkerICI/scfeatures)
6. [scaffold](https://github.com/ParkerICI/scaffold2)
7. [Rtsne](https://github.com/jkrijthe/Rtsne)

**Install a C++ compiler**
Some steps require a working C++ compiler for installation- please refer to the appropriate option depending on your system.

#### Mac OSX
You need to install the XCode software from Apple that is freely available on the App Store. Depending on the specific version of XCode you are using you might also need to install the "Command Line Tools" package separately. Please refer to the Documentation for your XCode version

#### Windows
Install the [Rtools](https://cran.r-project.org/bin/windows/Rtools/) package, which is required for building R packaged from sources

#### Linux
Install GCC. Refer to the documentation of your distribution to find the specific package name

### Install R Packages
To install these packages, open an R session and enter the following command lines:
```
install.packages("devtools")

source("http://bioconductor.org/biocLite.R")
biocLite("flowCore")

install.packages("premessa")

install.packages("scgraphs")

library(devtools)
devtools::install_github("ParkerICI/scfeatures")

library(devtools)
devtools::install_github("ParkerICI/scaffold2")
library(scaffold2)
scaffold2()

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

Assuming you're using the example [Science data](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) as your working directory and it contains two FCS files called BM_cells.FCS and SPL_cells.FCS, at the end of the workflow the following directory structure and output files will be generated:

```
Science data
|--- BM_cells.FCS 
|--- SPL_cells.FCS 
|--- LN_cells.FCS 
|--- BLD_cells.FCS
|--- normed
     |--- BM_cells_normalized.fcs
     |--- SPL_cells_normalized.fcs
     |--- LN_cells_normalized.FCS 
     |--- BLD_cells_normalized.FCS
     |--- beads_before_and_after.pdf
     |--- beads_vs_time
          |--- BM_cells.pdf
          |--- SPL_cells.pdf
          |--- LN_cells.pdf
          |--- BLD_cells.pdf
     |--- beads_removed
          |--- BM_cells_normalized_beadsremoved.fcs
          |--- SPL_cells_normalized_beadsremoved.fcs
          |--- LN_cells_normalized_beadsremoved.fcs
          |--- BLD_cells_normalized_beadsremoved.fcs
          |--- removed_events
               |--- BM_cells_normalized_removedEvents.fcs
               |--- SPL_cells_normalized_removedEvents.fcs
               |--- LN_cells_normalized_removedEvents.fcs
               |--- BLD_cells_normalized_removedEvents.fcs 
     |--- beads
          |--- BM_cells_beads.fcs
          |--- SPL_cells_beads.fcs
          |--- LN_cells_beads.fcs
          |--- BLD_cells_beads.fcs
```

- *BM_cells_normalized.fcs*: contains the normalized data, with an added parameter called *beadDist* representing the square root of the Mahalanobis distance of each event from the centroid of the beads population
- *beads_before_and_after.pdf*: a plot of the median intensities of the beads channels before and after normalization. This plot contains a single median value per sample. Therefore it will not be informative if you are normalizing a single sample
- *BM_cells.pdf*: a plot of the intensities of the beads channels along time, before and after normalization
- *BM_cells_normalized_beadsremoved.fcs*: the normalized data with the beads events removed
- *BM_cells_normalized_removedEevents.fcs*: the events that have been removed from the normalized data based on the Mahalanobis distance cutoff 
- *BM_cells_beads.fcs*: the beads events, as identified by gating

### Starting the normalization GUI and selecting the working directory and example data
You can start the normalizer GUI by typing the following commands in your R session:

```
library(premessa)
normalizer_GUI()
```

This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory (computer location) containing the data you want to analyze, and select any file in that directory. In this example, select a file from the [Bone marrow data](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets). The directory itself will then become the working directory for the software. Make sure that * ALL * files are selected in your window- this requires one-by-one selection from the drop down box.

**The GUI contains two tabs**:

**1. Normalize data:** used for beads gating and normalization 

This panel contains the following controls:

- *Select beads type*: select the type of normalization beads that have been used for the experiment. Here we will use *Beta Beads (139, 141, 159, 175)*. The numbers indicate the beads channels used for normalization.
- *Select FCS file*: the FCS  that is currently being visualized for gating. This dropdown will contain all the FCS files located in the working directory. If you want to select multiple files to work with at once, these must be selected one-by-one from the drop down box until all files appear in the window under "_You have gated beads for the following files_." The gating plots will appear under the row of buttons.
- *Select baseline for normalization*: the baseline beads intensities to be used for normalization. You can either use the median beads intensities of the FCS files that you are currently using for normalization (*Current files* option), or the median intensities of an existing set of beads files (*Existing folder of beads files*). If you select the latter a file dialog window will pop-up when you select the option. Use the window to navigate to a directory containing FCS files containing beads events only (for instance the *BM_cells_beads.fcs* file in the above example) and select one of the files. The software will then load *all* the files contained in the same directory as the file you selected. The currently selected folder will be displayed in a text box on the right. 

- *Identify beads*: clicking this button will color in red the events that are recognized as beads events in the gating plots.

- *Apply current gates to all files*: applies the current gates to all the files. At this point the screen should look like this:
![screen shot 2018-07-12 at 5 20 42 pm](https://user-images.githubusercontent.com/36977003/42660823-443255ba-85fa-11e8-937c-a4d6726d7944.png)

- *Normalize*: starts the normalization routine. 

When the process is completed a confirmation dialog will appear:
![screen shot 2018-07-12 at 5 36 18 pm](https://user-images.githubusercontent.com/36977003/42660766-22f11fe4-85fa-11e8-87a0-9b9edff83a9f.png)

The normalized versions of the selected files will be  saved in the directory you selected the files from. The workflow involves cycling through all the files (there are four of them in our example) and adjusting the beads gates in the plot, in order to identify the beads. Only events that are included in *all* the beads gates are identified as beads. As detailed in the dialog box that is above the row of buttons, only files for which the gates have been defined will be used as input for normalization. 

You can cycle back and forth between different files, as the GUI will remember the gates you have selected for each file.

**2. Remove beads**: used for beads removal 

This panel has the following controls:

- *Select beads type*: same as for the *Normalize data* panel: select the type of normalization beads that have been used for the experiment. In our example, we are using the Beta Beads option.
- *Select FCS file*: select the FCS file for plotting. The dropdown will contain all the FCS files located in the *normed* sub-folder of the working directory. The plots will appear below the row of buttons. See below for a description of what the plots represent
- *Cutoff for bead removal*: the Mahalanobis distance cutoff to be used for bead removal. This requires looking at the data and chosing an appropriate cut point (orange=beads, blue=no beads, more on this below)
- *Remove beads (current file)*: use the current cutoff to remove beads from the currently selected file.
- *Remove beads (all files)*: use the current cutoff to remove beads from all the files in the folder (i.e. all the files that are listed in the *Select FCS file* dropdown):
![screen shot 2018-07-12 at 5 52 07](https://user-images.githubusercontent.com/36977003/42661497-56b6293a-85fc-11e8-95ad-1ff105bfca39.png)

When the process is completed, the following screen will appear:

The **bead removal procedure** is based on the idea of looking at the distance between each event and the centroid of the beads population, and removing all the events that are closer than a given threshold to the beads population, and therefore are likely to represent beads as opposed to true cells.

To this end, during normalization, the software calculates the square root of the Mahalanobis distance of each event from the centroid of the beads population, and records this information in the *beadDist* parameter in the FCS file with the normalized data (i.e. the *_normalized.fcs* files in the *normed* sub-folder).

During the beads removal step, all the events whose *beadDist* is less or equal than the *Cutoff for bead removal* parameter are removed from the FCS. The removed events are saved in the *removed_events* sub-folder (see above).

The plots in the bottom half of the panel help you select an appropriate cutoff. They display all the pairs of beads channels. Beads should appear as a population in the upper right corner (as they will be double-positives for all the channel pairs). The color of the points represent the distance from the beads population. You should choose a cutoff so that most of the bead events (orange) are below the cutoff, and most of the non-beads events (blue) are above it. The legend for the color scale is located above the plots.

To stop the software simply hit the "ESC" key in your R session. _Note_: If the GUI does _not_ open a new web browser, hit the "ESC" key and re-enter the above command.

# De-barcoding
Combinatorial barcoding allows for identifying and removing errors from your dataset. [This paper](https://www.ncbi.nlm.nih.gov/pubmed/25612231) describes the concept of applying single cell-based debarcoding algorithms and it's advantages over population-based methods in terms of allowing for rapid and unbiased sample deconvolution.

Assuming the FCS file *BM_cells.fcs* is located in the directory *Science Data*, and the barcode key defines 3 barcoded populations (A, B, C), the following directories and output files will be created at the end of the debarcoding process:

```
Science Data
|--- BM_cells.fcs
|--- debarcoded
     |--- BM_cells.A.fcs.fcs
     |--- BM_cells.B.fcs.fcs
     |--- BM_cells.C.fcs
     |--- BM_cells_Unassigned.fcs
```

### Starting the de-barcoding GUI and selecting the working directory and example data
You can start the de-barcoding GUI by typing the following commands in your R session:

```
library(premessa)
debarcoding_GUI()
```
Upon launching the GUI you will have access to the following controls:

- *Current barcode key*: the CSV file containing the barcode key. To upload the key to follow our example dataset, press the *Select key* button and select the [file](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets).
![screen shot 2018-06-26 at 2 54 08 pm](https://user-images.githubusercontent.com/36977003/41941422-ce225790-7950-11e8-9a04-daf49ac46784.png)

- *Current FCS file*: the FCS that is currently being debarcoded. Here we will again continue with the bone marrow data file. Press the *Select FCS* button to select the FCS file you want to debarcode. Upon selecting both the FCS file and the key, the preliminary debarcoding process will start immediately. After a few seconds a number of diagnostic plots will appear in the right portion of the window (see [below](#plot-types))
- *Minimum separation*: the minimum seperation between the positive and negative barcode channels that an event needs to have in order to be assigned to a sample. Events where the separation is less than this threshold are left unassigned. This filtering is done after rescaling the intensity of the barcode channels, and therefore the threshold should be a number between 0 and 1. For our example we will use a minimum separation= 0.4.
- *Maximum Mahalanobis distance*: the maximum distance between a single cell event, and the centroid of the sample the event has been assigned to. Events with distance greather than the threshold are left unassigned. The distance is capped at 30, so the default value of this option does not apply a filter based on Mahalanobis distance. Note that in some cases (for instance when there are very few events in a sample), the Mahalanobis distance calculation may fail. In this case a diagnostic message is printed in the R console, and filtering based on Mahalanobis distance is not performed for the corresponding sample.
- *Plot type*: selects the type of plot to be displayed. Please see [below](#plot-types) for a description of the plots. Depending on the plot type, a few additional controls may be displayed:
  - *Select sample*: select a specific sample for plotting. Sample names are taken from the barcode key
  - *Select x axis*: select the channel to be displayed on the x axis
  - *Select y axis*: select the channel to be displayed on the y axis
- *Save files*: hitting this button will apply the current settings, performed the debarcoding, and save the resulting output files
![screen shot 2018-06-26 at 3 01 26 pm](https://user-images.githubusercontent.com/36977003/41941778-d5cb090a-7951-11e8-9d93-9a103d5b2b66.png)

### Plot types

There are four types of visualization that allow you to inspect the results of the debarcoding process, and choose the optimal seperation and Mahalanobis distance thresholds. Each plot window is subdivided into a top and bottom section, as described below:

- *Separation*
  - *Top*: A histogram of the separation between the positive and negative barcode channels for all the events
  - *Bottom*: Barcode yields as a function of the separation threshold. As the threshold increases, the number of cells assigned to each sample decreases, and more events are left unassigned. The currently selected threshold is displayed as a vertical red line.
- *Event*
  - *Top*: Bargraph or cell yields for each sample after debarcoding, given the current settings
  - *Bottom*: Scatterplot showing the barcode channel intensities for each event, ordered on the x axis. The left plot displays the original data, arcsinh transformed. The plot on the right displays the data rescaled between 0 and 1, which is actually used for debarcoding. Both plots only displays data for the selected sample (use the *Select sample* dropdown to pick the active sample)
- *Single biaxial*
  - *Top*: Bargraph of cell yields for each sample after debarcoding, given the current settings
  - *Bottom*: Scatterplot of barcode channel intensities for the selected sample. The channels displayed on the x and y axis are selected using the dropdown menus on the left (*Select x axis*, *Select y axis*). The points are colored according to the Mahalanobis distance from the centroid of the population. The color scale is displayed on top of the graph. The values plotted are arcsinh  transformed.
- *All barcode biaxials*
  - *Top*: Bargraph of cell yields for each sample after debarcoding, given the current setting
  - *Bottom*: a matrix of scatterplots of barcode channel intensities for the selected sample. All possible combinations are displayed, and the channels represented on the rows and columns are identified by the names on the axes of the bottom and left-most plots. The plots on the diagonal display the distribution of intensity values for the corresponding channel


# Gating
* Check data, clean to make sure doublets, dead cells and unwanted populations removed
* Manually gated data will establish landmark nodes, for this use desired software system (we use Cell Engine)

# Visualization

## Unsupervised visualization 
The first task is to get and idea of the structure of the sample— what cells are in there? How many different populations can we identify as a first pass? What are the markers combinations that define such populations? Rather than using standard dimension-reduction methods PCA or tSNE, both of which have limitations, we will use **Force-directed graphs** to achieve this.

Like other graphical methods, **Force-directed graphs** (_cite_) offer flexibility of visulization, as each node represents one cell (or cluster of cells) and each edge represents similarity between cells which can correspond to a variety of distance metrics or functions, but for our purposes we will use cosine (_link scgraphs package_).  In this layout the edges act as springs, whose strength is proportional to the weight of the edge (i.e. the similarity between the nodes). The algorithm then proceeds as a physical simulation by pulling together nodes that are similar (i.e. are connected by strong springs), and repelling dissimilar nodes. The end result is a layout where groups of similar cells are located close on the page, exactly what we set out to do for our visualization. (_cite Zunder:2015gp, Levine:2015ew, Samusik:2016ev, Spitzer:2015jd_?}

* Number of neighbours=15

## Clustering
Given our set of files, three modes of clustering are possible:
* Cluster each file/sample independently
* Pool all files from a given tissue type and cluster
* Pool all files then cluster and extract features from each group of cells (like change in expression, fold change) and look at association with endpoint

The choice in clustering here has important implications for feature generation and subsequent model building (more below).

Our example input directory called `Science Data` contains five files:
```
- BM_a_cells.fcs
- BM_b_cells.fcs
- BM_c_cells.fcs
- BM_d_cells.fcs
- BM_e_cells.fcs
```
If the choice was to run each sample independently, the following R code would apply:

```R
# These are the names of the columns in the FCS files that you want to use for clustering. 
# The column descriptions from the FCS files are used as name when available (corresponding
# to the $PxS FCS keyword). When descriptions are missing the channel names are used
# instead ($PxN keyword)

col.names <- c("(CD4)", "(CD8)", "((CD3)")

# Please refer to the documentation of this function for an explanation of the parameters
# and for a description of the output type. The output is saved on disk, and the function
# simply return the list of files that have been clustered
cluster_fcs_files_in_dir("Bone_marrow", num.cores = 1, col.names = col.names, num.clusters = 200,
    asinh.cofactor = 5, output.type = "directory")

# You can also specify a list of files directly using the cluster_fcs_files function,
# which takes essentially the same arguments
files.list <- c("Bone_marrow/BM_a_cells.fcs", "Bone_marrow/BM_b_cells.fcs")
cluster_fcs_files(files.list, num.cores = 1, col.names = col.names, num.clusters = 200,
    asinh.cofactor = 5, output.type = "directory")
```

If instead you wanted to pool some files together, you would setup the run as follows

```R
# Assuming for instance that you wanted to pool BM_a_cells.fcs and BM_b_cells.fcs in group 1, and BM_c_cells.fcs, 
# BM_d_cells.fcs & BM_e_cells.fcs in group 2 (once again please refer to the documentation for details)
files.groups <- list(
    group1 = c("Bone_marrow/ BM_a_cells.fcs", "Bone_marrow/ BM_b_cells.fcs")
    group2 = c("Bone_marrow/ BM_c_cells.fcs", " BM_d_cells.fcs", " BM_e_cells.fcs"))

cluster_fcs_files_groups(files.groups, num.cores = 1, col.names = col.names, 
    num.clusters = 200, asinh.cofactor = 5, output.type = "directory")
```

This can be done by tissue type (for example pooling lymph node and bone marrow files separately) or in whatever organization makes sense for your dataset.

### Output

Both clustering functions ouptut two types of data:
- A summary table of per-cluster statistics
- One or more RDS (R binary format) files containing cluster memberships for every cell event

The details of the RDS output depend on the `output.type` option, please refer to the R documentation for more details. The summary table contains one row for each cluster, and one column for each channel in the original FCS files, with the table entries representing the median intensity of the channel in the corresponding cluster.

If multiple files have been pooled together this table also contains columns in the form `CD4@BM_a_cells.fcs`, which contain the median expression of `CD4`, calculated only on the cells in that cluster that came from sample `BM_a_cells.fcs`

### Features generation

This package also contains functions to rearrange the clustering output to calculate cluster features that can be used to build a predictive model (similar to the approach used in the (Citrus)[https://github.com/nolanlab/citrus] package). These functions operate on the clusters summary table described above, and require data to have been pooled together before clustering (i.e. the clustering should have been run with the `cluster_fcs_files_groups` function). In other words, if you want to build a model that includes data from the four files in the example above, you need to cluster them as a single group.

The general approach for features generation for model building, is that you want to generate a table where each row represents a cluster feature (e.g. the abundance of a cluster, or the expression of a marker in a cluster), and each column represent an observation (e.g. a different sample), for which you have a categorical or continuous endpoint of interest that you want to predict using the cluster features.

This package allows you to gather data that has been collected in multiple FCS files and, after clustering, integrate all the different pieces together to generate that matrix.

The two main functions are (please refer to the R documentation for all the details):
- `get_cluster_features`: this function takes a model specification and rearranges the clustering output to produce features that are suitable for model building
- `multistep_normalize`: this function can be used to do complex normalization operations on the features

 Suppose that you have the following dataset !!

|file   |timepoint  |condition  |subject    |label  |tumor_size |
|-------|-----------|-----------|-----------|-------|-----------|
|A.fcs  |baseline   |stim1      |subject1   |R      |0.1        |
|B.fcs  |baseline   |stim2      |subject1   |R      |0.1        |
|C.fcs  |baseline   |unstim     |subject1   |R      |0.1        |
|D.fcs  |week8      |stim1      |subject1   |R      |1.5        |
|E.fcs  |week8      |stim2      |subject1   |R      |1.5        |
|F.fcs  |week8      |unstim     |subject1   |R      |1.5        |
|G.fcs  |baseline   |stim1      |subject2   |NR     |0.2        |
|H.fcs  |baseline   |stim2      |subject2   |NR     |0.2        |
|I.fcs  |baseline   |unstim     |subject2   |NR     |0.2        |
|L.fcs  |week8      |stim1      |subject2   |NR     |3.2        |
|M.fcs  |week8      |stim2      |subject2   |NR     |3.2        |
|N.fcs  |week8      |unstim     |subjcet2   |NR     |3.2        |

There are multiple ways that you could envision leveraging this data for model construction. The two key parameters of `get_cluster_features` that allow you to specify different models are:
- `predictors`: this specifies which variables are going to be used as predictors
- `endpoint.grouping`: this specifies which variables are used to group together files that are associated with the same endpoint

************* A few examples should clarify how to use this function ************* (ADD)

#### Example model 1

(create from our dataset?)

## Creating an unsupervised visualization in R

After clusters have been generated from the steps above in (scfeatures)[#scfeatures] (though any kind of tabular input data can be used), this package and the following steps enable the analysis of single-cell data using graphs, both unsupervised graphs as well as scaffold maps. 

Open a window in R and enter the following commands:


```R
# Use as input files that have been generated using scfeatures
input.files <- c("A.clustered.txt", "B.clustered.txt")

# Optional: Define a table of sample-level metadata. All the nodes derived from the corresponding cluster file will
# have vertex properties corresponding to this metadata ("response" and "pfs" in this example)
metadata.tab <- data.frame(filename = input.files, response = c("R", "NR"), pfs = c(12, 7))

# Define which columns contain variables that are going to be used to calculate similarities between the nodes
col.names <- c("foo", "bar", "foobar")


# The clusters in each one of the input files will be pooled together in a single graph
# This function also performs graph clustering by community detection. The community assignments are contained in
# the "community_id" vertex property of the resulting graph
G <- scgraphs::get_unsupervised_graph_from_files(input.files, metadata.tab = metadata.tab, 
            metadata.filename.col = "filename", col.names = col.names, filtering.threshold = 15)

# Write the resulting graph in graphml format. The Gephi software package (https://gephi.org/) is an excellent 
# solution to interactively manipulate the data
igraph::write.graph(G, "unsupervised.graphml", format = "grpahml")


## Supervised visualization 

### Running a Scaffold analysis

This code snippet demonstrates how to construct scaffold maps. In this example, the data for the landmark nodes, i.e. the gated populations, are given to you in the subfolder called [gated](https://github.com/ParkerICI/July-2018-single-cell-workshop/tree/master/Science%20datasets) as single FCS files (one for each population). The software will split the name of the FCS files using `"_"` as separator and the last field will be used as the population name. For instance if a file is called `foo_bar_foobar_Bcells.fcs` the corresponding population will be called `Bcells` in the scaffold analysis. Enter the following code into R:

```R
# Use as input files that have been generated using scfeatures
input.files <- c("A.clustered.txt", "B.clustered.txt")

# Define which columns contain variables that are going to be used to calculate similarities between the nodes
col.names <- c("foo", "bar", "foobar")

# Load the data for the landmarks
landmarks.data <- load_landmarks_from_dir("gated/", asinh.cofactor = 5, transform.data = T)
    
# Run the analysis. By default results will be save in a directory called "scaffold_result"
run_scaffold_analysis(input.files, input.files[1], landmarks.data, col.names)
```

#### Scaffold output

By the default the output of the analysis will be saved in a folder called `scaffold_result`. The directory will contain a `graphml` file for each Scaffold map and two sub-folders called `clusters_data` and `landmarks_data`.

These folders contain downsampled single-cell data for the clusters and landmarks, to be used for visualization. The `clusters_data` folder will contain a separate sub-folder for each `graphml` file, containing the data specific to that sample. The data is split in multiple `rds` files, one for each cluster (or landmark in `landmarks_data`). 

If the Scaffold analysis was constructed from data that was pooled before clustering (i.e. using `scfeatures::cluster_fcs_files_groups`), the `clusters_data` folder will also contain a subfolder called `pooled`, containing the pooled data, in addition to the sample-specific folders described above.

### Using the GUI

A GUI is available to launch either an unsupervised graph analysis or a Scaffold analysis. The GUI allows you to specify all the input options in a graphical environment, instead of having to write R code.

To launch the GUI type the following in your R console

```R
scgraphs::scgraphs_GUI()
``'

Like in earlier steps, this will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory (computer location) containing the data you want to analyze, and select any file in that directory. In this example, select a file from the [scfeatures](#scfeatures) output before. The directory itself will then become the working directory for the software. Make sure that * ALL * files are selected in your window- this requires one-by-one selection from the drop down box. Note: if the GUI does not launch, hit 'esc' and try to run the code again.


##Scaffold

The `scaffold2` R package (installed previously) puts together with all the required dependencies to run your Scaffold analysis. If evertyhing was successful you should be able to start the GUI with the following commands:

```R
library(scaffold2)
scaffold2()
```
Again, note that if it nothing pops up on your screen, or to stop the GUI simply hit the `ESC` key in your R session.

# Usage

When you launch the GUI you will be prompted to select a file. You can select *any* file in what you want to be your working directory and this will set the working directory for the remainder of the session.

The working directory must contain all the `graphml` files you want to visualize, plus a sub-folder named `clusters_data` containing the single-cell data for each cluster. If the graphml files represent Scaffold maps, a sub-folder called `landmarks_data` must also be present. 

If the `graphml` files were generated using the [scgraphs](https://github.com/ParkerICI/scgraphs) package these directories were generated for you as long as the `process.clusters.data` option was set to `TRUE` in `scgraphs::run_scaffold_analysis` or `scgraphs::get_unsupervised_graph_from_files` (please refer to the documentation of the `scgraphs` packages for details)

Once the working directory has been selected two browser windows will be opened: a main window containing the graph visualization, and a separate plotting window. Please note the following, depending on your browser settings:
- If your browser is configured to block pop-ups you need to allow pop-ups coming from the address `127.0.0.1:8072` (8072 is the default `scaffold2` port, you will have to enable pop-ups coming from a different port if you change this default)
- If your browser is configured to open new windows in a new tab, the last tab shown in the browser will be the plotting window, which is initially empty. The main `scaffold2` interface will be in a different tab

## Description of the GUI functionality

- **Choose a graph**: select the `graphml` file you want to visualize from a list of files contained in your working directory

You can interact with the graph using the mouse as follows:
- Scrolling: zoom in/out. 
- Left click + drag: panning
- Click on a node + Shift key: add the node to the current selection, or create a new selection if none existed (selected nodes are displayed in red)
- Left click + drag + Alt key: select all nodes inside a rectangle. To clear the current selection use this key combination to create a selection on an empty area of the graph

The appearence of the graph can be modified with the following controls:

- **Active sample**: whether to display data for All the samples (i.e. the pooled data) or a specific sample (this control is only available if the graph represents multiple samples, i.e. it was generated from pooled clustering). Selecting a different sample changes the size and color of the nodes, to reflect statistics calculated using only data from the current active sample.
- **Display edges**: select which edges to display:
   - All: displays all the edges
   - Highest scoring: for each node, display the highest scoring connection to a landmark (i.e. this is the most similar landmark)
   - Inter cluster: only display edges between clusters
   - To landmark: only display edges between clusters and landmarks
- **Nodes color**: use this dropdown to color the nodes according to the expression of a specific marker, or with "Default" colors (clusters in Blue, landmarks in red)
- **Stats type**: only available if the graph represents mulitple samples:
  - Absolute: the node colors represent the absolute value of the selected marker in the active sample
  - Ratio: the node colors represent the ratio between the value of the selected marker in the active sample and the value of the selected marker in the sample selected from the **Stats relative to** dropdown. The ratio is calculated on the asinh transformed values
  - Difference: the node colors represent the difference between the value of the selected marker in the active sample and the value of the selected marker in the sample selected from the **Stats relative to** dropdown. The difference is calculated on the asinh transformed values
- **Stats relative to**: only available if the graph represents multiple samples and **Stats type** is different from `Absolute`. The sample with respect to which stats are calculated (see **Stats type**)
- **Number of colors**: number of colors to use in the color scale
- **Under**: the color to use for values that are below the minimum of the scale
- **Over**: the color to use for value that are above the maximum of the scale
- **Min**: the color corresponding to the minimum of the scale
- **Mid**: the color corresponding to the midpoint of the scale (only available if the number of colors is 3)
- **Max**: the color corresponding to the maximum of the scale
- **Color scale midpoint**: the value corresponding to the midpoint of the color scale (only available if the number of colors is 3)
- **Color scale limits**: the numerical values that correspond to the domain of the color scale
- **Color scale min**: the minimum value available in the **Color scale limits** slider
- **Color scale max**: the maximum value available in the **Color scale limits** slider
- **Nodes size**: select whether you want the size of the nodes to be constant (Default) or Proportional to the number of cells in each cluster. 
- **Minimum / Maximum / Landmark node size**: the minimum and maximum size for the cluster and the size of the landmark (red) nodes
- **Reset graph position**: this button will reset the graph to its initial position, which is intended to display most of the nodes in a single image
- **Toggle landmark labels**: toggle the display of the landmark labels on/off
- **Toggle cluster labels**: toggle the display of the cluster labels on/off
- **Export selected clusters**: click this button to export the events in the selected clusters in a separate FCS file. For this to work, the original RData files corresponding to the clustered files in use must be located in the working directory. A new FCS file will be created in the working directory, with a name starting with *scaffold_export*, and ending with a random string of alpha-numeric characters, to prevent naming conflicts.

One of the most useful ways to inspect a cluster is to plot the expression values for the cells that comprise the cluster. The controls below allow you to control the appeareance of the plot. Only data for the clusters that have been selected will be plotted. If the graph represents a Scaffold map, the plot will also include the data for the landmarks that are connected to the selected clusters

- **Plot type**: the type of plot to display. Either a boxplot, a density plot, or a scatteplot (biaxial).
- **Facet by**: only availabe if the graph represents data from multiple samples. Whether the plot should be faceted by Sample or Variable, i.e. wether each Sample or each Variable should be its separate plot in the panel
- **Pool clusters data**: whether to pool all the data from the selected clusters for plotting. If the option is not selected, each cluster will be plotted individually as a separate boxplot, or density plot. Selecting this option will pool all the clusters data together for plotting. Note that if this option is selected, and the graph represents data from multiple samples, the data for the different samples will be pooled as well
- **Pool samples data**: only available if the graph represents data from multiple samples and **Pool clusters data** is not selected. Whether to pool the data from multiple samples for plotting.
- **Markers to plot**: Select which markers you want to display in the plot.
- **Samples to plot**: only available if the graph represents data from multiple samples. If this is left empty, the pooled data is plotted, otherwise only the data from the specified samples
- **Plot clusters**: plot the selected clusters. The plots appear in the separate plotting window


