# OmniUnet

[![arxiv](https://img.shields.io/badge/arXiv-2508.00580-b31b1b.svg)](https://doi.org/10.48550/arXiv.2508.00580)
[![Data](https://img.shields.io/badge/Data-DOI-green)](https://doi.org/10.5281/zenodo.15496884)

Code associated with the article: **"OmniUnet: A Multimodal Network for Unstructured Terrain Segmentation on Planetary Rovers Using RGB, Depth, and Thermal Imagery".**

*Author:* [R. Castilla Arquillo](https://github.com/raulcastar) [![orcid](https://orcid.org/sites/default/files/images/orcid_16x16.png)](https://orcid.org/0000-0003-4203-8069)

*Supervisor:* [Carlos J. Pérez del Pulgar](https://github.com/carlibiri) [![orcid](https://orcid.org/sites/default/files/images/orcid_16x16.png)](https://orcid.org/0000-0001-5819-8310)

*Contact info:* <raulcastar@uni.lu>

## Links

- arXiv article: <https://doi.org/10.48550/arXiv.2508.00580>
- dataset: <https://doi.org/10.5281/zenodo.15496884>

## Citation

If this work was helpful for your research, please consider citing the following BibTeX entry:

```bibtex
@article{castilla2025omniunet,
      title={OmniUnet: A Multimodal Network for Unstructured Terrain Segmentation on Planetary Rovers Using RGB, Depth, and Thermal Imagery}, 
      author={Raul Castilla-Arquillo and Carlos Perez-del-Pulgar and Levin Gerdes and Alfonso Garcia-Cerezo and Miguel A. Olivares-Mendez},
      year={2025},
      eprint={2508.00580},
      archivePrefix={arXiv},
      primaryClass={cs.RO},
      journal={arXiv},
      doi={10.48550/arXiv.2508.00580},
}
```

## System information

This repository contains the code for training and executing OmniUNet, a multimodal neural network based on transformers designed for semantic segmentation of images that combine color, depth, and thermal data. OmniUNet enables robust perception in complex environments by leveraging complementary sensor modalities. It is especially suited for robotics applications.

A diagram of the OmniUNet architecture is shown below:

<div align="center">
<img src="docs/omniunet_diagram.png" alt="OmniUnet diagram" width="600"/>
</div>

## Docker configuration

A Ubuntu host system is needed to run the files located in this repo, as we use an Nvidia GPU to train the network. First of all, we must install the docker core:

```bash
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

We install our NVIDIA card's drivers and the modules that let us use them in our docker environment:

```bash
sudo ubuntu-drivers autoinstall
sudo apt-get install -y nvidia-docker2 nvidia-container-runtime
```

After that, we must build the configured docker container for this project:

```bash
docker build . -f ./dockerfile/omniunet.dockerfile -t omniunet 
```

We run the docker image:

```bash
xhost +local:docker  ## To let docker use the screen

docker run -e DISPLAY=$DISPLAY -v /home/pc/Escritorio/omnivore_orig_tests:/home/omnivore \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  --ipc=host \
  --rm --gpus all \
  -it omniunet \
  -d -t \
  /bin/bash
```

and:

```bash
docker exec -it $ID$  /bin/bash
```

## Commands

### Train mode

`train.py` script will be used to train our neural network. For doing so, many arguments can be used

- d: in which dataset we will train. Explore data_loading.py to know more
- b: batch size
- e: number of epochs. At the end of every epoch, a .pth will be saved in order to avoid losing data if training gets interrupted
- l: learning rate
- f: load model from a .pth file as a pretrain. If none is given, will train from scratch
- s: Downscaling factor of the images
- v: Percent of images in dataset used for validation
- c: Number of classes in the dataset. This parameter must be given

Example:

```bash
python3 train.py -u=1 -m=5 -d=rgbdt -c=6 -b=16 -e=30 -l=2e-5
```

If you want to continue a training:

```bash
python train.py -d=rgbdt -u=1 -c=6 -b=16 -e=30 -l=2e-5 -f=path_to_your/file.pth
```

### Predict mode

- m: Number of max channels architecture
- i: RGB input
- p: Depth input
- T: Thermal input
- d: Dataset
- c: Number of classes in the dataset

#### Predict

```bash
python3 predict.py -m=path_to_your_weights.pth -d=rgbdt -c=6 -i=path_to_rgb_img.png  -p=path_to_depth_img.csv  -T=path_to_thermal_img.csv
```

#### Single images metrics mode

```bash
python3 predict_metrics.py -m=path_to_your_weights.pth -d=rgbdt -c=6 -i=path_to_rgb_img.png -p=path_to_depth_img.csv  -T=path_to_thermal_img.csv -g=path_to_groundtruth_mask.png
```

#### Multi-image metrics mode

```bash
python3 predict_multi_metrics.py -m=path_to_your_weights.pth -d=rgbdt -c=6 -i=path_to_rgb_img_folder -p=path_to_depth_img_folder -T=path_to_thermal_img_folder -g=path_to_groundtruth_mask_folder
```

### Training missing modalities

RGDT needs to be defined by default.

```bash
python3 train_missing_mod.py -u=1 -m=5 -d=rgbdt -b=16 -e=30 -l=2e-5 -c=6 
```
