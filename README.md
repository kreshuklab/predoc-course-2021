# EIPP Theory@EMBL 2024<!-- omit in toc -->

_Course material in previous years/events are managed by other branches._

You will design an image segmentation pipeline for immunofluorescence images of COVID-infected cells published in Microscopy-based assay for semi-quantitative detection of SARS-CoV-2 specific antibodies in human sera[[1]](https://www.biorxiv.org/content/10.1101/2020.06.15.152587v3). Please read the [Introduction](#introduction) before signing up to this challenge.

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
You can download the Covid assay dataset from [here](https://oc.embl.de/index.php/s/zOl3CFIApyTmvL1?path=%2FWeek%205%20-%20Theory%40EMBL%2FTeam2-Kreshuk).
The dataset consist of 6 files containing the raw data together with ground-truth labels. I have made the data available as both HDF5 file format and tif file format. Each HDF5 file contains three internal datasets:

- `raw` - containing the 3 channel input image; dataset shape: `(3, 1024, 1024)`: 1st channel - serum, 2nd channel - infection (**ignored**), 3 - nuclei
- `cells` - containing the ground truth cell segmentation `(1024, 1024)`
- `infected` - containing the ground truth for cell infection (at the nuclei level); contains 3 labels: `0 - background`, `1 - infected cell/nuclei`, `2 - non-infected cell/nuclei`

I have also seperated out the nuclei and serum channels from the raw and saved these seperately for you to use in the stages of the challenge. The seperated channel images can be found in `nuclei` and `serum` folders. Each file contains a image of shape `(1, 1024, 1024)`. 

We recommend [ilastik4ij ImageJ/Fiji](https://github.com/ilastik/ilastik4ij) or [napari](https://napari.org/) for loading and exploring the data.

The actual segmentation task can be split in three parts:

1. Segmentation of the nuclei using the **nuclei channel**
2. Predicting cell boundaries using the **serum channel**
3. Segmentation of individual cells with a seeded watershed algorithm, given the segmented nuclei and the boundary mask

After successfully executing the 3 pipeline steps, you can qualitatively compare the segmentation results to the ground truth images (`cells` dataset).
For quantitative comparison one may use one of the common instance segmentation metrics, e.g. [Adapted Rand Error](https://scikit-image.org/docs/dev/api/skimage.metrics.html#skimage.metrics.adapted_rand_error).

More detailed description of the 3 steps can be found below.

## How is this going to work?
For the following challenge there is a lot of flexibility depending on how you want to work and how much experience you already have with python and image analyis. All the exercises can be run locally on your own computer, although this will require some installation of various tools and also usage of conda environments. If you would rather not run things locally or don't feel confident with how that would work then we also have several virtual machines (VMs) which we can run through your computer. Another advantage of using the VMs is that most of the software we want to use has been pre-installed. I will run through how to access these in a short demo, but you should be able to follow these links and then just use your embl credentials to access the VMs.
- [Jupyterhub](https://jupyterhub.embl.de/hub/spawn)
- [BARD](https://bard.embl.de/)

When you have gained access to the VMs we still need to get access to the data, you can do this through the [ownCloud](https://oc.embl.de/index.php/s/zOl3CFIApyTmvL1?path=%2FWeek%205%20-%20Theory%40EMBL%2FTeam2-Kreshuk) as described below (although annoyingly copy and pasting might be annoying, so you might have to type out the url :unamused:) or I have downloaded the relevant data already onto my personal g-drive so you can just copy it from there to whatever location you like, for example your /home/USER_NAME folder. To copy the data to your desired location navigate to the desired location using the `cd` command. To check your current location you can use the `pwd` command. Use the following command to copy the data to your current location with the same filename.
- `cp -r /g/kreshuk/talks/predoc-course ./`

### 1. Nuclei segmentation

Explore algorithms for instance segmentation of the nuclei from the 'nuclei channel'.
After successfully segmenting the nuclei from the covid assay dataset, save the results in the appropriate format (tiff of hdf5), since you'll need it in step 3.

There are multiple options for nuclei instance segmentation that you could explore, this includes but is not limited to
- ilastik
- StarDist
- Cellpose
- PlantSeg

You are welcome to explore whatever approach you would like and can even compare the results of mulitple methods if you have time.

#### ilastik
If you decide to investigate ilastik you have multiple workflows that you could try
1. [`Pixel Classification`](https://www.ilastik.org/documentation/pixelclassification/pixelclassification) followed by [`Object Classification`](https://www.ilastik.org/documentation/objects/objects).
2. [`Neural Network Classification (Local)`](https://www.ilastik.org/documentation/nn/nn) followed by [`Boundary-based Segmentation with Multicut`](https://www.ilastik.org/documentation/multicut/multicut). !See more information below!

More information about how to perform all of the workflows or any general information about ilastik can be found in the [documentation](https://www.ilastik.org/documentation/).

:exclamation:Neural Network Classification (Local):exclamation:

If you decide to investigate the 2nd option listed above you will be making use of a pre-trained CNN to predict the nuclei. You can follow the instructions given in the next stage (Cell boundary segmentation). However, you should select a different model, one that is more suitable to your current task [NucleiSegmentationBoundaryModel](https://bioimage.io/#/?tags=nuclei&id=10.5281%2Fzenodo.5764892).

:exclamation:Intallation Advice:exclamation:
You can either install ilastik locally on your machine and run everything there, or login to a VM (as discussed above) and run the workflows directly in the pre-installed ilastik.

#### StarDist
[StarDist](https://github.com/stardist/stardist) is a method based on neural networks that can perform segmentation. More deatils on StarDist can be found on the github or in the original [publication](https://arxiv.org/abs/1806.03535). If you chose to investigate StarDist we can utalise a pretrained model. As detailed on the github page StarDist can be installed as a package and used programmatically, you are welcome to do this and explore the results. Alternatively, if you would rather interact with a gui, StarDist can be run through a Fiji plugin [StarDist Fiji](https://imagej.net/plugins/stardist). If you are using the Fiji plugin I recommend using the model called `Versatile (fluorescent nuclei)`.

:exclamation:Intallation Advice:exclamation:
If you are working on a Virtual Machine the pre-existing Fiji is a bit of a problem as it is read only ... yada yada yada. Basically you will need to install your own version of Fiji to allow you to add the StarDist plugin. **Alternatively** you can use my pre-installed version that has the StarDist plugin already installed :star:. To access this version of Fiji run the following command in the VMs terminal. 
`/g/kreshuk/talks/Fiji.app/ImageJ-linux64`


#### Cellpose
[Cellpose](https://github.com/MouseLand/cellpose) is another neural netowrk based method that can perform object segmentaiton.Instructions for the use and installation of Cellpose can be found on the github page. You can interact with cellpose either programmatically or via a gui. Once again cellpose provides pretrained models that you can make use of. :exclamation: WARNING :exclamation: this does require the setting up of a python conda environment, if you are not already familiar with conda environments I recommend installing [miniforge](https://github.com/conda-forge/miniforge).

#### PlantSeg
[PlantSeg](https://github.com/kreshuklab/plant-seg) is a tool for cell instance aware segmentation in densely packed 3D volumetric images. The pipeline uses a two stages segmentation strategy (Neural Network + Segmentation). The pipeline is tuned for plant cell tissue acquired with confocal and light sheet microscopy. Pre-trained models are provided. Follow the github page for installation instructions, :exclamation: WARNING :exclamation: Requires setting up a conda environment follow installation advice for [miniforge](https://github.com/conda-forge/miniforge).

Once you have PlantSeg installed can follow the instructions in the [documentation](https://kreshuklab.github.io/plant-seg/) to run PlantSeg through the napari plugin. You will once again load a pre-trained model and can explore the functionality of PlantSeg. I recommend using the `lightsheet_2D_unet_root_nuclei_ds1x` model.

If you are using my conda environment following the instructions the Conda Environment section you may get an error referring to your QT_API. Try running the following command.
`export QT_API=pyqt5`

## :exclamation: Conda Environment :exclamation:
If you want to use conda environments for any of the stages of this practical you have a few options depending on your familiarity level.
1. You know what a Conda environment is and want to make yout own, go ahead no need to read more!!!
2. You sort of know what a Conda environment is and want to install and set up your own. Hats off to you, give it a go its not that scary :ghost: :ghost:. I advise installing [mininforge](https://github.com/conda-forge/miniforge) you can follow the instructions on their github and ask if you need help.
3. What is a Conda??? Isn't that a type of snake :snake:? Fear not I have set up some conda environments for you that you can use without installing anything by following the instructions below.
  - For this to work you need to be logged into one of the VMs through [Jupyterhub](https://jupyterhub.embl.de/hub/spawn) or [BARD](https://bard.embl.de/). If you decide to use Jupyterhub then I recommend the `Image Analysis: Special Purpose Desktops`.
  - Open a terminal, you should be in your home folder e.g. /home/username.
  - Now need to initialise Conda with the command 
  `/g/kreshuk/talks/miniforge3/bin/conda init`
  - Now close and re-open the terminal to allow this to take afffect
  - To activate a Conda environment enter the following command into the terminal, replacing ENV_NAME with the desired environment name 
    `conda activate ENV_NAME`
    I have made a few environments for you to use, the general environment needed to run the notebook is named `predoc-challenge`. An environment to run PlantSeg is called `plant-seg`. An environment to run Cellpose is called 
  - Once your environment is activated you can run whatever commands are neccesary in that same terminal, as per the instructions of the tool you are investigating.
  

:exclamation: IMPORTANT :exclamation: If you want to run a notebook you will need to activate the python environment as a kernel in the Jupyter Notebook. First we need to add the conda environment to jupyter notebook.
1. Go to the terminal and activate the environment as before, 
`conda activate predoc-challenge`
2. Run the following command to add the environment to the notebook 
`python -m ipykernel install --user --name=predoc-challenge`
3. Now when you open up a notebook you can select kernel in the top right and select the `predoc-challenge` kernel.

### 2. Cell boundary segmentation

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

### 3. Segmentation with seeded watershed

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

### 4. Segmentation results evaluation

Compare your cell segmentation results with the ground truth (saved as `cells` dataset in the HDF5 files), using one/all of the metrics described below:

- [Adapted Rand Error](https://scikit-image.org/docs/dev/api/skimage.metrics.html#skimage.metrics.adapted_rand_error)
- [Variation of Information](https://scikit-image.org/docs/dev/api/skimage.metrics.html#skimage.metrics.variation_of_information)
- [Average Precision](https://github.com/hci-unihd/plant-seg/blob/master/evaluation/ap.py#L131)
