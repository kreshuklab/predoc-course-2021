# EIPP Theory@EMBL 2024<!-- omit in toc -->

_Course material in previous years/events are managed by other branches._

You will design an image segemntation pipeline for immunofluorescence images of COVID-infected cells published in Microscopy-based assay for semi-quantitative detection of SARS-CoV-2 specific antibodies in human sera[[1]](https://www.biorxiv.org/content/10.1101/2020.06.15.152587v3). Please read the [Introduction](#introduction) before signing up to this challenge.

## Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Challenge: Cell segmentation](#challenge-cell-segmentation)
  - [Nuclei segmentation](#nuclei-segmentation)
  - [Cell boundary segmentation](#cell-boundary-segmentation)
  - [Segmentation with seeded watershed](#segmentation-with-seeded-watershed)
  - [Segmentation results evaluation](#segmentation-results-evaluation)

## Introduction

- **Lecture**: You will learn the basic concepts of supervised machine learning and its application to the problem of image analysis. You will learn the difference between feature-based and deep machine learning, how to avoid some common pitfalls and where to begin to apply it to your own data.
- **Challenge**: You will design an image analysis pipeline for immunofluorescence images of COVID-infected cells (https://www.biorxiv.org/content/10.1101/2020.06.15.152587v2). In the first part of the challenge, you will explore algorithms to segment the cell nuclei. Then, you will investigate how segmented nuclei can help in the more challenging task of segmenting the entire cell membrane. In this challenge, you will learn how to use and adapt state-of-the-art bioimage analysis algorithms and combine them in a custom pipeline to quantify visual information from microscopy images.
- **Pre-requisites**: you will ace this challenge if you either have some prior experience with images (used Fiji before, for example) or some prior experience with coding in Python. If you don’t have either, try to get on a team where at least one member has it and you’ll learn along the way.

## Challenge: Cell segmentation

You will explore algorithms to segment individual cells in the IF images from the above
study as shown in the picture below:

![cell_segm](img/cell_segm.png?raw=true "Serum cells segmentation pipeline")

The input to the pipeline is an image consiting of 3 channels: the 'nuclei channel' (containing DAPI stained nuclei), the 'serum channel' (dsRNA antibody staining) and the 'infection channel' (ignored in this challenge).
The output from the pipeline is a segmentation image where each individual cell is assigned a unique label/number.
You can download the Covid assay dataset from [here](https://oc.embl.de/index.php/s/gfpnDykYgcxoM7y).
The dataset consist of 6 files containing the raw data together with ground-truth labels.
The data is saved using the HDF5 file format. Each HDF5 file contains two internal datasets:

- `raw` - containing the 3 channel input image; dataset shape: `(3, 1024, 1024)`: 1st channel - serum, 2nd channel - infection (**ignored**), 3 - nuclei
- `cells` - containing the ground truth cell segmentation `(1024, 1024)`
- `infected` - containing the ground truth for cell infection (at the nuclei level); contains 3 labels: `0 - background`, `1 - infected cell/nuclei`, `2 - non-infected cell/nuclei`

We recommend [ilastik4ij ImageJ/Fiji](https://github.com/ilastik/ilastik4ij) or [napari](https://napari.org/) for loading and exploring the data.

The actual segmentation task can be split in three parts:

1. Segmentation of the nuclei using the **nuclei channel**
2. Predicting cell boundaries using the **serum channel**
3. Segmentation of individual cells with a seeded watershed algorithm, given the segmented nuclei and the boundary mask

After successfully executing the 3 pipeline steps, you can qualitatively compare the segmentation results to the ground truth images (`cells` dataset).
For quantitative comparison one may use one of the common instance segmentation metrics, e.g. [Adapted Rand Error](https://scikit-image.org/docs/dev/api/skimage.metrics.html#skimage.metrics.adapted_rand_error).

More detailed description of the 3 steps can be found below.

### Nuclei segmentation

Explore algorithms for instance segmentation of the nuclei from the 'nuclei channel'.
After successfully segmenting the nuclei from the covid assay dataset, save the results in the appropriate format (tiff of hdf5),
since you'll need it in step 3.

There are multiple options for nuclei instance segmentation that you could explore, this include but are not limited to
- ilastik workflows
- StarDist Model
- Cellpose Model
- PlantSeg Model
- InstanSeg

You are welcome to explore whatever approach you would like and can even compare the results of mulitple methods.

#### ilastik
If you decide to investigate ilastik you have multiple workflows that you could try
1. [`Pixel Classification`](https://www.ilastik.org/documentation/pixelclassification/pixelclassification) followed by [`Object Classification`](https://www.ilastik.org/documentation/objects/objects).
2. [`Neural Network Classification (Local)`](https://www.ilastik.org/documentation/nn/nn) followed by [`Boundary-based Segmentation with Multicut`](https://www.ilastik.org/documentation/multicut/multicut). !See more information below!

More information about how to perform all of the workflows or any general information about ilastik can be found in the [documentation](https://www.ilastik.org/documentation/).

:exclamation:Neural Network Classification (Local):exclamation:

If you decide to investigate the 2nd option listed above you will be making use of a pre-trained CNN to predict the nuclei. You can follow the instructions given in the next stage (Cell boundary segmentation). However, you should select a different model, one that is more suitable to your current task [`NucleiSegmentationBoundaryModel](https://bioimage.io/#/?tags=nuclei&id=10.5281%2Fzenodo.5764892).

#### StarDist
[StarDist](https://github.com/stardist/stardist) is a method based on neural networks that can perform segmentation. More deatils on StarDist can be found on the github or in the original [publication](https://arxiv.org/abs/1806.03535) approaches have pretrained models available to use on your own data. As detailed on the github page StarDist can be installed as a package and used programmatically, you are welcome to do this and explore the results. Alternatively, if you would rather interact with a gui StarDist can be run through a Fiji plugin [StarDist Fiji](https://imagej.net/plugins/stardist).


#### Cellpose
[Cellpose](https://github.com/MouseLand/cellpose) is another neural netowrk based method that can perform object segmentaiton.Instructions for the use and installation of Cellpose can be found on the github page. You can interact with cellpose either programmatically or via a gui. Once again cellpose provides pretrained models that you can make use of. :exclamation: Warning :exclamation: this does require the setting up of a python conda environment, if you are not already familiar with conda environments I reccomend installing [miniforge](https://github.com/conda-forge/miniforge).



### Cell boundary segmentation

In order to simplify the task of cell boundary prediction you will also use a pre-trained CNN.
This time we encourage you to use the [ilastik Neural Network Classification Workflow](https://www.ilastik.org/documentation/nn/nn).
Please download and install the **latest beta version** of ilastik in order to use the Neural Network Classification workflow (see: https://www.ilastik.org/download.html).

Then:

- open ilastik and create the `Neural Network Classification (Local)` project
- load a sample H5 image: `Raw Data -> Add New -> Add separate image -> (choose h5 file)`
  make sure to load only the serum channel (you need to extract the serum channel and save it in a separate h5 file beforehand). The size of the input should be `(1, 1024, 1024)`; **do not skip the singleton dimension**
- go to [BioimageIO](https://bioimage.io/), find [`CovidIFCellSegmentationBoundaryModel`](https://bioimage.io/#/?id=10.5281%2Fzenodo.5847355). We are going to use this pre-trained network to predict the cell boundary segmentation. Copy the name (powerful-chipmunk) and paste it into the box in the `NN Prediction` section.
- go to `NN Prediction` and click the green arrow (`Load model`); this will load the weights of the selected model.
- after the model has been loaded successfully, click `Live Predict`; after the prediction is finished you can see the two output channels
  predicted by the network (i.e. foreground channel and cell boundaries channel) by switching between the layers in `Group Visibility` section (bottom left);
  you should see something like the image below:
  ![cell_segm](img/ilastik_nn_workflow.png?raw=true "NN workflow")
- go to `Export Data` and save the output from the network in hdf5 format for further processing

**Important note**
The network predicts 2 channels: the 1st channel contains a foreground(cells)/background prediction and the 2nd channel
contains the cell boundaries. You will need both channels for step 3.

### Segmentation with seeded watershed

Given the nuclei segmentation (step 1), the foreground mask and the the boundary prediction maps (step 2),
use the seeded watershed algorithm from the `skimage` (see [documentation](https://scikit-image.org/docs/stable/api/skimage.segmentation.html#skimage.segmentation.watershed))
library in order to segment the cells in the serum channel.

Tip:
The watershed function is defined as follows:

```python
skimage.segmentation.watershed(image, markers=None, mask=None)
```

use boundary probability maps as `image` argument, nuclei segmentation as `markers` argument, and the foreground mask as the `mask` argument.

[environment.yml](environment.yml) can be used to create a conda environment with the necessary dependencies:

```bash
conda env create -f environment.yml
```

### Segmentation results evaluation

Compare your cell segmentation results with the ground truth (saved as `cells` dataset in the HDF5 files), using one/all of the metrics described below:

- [Adapted Rand Error](https://scikit-image.org/docs/dev/api/skimage.metrics.html#skimage.metrics.adapted_rand_error)
- [Variation of Information](https://scikit-image.org/docs/dev/api/skimage.metrics.html#skimage.metrics.variation_of_information)
- [Average Precision](https://github.com/hci-unihd/plant-seg/blob/master/evaluation/ap.py#L131)
