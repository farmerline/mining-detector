# Amazon Gold Mining Detector
<!--![mining-header](https://user-images.githubusercontent.com/13071901/146877405-3ec46c73-cc80-4b1a-8ad1-aeb189bb0b38.jpg)-->
![mining-header-planet](https://user-images.githubusercontent.com/13071901/146877590-b083eace-2084-4945-b739-0f8dda79eaa9.jpg)

This repo contains the models, code, and outputs created for the automated detection of gold mining in the Amazon rainforest using satellite imagery.

The system works by taking a 440 m² patch of data from the [Sentinel 2 L1C product](https://sentinel.esa.int/web/sentinel/missions/sentinel-2). A neural network then analyzes the patch, assessing whether recent mining activity is seen within the tile. The region of interest is shifted by 140 m, and the network makes another classification. This process is repeated to cover the entire region of interest, producing a database of likely mining locations. This system was developed for use in the Amazon, though it has also been seen to work in other tropical biomes.

## Overview
The primary output of this work is a map of likely mining locations in the Amazon basin assessed in 2020. This analysis evaluated more than 200 billion pixels and made more than 326 million neural network classifications to cover the 6.7 million square kilometers of the Amazon basin.

#### Users should be aware of the following limitations:

**Temporal Resolution**

Mining in the Amazon is growing rapidly. Most basemap imagery in the Amazon is not current, thus some regions classified as containing mines will not appear to have mining activity in the imagery. See example below. Regions in question can be assessed by viewing recent [Sentinel 2 imagery on SentinelHub EO Browser](https://apps.sentinel-hub.com/eo-browser/?zoom=14&lat=-7.13214&lng=-57.36245&visualizationUrl=https%3A%2F%2Fservices.sentinel-hub.com%2Fogc%2Fwms%2Fbd86bcc0-f318-402b-a145-015f85b9427e&datasetId=S2L2A&fromTime=2020-09-16T00%3A00%3A00.000Z&toTime=2020-09-16T23%3A59%3A59.999Z&layerId=1_TRUE_COLOR), or Planetscope data accessible through the [Planet NICFI program](https://www.planet.com/nicfi/).
![mining-imagery-comparison](https://user-images.githubusercontent.com/13071901/146989519-d1e537c4-7d70-438d-b4a5-06b2a41a8482.jpg)

**Model Accuracy**

In order to run across the full breadth of the Amazon basin, the model's sensitivity and precision have been reduced in order to improve generalization. This means that the model outputs have some false positives (mining classification where none is present) and false negatives (mines that are rejected by the model). Common false negative failure modes include older mining sites that may be inactive, and edges of active mining regions. False positives can at times be triggered by natural sedimented pools, manmade water bodies, and natural earth-scarring activities such as landslides.

The aggregate assessment of mining status should be trusted, but users should attempt to validate results by eye if precise claims of mined regions are needed. Additionally, there are a few regions that we could not assess mining activity by eye in high resolution satellite imagery. We have decided to leave these regions in the output data and maps.

**Area Overestimation**

The goal of this work is mine detection rather than area estimation, and our classification operates on the classification of 440 m² patches. If the network assesses that mining exists anywhere within the patch, then the full patch is declared a mine. This leads to a systematic overestimation of mined area if it is naively computed from the polygon boundaries.

Because the polygon area estimation is consistent, measuring relative year-to-year change is a viable option.


## Results
### 2020 Amazon Basin Mining
The results can be visualized on the [2020 Map of Mining in the Amazon Basin]() and the data can be downloaded in GeoJSON form from the [2020 Mining in the Amazon basin dataset]().

### 2016-2020 Tapajos Basin Mining Progression

### 2020 Bolivar and Amazonas Validated Mining




## Setup

Download and install [Miniforge Conda env](https://github.com/conda-forge/miniforge/) if not already installed:


| OS      | Architecture          | Download  |
| --------|-----------------------|-----------|
| Linux   | x86_64 (amd64)        | [Miniforge3-Linux-x86_64](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh) |
| Linux   | aarch64 (arm64)       | [Miniforge3-Linux-aarch64](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh) |
| Linux   | ppc64le (POWER8/9)    | [Miniforge3-Linux-ppc64le](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-ppc64le.sh) |
| OS X    | x86_64                | [Miniforge3-MacOSX-x86_64](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-x86_64.sh) |
| OS X    | arm64 (Apple Silicon) | [Miniforge3-MacOSX-arm64](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh) |

Then run 
```
chmod +x ~/Downloads/Miniforge3-{platform}-{architecture}.sh
sh ~/Downloads/Miniforge3-{platform}-{architecture}.sh
source ~/miniforge3/bin/activate
```

Next, create a conda environment named `mining-detector` by running `conda env create -f environment.yml` from the repo root directory. Activate the environment by running `conda activate mining-detector`. Code has been developed and tested on a Mac with python version 3.9.7. Other platforms and python releases may work, but have not yet been tested.

If desired, the data used for model training may be accessed and downloaded from s3 at `s3://mining-data.earthrise.media`.

## Notebooks
The system runs from three core notebooks. 

As described previously, creating datasets and deploying the model require access to the [Descartes Labs](https://descarteslabs.com/) platform.

#### `create_dataset.ipynb` (requires Descartes Labs access)
Given a GeoJSON file of sampling locations, generate a dataset of Sentinel 2 images. Dataset is stored as a pickled list of numpy arrays.

#### `train_model.ipynb`
Train a neural network based on the images stored in the `data/training_data/` directory. Data used to train this model is stored at `s3://mining-data.earthrise.media`.

#### `deploy_model.ipynb` (requires Descartes Labs access)
Given a model file and a GeoJSON describing a region of interest, run the model and download the results. Options exist to deploy the model on a directory of ROI files.

## Data
The data directory contains two directories.
- `data/boundaries` contains GeoJSON polygon boundaries for regions of interest where the model has been deployed.
- `data/sampling_locations` contains GeoJSON datasets that are used as sampling locations to generate training datasets. Datasets in this directory should be considered "confirmed," and positive/negative class should be indicated in the file's title.
- `data/outputs` will be populated with results from `deploy_model.ipynb` in a folder corresponding to the model version used in the run. By default, it is populated with GeoJSON files from our runs and analyses.

## Models
The models directory contains keras neural network models saved as `.h5` files. The model names indicate the patch size evaluated by the model, followed by the model's version number and date of creation. Each model file is paired with a corresponding config `.txt` file that logs the datasets used to train the model, some hyperparameters, and the model's performance on the test dataset.

The model `44px_v2.8_2021-11-11.h5` is currently the top performer overall. However, different models have different strengths/weaknesses. There are also versions of model v2.6 that operate on RGB and RGB+IR data. These may be of interest when evaluating whether multispectral data from Sentinel is required.

## Docs
This directory does not store docs. Instead, it hosts .html files that are displayed on the repo's github pages site at [https://earthrise-media.github.io/mining-detector/{file_name}.html](https://earthrise-media.github.io/mining-detector/amazon-v2.4.html).