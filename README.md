# Amazon Gold Mining Detector
<!--![mining-header](https://user-images.githubusercontent.com/13071901/146877405-3ec46c73-cc80-4b1a-8ad1-aeb189bb0b38.jpg)-->
![mining-header-planet](https://user-images.githubusercontent.com/13071901/146877590-b083eace-2084-4945-b739-0f8dda79eaa9.jpg)


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

Install dependencies 

If desired, the data used for model training may be accessed on s3 at `s3://mining-data.earthrise.media`.

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
This directory does not store docs. Instead, it hosts .html files for repos github repos site. 
